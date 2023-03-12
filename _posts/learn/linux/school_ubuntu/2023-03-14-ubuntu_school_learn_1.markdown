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

1. [CentOS 7 설치](#centos-7-설치)
2. [Apache 설치](#apache-설치)
3. [Tomcat 설치](#tomcat-설치)
4. [Jenkins 설치](#jenkins-설치)
5. [PostgreSQL 설치](#postgresql-설치)

### CentOS 7 설치
VMware에 새로운 

### Apache 설치


### Tomcat 설치


### Jenkins 설치


### PostgreSQL 설치


### 참고
#### 

#### Web Server와 WAS의 차이점


#### Jenkins는 어떤 역할을 할까?

### 참고 자료
1. 