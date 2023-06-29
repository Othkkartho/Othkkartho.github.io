---
# multilingual page pair id, this must pair with translations of this page. (This name must be unique)
lng_pair: Ubuntu
title: Ubuntu 서버 구축(7)-X 윈도우 응용 프로그램

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
date: 2023-06-28 22:00:00 +0900

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

X 윈도우에서 사용하는 각종 응용프로그램에 대한 설명과 GNOME 데스크톱을 사용할 때 어떤 패키지를 설치 해야 하는지를 살펴보고, KDE 데스크톱을 설치해본다.

<!-- outline-end -->

* * *

### 목차

1. [응용 프로그램 설명](#응용-프로그램-설명)
2. [CUI 환경에 GNOME 데스크톱 환경 추가하기](#serverb에-gnome-데스크톱-환경-추가하기)
3. [KDE 데스크톱 설치해보기](#kde-데스크톱-설치해보기)

### 응용 프로그램 설명
#### 노틸러스 사용하기
- 노틸러스는 현재 활동 아래의 파일 아이콘을 선택하거나 터미널에서 nautilus 명령을 입력하면 그래픽 환경의 노틸러스가 열립니다.
- 노틸러스는 윈도우의 파일 탐색기와 비슷하게 사용이 가능하며, 윈도우의 복사(Ctrl+C), 잘라내기(Ctrl+X), 붙여넣기(Ctrl+V) 단축키도 사용이 가능합니다.

#### 브라세로 사용하기
- `apt-get -y install brasero`를 입력해 브라세로를 설치하면 아래와 같은 브라세로 아이콘을 누르거나 터미널에 `brasero`를 입력하면 실행됩니다.
![브라세로 아이콘](:/linux/ubuntu/7/brasero.png){:data-align="center"}
- 여기서 아래 사진과 같이 여러 파일들을 CD나 DVD로 만들 수 있습니다.
![브라세로 프로젝트](:/linux/ubuntu/7/brasero_project.png){:data-align="center"}

#### 인터넷 응용 프로그램
- **파이어폭스**: 우분투에서 기본으로 제공하는 웹 브라우저로 많은 기능과 안전성이 장점입니다.
- **선더버드**: 마이크로소프트의 아웃록과 비슷한 기능을 하는 이메일 클라이언트입니다. 설치하지 않아도 자동으로 깔려있고, Mozilla Thunderbird를 선택하거나, 터미널에서 `thunderbird`를 입력해 실행할 수 있습니다.
- **gftp**: `apt-get install gftp`로 설치해 `gftp`명령이나 아이콘에서 gFTP 아이콘을 눌러 실행할 수 있습니다.
    - gFTP는 다중 스레드 파일 전송 프로토콜 클라이언트 프로그램으로 유닉스 계열 시스템에서 가장 많이 사용됩니다. 해당 프로그램에는 [GTK+](#gtk) 그래픽 툴킷을 사용하는 GUI와 CUI가 모두 포함되어 있습니다.
![gftp](:/linux/ubuntu/7/gftp.png){:data-align="center"}

#### 이외 다양한 응용 프로그램들
- 사운드 설정: Client 가상 머신의 시작에서 '시스템 설정 - 멀티미디어 - 오디오 음량 - 고급'을 선택하면 사운드 설정을 진행할 수 있습니다.
- 음악 재생 및 인터넷 라디오 실행: 리듬박스를 선택하거나 `rhythmbox` 명령을 실행합니다.
- DVD 재생 및 동영상 플레이어: 동영상을 선택하거나 `totem`명령을 실행합니다.
- 문서 보기: 유틸리티-문서 보기를 선택하거나 `evince`명령으로 실행할 수 있음. PDF, XPS, TIFF등의 다중 문서를 **볼 수 있는** 뷰어 프로그램입니다. 
- 그래픽 프로그램
    - 윈도우의 포토샵과 비슷한 그래픽 편집 프로그램으로써 `apt install gimp`명령으로 GIMP를 설치해 사용할 수 있습니다.
    - 윈도우의 알씨처럼 그림 파일을 보여주는 그래픽 뷰어 프로그램으로 `eog`명령으로 실행 가능합니다.
    - 해당 책에는 유틸리티에 스크린샷이 있지만 현재 제가 사용하고 있는 22.04에서는 스크린샷이 안보이기 때문에 `apt-get install gnome-screenshot`으로 스크린샷 툴을 추가하고, `gnome-screenshot`으로 실행하니 실제 사진 폴더에 스크린샷이 저장되는 것을 확인할 수 있었습니다.
- 리브레오피스 프로그램
    - 리브레오피스: 문서작성 프로그램, MS 오피스와 호환성이 뛰어납니다. MS 오피스처럼 여러 문서들을 선택해 사용할 수 있게 되어있습니다. `libreoffice`명령으로 이용 가능합니다.
    - 라이터: MS 워드의 doc와 docx를 지원하고, PDF 파일로 저장하는 기능도 있습니다. `libreoffice --writer` 명령으로 실행할 수 있습니다.
    - 캘크: MS 엑셀의 xls와 xlsx를 지원하고, PDF 저장 기능도 있습니다. `libreoffice --calc` 명령으로 실행 가능합니다.
    - 임프레스: 파워포인트의 ppt와 pptx를 지원하며, PDF 파일로 저장하는 기능도 있습니다. `libreoffice --impress` 명령으로 실행할 수 있습니다.
![libreoffice](:/linux/ubuntu/7/libreoffice.png){:data-align="center"}

### Server(B)에 GNOME 데스크톱 환경 추가하기
1. 우분투 저장소 파일에 sources.list를 열어 'jammy-updates'를 추가하거나 변경한 후 저장합니다.
2. apt-get update 명령으로 변경 내용을 적용합니다.
3. `apt-get -y install gnome-session gdm3 gnome-terminal language-pack-ko language-pack-gnome-ko`를 통해 관련 패키지를 설치합니다. 설치가 완료되면 reboot 명령으로 재부팅하면 GNOME 초기 화면이 보입니다.
4. 본인에 맞게 언어, 키보드등 세팅을 완료합니다.
5. 터미널에서 `sudo apt-get -y install tasksel`명령을 입력해 tasksel 패키지를 설치하고, `sudo tasksel install ubuntu-desktop`명령을 입력해 관련 패키지를 설치합니다.
    - [tasksel](#tasksel) 대신 apt-get을 사용해도 됩니다.
6. 설치가 완료되어 `reboot` 명령으로 재부팅하면 우분투 데스크톱을 설치한 것과 동일한 환경으로 사용이 가능합니다.

### KDE 데스크톱 설치해보기
1. sources.list 파일에 'jammy-updates'를 추가합니다.
2. `apt-get update`명령으로 변경 내용을 적용하고, `apt-get -y install kubuntu-desktop`으로 KDE Plasma를 설치합니다.
3. 기본 화면 관리자는 sddm으로 선택하고 확인을 누른 뒤 설치가 완료되면 `reboot` 명령으로 재부팅합니다.

### 참고
#### GTK
GTK는 GUI를 만들기 위한 무료 오픈소스 크로스 플랫폼 위젯 툴릿입니다.  
C언어로 작성된 객체 지향 위젯 툴킷으로써 주로 X11 및 Wayland를 기반으로 하는 윈도우 시스템을 위한 것이지만 MS Window 및 macOS를 포함한 다른 플랫폼에서도 작동합니다.

#### tasksel
Tasksel은 Debian/Ubuntu용으로 개발된 간단하고 사용하기 쉬운 도구로 사용자에게 관련 그룹을 설치할 수 있는 GUI 인터페이스를 제공합니다. 메타 패키지와 유사하게 작동하며 메타 패키지에 있는 tasksel의 거의 모든 작업을 찾을 수 있습니다.   \n
**tasksel**을 설치하려면 `sudo apt-get install tasksel`을 실행하고, 명령줄에서 tasksel을 실행하는 일반적인 구문은 다음과 같습니다.

```
sudo tasksel install task_name
sudo tasksel remove task_name
sudo tasksel command_line_options
```   
tasksel GUI를 사용하려면 `sudo tasksel`을 실행하면 됩니다. 별표가 표시된 부분은 소프트웨어가 이미 설치되어 있음을 의미합니다. 본인이 설치를 원하는 곳에 위, 아래 화살표를 사용해 빨간팬을 이동시키고 스페이스 바를 눌러 소프트웨어를 선택한 다음 Tab치를 눌러 ok로 이동하거나 그곳에서 Enter를 누르면 소프트웨어를 설치합니다.   \n

`sudo tasksel --list-tasks`를 입력하면 모든 작업을 나열할 수 있습니다. 첫 번째 열의 u는 소프트웨어가 설치되지 않았음을 의미하고, i는 소프트웨어가 설치되었음을 의미합니다.

### 참고 자료
1. [리눅스 실습 for Beginner](https://www.hanbit.co.kr/store/books/look.php?p_code=B7654754187)
2. [Wikipedia - gFTP](https://en.wikipedia.org/wiki/GFTP)
3. [Linux-Console.net - tasksel](https://ko.linux-console.net/?p=1873#gsc.tab=0)