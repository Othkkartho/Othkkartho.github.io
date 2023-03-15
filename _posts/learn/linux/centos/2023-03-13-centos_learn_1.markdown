---
# multilingual page pair id, this must pair with translations of this page. (This name must be unique)
lng_pair: centos
title: CentOS 서버 구축(1)-가상머신에 리눅스 설치 및 기본 설정

# post specific
# if not specified, .name will be used from _data/owner/[language].yml
author: Othkkartho's Workshop
# multiple category is not supported
category: CentOS 서버 구축
# multiple tag entries are possible
tags: [Linux, CentOS 서버 구축]
# thumbnail image for post
img: ":/linux/centos/centos.avif"
# disable comments on this page
#comments_disable: true

# publish date
date: 2023-03-13 22:00:00 +0900

# seo
# if not specified, date will be used.
meta_modify_date: 2023-03-15 22:00:00 +0900
# check the meta_common_description in _data/owner/[language].yml
#meta_description: ""

# optional
# if you enabled image_viewer_posts you don't need to enable this. This is only if image_viewer_posts = false
#image_viewer_on: true
# if you enabled image_lazy_loader_posts you don't need to enable this. This is only if image_lazy_loader_posts = false
#image_lazy_loader_on: true
# exclude from on site search
#on_site_search_exclude: true
# exclude from search engines
#search_engine_exclude: true
# to disable this page, simply set published: false or delete this file
# published: false
---

<!-- outline-start -->

Spring 프로젝트를 리눅스에서 동작할 수 있도록\n
VMware에 CentOS 7을 올리고, 서버를 동작하는데 필요한 Apache(Web Server), Tomcat(WAS), Jenkins(CI), PostgreSQL(DBMS) 등을 다운받는 것이 이번 포스트의 목표입니다.

<!-- outline-end -->

* * *

### 개요
학교에서 현재 데비안(APT)의 우분투와 쿠분투를 이용해 Server, Client를 공부하고 있어 그에 대한 흥미로 리눅스 마스터 자격증에 대해 관심이 생겼습니다.\n
그런데 해당 자격증은 페도라 방식의 RHEL 계열의 CentOS를 사용한다는 내용을 확인했고, 실제로 이쪽 계열도 많이 사용한다는 것을 알게되어 자격증 공부용으로 제가 전에 만든 Spring 프로젝트 패키지를 올려 서비스되는 서버를 구축할 예정입니다.\n\n

그러기 위해서 CentOS ISO를 다운받을 수 있는 마지막 버전인 CentOS 7의 7.8.2009 버전과 Apache, Tomcat, Jenkins, PostgreSQL을 다운 받는 과정과 마지막으로는 서버의 동작까지 열심히 포스트해보겠습니다. \n\n

학교에서 배우는 ubuntu 또한 학교 진도에 맞춰 포스트를 진행할 계획입니다.

### 목차

