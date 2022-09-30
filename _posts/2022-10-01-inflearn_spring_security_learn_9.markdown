---
# multilingual page pair id, this must pair with translations of this page. (This name must be unique)
lng_pair: springlearn
title: 인프런 스프링 시큐리티 강의 학습-9

# post specific
# if not specified, .name will be used from _data/owner/[language].yml
author: Othkkartho's Workshop
# multiple category is not supported
category: 인프런 스프링 시큐리티 강의 정리
# multiple tag entries are possible
tags: [Spring, 인프런 스프링 시큐리티 강의 정리]
# thumbnail image for post
img: ":/inflearn_spring_security_learn/post_spring_security.png"
# disable comments on this page
#comments_disable: true

# publish date
date: 2022-10-01 22:00:00 +0900

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

실전 프로젝트 구현전 구상
----------------------------------------------------------------------------

출처는 인프런의 스프링 시큐리티 - [Spring Boot 기반으로 개발하는 Spring Security](https://www.inflearn.com/course/%EC%BD%94%EC%96%B4-%EC%8A%A4%ED%94%84%EB%A7%81-%EC%8B%9C%ED%81%90%EB%A6%AC%ED%8B%B0)강의를 바탕으로 이 포스트를 작성하고 있습니다.<br>
강의의 세션 3의 1번 강의내용에 대한 정리입니다.

<!-- outline-end -->

### 실전 프로젝트 구성
#### 프로젝트 기본 구성
- 의존성 설정, 환경설정, UI 화면 구성, 기본 CRUD 기능을 구현합니다.
- 스프링 시큐리티 보안 기능을 점진적으로 구현 및 완성할 계획입니다.
- 코드 중 resource에 들어가는 코드는 강의의 코드를 그대로 가져왔습니다. 각 프론트앤드 부분 코드들은 강의에서 자세히 다루지 않을경우 코드를 확인해 포스트를 작성하겠습니다.
- 강의에서는 각각의 강의에 따른 코드를 다른 브랜치로 관리했지만 저는 포스트에 코드를 조금 더 자세히 작성하는 것으로 대체하도록 하겠습니다.

#### 실제 코드
**build.gradle**{:data-align="center"}
```java
implementation 'org.springframework.boot:spring-boot-starter-security'
implementation 'org.springframework.boot:spring-boot-starter-thymeleaf'
implementation 'org.springframework.boot:spring-boot-starter-web'
implementation 'org.thymeleaf.extras:thymeleaf-extras-springsecurity5'
testImplementation 'org.springframework.boot:spring-boot-starter-test'
testImplementation 'org.springframework.security:spring-security-test'
```
위 코드를 보시면 어떤 의존성을 추가했는지 아실 수 있습니다.<br>
추가한 의존성은 Spring Security, Thymeleaf, Spring Web입니다. 강의에서 JPA 사용한다고 먼저 [JPA 의존성을 추가하시면 안됩니다](#failed-to-configure-a-datasource-에러).

### 참고
#### Failed to configure a DataSource 에러
전체 에러 코드를 보면 아래와 같이 되어있습니다.
```
Failed to configure a DataSource: 'url' attribute is not specified and no embedded datasource could be configured.

Reason: Failed to determine a suitable driver class
```
에러가 DataSource를 설정해주지 않아서 발생한 문제임을 위 코드를 통해 알 수 있습니다.<br>
스프링 라이브러리를 소개한 프로젝트에서 자세히 소개하겠지만 JPA는 매핑된 관계를 이용해 SQL을 생성, 실행합니다.<br>
즉, DB를 건드리기 때문에 DataSource 설정이 필요한것입니다. 하지만 세션 3의 1번 강의에서는 DBMS연결을 하지 않아 datasource 설정을 하지 않았고, 그로 인해 오류가 발생한 것입니다.

#### Gradle VS Maven


#### Postgresql 설명 및 설치


#### thymeleaf 설명


### 출처
1. [학습중인 강의](https://www.inflearn.com/course/%EC%BD%94%EC%96%B4-%EC%8A%A4%ED%94%84%EB%A7%81-%EC%8B%9C%ED%81%90%EB%A6%AC%ED%8B%B0)