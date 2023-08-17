---
# multilingual page pair id, this must pair with translations of this page. (This name must be unique)
lng_pair: Ubuntu
title: Ubuntu 서버 구축(11)-웹 서버와 FTP 서버

# post specific
# if not specified, .name will be used from _data/owner/[language].yml
author: Othkkartho's Workshop
# multiple category is not supported
category: Ubuntu 서버 구축
# multiple tag entries are possible
tags: [Linux, Ubuntu 서버 구축]
# thumbnail image for post
img: ":/linux/ubuntu/ubuntu.avif"
# disable comments on this page
#comments_disable: true

# publish date
date: 2023-07-01 22:00:00 +0900

# seo
# if not specified, date will be used.
meta_modify_date: 2023-08-17 22:00:00 +0900
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

APM의 개요를 알아보고 설치하며, XE를 활용한 웹 사이트를 구축해 보고, FTP 서버를 설치하고 운영해 보는 포스트입니다.

<!-- outline-end -->

* * *

### 목차

1. [APM의 개요, 설치](#apm의-개요-설치)
2. [XE를 활용한 웹 사이트 구축](#xe를-활용한-웹-사이트-구축)
3. [FTP 서버 설치와 운영](#ftp-서버-설치와-운영)

### APM의 개요, 설치
1. 외부에서 웹 서버에 접속하게 하기 위해서는 고정 IP를 할당해야 합니다.
    - 터미널을 열어 `ifconfig ens33` 명령으로 IP 주소의 앞에서 세번째 자리를 확인한 후 `nm-connection-editor` 명령으로 [네트워크 연결] 창을 불러옵니다.
2. 네트워크 연결에서 유선 연결 1의 설정을 누르고, IPv4 칸으로 갑니다.
    - 그럼 그곳에 방식이 자동으로 설정되어 있을탠데 이것을 수동으로 바꾸고, 추가를 눌러 주소에는 '192.168.OOO.100', 넷마스크에는 '255.255.255.0', 게이트웨이에는 '192.168.OOO.2'를 입력합니다. 여기서 OOO에는 전에 확인했던 IP 주소 3번째 자리 수가 들어가면 됩니다.
    - 또한 DNS 서버에는 8.8.8.8을 입력합니다. 다 잘 입력했으면 저장을 누르고 네트워크 연결 창을 닫습니다.
    - 설정을 적용하기 위해 reboot 명령으로 컴퓨터를 재부팅합니다.
3. `dpkg -l apache2`, `dpkg -l php-common`, `dpkg -l mysql-server` 명령으로 APM이 설치되어 있는지 확인해보면 설치되어 있지 않은것을 알 수 있습니다.
4. `sudo apt -y install lamp-server^` 명령으로 아파치, PHP, MySQL을 한 번에 설치할 수 있습니다.
    - 만약 설치가 안되거나 중간에 오류가 뜨면 `apt update` 명령으로 repository를 갱신한 뒤 다시 시도해 보시기 바랍니다.
5. 다 설치가 완료되면 3번 명령어를 한 번 더 쳐보면 정상적으로 설치가 되어 있는것을 확인할 수 있습니다.
6. `systemctl restart apache2`, `systemctl enable apache2`, `systemctl status apache2`로 웹 서비스를 작동하고 확인합니다.
    - 같은 방식으로 mysql도 서비스를 시작하고 상시 작동하도록 설정합니다.
7. 파이어폭스를 실행하고 주소 창에 http://localhost/ 또는 http://127.0.0.1/을 입력합니다. Apache2 Ubuntu Default Page와 같은 창이 나타나면 apache2 서비스가 정상적으로 작동하는 것입니다.
8. `vi /var/www/html/phpinfo.php`로 새 파일에 다음 PHP 코드를 입력하고 저장합니다.
    - `<?php phpinfo(); ?>` 해당 명령어는 웹 서버에 설치된 PHP의 정보를 출력하는 것입니다.
9. 웹 브라우저에 http://localhost/phpinfo.php를 입력했을 때 php 정보창이 나타나면 php 모듈도 정상적으로 작동하는 것입니다.
10. 외부에서 웹 서버에 접근할 수 있도록 `ufw allow` 80 명령으로 포트를 허용합니다. `ifconfig ens33`으로 server 가상머신의 IP 주소를 확인합니다.
11. Client에 접속해 웹 브라우저를 실행하고 http://서버IP주소/phpinfo.php를 입력하면 server에서 봤던 PHP 페이지가 그대로 출력되는 것을 확인할 수 있습니다. 이로써 Apache, PHP, MySQL이 정상적으로 작동하는 것을 확인했습니다.
    - 이제 웹 페이지 또는 php 소스를 /var/www/html/ 디렉터리에 가져다놓으면 웹 사이트를 운영할 수 있습니다.

**APM이 설치 되었는지 확인**{:data-align="center"}
![설치되어 있지 않아 찾지 못하는 것을 확인할 수 있음](:/linux/ubuntu/11/checkapm.jpg){:data-align="center"}

**lamp-server^ 설치 이후 정상적으로 설치 되었는지 확인**{:data-align="center"}
![APM 모두 정상적으로 설치 되었음](:/linux/ubuntu/11/checkapm2.jpg){:data-align="center"}

**apachestatus 확인**{:data-align="center"}
![active 상태](:/linux/ubuntu/11/apachestatus.jpg){:data-align="center"}

**mysql의 상태 확인**{:data-align="center"}
![active 상태](:/linux/ubuntu/11/mysqlstatus.jpg){:data-align="center"}

**Server에서 apache가 정상 작동하는지 확인**{:data-align="center"}
![정상적으로 작동함](:/linux/ubuntu/11/apache2.jpg){:data-align="center"}

**Server에서 PHP가 정상 작동하는지 확인**{:data-align="center"}
![정상적으로 작동함](:/linux/ubuntu/11/php.jpg){:data-align="center"}

**Client에서 Apache와 PHP가 정상 작동하는지 확인**{:data-align="center"}
![정상적으로 작동함](:/linux/ubuntu/11/clientphp.jpg){:data-align="center"}

### XE를 활용한 웹 사이트 구축
XE가 바뀌고 나서 때문인지 정확한 이유는 알 수 없지만 xe의 php가 화면이 뜨지 않는 문제가 발생 문제를 해결하면 포스트 수정하겠습니다.

### FTP 서버 설치와 운영
1. Server를 초기화하고 `apt -y install vsftpd`를 입력합니다.
2. anonymous 사용자 접속 및 파일 업로드를 허용하도록 설정하기 위해 아래 명령어를 실행하고, 작업을 수행합니다.
    - `vi /etc/vsftpd.conf`
    - `annonymous.enable=NO` -> `annonymous.enable=YES`
    - `#write_enable=YES` -> `#write_enable=NO`
    - `#anon_upload_enable=YES` -> `anon_upload_enable=YES`
    - `#anon_mkdir_write_enable=YES` -> `anon_mkdir_write_enable=YES`
3. vsftpd에 anonymous로 접속되는 디렉터리는 /srv/ftp입니다. 이 디렉터리 아래 pub 디렉터리를 만들고 모든 사용자의 r,w 권한을 허용합니다.
    - `cd /srv/ftp`
    - `sudo mkdir pub`
    - `sudo chmod 777 pub`
    - `cd pub`
    - `sudo cp /boot/vm* file1`
    - `ls -l`
4. vsftpd를 systemctl restart, enable, status vsftpd를 차레로 입력해 서비스를 시작합니다.
5. 외부 접근이 되도록 `sudo ufw allow ftp` 명령으로 포트를 허용합니다. 원활한 외부 접속을 허용하기 위해 `systemctl stop ufw`로 잠시 방화벽을 끕니다.
6. `ifconfig ens33`으로 서버 IP 주소를 확인합니다.
7. Client에서 터미널을 열고 `sudo apt -y install filezilla`로 클라이언트를 설치합니다.
8. `filezilla`명령으로 실행하고 접속합니다. 호스트에는 Server의 IP 주소, 사용자명에는 anonymous, 비밀번호는 임의, 포트에는 21을 입력하고 빠른 연결을 클릭하면 아래 파일들이 나옵니다.
9. 로컬 사이트 창의 /home/tony 아래 디렉터리 만들고 그 안으로 이동을 선택해 /home/ubuntu/download 디렉터리를 만듭니다.
10. 위 과정이 완료된 상태에서 pub를 선택해 file1을 이동시켜 보고, client에도 적당한 걸 골라 이동시켜 보면 잘 이동이 되는 것을 확인할 수 있습니다.

#### proftpd 설치와 운영
proftpd는 대형 사이트에서 오랫동안 인기가 많았던 ftp 서버입니다.

1. Server를 초기화하고 `sudo apt -y install proftpd`를 실행합니다.
2. anonymous 사용자 접속 및 파일 업로드를 허용하도록 설정하기 위해 아래 명령어를 실행하고, 작업을 수행합니다.
    - `sudo vi /etc/proftpd/proftpd.conf`
    - `<Anonymous ~ftp>` 부터 `</Anonymous>` 까지 첫 행의 모든 주석 제거
    - `<Directory incomming>` 부터 `</Directory>` 까지 각 행의 주석 제거
    - 약 171행과 181행의 `DenyAll` -> `AllowAll`
3. `sudo systemctl restart proftpd` 명령과 `sudo systemctl enable proftpd`로 서비스를 시작합니다.
4. 이후 방화벽부터의 방법은 FTP 앞 부분과 동일합니다.

### 참고
#### GRANT로 유저 만들기
원래 책에서는 `GRANT ALL ON xeDE.* TO xeUser@localhost IDENTIFIED BY '1234'`로 되어 있지만 위에 [XE를 활용한 웹 사이트 구축](#xe를-활용한-웹-사이트-구축)을 보시면 `create user 'xeUser'@localhost identified by '1234';`와 `grant all on xeDB.* to 'xeUser'@localhost;`로 나눠서 작성한 것을 알 수 있습니다. 이는 [MySQL 8부터 GRANT 명령문으로 유저를 생성하는 것이 막혔기](https://stackoverflow.com/questions/50177216/how-to-grant-all-privileges-to-root-user-in-mysql-8-0) 때문입니다.

### 참고 자료
1. [리눅스 실습 for Beginner](https://www.hanbit.co.kr/store/books/look.php?p_code=B7654754187)