0. [CentOS 기본 설명](#centos-기본-설명)
1. [CentOS 7 설치](#centos-7-설치)
2. [Apache 설치](#apache-설치)
3. [Tomcat 설치](#tomcat-설치)
4. [Apache & Tomcat 연동](#apache와-tomcat-연동)
5. [Jenkins 설치](#jenkins-설치)
6. [PostgreSQL 설치](#postgresql-설치)

### CentOS 기본 설명
CentOS는 CentOS 프로젝트에서 레드햇과의 제휴로 개발한 OS입니다. 페도라 기반인 RHEL(레드햇 엔터프라이즈 리눅스)와 완벽하게 호환되는 무료 기업용 컴퓨팅 플랫폼을 제공할 목적으로 만들어진 리눅스계열 OS입니다.   \n
2021년, CentOS 8의 지원기간이 2021년 12월 31일까지로 단축되고, 베타격인 CentOS Stream으로 전환이 발표되어 현재는 CentOS 8 이후의 ISO는 공식적으로 없습니다.

### CentOS 7 설치
먼저 CentOS 7을 설치할 가상머신을 먼저 만들어 보겠습니다.
1. Create a New Virtual Machine 클릭
2. I will install the operating system later 클릭
3. Guest operating System의 Linux를 선택하고, Version은 CentOS 7 64-bit 선택 (선택창에서 C를 누르면 좀 더 빠르게 찾을 수 있음)
4. 가상머신의 이름을 정하고, 해당 머신의 설치 위치를 설정
5. 최대 디스크 크기는 우선 20GB로 놓고, Store virtual disk as a single file로 두어 용량을 고정시킴
6. 본인이 원하는 하드웨어 설정을 한 후 Finish 클릭
\n

* * *

일련의 작업이 종료되면 다음으로는 CentOS 7을 설치해 보겠습니다.
1. 본인이 설정한 가상머신에 들어가서 Edit virtual machine settings나 우클릭 후 Settings 클릭
2. CD/DVD에서 본인이 다운로드 받은 ISO 파일을 삽입후 OK 클릭
    - 만약 ISO가 없으신 분들은 [카카오 미러](http://mirror.kakao.com/centos/7.9.2009/isos/x86_64/)나 [네이버 미러](http://mirror.navercorp.com/centos/7.9.2009/isos/x86_64/)에서 빠르게 다운받으실 수 있습니다. 저는 해당 사이트의 CentOS-7-x86_64-Everything-2009.iso를 다운받았습니다.
3. Play virtual machine 클릭
4. Install CentOS 7과 Test this~, Troubleshooting이 있는데 화살표 위 버튼을 한번 눌러 Install CentOS 7을 선택 후 Enter 누름
5. 시간이 좀 지나면 언어 설정 화면이 나오는데 전 English(US) 기본 설정 그대로 Continue를 눌렀습니다.
    - 한국어가 있긴 하지만 어차피 명령어 공부를 겸하기 위해 CUI방식으로 진행할 것이고, 갑자기 알지도 못하는 오류가 날 것을 방지하기 위해 영어로 설정하고 진행했습니다. 만약 한국어로 진행하실 분들은 아래 검색창에 korean 검색 후 설정해 사용하시면 됩니다.
6. 계속 진행을 하면 지역, 소프트웨어, 시스템 설정하는 메뉴가 있는데 대부분 알아서 진행이 됩니다만 Installation Destination은 오류가 떠있을 것입니다. 해당 오류는 클릭해 들어가서 설치 위치가 선택이 되어 있으므로 바로 Done을 눌러도 되고, 본인이 설치하고 싶은 위치를 선택 하고 Done을 누르시면 됩니다. 저는 그것과 Date & Time 부분의 timezone을 서울 시간으로 변경하고, Network & Host name을 선택해 기본 off인 네트워크 연결을 On시킨 후 Begin Installation을 눌러 계속 진행했습니다.
    - 만약 연결을 설정하지 않았다면 추후 [참고 - 네트워크 연결](#centos-7-네트워크-연결)에 내용과 같이 설정해 줄 수 있습니다.
7. 이후 ROOT 비밀번호와 사용자를 만들고 설치가 완료 될 때까지 기다린 후 완료가 되면 Reboot 버튼이 나오면 reboot를 클릭해 주시면 자동으로 reboot 되면서 리눅스가 실행됩니다.
    - User는 만들지 않아도 괜찮습니다.
8. 그럼 localhost에 root, 비밀번호에 root 비밀번호 설정한 것을 입력하면 CUI 환경에서 진행이 됩니다.

### Apache 설치
위의 순서대로 하셨다면 Apache가 설치가 안되있을것입니다. 그럼 Apache를 설치하겠습니다.
1. `# yum install -y httpd` 을 입력해 설치
2. 방화벽 설정
```linux
# firewall-cmd --permanent --add-service=http
# firewall-cmd --permanent --add-service=https
# firewall-cmd --reload
```
3. `systemctl enable httpd` 서비스를 활성화 시키고 부팅시 실행되게 설정
4. `systemctl start httpd` 서비스 시작
5. `ps -ef lgrep httpd` or `systemctl -l status httpd` httpd 실행상태 확인 아래 캡쳐에서 볼 수 있듯이 active에 초록색으로 active (running)이 뜨면 실행이 완료 된 것입니다.
![실행 완료](:/linux/centos/1/status.avif){:data-align="center"}
6. http://centos IP 주소 를 입력해 아래 사진과 같이 Testing 123..이 뜨고, This server is powered by centos. 라는 문구가 뜨면 잘 실행이 된 것을 확인할 수 있습니다.
    - os의 IP 주소는 `hostname -I`를 입력하면 나옵니다.
- 혹시 Apache가 설치되어 있는지 확인하시려면 `yum list installed | grep httpd`를 입력하시면 됩니다.
- Apache의 파일 경로는 /etc/httpd입니다.

### Tomcat 설치
Tomcat 설치도 바로 진행하겠습니다.
1. `yum install -y tomcat*` 으로 tomcat과 의존성 있는 것 또한 설치를 해줍니다.
- 그리고 `yum list installed | grep java`로 현재 OS의 java가 오라클사의 jdk가 아닌 openjdk로 이루어져 있기 때문에 오라클사의 jdk를 다시 설치하도록 하겠습니다. 그리고 Jenkins의 설치를 위해 최신버전인 19가 아닌 Java11 버전을 설치할 것입니다.
2. 우선 여러분들이 java를 다운받으려면 wget를 사용해야 하는데 아마 해당 명령어를 실행할 수 없을 것입니다. 그렇기 때문에 다운에 앞서 `yum install wget`를 이용해 wget를 설치해 주시면 됩니다.
3. [https://www.oracle.com/java/technologies/downloads/#java11-linux](https://www.oracle.com/java/technologies/downloads/#java11-linux)에서 jdk-11.0.18_linux-x64_bin.rpm을 눌러 다운로드를 하고 해당 다운로드 링크를 복사 후, `wget -c 다운로드링크주소` 명령어를 입력해 주시면 설치를 시작합니다.
    - 링크를 붙여넣기 까지 너무 오래걸리면 오라클의 AuthParam 값이 변경되 다운로드 할 수 없기 때문에 바로 복사 붙여넣기 해주시는 것이 좋습니다.
4. 다운이 완료되면 다운 파일명이 보이실 것입니다. 해당 파일명이 저희가 입력하기에 상당히 어렵기 때문에 `mv 다운로드된파일명 바꿀파일명` 을 입력해 파일명을 변경하시길 바랍니다.
    - 파일명을 일정 이상 입력하고, TAB을 누르면 자동완성을 지원합니다.
5. `rpm -ivh jdk파일명`을 입력하면 jdk 설치를 진행합니다.
6. `alternatives --config java` 명령어로 설치된 자바를 확인하실 수 있습니다. 그리고 현재 선택된 java를 변경하시려면 나오시기 전에 해당 번호를 입력하고, Enter를 누르면 적용됩니다.
7. 아래의 명령어로 방화벽 설정을 해줍니다.
```linux
firewall-cmd --permanent --add-port=8080/tcp
firewall-cmd --reload
```
8. `systemctl enable tomcat`으로 서비스를 활성화 시키고 부팅시 실행되게 설정합니다.
9. `systemctl start tomcat`으로 서비스를 실행하고, http://아이피:8080으로 접속하면 익숙한 Tomcat화면이 나오는 것으로 정상 작동을 확인할 수 있습니다.
- Tomcat의 파일 경로는 /usr/share/tomcat 입니다.
- 마찬가지로 Tomcat이 설치되어 있는지 확인하려면 `yum list installed | grep tomcat`을 입력하면 됩니다.

### Apache와 Tomcat 연동
1. `yum install gcc gcc-c++ httpd-devel`을 입력해 gcc, gcc-c++, httpd-devel 패키지를 설치해 mod_jk를 설치할 준비를 합니다.
2. [https://tomcat.apache.org/download-connectors.cgi](https://tomcat.apache.org/download-connectors.cgi)으로 접속해 중간에 있는 Tomcat Connectors JK 중 Unix 계열 용 tar.gz를 우클릭으로 링크를 복사해 `wget -c 링크주소` 로 파일을 다운받습니다.
3. `tar zxvf tomcat-connector*`으로 압축을 풀어줍니다.
4. ls로 풀린 디렉토리를 확인하고, 생성된 디랙토리 안의 native 디렉토리로 들어갑니다.
5. `./configure --with-apxs=/usr/bin/apxs` 로 Makefile을 생성합니다.
6. `make` 명령어로 컴파일을 진행 후, `make install` 로 설치 합니다.
7. 설치 완료 후 root 파일로 돌아가 /etc/httpd/modules 경로에 mod_jk.so 파일이 생성되었음을 확인할 수 있습니다.
8. Selinux의 보안 설정 변경을 위해 `chcon -u system_u -r object_r -t httpd_modules_t /etc/httpd/modules/mod_jk.so`를 실행합니다.
9. `vi /etc/httpd/conf/httpd.conf`로 파일을 연 후 /을 누르고 LoadModule을 찾아 LoadModule jk_module modules/mod_jk.so 를 추가합니다.
10. `vi /etc/httpd/conf.modules.d/mod_jk.conf`로 설정 파일을 연 후 아래의 내용을 추가합니다.
```text
<IfModule mod_jk.c>
    # worker configuration file location
    JkWorkersFile conf/workers.properties
    # shared memory file location    
    JkShmFile run/mod_jk.shm     
    # log location
    JkLogFile logs/mod_jk.log     
    # log level setting     
    JkLogLevel info     
    # specifies the time format to use for log formatting     
    JkLogStampFormat "[%y %m %d %H:%M:%S] " 
</IfModule>
```
    - Esc와 :wq를 차레로 눌러 저장하고 빠져 나옵니다.


### Jenkins 설치


### PostgreSQL 설치


### 참고
#### Apache와 Tomcat을 연동한 이유


#### CentOS 7 네트워크 연결


#### Web Server와 WAS의 차이점


#### Jenkins는 어떤 역할을 할까?


### 참고 자료
1. [CentOS - 위키백과](https://ko.wikipedia.org/wiki/CentOS)
2. 