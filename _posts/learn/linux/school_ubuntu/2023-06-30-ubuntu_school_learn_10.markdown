---
# multilingual page pair id, this must pair with translations of this page. (This name must be unique)
lng_pair: Ubuntu
title: Ubuntu 서버 구축(10)-원격 접속 서버

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
date: 2023-06-30 22:00:00 +0900

# seo
# if not specified, date will be used.
# meta_modify_date: 2023-04-14 17:00:00 +0900
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

텔넷, SSH, VNC 서버의 작동 개념을 이해햐고, 설정법을 익히는 포스트입니다.

<!-- outline-end -->

* * *

### 목차

1. [텔넷 서버](#텔넷-서버)
2. [SSH 서버](#ssh-서버)
3. [VNC 서버](#vnc-서버)

### 텔넷 서버
- 텔넷 서버
    - 리눅스에서 원격 접속을 위해 리눅스 서버에 텔넷 서버를 설치하고, 원격지 PC는 텔넷 클라이언트 프로그램을 설치해야 합니다.
    - 전통적인 원격 접속 방법인 텔넷은 보안에 취약해 최근에는 보안 기능을 추가해 사용합니다.
    - 클라이언트에서 리눅스 서버에 접속하면, 서버에서 직접 텍스트 모드로 작업하는 것과 동일하게 작업할 수 있습니다.

1. `dpkg -l telnetd`로 텔넷 서버 패키지가 설치되어 있는지 확인 후 설치되어 있지 않다면 `apt -y install xinetd telnetd`로 관련 패키지를 설치합니다.
2. `cd /etc/xinetd.d` 폴더로 이동해, `touch telnet`으로 빈 파일을 생성합니다.
3. `vi telnet`으로 아래 내용 입력 후 저장합니다.
```
service telnet
{
    disable = no
    flags = REUSE
    socket_type = stream
    wait = no
    user = root
    server = /usr/sbin/in.telnetd
    log_on_failure += USERID
}
```
4. `adduser teluser`로 사용자를 만듭니다.
5. `systemctl restart xinetd`로 텔넷 서비스를 가동, `systemctl enable xinetd`로 컴퓨터를 재부팅해도 텔넷 서비스가 가동되도록 설정한 후 `systemctl status xinetd`로 가동 여부를 확인해보면 Active: active가 되어 있는것을 확인할 수 있습니다. 확인 완료 후 q를 누르면 프롬프트가 나타납니다.
6. `ufw allow 23/tcp`로 텔넷 23번 포트의 방화벅을 열고, `ifconfig`를 통해 본인의 IP 주소를 확인합니다.
7. 자신의 컴퓨터에서 `telnet 서버IP주소`로 telnet 서버로 접속해 보면 'server login' 창이 뜨고, 거기에서 teluser와 비밀번호를 입력해 들어가면 정상적으로 진입되는 것을 확인할 수 있습니다.
8. 이번에는 클라이언트로 들어가 터미널을 엽니다. 거기서 `ping -c 3 서버IP주소`를 입력해 Server와 네트워크로 연결되어 있는지 확인합니다.
9. 이후 `telnet 서버IP주소`로 텔넷에 접속을 시도하고, 사용자 이름에 teluser, 비밀번호에 본인이 정한 비밀번호를 입력하면 정상적으로 접속되는 것을 확인할 수 있습니다.
10. 참고로 root사용자로 들어가려 하면 접속되지 않을 것입니다. 그 이유는 만약 root 사용자의 텔넷 접속을 허용하게 되면 시스템 보안에 문제가 발생할 수 있기 때문엡니다.
11. 만약 root 사용자로 접속하고 싶다면, Server에서 `mv /etc/securetty /etc/securetty.bak` 명령을 입력해 우분투에 있는 보안 항목을 잠시 치워두면 접속이 가능해집니다.

**xinetd가 active된 화면**{:data-align="center"}
![xinetd가 active된 화면](:/linux/ubuntu/10/xinetd_active.png){:data-align="center"}
**Client에서 텔넷 서버에 접속한 모습**{:data-align="center"}
![Client에서 텔넷 서버에 접속한 모습](:/linux/ubuntu/10/teluser.png){:data-align="center"}

### SSH 서버
- SSH 서버
    - 텔넷의 테이터 전송시 암호화하지 않아 해킹에 위험을 해결하기 위해 리눅스에서는 OpenSSH 서버를 지원합니다.
    - OpenSSH 서버는 텔넷 서버와 비슷하지만 데이터를 전송할 때 패킷을 암호화해 전송합니다.

1. `apt -y install openssh-server`로 SSH 서버를 설치합니다.
2. 텔넷때와 마찬가지로 `systemctl restart ssh`, `systemctl enable ssh`, `systemctl status ssh`를 차례로 눌러 서비스 재가동, 상시 가동 허용, 가동 여부 확인을 진행합니다.
3. `ufw allow 22/tcp`로 SSH가 사용하는 22번 포트의 방화벽을 허용하고, `ifconfig`로 서버 주소를 확인합니다.
4. Client를 열어 터미널을 엽니다. 그 후 `ssh teluser@IP주소`를 입력하고, 접속이 확실한지 물어보면 yes를 입력합니다. 이후로는 텔넷과 완전히 동일하게 사용이 가능합니다.

**Client에서 SSH 서버에 접속한 모습**{:data-align="center"}
![Client에서 SSH 서버에 접속한 모습](:/linux/ubuntu/10/ssh.png){:data-align="center"}

### VNC 서버
- VNC 서버
    - 그래픽 모드로 원격 관리를 지원하는 서버입니다.
    - 텍스트만 전송하는 텔넷이나 SSH 서버와 비교하면 속도가 많이 느린것이 단점입니다.

1. 설정창에 공유를 선택합니다.
2. 해당 공유 화면에서 원격 데스크톱을 누르고, 원격 데스크톱을 설정, Enable Legacy VNC Protocol 선택 후 옆의 설정에 암호  필요 클릭, 원격 조작 허용, 맨 밑에 인증에 암호를 본인이 원하는 암호로 바꾸고, X를 클릭해 창을 닫으면 원격 데스크톱이 켬으로 변경됩니다.
3. 그 상태에서 `ufw allow 5900/tcp`로 VNC에서 사용하는 5900포트를 열고, `ifconfig`로 Server Ip주소를 확인합니다.
4. Client로 들어가서 터미널에 `sudo apt -y install xtightvncviewer`를 입력해 설치하고, `vncviewer 서버IP주소를 입력해 Server에 접속합니다. 그럼 비밀번호를 치라고 나오는데 거기에 원격 데스크톱에 작성했던 비밀번호를 입력하면 해당 환경으로 들어가는 것을 확인할 수 있습니다.

**설정에서 공유 선택**{:data-align="center"}
![설정에서 공유 선택](:/linux/ubuntu/10/vnc1.png){:data-align="center"}
**원격 데스크톱 허용**{:data-align="center"}
![원격 데스크톱 허용](:/linux/ubuntu/10/vnc2.png){:data-align="center"}
**Client에서 서버에 들어가짐**{:data-align="center"}
![Client에서 서버에 들어가짐](:/linux/ubuntu/10/vnc3.png){:data-align="center"}

### 참고
#### 텔넷 서버, SSH 서버, VNC 서버의 비교

| 구분 | 텔넷 서버 | SSH 서버 | VNC 서버 |
| :--: | :--: | :--: | :--: |
| 접속 속도 | 빠름 | 빠름 | 느림 |
| 그래픽 지원 | X | X | O |
| 보안 | 취약 | 강함 | 취약하지만 SSH와 연동하여 보완 가능 |
| 사용 가능 명령 | 텍스트 | 텍스트 | 텍스트와 그래픽 환경 |

### 참고 자료
1. [리눅스 실습 for Beginner](https://www.hanbit.co.kr/store/books/look.php?p_code=B7654754187)
