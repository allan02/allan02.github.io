---
layout: post
title: "로컬에서 MySQL 설치하기"
author: "Jaehyeon"
---

<h3>What? 무엇을 진행하나요?</h3>

로컬에서 Vagrant를 이용해 MySQL을 설치할 예정입니다.


<h3>Why? 왜 Vagrant를 이용하나요?</h3>

가상화 기술로 손쉽게 개발 환경을 구축할 수 있기 때문입니다.


<h4>여기서 잠깐! Vagrant란?</h4>


Vagrant는 포터블 가상화 소프트웨어 개발 환경의 생성 및 유지보수를 위한 오픈 소스 소프트웨어 제품의 하나입니다.


Vagrant는 독립적으로 사용되는 도구가 아니며, 가상 머신을 생성하거나 조작하는 기능을 직접 제공하지는 않습니다. 


Vagrant에는 Provider라는 개념이 있어서 VirtualBox, VMWare, Docker, Hyper-V와 같은 도구들을 가상 머신을 관리하는 도구로 조합해서 사용할 수 있습니다.


<h4>Vagrant를 사용하기 위해 다음 코드들을 순차적으로 Mac 터미널에서 입력해 주시면 됩니다.</h4>

<h3>How? 제 코드를 따라오시면 됩니다.</h3>

<ul>

  
<li>/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"<br>
  (brew를 설치합니다.)
  </li><br>

  
<li>brew install --cask visual-studio-code<br>
  (visual studio code를 설치합니다.)</li><br>

  
<li>brew install virtualbox virtualbox-extension-pack vagrant<br>
  (vagrant 사용을 위해 virtualbox를 설치합니다.)
  </li><br>

  
<li>mkdir -p ~/workspace/VM/centos7<br>
(Vargant 관련 파일이 만들어질 작업 폴더를 구성합니다.)</li><br>

  
<li>cd /workspace/VM/centos7<br>
  (해당 워크스페이스로 이동합니다.)
  </li><br>

  
<li>vagrant plugin install vagrant-vbguest<br>
  (vagrant를 통해 provisioning하는 가상머신의 모든 기능을 사용하기 위해 plugin을 설치합니다.)
  </li><br>

  
<li>vagrant init<br>
(vagrant 초기파일을 생성합니다.)</li><br>

  
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
        - centos
    - name: ssh 내용 추가
      authorized_key:
        user: "{{ item }}"
        state: present
        key: "{{ lookup('file', '~/.ssh/id_rsa.pub') }}"
      with_items:
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

  
<h3>Vagrant 가상 이미지 실행 및 SSH 접근</h3>

다음 코드들을 참고하시면 됩니다.

<ul>

<li>ssh-keygen<br>
  (SSH 접근을 위해 SSH 공개키를 먼저 만들어 줍니다.)</li><br>

  
<li>이렇게 만들어진 공개키 접속하려는 서버에 등록하면 됩니다.</li><br>
  
<li>vagrant ssh<br>
  (vagrant 명령어를 통해 SSH로 접근합니다. vagrant up 명령어가 선행 되어야 합니다.)
  </li><br>
  
<li>cat ~/.ssh/authorized_keys<br>
  (인증키가 제대로 등록 되었는지 확인하는 코드입니다.)</li><br>

</ul>  

Vagrant 가상 이미지 실행 및 SSH 접근이 올바르게 수행된 모습입니다.

<img src="/assets/ssh_final.png" width="90%" height="150px" title="제목" alt=""/><br><br><br>




<h3>지금부터 2가지 방법으로 MySQL을 설치해 보겠습니다.</h3>

<h3>첫 번째로 yum을 이용하여 MySQL을 설치해 보겠습니다.</h3>

<h4>다음 코드들을 순차적으로 CentOS 환경에서 입력해 주시면 됩니다.</h4>

<ul>
  
<li>su root<br>
  (<strong>you need to be root to perform this command</strong> 에러 메시지를 방지 하기 위해 root로 이동합니다.)
  </li><br>
  
<li>yum -y install http://dev.mysql.com/get/mysql57-community-release-el7-11.noarch.rpm<br>
(MySQL 5.7을 설치합니다.)</li><br>

  
<li>yum -y install mysql-community-server</li><br>

  
<li>systemctl start mysqld<br>
(MySQL을 실행합니다.)</li><br>

  
<li>vi /var/log/mysqld.log<br>
(MySQL을 실행하면 임시 비밀번호가 생성되고 mysqld.log 파일 안에서 임시 비밀번호를 확인 할 수 있습니다.)<br>
<img src="/assets/first_password.png" width="90%" height="90%" title="제목" alt=""/>
  </li><br>
  
  
