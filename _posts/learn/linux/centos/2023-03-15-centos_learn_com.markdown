---
# multilingual page pair id, this must pair with translations of this page. (This name must be unique)
lng_pair: centos
title: CentOS 서버 구축 - 명령어 정리

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
meta_modify_date: 2023-03-13 22:00:00 +0900
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

CentOS를 사용하면서 써봤던 명령어들을 정리해 놓은 포스트입니다.

<!-- outline-end -->

* * *

### 목차

1. [yum](#yum-명령어)
2. [](#)
3. [](#)
4. [](#)
5. [](#)
6. [](#)

### yum 명령어
#### 명령어 개요
Yellowdog Updater Modified의 약자로 대화형 패키지 관리 시스템입니다.   
yum은 Central Repository를 통해 rpm을 배포하고 의존성 관리를 하기 때문에 손쉽게 패키지를 관리할 수 있습니다. 즉 만약 패키지를 하나 다운받는다면 그 패키지의 의존성 패키지들을 일일이 다운받을 필요가 없고, rpm이 업데이트 됬을 경우 쉽게 확인할 수 있습니다.  
#### 주요 명령어와 옵션
| 옵션 | 설명 |
| :---: | :---: |
| --enablerepo=[repo] | 여러개의 yum repository가 있을 경우 사용할 repos를 지정합니다. 같은 wildcard를 사용할 수 있습니다. |
| --disablerepo=[repo] | 사용하지 않을 repos를 지정합니다. wildcard를 사용 가능합니다. |
| --nogpgcheck | GPG 서명검증을 사용하지 않습니다. 해당 repos의 공개키가 없어서 서명검증에 실패한 경우 사용합니다. |
| -d [debug level] | debugging level을 지정합니다. 0-10 까지 가능하며 숫자가 클수록 자세한 정보를 출력합니다. |
| -y, --assumeyes | yum 진행중 나오는 질문을 모두 긍정 응답으로 처리합니다. |
| install | 패키지와 의존성 있는 패키지를 설치합니다. |
| update | 해당 패키지의 새 버전이 있으면 update 합니다. |
| check-update | update 될 패키지의 목록을 출력합니다. |
| clean |  |

### 참고 자료
1. [yum 주요 사용법 및 고급 사용법 - System Administrator](https://www.lesstif.com/system-admin/yum-history-plugin-undo-6979667.html)