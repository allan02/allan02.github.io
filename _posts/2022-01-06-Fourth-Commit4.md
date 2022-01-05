---
layout: post
title: "Vagrant CentOS 7에 MySQL 설치하기"
author: "Jaehyeon"
---

<h3>먼저 제 블로그의 'Vagrant에 MySQL 설치하기'를 참고하여 환경을 세팅합니다.</h3>

<h4>그 후, 다음 코드들을 따라가시면 됩니다.</h4>

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
<img src="/assets/second/user_pr.png" width="90%" height="90%" title="제목" alt=""/>
<br>-> User를 할당해 준 코드입니다.<br>
<img src="/assets/second/final.png" width="90%" height="90%" title="제목" alt=""/><br>

