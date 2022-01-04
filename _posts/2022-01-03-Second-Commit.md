---
layout: post
title: "Vagrant에 Mysql 설치"
author: "Jaehyeon"
---

<h3>오늘은 **Vagrant에 Mysql를 설치해 보겠습니다.**</h3>


<h4>Vagrant란?</h4>


Vagrant는 포터블 가상화 소프트웨어 개발 환경의 생성 및 유지보수를 위한 오픈 소스 소프트웨어 제품의 하나입니다.


Vagrant는 독립적으로 사용되는 도구가 아니며, 가상 머신을 생성하거나 조작하는 기능을 직접 제공하지는 않습니다. 


Vagrant에는 Provider라는 개념이 있어서 VirtualBox, VMWare, Docker, Hyper-V와 같은 도구들을 가상 머신을 관리하는 도구로 조합해서 사용할 수 있습니다.


<h4>Vagrant를 사용하기 위해 다음 코드들을 순차적으로 Mac 터미널에서 입력해 주시면 됩니다.</h4>


<ul>

  
<li>/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"</li>

  
<li>brew install --cask visual-studio-code</li>

  
<li>brew install virtualbox virtualbox-extension-pack vagrant</li>

  
<li>mkdir -p ~/workspace/VM/centos7</li>
(Vargant 관련 파일이 만들어질 작업 폴더를 구성합니다.)

  
<li>cd /workspace/VM/centos7</li>

  
<li>vagrant plugin install vagrant-vbguest</li>

  
<li>vagrant init
(초기파일을 생성합니다.)</li>

  
<li>touch init.yml</li>
  
  
</ul>


그 후, vi 에디터를 통해 생성된 Vagrantfile과 init.yml을 다음과 같이 수정해줍니다.


<h4>Vagrantfile</h4>


```c
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

```c
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
    - name: ssh내용 추가
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

그 후, `vagrant up` 명령어를 통해 가상 머신을 실행시킵니다.














`시작이 반이다!`