<li>ALTER USER 'root'@'localhost' IDENTIFIED BY '새 비밀번호';<br>
  (비밀번호를 재설정 합니다.)
  <img src="/assets/new_password.png" width="90%" height="90%" title="제목" alt=""/>
  </li><br>
  
<li>FLUSH PRIVILEGES;</li><br>

</ul>

지금까지 잘 따라오셨다면, 가상머신에서 MySQL이 이상 없이 잘 작동합니다. 
  
<img src="/assets/new_database.png" width="90%" height="150px" title="제목" alt=""/> 
<img src="/assets/new_table.png" width="90%" height="90%" title="제목" alt=""/> 
<img src="/assets/final.png" width="90%" height="90%" title="제목" alt=""/><br><br>




<h3>두 번째로 wget을 이용하여 직접 압축 파일을 다운로드하고, 이를 통해 MySQL을 설치해 보겠습니다.</h3>

 
<ul>

  
<li>sudo yum install wget<br>
  (Wget 명령어 실행을 위해 라이브러리를 설치합니다.)
  </li><br>

  
<li>https://downloads.mysql.com/archives/community/로 이동합니다.<br>
  (원하는 버전의 다운로드를 마우스 우클릭 하여, 링크 주소를 복사합니다.)
  <img src="/assets/second/link_copy.png" width="90%" height="90%" title="제목" alt=""/>
  </li><br>

  
<li>wget https://downloads.mysql.com/archives/get/p/23/file/mysql-8.0.26-1.el7.x86_64.rpm-bundle.tar<br>
  (wget을 통해 다운로드를 진행합니다.)
  <img src="/assets/second/down_zip.png" width="90%" height="90%" title="제목" alt=""/>
  </li><br>
  
<li>tar -xvf mysql-8.0.26-1.el7.x86_64.rpm-bundle.tar<br>
  (명령어를 통해 압축을 풀어줍니다.)
  </li><br>
  
<li>sudo yum localinstall mysql-community-*<br>
  (yum을 사용하여 의존성 설치를 진행합니다.)
  </li><br>
  
<li>sudo systemctl start mysqld<br>
  (sudo systemctl status mysqld 코드를 통해 확인해 보시면, SQL이 정상적으로 작동하는 것을 알 수 있습니다.)
  <img src="/assets/second/ok_mysql.png" width="90%" height="90%" title="제목" alt=""/>
  
  </li><br>
  
<li>sudo systemctl stop mysqld<br>
  (다음 코드 실행을 위해 잠시 MySQL을 종료시킵니다.)
  </li><br>
  
<li>sudo systemctl set-environment MYSQLD_OPTS="--skip-grant-tables" <br>
  (무제약모드로 MySQL을 실행하기 위해 작성한 코드입니다. -> 이렇게 해야 root에 접근하여 user를 할당할수있습니다.)
  </li><br>
 
</ul>


여기까지 잘 따라오셨다면, MySQL이 이상 없이 동작합니다. 다음 사진들은 User를 할당해 준 코드입니다.
<img src="/assets/second/test_mysql.png" width="90%" height="90%" title="제목" alt=""/><br>
<img src="/assets/second/create_test.png" width="90%" height="90%" title="제목" alt=""/><br>

<br>-> User를 할당해 준 코드입니다.<br>
<img src="/assets/second/final.png" width="90%" height="90%" title="제목" alt=""/><br><br>


<h3>yum을 이용한 방식과 압축 파일을 다운로드하는 방식의 차이는 무엇이 있을까요?</h3>

yum을 이용할 경우, 설치가 빠르고 편리하다는 장점이 있습니다.<br>

우선 yum은 이전에 사용하던 다음 rpm의 두 가지의 단점을 개선한 방식입니다.<br>

1, 특정 rpm에 의존성 있는 패키지가 있을 경우, 일일이 다운로드해서 의존성 있는 rpm을 설치해야 합니다.<br>
2, rpm 이 update 됐을 경우  update 됐다는 사실을 알기가 어렵습니다. (패키지마다 직접 확인이 필요합니다.)<br>

엄청 좋은 yum... 하지만! yum central repository에 등록되지 않은 패키지라면 다운로드가 불가능합니다.<br>

<h3>최신 버전 다운을 원하거나, yum central repository에 없는 패키지라면,<br><br> wget을 이용해서 직접 다운로드 받도록합시다!</h3><br>

 


