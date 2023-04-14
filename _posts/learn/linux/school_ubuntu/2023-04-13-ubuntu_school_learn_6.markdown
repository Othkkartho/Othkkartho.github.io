---
# multilingual page pair id, this must pair with translations of this page. (This name must be unique)
lng_pair: Ubuntu
title: Ubuntu 서버 구축(6)-리눅스 패키지 설치와 응급 복구

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
date: 2023-04-13 22:00:00 +0900

# seo
# if not specified, date will be used.
# meta_modify_date: 2023-03-24 15:00:00 +0900
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



<!-- outline-end -->

* * *

### 개요


### 목차

1. [dpkg 명령어에 관해](#dpkg-명령어의-정보)
2. [apt 명령어에 관해](#apt-명령어의-정보)
3. [응급 복구]()
4. [GRUB 부트 로더]()

### dpkg 명령어의 정보
#### dpkg 설명
- 우분투에서 패키지를 설치할 때 가장 많이 사용되는 명령어로 apt 명령어가 나오기 전에 주로 썼습니다.
\n

#### dpkg 명령어 옵션

| 옵션 | 설명 |
| :---: | :---: |
| -i or --install | 패키지 설치 |
| -r or --remove | 설치되어 있는 패키지 삭제(설정 파일은 유지) |
| -P or --purge | 설치되어 있는 패키지와 설정 파일을 모두 삭제 |
| -ㅣ or -L | 설치된 패키지 관련 정보와 설치한 패키지로부터 모든 파일 목록을 보여줌 |
| --info 패키지파일명.deb | deb 파일 정보를 조회하는 옵션 |

\n
#### 예시 명령어
- dpkg -i mc_4.8.27-1_amd64.deb: mc 패키지을 설치합니다.
    - 18.04 버전의 mc 패키지(mc_4.8.19-1_amd64.deb)의 경우 의존성이 없어 설치가 되지만 22.04 mc 최신 패키지인 mc_4.8.27-1_amd64.deb의 경우 mc-data가 의존성으로 존재해 설치 시 오류가 발생합니다.
- dpkg -r mc: mc 패키지를 삭제합니다.
- dpkg -P mc: mc 패키지를 설정 파일까지 삭제합니다.
- dpkg -l mc: 설치된 패키지의 정보를 보여줍니다.
    - 결과: ii  mc             3:4.8.27-1   amd64        Midnight Commander - a powerful file manager
- dpkg -L mc: 패키지가 설치된 파일 목록을 보여줍니다.
- dpkg --info mc_4.8.27-1_amd64.deb: mc패키지의 패키지 이름, 버전, 아키텍쳐, 사이즈, 의존성 등의 패키지 정보를 보여줍니다.
\n
#### dpkg의 단점
- 가장 큰 문제는 mc를 다운로드 받으면서 느끼셨지만 의존성을 알아서 설치해 주지 못합니다. 의존성이 없는 간단한 패키지나, 저장소 문제로 다운로드되지 않는 몇몇 패키지를 다운로드 받을 때를 제외하고는 의존성 문제에 의해 귀찮아질 확률이 높습니다.  
\n
#### dpkg 실습  
1. `wget http://archive.ubuntu.com/ubuntu/pool/universe/m/mc/mc_4.8.27-1_amd64.deb` mc 패키지 설치 바이너리 파일인 mc_4.8.27-1_amd64.deb를 다운로드 받습니다.
2. `wget http://archive.ubuntu.com/ubuntu/pool/universe/n/ncftp/ncftp_3.2.5-2.2_amd64.deb`으로 ncftp 패키지를 다운로드 받습니다.
3. `dpkg -l ncftp`를 입력해 이미 패키지가 설치되어 있는지 확인합니다. `no packages found matching ncftp`라는 오류 메시지가 뜨면 설치가 안되있기 때문에 `dpkg -i ncftp_3.2.5-2.2_amd64.deb`를 통해 ncftp 패키지를 설치합니다.
    - `dpkg --info ncftp_3.2.5-2.2_amd64.deb`를 입력해보면 depends에 `Depends: libc6 (>= 2.15), libncurses6 (>= 6), libtinfo6 (>= 6)`라고 의존성 정보가 나와있는 것을 확인할 수 있습니다.
    - 없는 의존성이 없기 때문에 정상적으로 설치되는 것을 확인할 수 있습니다.
    - 정상적으로 실행되는 것은 [참고](#ncftp)에 작성해 놓았습니다.
4. ncftp가 정상적으로 사용되었음을 확인하였기 때문에 `dpkg -r ncftp`로 ncftp 패키지를 삭제합니다.
5. 다운로드 하기 전 `dpkg --info mc_4.8.27-1_amd64.deb`로 mc 패키지 정보를 확인해 보면 여러 lib의존성과 함께 mc-data 의존성이 있는 것을 확인하실 수 있습니다.
6. 실제로 `dpkg -i mc_4.8.27-1_amd64.deb`로 패키지를 다운로드 하면 의존 패키지가 없어 설치되지 않습니다.
    - `dpkg -l mc`로 보면 설치가 되있는 것을 확인할 수 있지만 `mc`를 입력하면 error가 뜨고, 정상적인 화면 출력이 안됩니다.
7. `dpkg -r mc`로 mc 패키지를 지웁니다. 위 과정을 통해 dpkg가 의존성에 관련해 불편한 부분이 있다는 것을 할 수 있었습니다. 그래서 개발자들은 apt 명령어를 만들게 되었습니다.

### apt 명령어의 정보
#### apt의 설명
- `apt` 또는 `apt-get` 명령어는 *.deb 패키지를 설치하는 편리한 도구로써 만들어졌습니다.
- 우분투가 제공하는 deb 파일 저장소에서 자동으로 deb 파일을 다운로드해 설치합니다. 이 과정에서 의존성이 있다면 해당 의존성 패키지까지 모두 가져와 설치합니다.
1. `apt install 패키지명`을 입력하면 /etc/apt/sources.list 파일에서 저장소 URL을 확인합니다.
2. 그럼 해당 URL의 저장소에서 설치와 관련된 패키지 목록을 요청하면 저장소는 설치와 관련된 패키지 목록을 컴퓨터에 줍니다.
3. 그럼 컴퓨터는 설치할 패키지와 관련된 패키지의 이름을 화면에 출력합니다.
4. 사용자가 y를 눌러 패키지 다운로드에 동의하면 설치에 필요한 패키지 파일을 우분투 패키지 저장소에 요청하고, 저장소는 설치할 패키지 파일을 다운로드해서 자동으로 설치합니다.

#### apt 명령어중 일부

| 명령어 | 설명 |
| :---: | :---: |
| apt install or apt-get install 패키지명 | 패키지 설치 명령어 |
| apt update or apt-get update | /etc/apt/sources.list 파일의 내용이 수정되면 다운로드할 패키지 목록을 업데이트 |
| apt remove or apt-get remove 패키지명 | 설치되어 있는 패키지를 삭제(설정 파일 유지) |
| apt purge or apt-get purge 패키지명 | 설치되어 있는 패키지와 설정 파일을 모두 삭제 |
| apt autoremove or apt-get autoremove | 사용하지 않는 패키지를 모두 삭제 |
| apt clean(autoclean) or apt-get clean(autoclean) | 설치할 때 다운로드한 파일과 과거의 파일을 삭제 |
| apt show or apt-cache show 패키지명 | 패키지의 정보를 보여줌 |
| apt depends or apt-cache depend 패키지명 | 패키지의 의존성을 보여줌 |
| apt rdepends or apt-cache rdepends 패키지명 | 패키지에 의존하는 다른 패키지의 목록을 보여줌 |

\n
#### apt 명령어 실습
1. 위 dpkg 명령어로 다운로드에 실패할 mc 명령어를 다운로드 합니다.
    1. 설치할 mc 패키지의 정보를 `apt show mc` 명령어로 확인합니다.
    2. `apt depends mc`로 의존성 정보를 확인합니다.
    3. `apt -y install mc`로 mc를 다운로드 합니다.
        - 만약 위에서 제대로 mc를 지우지 않아 오류가 나면 `apt --fix-broken install`을 입력하면 알아서 지워줍니다. 이후 위 명령어로 다시 mc 패키지를 다운로드하면 됩니다.
    4. `mc`를 입력하면 정상적으로 파일 관리자가 실행되는 것을 확인할 수 있습니다.
        - mc의 경우 UI가 강화된 파일 관리자입니다.

#### apt의 작동 방식과 설정 파일
- `apt show 패키지명`에서 section을 확인해 보면 universe나 main과 같은 것들이 적혀 있습니다.
    - main: 우분투에서 공식적으로 지원하는 무료 소프트웨어를 의미합니다
    - universe: 우분투에서 지원하지 않는 무료 소프트웨어입니다.
    - restricted: 우분투에서 공식적으로 지원하는 유료 소프트웨어입니다.
    - multiverse: 우분투에서 지원하지 않는 유료 소프트웨어입니다.
- 기본적으로 우분투 패키지 저장소는 우분투 사이트에서 저공합니다. 가장 처음 /etc/apt/sources.list 파일을 보거나, /etc/apt/sources.list.bak 파일을 본다면 우분투 아카이브 링크가 적혀있습니다.
    - 하지만 이 블로그를 따라 오신분들의 경우 /etc/apt/sources.list에 다음 카카오도 적혀 있는 것을 확인하실 수 있는데 이런 저장소를 미러 사이트라고 합니다.
    - 이외의 미러 사이트는 [https://launchpad.net/ubuntu/+cdmirrors](https://launchpad.net/ubuntu/+cdmirrors)에서 확인할 수 있습니다.
    - 일반 사용자가 `apt install 패키지명`을 실행하면 sources.list에 기록된 사이트에서 자동으로 접속해 다운로드를 진행합니다.
- /etc/apt/sources.list 파일 구성
    - 각 행은 `deb 우분투패키지저장소url 버전코드명 저장소종류`를 의미합니다.
    - 맨 처음 다운로드 받으면 18.04를 의미하는 bionic으로 버전 코드명이 적혀 있지만, 저는 지금 22.04로 진행하고 있으므로 해당 버전을 의미하는 jammy로 버전 코드명을 변경해 업데이트 했습니다.
    - 만약 무료 제품만 사용하고 싶다면, main과 universe 행만 남기고 나머지는 맨 앞에 #을 붙여 주석 처리를 하면 됩니다.
    - `jammy`는 우분투 22.04 LTS가 출시된 시점에 제공되는 패키지 버전만 설치하겠다는 것을 의미합니다. 이후 업그래이드된 최신 버전의 패키지를 설치하고 싶다면 `jammy`를 `jammy-updates`로 수정하거나 아래 행에 추가하면 됩니다.

#### apt 명령어로 추가 기능 설정하는 실습
1. 새로운 url로 저장소를 업데이트 합니다.
    1. `vi /etc/apt/sources.list`명령으로 파일을 열어 모두 주석 처리한 뒤 저장한 후 닫습니다.
    2. 터미널을 열고 `apt update`로 설정 내용을 적용하고, `apt install mc`명령으로 패키지를 설치하면 저장소 URL이 없기 때문에 패키지가 없다고 에러 메시지가 나오고 설치되지 않습니다.
    3. [https://launchpad.net/ubuntu/+cdmirrors](https://launchpad.net/ubuntu/+cdmirrors)에서 쓸만한 미러 사이트를 찾습니다.
    4. 저는 [http://mirror.aarnet.edu.au/pub/ubuntu/archive/](http://mirror.aarnet.edu.au/pub/ubuntu/archive/)를 이용했습니다.
    5. `vi /etc/apt/sources.list` 명령으로 파일을 열어 주석 아래에 URL 주소만 다르게 작성해 입력하면 됩니다. 설정 파일의 경우 설명 마지막에 사진이 있습니다.
    6. `apt update` 명령으로 저장소를 업데이트하고, `apt remove mc` 후, `apt install mc`로 패키지를 설치하면 정상적으로 설치가 되는 것을 확인할 수 있습니다.
2. 업데이트된 패키지 설치
    1. `apt upgrade` 명령으로 전체 시스템을 업그레이드 하려고 하면 업그레이드할 것이 없습니다.
    2. `vi /etc/apt/sources.list` 명령으로 파일을 열어 각 행을 복사 후 각 아래 행에 복사한 뒤 jammy를 jammy-updates로 변경한 뒤 저장, 종료를 합니다.
    3. `apt update` 명령으로 sources.list의 수정 내용을 적용하고, `apt upgrade`를 입력하면 업그레이드 항목들이 나오고, 계속하면 업그래에드를 정상적으로 진행하게 됩니다.

<!-- 사진 바꿔야 함 -->
![sources.list](:/linux/ubuntu/6/sources_list.avif){:data-align="center"}

### root 사용자 비밀번호 분실 시
1. 비밀번호 변경 준비하기
    1. Server(B)를 부팅하자마자 바로 검은 화면에서 마우스를 클릭해 마우스 커서가 없어지면 Server(B) 화면이 선택된 상태입니다. 이 상태에서 Esc를 여러 번 누르게 되면 GRUB의 메뉴 화면이 나타납니다. `*Ubuntu`와 `Advanced options for Ubuntu`중 `*Ubuntu`를 선택한 상태에서 'E'를 누릅니다.
    2. 아래 화살표를 눌러 맨 아래에서 2번째 줄에서 `init=/bin/bash`를 입력합니다.
    ```
    fi
    linux   /vmlinuz-4.15.5-206~ maybe-ubiquity(nomodeset) init=/bin/bash
    ```
2. 비밀번호 변경하기
    1. 그 후 Ctrl+x나 F10을 눌러서 부팅합니다.
    2. 별도의 로그인 절차 없이 부팅되고, root@(none):/# 프롬프트가 나타나면 `whoami`로 현재 root 사용자로 로그인 되있는 것을 확인할 수 있습니다.
    3. root 사용자의 비밀번호를 변경하기 위해 `passwd` 명령을 입력 후 원하는 비밀번호를 설정합니다.
        - `Authentication token manipulation error`가 일어나며 비밀번호가 바뀌지 않습니다. 비밀번호를 정상적으로 바꾸기 위해 아래와 같이 마운트된 파티션을 읽고 쓰기가 가능하도록 변경해야 합니다.
    4. 마운트된 파티션을 `mount` 명령을 입력해 끝 부분을 확인해 보면 `/ type ext4 (ro,~)`로 / 파티션이 ro(Read-Only)로 마운트되어 있음을 확인할 수 있습니다.
    5. 이를 읽고 쓰기가 가능하도록 `mount -o remount,rw /` 명령을 입력해 / 파티션을 읽기/쓰기(rw) 모드로 다시 마운트 합니다.
    6. mount 명령을 다시 입력하면 읽기/쓰기 모드로 변경된 것을 확인 가능합니다.
    7. 이후 passwd 명령을 입력해 연재 사용자인 root의 비밀번호를 간단히 변경하면 성공적으로 변경됩니다.
    8. 이후 시스템을 강제로 재부팅하고, 로그인을 해보면 정상적으로 로그인이 되는 것을 확인할 수 있습니다.
        - 화면에서 reboot를 하면 `failed to talk to init daemon.` 에러가 나기 때문에 Player -> Power -> Restart Guest로 재시작 해야 합니다.

#### GRUB 부트 로더
**GRUB 부트 로더에 관해**
- 우분투를 부팅할 때 처음 나오는 선택화면으로 비밀번호를 설정할때와 같이 강제로 불러올 수 있습니다.
- 부트 정보를 사용자가 임의로 변경하여 부팅할 수 있습니다.
- 다른 운영체제와 멀티부팅이 가능합니다.
- 대화형 설정이므로 커널의 경로와 파일 이름만 알면 부팅이 가능합니다.
- 동적 모듈 로딩이 가능합니다.
- ISO 이미지를 이용해 바로 부팅이 가능합니다.
- GRUB의 설정 파일은 /boot/grub/grub.cfg입니다.
\n
**/etc/default/grub 파일**
- grub.cfg 파일은 일반 사용자에게는 읽기 전용이며, root 사용자도 직접 편집해서는 안됩니다.
- 설정된 내용을 변경하려면 /etc/default/grub 파일과 /etc/grub.d/ 디렉터리의 파일을 수정한 후 grub-mkconfig 명령을 실행해야 합니다.
\n
```
GRUB_DEFAULT=0
GRUB_TIMEOUT_STYLE=hidden
GRUB_TIMEOUT=0
GRUB_DISTRIBUTOR=`lsb_release -i -s 2> /deb/null || echo Debian`
GRUB_CMDLINE_LINUX_DEFAULT="maybe-ubiquity"
GRUB_COMDLINE_LINUX=""
```
- 1행: GRUB 목록 중에서 0번째가 기본으로 선택되게 합니다.
- 2행: 3행의 시간 동안 화면에 GRUB 목록이 보이지 않게 합니다.
- 3행: 처음 화면이 나오고 자동으로 부팅되는 시간을 초 단위로 설정합니다.
- 4행: 초기 부팅 화면의 각 엔트리 앞에 붙을 배포판 이름을 추출합니다.
- 5~6행: 부팅 시 커널에 전파할 파라미터를 지정합니다.

#### GRUB 부트로더 변경 및 비밀번호 설정하기
1. GRUB의 내용을 변경하고 부팅 화면 실행하기
    1. vi 에디터로 /etc/default/grub 파일을 열어 아래 3개의 행을 변경한 뒤 저장하고 닫습니다.
    ```
    GRUB_TIMEOUT_STYLE=countdown    # 부팅 시 대기시간을 보여줌
    GRUB_TIMEOUT=20     # 부팅 대기 시간을 20초로 변경
    GRUB_DISTRIBUTOR="OTHKKARTHO LINUX" # 초기 화면의 글자를 변경
    ```
    2. 변경한 내용을 적용하기 위해 `update-grub`을 입력합니다.
    3. reboot 명령으로 재부팅하면 20초 동안 카운트를 합니다. ESC를 누르면 글자가 바뀐 GRUB 초기 화면이 나타납니다.
    4. 첫 행에서 Enther를 누르고 root 사용자로 접속합니다.
2. GRUB 전용 사용자와 비밀번호 생성하기
    1. vi 에디터로 /etc/grub.d/00_header 파일을 열어 마지막 다음 줄에 아래의 4줄을 추가한 뒤 저장하고 닫습니다.
    ```
    cat << EOF
    set superusers="grubuser"   # grubuser는 새로운 GRUB 사용자의 이름
    password grubuser 1234  # 'GRUB사용자 새비밀번호' 형식으로 비밀번호 설정
    EOF
    ```
    2. 변경한 내용 적용을 위해 `update-grub` 명령을 입력합니다.
    3. reboot 명령으로 시스템을 재부팅한 후 Esc를 누르고 GRUB 화면으로 들어가 e를 누르면 사용자와 비밀번호를 입력하는 창이 나옵니다.
    4. 해당 화면에서 설정한 사용자와 비밀번호를 입력하면 GRUB 편집 모드가 나옵니다. 설정이 제대로 된 것을 확인하실 수 있습니다.

### 참고
#### ncftp에 관해
1. ncftp 패키지의 설명을 간단히 보면 ncftp는 유저 친화적이고, 기능이 좋은 FTP 클라이언트입니다.
2. `ncftp`를 입력하면 ncftp 콘솔로 접속됩니다.
3. `open ftp.ubuntu.com` 우분투의 ftp 서버에 접속합니다.
    - ncftp로 다운로드 받고, 업로드 하는 것은 [[linux]ncftp 명령어로 파일 다운로드/업로드 방법-투원대디](https://seculog.tistory.com/6)에서 확인하실 수 있습니다.
4. `ls`, `cd ubuntu` 등 여러 명령어를 실행해 보고, 잘 작동되는 것을 확인한 후 exit으로 ftp 접속을 종료합니다.

### 참고 자료
1. [리눅스 실습 for Beginner](https://www.hanbit.co.kr/store/books/look.php?p_code=B7654754187)
