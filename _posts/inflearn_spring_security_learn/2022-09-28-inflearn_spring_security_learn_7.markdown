---
# multilingual page pair id, this must pair with translations of this page. (This name must be unique)
lng_pair: springlearn
title: 인프런 스프링 시큐리티 강의 학습-7

# post specific
# if not specified, .name will be used from _data/owner/[language].yml
author: Othkkartho's Workshop
# multiple category is not supported
category: Spring Security 공부
# multiple tag entries are possible
tags: [Spring, Spring Security 공부, 강의 정리]
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

[인증 저장소 필터](#인증-저장소-필터---securitycontextpersistencefilter), [인증 흐름 이해](#인증-흐름의-이해---authencitation-flow), [인증 관리자](#인증-관리자---authenticationmanager), [인증 처리자](#실질적-인증-처리자---authenticationprovider)에 관해 포스팅했습니다.
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

### 인증 흐름의 이해 - Authencitation Flow
#### 인증 처리 과정의 흐름의 순서
1. 클라이언트가 Form 인증 방식으로 로그인 요청을 하면 UsernamePassword AuthenticationFilter가 요청을 받아 id와 password가 담긴 인증 전 토큰 객체인 Authentication 객체를 생성합니다. 그리고 필터는 AuthenticationManager에게 인증 객체를 넘기며 인증 처리를 맡깁니다.
2. 메지저는 객체를 전달받고, 인증의 전반적인 관리를 하지만, 실제 인증 역할을 하지 않고 적절한 AuthenticationProvider에 인증을 위임합니다.
    - 내부적으로 리스트 변수가 있습니다. 이 리스트 변수 안에는 1개 이상의 AuthenticationProvider 객체가 있고, Manager는 현재 인증에 사용할 수 있는 Provider를 찾아 인증을 위임합니다.
3. AuthenticationProvider은 메니저에게 인증 객체를 전달 받고, ID, password와 같은 정보를 검증합니다.
    - Provider은 Id를 전달하면서 loadUserByUsername(username) 메소드를 호출하면서 유저 객체를 요청합니다.
4. 그럼 UserDetailService가 실질적으로 유저 객체를 조회합니다. 실제로 유저가 있다면, 유저 객체를 리턴합니다.
    - 만약 유저 정보가 없다면 예외를 발생시키고, 예외는 UsernamePasswordAuthenciationFilter가 예외를 처리합니다.
5. User가 Return되면 Service는 UserDetail Type으로 AuthenticationProvider에 반환합니다.
    - 그럼 ID 검정은 완료됬고, 반환받은 UserDetail에 있는 Password와 입력받은 Password를 매치해서 일치하는지 검사합니다.
        - 만약 password가 다르다면 예외를 날리고, 인증은 실패합니다.
6. password도 일치하면 인증에 성공한 것입니다. 그럼 Provider은 성공한 인증 결과를 담은 Authentication객체를 생성하고, Manager에게 반환합니다.
7. Manager은 Authentication 객체를 Filter에게 전달합니다. 그럼 Filter은 SecurityContext에 인증 객체를 저장합니다.
그럼 이제 인증 객체를 전역적으로 참조가 가능합니다.

### 인증 관리자 - AuthenticationManager
#### AuthenticationManger 설명 및 진행도
AuthenciationManager는 인터페이스로 제공이 되고, 이 인터페이스 구현체는 ProviderManager입니다. ProviderManager은 AuthenticationProvider 클래스들을 관리하는 클래스입니다.
- AUthenciationProvider 목록 중에 인증 처리 요건에 맞는 Provider를 찾아 인증 처리를 위임합니다.
- 부모 ProviderManager를 설정해 AuthenticationProvider를 계속 탐색할 수 있습니다.
1. 사용자가 Form 인증 방식으로 인증을 요청합니다.
    - Spring Security에는 여러 방식의 인증이 있습니다. Form, RememberMe, Oauth 인증 등이 있습니다.
2. Form 인증 필터가 메니저를 호출하면서 Authentication 객체를 전달합니다.
3. ProviderManager는 전달받은 인증 객체를 관련 처리를 할 수 있는 AuthenticationProvider에게 전달하여 인증을 위임하게 됩니다.
    - Form 방식의 경우 Dao AuthenticationProvider에게 위임하고, RemeberMe 방식의 경우 RemeberMe Provider에게 위임합니다.
    - 만약 Oauth 인증이라면 관련 필터가 ProviderManager에게 인증 처리를 위임하고, 만약 Manager에 관련 Provider가 없다면 부모 속성에 저장되 있는 ProviderManager에도 Oauth Provider를 찾기 위한 탐색을 합니다. 찾았다면 그 부모격의 ProviderManager가 Oauth Provider에 위임합니다.
4. 인증 처리를 위임받은 Provider는 인증에 성공한다면 그 성공한 인증 객체를 다시 ProviderManager에게 반환하고, Manager는 그 인증 객체를 자신을 호출한 Filter에게 전달합니다.

### 실질적 인증 처리자 - AuthenticationProvider
#### AuthenticationProvider 처리의 흐름
AuthenticationProvider는 인터페이스로 보통 서버 제작 실정에 맞게 구현해 많이 사용합니다.<br>
아래의 2개의 메소드가 제공이 됩니다.
- authenticate 메소드는 실질적인 인증 처리를 위한 검증을 합니다.
    - AuthenticationManager로 부터 authentication 객체를 전달 받습니다.
    - 인증 객체의 정보를 가지고 시스템에 저장된 사용자의 계정과 일치하는지 검정합니다.
        - ID 검정은 UserDetailsService 인터페이스에서 데이터에서 사용자 계정이 있는지 확인합니다. 있다면 사용자의 User 객체를 생성해 UserDetails 타입으로 변경해 Provider에게 전달합니다.
        - password 검정은 위 UserDetails 객체에 저장된 비밀번호와 authentication에 저장된 비밀번호를 비교를 합니다.
        - 추가 검정이 필요하다면 그 검정도 하게 됩니다.
    - 모든 검정이 끝나면 Provider는 Authentication 객체를 생성하고, 성공 결과를 저장합니다. 이 인증 객체를 AuthenticationManager에게 전달합니다.
- supports 메소드는 인증을 처리할 조건이 되는지 검사합니다.

### 출처
1. [학습중인 강의](https://www.inflearn.com/course/%EC%BD%94%EC%96%B4-%EC%8A%A4%ED%94%84%EB%A7%81-%EC%8B%9C%ED%81%90%EB%A6%AC%ED%8B%B0)