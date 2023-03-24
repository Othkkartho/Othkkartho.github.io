---
# multilingual page pair id, this must pair with translations of this page. (This name must be unique)
lng_pair: Ubuntu
title: Ubuntu 서버 구축(1)-가상머신에 리눅스 설치 및 기본 설정

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
date: 2023-03-14 22:00:00 +0900

# seo
# if not specified, date will be used.
meta_modify_date: 2023-03-17 13:15:00 +0900
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

학교에서 공부하는 Ubuntu 서버를 정리하는 포스트입니다.

<!-- outline-end -->

* * *

### 개요
학교에서 사용하는 Ubuntu 서버의 경우 [리눅스 실습 for Beginner](https://www.hanbit.co.kr/store/books/look.php?p_code=B7654754187)에서 제공하는 학습을 따라가기 때문에 Ubuntu 18.04 Desktop, Server와 Kubuntu 18.04 버전을 사용하지만 포스트에서는 현재 포스트를 쓸 때 최신 버전인 22.04 버전을 기준으로 학교의 수업을 따라 포스트를 작성하겠습니다.   
만약 학교의 내용과 버전 때문에 다른 점이 있다면 제 PC에서는 22.04를 사용하고, 노트북에서는 18.04로 수업을 따라가고 있기 때문에 참고에서 해당 내용을 다루도록 하겠습니다.   \n
Ubuntu Desktop, Server 버전의 경우 둘 다 서버로 사용하고, Kubuntu로 클라이언트로 사용할 예정입니다.

### 목차

0. [Ubuntu, Kubuntu 기본 설명](#ubuntu-kubuntu-기본-설명)
1. [Ubuntu Desktop 설치](#ubuntu-설치)
2. [Ubuntu Server 설치](#ubuntu-server-설치)
3. [Kubuntu 설치](#kubuntu-설치)

### Ubuntu, Kubuntu 기본 설명
데비안 리눅스를 Fork해 개발되며 데비안에 비해 사용 편의성에 초점을 맞춘 리눅스 배포판입니다.   
일반적으로 매 6개월마다 새로운 판이 공개되며, 장기지원판(LTS)은 2년에 한번씩 출시됩니다.   
Ubuntu의 경우 Desktop 버전은 그놈의 어플리케이션 매니저 GUI 환경을 사용하고 있으며, Server 버전은 CUI 방식을 사용하고 있습니다.   \n

Kubuntu는 우분투를 기반으로 KDE(다국적 자유 소프트웨어 커뮤니티) 데스크톱 환경을 사용하는 리눅스 배포판입니다.   
쿠분투는 우분투 프로젝트의 일부분이며 같은 기반 시스템을 사용합니다.   
우분투의 모든 기능을 포함하지만 KDE Desktop을 기반으로 하여 반년마다 새 릴리스가 나오며, KDE 버전에 맞춰 제공합니다.

### Ubuntu 설치
우분투 데스크톱은 우분투 공식 홈페이지에 [데스크톱 다운로드](https://ubuntu.com/download/desktop)에서 다운받을 수 있고, 우분투 서버는 공식 홈페이지에 [서버 다운로드](https://ubuntu.com/download/server)에서 ISO 파일을 다운로드 받으실 수 있습니다. 해당 포스트에서는 22.04.2 LTS를 다운받아 진행하고 있습니다.

#### Ubuntu Desktop 설치
GUI로 동작하는 서버를 VMware에 설치합니다.

1. VMware를 실행 후 CentOS를 깔았던 것과 같이 Server라는 가상 환경을 만들고 ubuntu-desktop ISO를 넣으시면 됩니다.
2. 지역성 설정과 디스크 설정, 유저 설정이 완료되면 설치가 진행 되고, 설치가 완료되면 지금다시 시작을 클릭해 재부팅합니다.
3. DVD 장치 제거 후 Enter 메시지가 나오면 그냥 Enter를 누르면 VMware가 자동으로 DVD를 제거합니다.
4. 해상도 설정은 오른쪽 위의 종료 버튼이 있는 부분을 클릭하면 설정 아이콘을 클릭합니다. 디후 디스플레이에서 해상도 1024x768을 선택한 후 Tab을 세 번 누른 다음 Enter를 누릅니다.
5. 실습 환경이 변경되지 않도록 프로그램 표시 아이콘을 클릭 후 소프트웨어 & 업데이트 아이콘을 클릭해 업데이트의 업데이트 관련 설정을 끄거나 최소화 합니다.
6. root 사용자를 활성화 하기 위한 설정을 진행합니다.
    - 바탕화면에서 우클릭 후 테미널 열기를 선택합니다.
    - `sudo su - root` 명령어로 사용자를 관리자로 변경합니다.
    - `passwd`를 통해 root의 암호를 설정합니다.
    - `gedit /etc/gdm3/custom.conf` 를 통해 설명 아래 작성한 설정을 저장 후 gedit을 닫습니다.
    - `gedit /etc/pam.d/gdm-password` 로 파일을 열어 3행 앞에 #을 붙여 주석 처리 후 저장합니다.
    - `gedit /etc/pam.d/gdm-autologin` 으로 파일을 열어 3행 앞에 #을 붙여 주석 처리 후 저장합니다.
    - `gedit /root/.profile` 로 맨 아래의 ``앞에 #을 붙여 주석 처리 후 gedit을 닫습니다.
    - reboot 명령으로 서버를 재부팅합니다.
```text
10행 주석제거: AutomaticLoginEnable = true
11행 주석제거 및 설정 변경: AutomaticLogin = root
19행 추가: AllowRoot = true
```
7. root 사용자로 앞으로 공부를 위한 기능을 설정합니다.
    - 자동으로 root로그인이 되면 6번까지의 과정을 잘 따라 오신겁니다. 이후 터미널을 열고, `cd /etc/apt`, `mv sources.list sources.list.back`, `ls`를 통해 원래 있던 sources 파일을 백업 파일로 만들고, 잘 적용 되었는지 확인합니다.
    - `wget http://download.hanbit.co.kr/ubuntu/18.04/sources.list`로 저자가 미리 만들어 놓은 list 파일을 다운로드 하고, `apt-get update`를 통해 적용합니다.
    - `ufw enable`로 방화벽을 켭니다.
    - `apt-get -y install vim net-tools` 로 앞으로 실습에 필요한 프로그램을 미리 설치합니다.
    - 터미널에서 `halt -p` or `init 0`, `shutdown -h 0`, `poweroff` 명령으로 시스템을 종료합니다.
    - VMware에서 Server 가상 머신을 선택한 후 세팅에 CD/DVD를 선택 후 Connect at power on의 체크를 해제하고, Use physical drive를 선택해 설치가 완료된 iso 대신 설치한 OS가 동작하도록 설정합니다.

### Ubuntu Server 설치
CUI로 동작하는 Server를 설치합니다.

1. Server(B) 이름의 가상 환경을 만들어 ubuntu-server ISO를 넣고, 동작시킵니다.
2. 부팅 후 지역성과 설치에 필요한 디스크 설정 및 유저 설정을 완료 후 재부팅을 시킵니다.
3. 컴퓨터가 다시 켜지면 로그인하라는 곳에 생성한 이름, 비밀번호에 설정한 비밀번호를 입력하고, Enter를 누릅니다.
    - 보안상의 이유로 Password에 입력해도 입력하는지 보이지 않습니다.
4. 이후 실습을 위해 Ubuntu 소프트웨어 설치와 관련된 설정을 진행합니다.
    1. `cd /etc/apt`로 소프트웨어 설치 관련 디렉터리로 이동한 후 `ls`로 sources.list 파일이 있는지 확인합니다.
    2. `sudo mv sources.list sources.list.bak` 으로 sources.list 파일 이름을 변경합니다.
    3. `sudo wget http://download.hanbit.co.kr/ubuntu/18.04/sources.list`로 저자가 만든 새로운 sources.list 파일을 받아옵니다.
    4. `sudoapt-get update`로 명령을 설정합니다.
        - 이는 앞으로 우분투에서 패키지를 설치할 때 sources.list 파일의 지정된 사이트에서 다운로드 및 설치하도록 설정하는 것입니다.
    5. root 사용자를 활성화합니다.
        - `sudo su - root`로 root로 사용자를 변경하고, `passwd`를 통해 root 사용자의 비밀번호를 설정합니다.
    6. `ufw enable`로 방화벽을 키는 것으로 설정을 완료했기 때문에 `init 0` 명령어로 컴퓨터를 종료합니다.
    7. 마찬가지로 Connect at power on의 체크를 해제하고, ISO파일 사용에서 물리드라이브를 사용하도록 변경합니다.
    - 혹시 해상도를 변경하고 싶으신 분들은 아래 [참고](#ubuntu-server에서-해상도-변경)에 작성해 놨습니다.

### Kubuntu 설치
클라이언트로 사용할 kubuntu-desktop을 설치합니다.

1. Client 가상 환경을 만들고, kubuntu-desktop ISO를 넣고, 실행합니다.
2. 쿠분투 또한 지역성과 디스크, 사용자 설정들을 마치고, 재시작하면 쿠분투가 잘 설치되고, 들어가 지는 것을 확인할 수 있습니다.
3. 우선 프로그램 실행기 아이콘을 클릭한 후 시스템 설정에 들어가서 디스플레이와 모니터의 해상도를 1024x768로 바꾸고, 아래의 적용을 누릅니다.
4. 위의 ubuntu 서버와 마찬가지로 `cd /etc/apt`, `ls`로 sources.list 파일을 확인하고, `mv sources.list sources.list.bak`으로 원래 파일 이름을 변경합니다.
5. `sudo wget http://download.hanbit.co.kr/ubuntu/18.04/sources.list`로 새 sources.list를 다운로드 하고, `sudo apt-get update`로 설정을 완료 후 `exit`으로 터미널을 닫습니다.
6. 설치와 기본 설정이 완료 되었기 때문에 터미널에서 `init 0`으로 시스템을 종료하거나, 떠나기 버튼 클릭 후 종료를 눌러 시스템을 종료합니다.
7. 마찬가지로 CD/DVD의 Connect at power on 체크 해제 후 Use physical drive를 선택하고, 종료합니다.

혹시 실습하다가 문제가 생길지도 모르니 모든 가상머신들을 백업 폴더를 만들어 복사 붙여넣기로 백업해 놓으시기 바랍니다.

### 참고
#### 우분투와의 비교
커널과 코어는 둘 모두 리눅스 커널과 우분투 코어를 사용합니다.   
그래픽은 X.Org 서버, 사운드는 펄스오디오, 멀티미디어는 G스트리머, 오피스 제품군은 리브레오피스, 이메일과 PIM은 선더 버드를 둘 모두 사용합니다.

| 소프트웨어 | 우분투 | 쿠분투 |
| :---: | :---: | :---: |
| 데스크톱 | 그놈 | 플라즈마 데스크톱 |
| 기본 툴킷 | GTK+, Nux & Qt | Qt |
| 브라우저 | 파이어폭스 | 13.10까지는 리콩트, 14.04부터는 파이어 폭스 |

#### Ubuntu Server에서 해상도 변경
1. `cd /etc/default/`에 gurb 파일이 있는지 ls로 확인한 후 `sudo vi grub`로 grub을 편집기로 엽니다.
2. 파일이 화면에 출력되면 ESC로 명령 모드로 들어간 후 a나 i를 눌러 편집 모드로 들어갑니다.
    - 10행에 `GRUB-CMDLINE_LINUX_DEFAULT="머신마다 다를 수 있음"` 텍스트를 "nomodeset"으로 수정합니다.
    - 12행에 `GRUB_GFXPAYLOAD_LINUX=800x600`을 추가해 해상도를 설정합니다. 800x600은 원하는 해상도 값을 넣으셔도 되며 x는 소분자입니다.
3. 입력을 마친 후 ESC를 누르고 :wq를 입력 후 ENTER를 눌러 저장하고 편집기를 종료합니다.
4. 설정 변경을 모두 완료했으면 `reboot`로 재시작합니다.

#### 망가진 고정 패키지가 있습니다. 오류 해결법
apt를 이용해 패키지 설치시 오류가 발생했는데 오류 내용은 아래와 같습니다.
```
다음 패키지의 의존성이 맞지 않습니다:
vim : 의존: vim-common(=2:8.0.1454-1ubuntu1) 하지만 ~
E: 문제를 바로잡을 수 없습니다. 망가진 고정 패키지가 있습니다.
```
즉 vim 의존 패키지 중 vim-common이 제대로 설치되지 않았다는 것을 의미합니다. 이런 경우 `apt -y install vim-common=2:8.0.1454-1ubuntu1`을 입력해 직접 의존성 패키지를 설치해 줍니다.   
그 후 원래 설치하려고 했던 vim을 `apt install vim`을 입력해 다시 설치합니다.   
\n
저는 이 단계에서 바로 되었는데 참고한 블로그에서 여전히 오류가 난다면 `apt install vim.*`를 실행한 뒤 다시 위와 같은 과정을 반복하라고 나와 있어 적어놓습니다.

#### apt vs  apt-get
apt와 apt-get은 어떤걸 써도 내부 동작에 차이는 거의 없습니다.  
다만 apt-get에서는 옵션들이 많아지다보니 apt에서는 자주 사용하는 옵션들을 추출해 사용자들이 사용하고 보기 편하게 만들어 apt가 더 예쁘고 추가적인 정보를 출력해 준다고 합니다.

### 참고 자료
1. [우분투 - 위키백과](https://ko.wikipedia.org/wiki/%EC%9A%B0%EB%B6%84%ED%88%AC)
2. [쿠분투 - 위키백과](https://ko.wikipedia.org/wiki/%EC%BF%A0%EB%B6%84%ED%88%AC)
3. [우분투 APT-GET 망가진 고정 패키지가 있습니다 해결방법-dololak](https://dololak.tistory.com/114)
4. [apt와 apt-get 차이점-나도 개발자다](https://ksbgenius.github.io/linux/2021/01/13/apt-apt-get-difference.html)