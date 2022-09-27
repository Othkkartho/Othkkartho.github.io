---
# multilingual page pair id, this must pair with translations of this page. (This name must be unique)
lng_pair: springlearn
title: 인프런 스프링 시큐리티 강의 학습-7

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
date: 2022-09-28 22:00:00 +0900

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

인증 저장소 필터, 인증 흐름 이해, 인증 관리자, 인증 처리자에 관해 포스팅했습니다.
----------------------------------------------------------------------------

출처는 인프런의 스프링 시큐리티 - [Spring Boot 기반으로 개발하는 Spring Security](https://www.inflearn.com/course/%EC%BD%94%EC%96%B4-%EC%8A%A4%ED%94%84%EB%A7%81-%EC%8B%9C%ED%81%90%EB%A6%AC%ED%8B%B0)강의를 바탕으로 이 포스트를 작성하고 있습니다.<br>
강의의 세션 2의 5,6,7,8번 강의내용에 대한 정리입니다.

<!-- outline-end -->

### 인증 저장소 필터 - SecurityContextPersistenceFilter
#### 필터의 역할
**SecurityContext 객체의 생성, 저장, 조회.** 를 하는 필터입니다.
- **익명 사용자** 가 요청하면
    - 새로운 SecurityContext 객체를 생성하여 SecurityContextHolder에 저장하고 다음 필터로 이동합니다.
    - AnonymousAuthenticationFilter(익명 사용자 처리 필터)에서 AnonymousAuthenticationToken(익명 사용자 인증 객체) 객체를 SecurityContext에 저장합니다.
- **인증 시** (인증이 끝나지 않은 인증 도중)
    - 새로운 SecurityContext 객체를 생성하여 SecurityContextHolder 에 저장한 후 다음 필터로 이동합니다.
    - UsernamePasswordAuthenticationFilter(Form 인증 처리 필터)에서 인증 성공 후 SecurityContext 에 UsernamePasswordAuthenticationToken 객체를 SecurityContext에 저장합니다.
    - 인증이 최종 완료된 후 Session에 SecurityContext를 저장합니다.
- **인증 후**
    - Session에서 SecurityContext를 꺼내어 SecurityContextHolder에서 저장합니다.
    - SecuirtyContext 안에 Authentication 객체가 존재하면 계속 인증을 유지합니다.
- 위 모든 인증의 **최종 응답 시 공통** 적으로
    - SecurityContextHolder.clearContext()를 통해 SecurityContext를 삭제합니다.
        - 매 요청마다 SecurityContextHolder 안에 SecurityContext를 매 요청마다 저장하기 때문에 응답할 때는 항상 저장된 SecurityContext를 SecurityContextHolder에서 삭제합니다.

FilterChainProxy가 관리하는 필터들 중 두번째에 위치해 SecurityContext 객체를 Holder에 저장한 후 다른 필터들이 Holder에서 SecurityContext를 꺼내 참조할 수 있도록 동작하고 있습니다.

#### 인증 절차의 순서도
1. 사용자가 요청을 할 때 인증 받은 사용자의 요청이든 익명 사용자의 요청이든 매 사용자의 요청을 SecurityContextPersistenceFilter가 받습니다. 이 필터는 내부적으로 HttpSecurityContextRepository 클래스를 가지고 있습니다.
    -HttpSecurityContextRepository는 SecurityContext 객체를 생성, 조회하는 역할을 합니다.
2. 인증 전이고, 인증을 받는 중이라면 새로운 SecurityContext를 생성합니다.
3. 인증 필터가 인증을 처리하고, 인증 필터가 SecurityContextHoler 안의 SecurityContext안에 인증 결과를 저장한 Authentication 객체를 저장합니다.
4. 최종적으로 클라이언트에게 응답하는 시점에 SecurityContextPersistenceFilter가 Sesson안에 SecurityContext를 저장시키고, SecurityContext를 Holder에서 삭제 시킵니다.
5. 그 후 다시 요청을 하게 되면, 인증 후기 때문에 Session에 SecurityContext가 있는지 확인하고, Context를 꺼내서 SecurityContextHolder에 저장하고, 다음 작업으로 이동합니다.
    - 그래서 사용자는 별도의 인증을 거치지 않아도 자원에 접근이 가능합니다.

### 참고
#### 

### 출처
1. [학습중인 강의](https://www.inflearn.com/course/%EC%BD%94%EC%96%B4-%EC%8A%A4%ED%94%84%EB%A7%81-%EC%8B%9C%ED%81%90%EB%A6%AC%ED%8B%B0)