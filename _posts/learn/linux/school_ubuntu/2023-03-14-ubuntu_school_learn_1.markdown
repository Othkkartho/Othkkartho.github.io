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
#meta_modify_date: 2022-02-10 08:11:06 +0900
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
1. [Ubuntu 설치](#ubuntu-설치)
2. [Ubuntu 기본 설정](#ubuntu-기본-설정)
3. [Kubuntu 설치](#kubuntu-설치)
4. [Kubuntu 기본 설정](#kubuntu-기본-설정)

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
1. VMware를 실행 후 CentOS를 깔았던 것과 같이 VMware를 만들고 ISO를 넣으시면 됩니다.
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
    - `wget http://download.hanbit.co.kr/ubuntu/18.04/sources.list`로 저자가 미리 만들어 놓은 list 파일을 다운로드 하고, `apt-get update`를 통해 적용하고, `exit`명령으로 터미널을 닫습니다.

#### Ubuntu Server 설치

### Ubuntu 기본 설정


#### Ubuntu Desktop 기본 설정


#### Ubuntu Server 기본 설정


### Kubuntu 설치


### Kubuntu 기본 설정


### PostgreSQL 설치


### 참고
#### 우분투와의 비교
커널과 코어는 둘 모두 리눅스 커널과 우분투 코어를 사용합니다.   
그래픽은 X.Org 서버, 사운드는 펄스오디오, 멀티미디어는 G스트리머, 오피스 제품군은 리브레오피스, 이메일과 PIM은 선더 버드를 둘 모두 사용합니다.

| 소프트웨어 | 우분투 | 쿠분투 |
| :---: | :---: | :---: |
| 데스크톱 | 그놈 | 플라즈마 데스크톱 |
| 기본 툴킷 | GTK+, Nux & Qt | Qt |
| 브라우저 | 파이어폭스 | 13.10까지는 리콩트, 14.04부터는 파이어 폭스 |

### 참고 자료
1. [우분투 - 위키백과](https://ko.wikipedia.org/wiki/%EC%9A%B0%EB%B6%84%ED%88%AC)
2. [쿠분투 - 위키백과](https://ko.wikipedia.org/wiki/%EC%BF%A0%EB%B6%84%ED%88%AC)