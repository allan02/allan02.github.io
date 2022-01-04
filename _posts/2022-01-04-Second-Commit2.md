---
layout: post
title: "Vagrant에 MySQL 설치하기"
author: "Jaehyeon"
---

<h3>먼저 Vagrant를 사용하여 CentOS를 Provisioning 해보겠습니다.</h3>


<h4>Vagrant란?</h4>


Vagrant는 포터블 가상화 소프트웨어 개발 환경의 생성 및 유지보수를 위한 오픈 소스 소프트웨어 제품의 하나입니다.


Vagrant는 독립적으로 사용되는 도구가 아니며, 가상 머신을 생성하거나 조작하는 기능을 직접 제공하지는 않습니다. 


Vagrant에는 Provider라는 개념이 있어서 VirtualBox, VMWare, Docker, Hyper-V와 같은 도구들을 가상 머신을 관리하는 도구로 조합해서 사용할 수 있습니다.


<h4>Vagrant를 사용하기 위해 다음 코드들을 순차적으로 Mac 터미널에서 입력해 주시면 됩니다.</h4>


<ul>

  
<li>/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"</li><br>

  
<li>brew install --cask visual-studio-code</li><br>

  
<li>brew install virtualbox virtualbox-extension-pack vagrant</li><br>

  
<li>mkdir -p ~/workspace/VM/centos7<br>
(Vargant 관련 파일이 만들어질 작업 폴더를 구성합니다.)</li><br>

  
<li>cd /workspace/VM/centos7</li><br>

  
<li>vagrant plugin install vagrant-vbguest</li><br>

  
<li>vagrant init<br>
(초기파일을 생성합니다.)</li><br>

  
<li>touch init.yml</li>
  
  
</ul>


그 후, vi 에디터를 통해 위 코드를 통해 생성된 Vagrantfile과 init.yml을 다음과 같이 수정해줍니다.


<h4>Vagrantfile</h4>


```
ENV["LC_ALL"] = "en_US.UTF-8"
Vagrant.configure("2") do |centos|

  # All servers will run cent 7
  centos.vm.box = "centos/7"
  centos.vm.box_check_update = false
  centos.disksize.size = "60GB"

  # Create the cent1 Server
  N = 1
  (1..N).each do |i|
    hostname = "cent7-#{i}"
    centos.vm.define hostname do |host1|
      host1.vm.hostname = hostname
      host1.vm.network "private_network", ip: "192.168.56.#{10 + i}"
      host1.vbguest.auto_update = false
      host1.vm.provider "virtualbox" do |v|
        v.name = hostname
        v.memory = "2048"
        v.cpus = "2"
        v.linked_clone = "true"
        v.gui = "false"
        v.customize ['modifyvm', :id, '--graphicscontroller', 'vmsvga']
        v.customize ['modifyvm', :id, '--vram', '20']
      end
    end
  end
  centos.vm.provision "ansible" do |ansible|
    ansible.playbook = "init.yml"
  end
  # centos.vm.provision "shell" do |shell|
  #   shell.path = "extend.sh"
  # end
end
```


<h4>init.yml</h4>

```
- name: init.yml
  hosts: all
  gather_facts: no
  become: yes
  tasks:
    - name: 사용자 이름 생성
      user:
        name: "{{ item }}"
        shell: /bin/bash
        home: "/home/{{ item }}"
        generate_ssh_key: true
        password_lock: yes
      with_items:
        - irteam
        - irteamsu
        - centos
    - name: sudoers.d 추가
      copy:
        content: |
          %{{item}} ALL=(ALL) NOPASSWD: ALL
        dest: "/etc/sudoers.d/{{item}}"
        owner: root
        group: root
        mode: 0440
        validate: "/usr/sbin/visudo -c -f '%s'"
      with_items:
        - irteam
        - irteamsu
        - centos
    - name: ssh 내용 추가
      authorized_key:
        user: "{{ item }}"
        state: present
        key: "{{ lookup('file', '~/.ssh/id_rsa.pub') }}"
      with_items:
        - irteam
        - irteamsu
        - centos
        - vagrant
        - root
    - name: Make sure a service unit is restarting
      ansible.builtin.systemd:
        state: restarted
        name: sshd.service
```

그 후, `vagrant up` 명령어를 통해 가상 머신을 실행시킵니다.<br>

발생할 수 있는 오류와 해결 방법은 다음과 같습니다.

<img src="/assets/brew_install_ansible.png" width="90%" height="90%" title="제목" alt=""/> 

-> ```brew install ansible``` 명령어를 통해 해결합니다.<br><br><br>

<img src="/assets/ip_error.png" width="90%" height="90%" title="제목" alt=""/> 

-> ```Vagrantfile```에 설정된 IP 주소를 범위에 맞게 변경해 줍니다.<br><br><br>

<img src="/assets/option_error.png" width="90%" height="90%" title="제목" alt=""/> 

-> https://blog.aeei.io/2021/07/24/occured-issues-in-k8s/ 블로그의 ```[문제 1]```을 참고합니다.<br><br><br>


<h3>Provisioning된 CentOS에 MySQL을 설치해 보겠습니다.</h3>

<h4>다음 코드들을 순차적으로 CentOS 환경에서 입력해 주시면 됩니다.</h4>

<ul>
  
<li>yum -y install http://dev.mysql.com/get/mysql57-community-release-el7-11.noarch.rpm<br>
(MySQL 5.7을 설치합니다.)</li><br>

  
<li>yum -y install mysql-community-server</li><br>

  
<li>systemctl start mysqld</li><br>

  
<li>vi /var/log/mysqld.log<br>
(MySQL을 실행하면 임시 비밀번호가 생성되고 mysqld.log 파일 안에서 임시 비밀번호를 확인 할 수 있습니다.)<br>
<img src="/assets/first_password.png" width="90%" height="90%" title="제목" alt=""/>
  </li><br>

  
<li>su root<br>
  (<strong>you need to be root to perform this command</strong> 에러 메시지를 방지 하기 위해 root로 이동합니다.)
  </li><br>
  
  
<li>ALTER USER 'root'@'localhost' IDENTIFIED BY '새 비밀번호';<br>
  (비밀번호를 재설정 합니다.)
  <img src="/assets/new_password.png" width="90%" height="90%" title="제목" alt=""/>
  </li><br>
  
<li>FLUSH PRIVILEGES;</li><br>

</ul>

지금까지 잘 따라오셨다면, 가상머신에서 데이터베이스가 이상 없이 잘 작동합니다. 
  
<img src="/assets/new_database.png" width="90%" height="150px" title="제목" alt=""/> 
<img src="/assets/new_table.png" width="90%" height="90%" title="제목" alt=""/> 
<img src="/assets/final.png" width="90%" height="90%" title="제목" alt=""/> 
  

