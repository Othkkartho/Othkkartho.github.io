---
# multilingual page pair id, this must pair with translations of this page. (This name must be unique)
lng_pair: Ubuntu
title: Ubuntu 서버 구축(2)-리눅스 기본 사용법

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
date: 2023-03-22 22:00:00 +0900

# seo
# if not specified, date will be used.
# meta_modify_date: 2023-03-17 13:15:00 +0900
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
이번에는 리눅스 셧다운 방법과 가상 콘솔 및 런타임의 이해. gedit, vi 에디터 사용법. 리눅스 마운트의 개념과 설정 방법에 대해 다뤘습니다.

<!-- outline-end -->

* * *

### 개요
학교에서 사용하는 Ubuntu 서버의 경우 [리눅스 실습 for Beginner](https://www.hanbit.co.kr/store/books/look.php?p_code=B7654754187)에서 제공하는 학습을 따라가기 때문에 Ubuntu 18.04 Desktop, Server와 Kubuntu 18.04 버전을 사용하지만 포스트에서는 현재 포스트를 쓸 때 최신 버전인 22.04 버전을 기준으로 학교의 수업을 따라 포스트를 작성하겠습니다.   
만약 학교의 내용과 버전 때문에 다른 점이 있다면 제 PC에서는 22.04를 사용하고, 노트북에서는 18.04로 수업을 따라가고 있기 때문에 참고에서 해당 내용을 다루도록 하겠습니다.   \n
Ubuntu Desktop, Server 버전의 경우 둘 다 서버로 사용하고, Kubuntu로 클라이언트로 사용할 예정입니다.

### 목차

0. [리눅스의 셧다운 방법의 이해](#리눅스의-셧다운-방법)
1. [가상 콘솔의 이해](#ubuntu-설치)
2. [런레벨](#런레벨)
3. [리눅스 마운트의 이해](#kubuntu-설치)

### 리눅스의 셧다운 방법
X윈도우의 경우 아래의 사진과 같이 사용자 기준 우측 상단의 버튼을 눌러 아래의 설정을 그대로 진행할 수 있습니다.    
![종료 및 로그아웃](:/linux/ubuntu/2/exit.avif){:data-align="center"}

리눅스의 경우 터미널에서 `poweroff`, `shutdown -P now`, `halt -p`, `init 0` 과 같은 종료 명령들을 실행해 즉시 종료할 수 있습니다.   
윈도우 또한 `shutdown -s`, `shutdown -p` 등의 명령어와 함께 예약 종료 명령어도 자주 사용하긴 하지만 기본적으로 싱글 유저로 만들어진 윈도우OS와는 다르게 멀티 유저를 기반으로 하는 리눅스의 경우 위와 같은 즉시 종료 명령어는 잘 사용하지 않습니다.   
만약 서버 관리자가 즉시 종료하게 될 경우 해당 서버에 연결되 작업중이던 모든 유저들의 작업이 종료될 수 있기 때문에 아래와 같은 예약 종료 명령어를 주로 사용합니다. 이런 예약 종료 명령어는 실행되면 다른 유저들에게 언제 종료 예정이라는 것을 알리게 됩니다.   
- `shutdown -P +10`: 10분 후에 종료합니다.
- `shutdown -r 22:00`: 22시에 서버를 재부팅 한다는 명령어입니다.
- `shutdown -c`: 만약 예약 종료를 잘못 설정하였을 경우 예약된 shutdown 명령을 취소시킵니다.
- `shutdown -k +15`: 15분 후에 종료할 거라는 메시지를 보내지만 실제로 종료하지는 않습니다.
\n
* 우분투 재부팅: `reboot`, `shutdown -r now`, `init 6` 명령으로 재부팅 가능합니다.
\n\n
보통 유저 본인만 종료할 경우 윈도우에서 처럼 종료가 아닌 로그아웃을 활용해 서버에서 나가야 합니다. 그 이유는 위에서도 설명했던 리눅스가 다중 사용자 시스템이기 때문입니다.   
명령어는 `logout`, `exit` 명령으로 가능합니다.

### 가상 콘솔
가상 콘솔 또는 가상 모니터라고 이야기를 합니다. 우분투의 경우 6개의 가상 콘솔을 제공합니다.      
각각의 콘솔로 이동하는 단축키는 Ctrl+Alt+F1~F6입니다.   
\n
그럼 해당 가상 콘솔 기술을 바탕으로 실제 종료 명령을 내릴 때 다른 사용자가 메시지를 받는지 확인해 보겠습니다.
1. Ctrl+Alt+F3을 눌러 root로 로그인 하고, Ctrl+Alt+F4를 눌러 다른 사용자로 로그인합니다.
2. 3번 콘솔로 이동 후 `shutdown -P +15`를 입력해 보겠습니다.
3. 4번 콘솔로 이동하면 화면에 아래 사진과 같은 메시지가 출력되었음을 확인할 수 있습니다.
![종료 메시지 출력](:/linux/ubuntu/2/msgexit.avif){:data-align="center"}
- 이외에도 `shutdown -c`, `shutdown -r 20:00` 과 같은 명령어를 입력하면 다른 사용자가 해당 동작에 맞는 명령어를 위 사진과 같은 형태로 받는것을 확인하실 수 있습니다.
- 실습이 종료된 후 시스템을 종료시키지 않고 해당 가상 콘솔만 종료시키고 싶다면 해당 콘솔들에서 `logout` 명령을 입력해 로그아웃을 실행하고, Ctrl+Alt+F2를 눌러 돌아올 수 있습니다.

### 런레벨
위 리눅스 셧다운 방법에서 이야기했던 init 뒤에 붙었던 숫자들이 런레벨이라고 하는 것입니다.   
리눅스의 경우 시스템 가동 방법을 아래 표와 같이 분류합니다.

| 런레벨 | 영문 모드 | 설명 | 비고 |
| :---: | :---: | :---: | :---: |
| 0 | Power Off | 종료 모드 |  |
| 1 | Rescue | 시스템 복구 모드 | 단일 사용자 모드 |
| 2 | Multi-User | Null | 사용하지 않음 |
| 3 | Multi-User | 텍스트 모드의 다중 사용자 모드 | Null |
| 4 | Multi-User | Null | 사용하지 않음 |
| 5 | Graphical | 그래픽 모드의 다중 사용자 모드 | Null |
| 6 | Reboot | 재부팅 모드 | Null |

현재 구동되고 있는 런레벨 모드를 확인하려면 /lib/systemd/system 디렉터리의 runlevel?.target 파일을 조회하면 됩니다.   
\n
런레벨 변경하기 실습
1. Server에서 터미널을 열어 `ls -l /lib/systemd/system/default.target` 명령을 입력합니다.
    - 만약 홈 디렉터리가 아니라면 `cd`를 입력해 홈 디렉터리로 이동합니다.
2. 그럼 `lrwxrwxrwx 1 root root 16 3월 17 17:11 /lib/systemd/system/default.target -> graphical.target`와 같은 기본으로 설정된 런레벨을 볼 수 있습니다.
    - lrwxrwxrwx: l(파일의 타입, 현제는 링크드 리스트 타입), rwx(소유주), rwx(그룹에 대한 접근 권한), rwx(기타 사람들의 접근권한) r은 read(파일 읽기 권한), w는 write(파일 변경 권한), x는 excute(파일 실행 권한)을 뜻합니다.
    - 1: 하드 링크 개수
    - root root: 소유주와 소유 그룹
    - 16: 파일 사이즈
    - 3월 17 17:11: 생성 날짜
    - /lib/systemd/system/default.target -> graphical.target: 현재 런레벨 5인 graphical 모드로 기본 설정되어 있다는 것을 의미합니다.
3. `ln -sf /lib/systemd/system/multi-user.target /lib/systemd/sstem/default.target` 명령을 입력하고, 다시 `ls -l /lib/systemd/system/default.target`를 입력하면

### 리눅스 마운트


### 참고
#### 가상 콘솔 18.04LST와 22.04의 차이점
18.04LST의 경우 F1이 처음 들어간 X윈도우가 구동되고 있는 콘솔이고, F2~6까지는 CUI의 가상 콘솔입니다.   
반면 22.04LST의 경우 F2가 처음 들어간 X윈도우가 구동되고 있는 콘솔이고, F1의 경우 새로운 X윈도우 콘솔입니다. F3~6는 CUI의 가상 콘솔입니다.   
즉 18.04의 경우 X윈도우 가상 콘솔이 1개이지만 22.04의 경우 X윈도우 가상 콘솔이 2개로 차이가 있음을 확인할 수 있었습니다.

### 참고 자료
1. 