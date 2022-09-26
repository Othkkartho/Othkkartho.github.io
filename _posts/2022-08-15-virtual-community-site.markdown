---
# multilingual page pair id, this must pair with translations of this page. (This name must be unique)
lng_pair: virtual
title: 버츄얼 유튜버 커뮤니티 사이트 요구분석 및 설계 

# post specific
# if not specified, .name will be used from _data/owner/[language].yml
author: Othkkartho's Workshop
# multiple category is not supported
category: 버츄얼 유튜버 커뮤니티 제작
# multiple tag entries are possible
tags: [Spring, virtual community]
# thumbnail image for post
img: ":post_virtual_pic1.jpg"
# disable comments on this page
#comments_disable: true

# publish date
date: 2022-08-15 10:00:00 +0900

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

버츄얼 유튜버 게시판을 만들기 위한 설계를 설명하는 페이지입니다.

<!-- outline-end -->

### 사이트 제작 이유
현재 버츄얼 유튜버 시장은 성장하고, 많은 사람들과 회사들이 나오고 있지만, 그 커뮤니티는 여러 커뮤니티 안에서 마이너 커뮤니티로서 활동되고, 거대한 버츄얼 회사들을 제외하면 그 커뮤니티 조차 없습니다.<br>
그래서 홀로라이브 뉴스라는 팬사이트를 보고 현재 성장해 가고 있는 버츄얼 시장에 버츄얼 전체를 아우르는 팬 커뮤니티인 사이트가 있었으면 좋겠다고 생각해 제작을 결정했습니다.<br>

### 제작 목표
여타 커뮤니티처럼 작동하는 커뮤니티 사이트를 만들어 서버 구동까지 하는 것이 목표입니다.

### 참고 사이트
[아카라이브](https://arca.live/)와 [홀로라이브 뉴스](https://hololive.kr/) 사이트를 참고해 제작 방향을 결정했습니다.

### UML
#### 엑터
아래 표는 각 액터들의 이름과 설명입니다.
| 이름 | 설명 |
| :-------: | :--------------: |
| 관리자 | 사이트의 관리자로서 개발자나 전체 사이트의 관리자 |
| 채널 관리자 | 각 버츄얼 커뮤니티 채널의 관리자 |
| 채널 활동자 | 각 버츄얼 커뮤니티의 활동 권한을 부여받은 사람들 |
| 번역가 | 커뮤니티 활동하는 사람들 중 번역을 하는 사람들 |
| 일러스트레이터 | 커뮤니티 활동하는 사람들 중 일러스트를 제작하는 사람들 |
| 게스트 | 커뮤니티에 회원가입을 했지만 어떤 역활도 부여받지 못한 사람들 |
{:data-align="center"}

### 페이지 설계
[오븐](https://ovenapp.io/project/A4Tm1teK90pM0YC9bPPiUVawvvcocM5D#QH6B0)에 페이지 설계를 함

### 데이터베이스 설계
![DB 스키마 사진](:/virtual_post/DB_design.jpg){:data-align="center"}

### API 명세서
![추후 업로드 예정]()
