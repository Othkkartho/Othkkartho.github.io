---
# multilingual page pair id, this must pair with translations of this page. (This name must be unique)
lng_pair: springlearn
title: 인프런 스프링 시큐리티 강의 학습-8

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
date: 2022-09-29 22:00:00 +0900

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

인가 개념 및 필터, 인가 결정 심의자, 스프링 시큐리티 필터 및 아키텍쳐 정리 포스트입니다.
----------------------------------------------------------------------------

출처는 인프런의 스프링 시큐리티 - [Spring Boot 기반으로 개발하는 Spring Security](https://www.inflearn.com/course/%EC%BD%94%EC%96%B4-%EC%8A%A4%ED%94%84%EB%A7%81-%EC%8B%9C%ED%81%90%EB%A6%AC%ED%8B%B0)강의를 바탕으로 이 포스트를 작성하고 있습니다.<br>
강의의 세션 2의 9,10,11번 강의내용에 대한 정리입니다.

<!-- outline-end -->

### 인가 개념 이해 - Authorization
#### Authorization이란?
인증을 받은 사용자가 특정한 자원에 접근하려 할 때 그 자격이 되는지를 판단하는 것이 **인가** 입니다.<br>
즉 사용자에게 무엇이 허가 되었는지를 증명하는 것이라고 할 수 있습니다.<br><br>
사용자가 서버 자원에 접근하려 할 때 먼저 인증을 받았는지를 먼저 판단하고, 인증을 받았다면 사용자가 가진 권한이 각각의 자원에 설정된 권한에 접근할 수 있는 자격이 있는지 없는지 권한 심사를 합니다.<br>
이처럼 Spring Security는 인증 영역과 인가 영역으로 나누어서 처리를 하고 있습니다.

#### Spring Security가 지원하는 권한 계층
- 웹 계층
    - URL 요청에 따른 메뉴 혹은 화면 단위의 레벨 보안입니다.
    - /user과 같은 경로로 서버 자원에 접근하려고, 요청을 했을 때 그 자원에 설정된 권한과 사용자의 권한을 확인해서, 해당 URL에 접근 가능 여부를 판단하는 계층입니다.
- 서비스 계층
    - 화면 단위가 아닌 메소드 같은 기능 단위의 레벨 보안입니다.
    - 만약 사용자가 user() 메소드에 접근하려 할 때, 메소드에 설정된 권한과 사용자의 권한을 확인해 접근 가능 여부를 판단하는 계층입니다. 
- 도메인 계층(Access Control List, 접근제어목록)
    - 객체 단위의 레벨 보안입니다.
    - 만약 user객체가 있다면 이 객체를 어플리케이션에 쓰고자(write) 할 때 그 도메인에 설정된 권한과 사용자의 권한을 확인해 도메인 객체를 쓸수 있을지 없을지 확인하는 계층입니다.

### 인가 필터의 이해 - FilterSecurityInterceptor
#### Filter의 역할은?
- 마지막에 위치한 필터로써 인증된 사용자에 대해 특정 요청의 승인/거부 여부를 최종적으로 결정합니다.
- 인증객체 없이 보호자원에 접근을 시도할 경우 AuthenticationException(인증 오류)을 발생시킵니다.
- 인증 후 자원에 접근 가능한 권한이 존재하지 않을 경우 AccessDeniedException(접근 거부 예외, 인가 예외)을 발생시킵니다.
- 권한 제어 방식 중 HTTP 자원의 보안을 처리하는 필터입니다.
- 권한 처리를 AccessDecisionManager에 맡깁니다.

#### 필터의 흐름 순서
1. 사용자가 자원에 접근을 시도하면, FilterSecurityInterceptor가 사용자가 인증객체가 존재하고 있는지의 여부를 판단합니다.
    - 인증 객체가 존재하지 않으면 AuthenticationException(인증 예외)가 발생하고, ExceptionTranslationFilter가 받아 처리합니다.
2. 인증객체가 있다면 SecurityMetadataSource가 사용자가 요청한 자원에 필요한 권한 정보 조회해서 전달합니다.
    - 만약 조회한 정보에 권한 정보가 Null일 경우 권한 심사를 하지 않고, 자원 접근을 허용합니다.
3. Source가 참조한 권한정보를 최종 심의 결정자인 AccessDecisionManager에 전달해 줍니다.
    - 내부적으로는 심의자인 AccessDecisionVoter 클래스를 통해 심의요청을 합니다.
    - 심의를 해 최종적으로 승인이나 거부 값을 전달해줍니다.
4. AccessDecisionVoter에게 Manager가 값을 받으면 접근 여부를 판단합니다
    - 접근이 거부되면 AccessDeniedException(인가 예외)를 ExceptionTranslationFilter가 처리합니다.
5. 접근이 승인되면 자원 접근이 허용됩니다.

#### 실제 코드 확인
**SecurityController.java**{:data-align="center"}
```java
@GetMapping("/")
public String index() {
    return "home";
}

@GetMapping("/user")
public String user() {
    return "user";
}
```

**SecurityConfig.java**{:data-align="center"}
```java
@Bean
public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
    http
            .authorizeRequests()
            .antMatchers("/user").hasRole("USER")       // 1
            .anyRequest().authenticated();
    http
            .formLogin();

    return http.build();
}
```
1. /user 경로에 접근하기 위해서는 USER 권한이 있어야 합니다.
위와 같이 코드를 작성하고, /user에 인증없이 접근하게 되면 아래의 사진과 같이 인증 객체가 익명 사용자로 만들어지게 됩니다.
**인증을 받지 않은 인증 객체**{:data-align="center"}
![인증을 받지 않고, 자원에 접근](:/inflearn_spring_security_learn/2s/8/anonymous_authentication.jpg){:data-align="center"}
권한은 사진을 확인해보면 anonymous 권한으로 익명 사용자 권한이기 때문에 인증 예외가 반환되고, 로그인 페이지가 뜹니다.
**인증 완료 후 인증 객체**{:data-align="center"}
![인증 완료 후 인증 객체](:/inflearn_spring_security_learn/2s/8/login_authentication.jpg){:data-align="center"}
로그인을 하면 위 사진과 같은 인증 객체가 생성되는데 Security에서 기본적으로 제공하는 계정은 권한 설정이 되어있지 않은 것을 확인할 수 있습니다.<br>
그래서 인가 예외를 던지고, 화면에는 There was an unexpected error (type=Forbidden, status=403) 에러가 출력됩니다.

**application.properties**{:data-align="center"}
```java
spring.security.user.roles=USER
```
위 코드로 변경 후 다시 로그인을 하고, 유저 경로에 접근하면 아래와 같은 인증 객체가 생성되는 것을 확인할 수 있습니다.
**user 권한 설정 후 인증 객체**{:data-align="center"}
![user 권한 설정 후 인증 객체](:/inflearn_spring_security_learn/2s/8/user_authentication.jpg){:data-align="center"}
USER 권한이 잘 설정되어 있는 것을 확인할 수 있고, /user 경로로 접근이 완료됩니다.

### 인가 결정 심의자 - AccessDecisionManager, AccessDecisionVoter
#### AccessDecisionManager의 역할 및 유형
- 인증 정보, 요청 정보, 권한 정보를 이용해 사용자의 자원접근을 허용할지 거부할지를 최종 결정하는 주체입니다.
- 여러 개의 Voter들을 가질 수 있으며 Voter 들로부터 접근 허용, 거부, 보류에 해당하는 각각의 값을 리턴받고 판단 및 결정합니다.
- 최종 접근 거부 시 예외를 발생시킵니다.
- 접근 결정의 세가지 유형
    - **AffirmativeBased** 는 여러개의 Voter 클래스 중 하나라도 접근 허가로 결론이 나면 접근 허가로 판단합니다.
    - **ConsensusBased** 는 다수결에 의해 최종 결정을 판단합니다. 동수일 경우 기본은 접근허가이나 allowIfEqualGrantedDeniedDecisions를 false로 설정할 경우 접근거부로 결정됩니다.
    - **UnanimousBased** 는 모든 Voter가 만장일치로 접근을 승인해야 하며 그렇지 않은 경우 접근을 거부합니다.

#### AccessDecisionVoter의 역할
- 판단을 심사하는 위원입니다. 각각의 Voter들 마다 사용자 요청에 대해 자원에 접근할 자격이 있는지를 결정해 Manager에게 return하는 객체입니다.
- Voter가 원한 부여 과정에서 판단하는 자료는 3개입니다.
    - **Authentication** - 인증 정보(user)
    - **FilterInvocatior** - 요청 정보(antMatcher("/user"))
    - **ConfigAttributes** - 권한 정보 (hasRole("USER"))
- 결정 방식도 3개가 있습니다.
    - **ACCESS_GRANTED** : 접근 허용(1)
    - **ACCESS_DENIED** : 접근 거부(-1)
    - **ACCESS_ABSTAIN** : 접근 보류(0)
        - Voter가 해당 타입의 요청에 대해 결정을 내릴 수 없는 경우 접근 보류를 결정합니다.

#### Manager, Voter의 진행 순서
1. FilterSecurityInterceptor가 AccessDecisionManager에게 decide(authentication, object, configAttributes) 객체 전달과 함께 인가처리를 맡깁니다.
    - **Authentication(authentication)** : 인증 정보, **FilterInvocation(object)** : 요청 정보, **ConfigAttributes(configAttributes)** : 권한 정보
2. Manager가 본인이 가지고 있는 여러 Voter들에게도 decide를 전달해 권한 판단 심사를 맡깁니다.
3. Voter들은 위 전달 파라미터들을 가지고 권한 판단을 심사합니다.
4. 심사 후 ACCESS_GRANTED, ACCESS_DENIED, ACCESS_ABSTAIN 중 하나를 Return합니다.
5. 성공하면 Manager는 Filter에게 ACCESS_GRANTED를 전달합니다.
    - 실패하면 AccessDeniedException을 발생해 ExceptionTranslationFilter가 처리하도록 합니다.

### 스프링 시큐리티 필터 및 아키텍쳐 정리
**총 정리 PPT 사진**{:data-align="center"}
![총 정리 PPT 사진](:/inflearn_spring_security_learn/2s/8/ppt.jpg){:data-align="center"}
위 사진의 출처는 강의 자료입니다. 강의의 출처는 강의 관련 포스트의 서론과 출처 란에 있습니다.<br><br>
간단한 설명과 함께 자세한 내용의 포스트를 링크로 만들었습니다.
1. [WebSecurityConfigurerAdapter](/posts/2022-09-20-inflearn_spring_security_learn_1#스프링-26-이전의-securityconfig)
    - SecurityConfig 제작시 extends하는 클래스이지만 현재 Deprecated가 되어 상속받지는 않는 클래스입니다. 현재는 [Bean](/_posts/2022-09-20-inflearn_spring_security_learn_1.markdown#스프링-27-이후의-securityconfig) 형태로 HttpSecurity를 생성합니다.
2. [DelegatingFilterProxy](/posts/2022-09-27-inflearn_spring_security_learn_6#delegatingfilterproxy의-간단한-설명)
    - 서블릿 필터로 부터 받은 요청을 DelegatingFilterProxy는 Spring Bean에게 요청을 위임하고, 스프링 시큐리티는 필터 기반으로 보안처리를 하고, 스프링의 기술도 사용할 수 있게 됩니다.
3. [FilterChainProxy](/posts/2022-09-27-inflearn_spring_security_learn_6#filterchainproxy-설명)
    - DelegatingFilterProxy로 부터 요청을 위임받고, 실제 보안 처리를 합니다.
4. [SecurityContextPersistenceFilter](/posts/2022-09-28-inflearn_spring_security_learn_7#인증-저장소-필터---securitycontextpersistencefilter)
    - **SecurityContext 객체의 생성, 저장, 조회.** 를 하는 필터입니다.
5. [LogoutFilter](/posts/2022-09-24-inflearn_spring_security_learn_3#logoutfilter)
    - 클라이언트가 로그아웃 요청을 하면, 서버는 세션 무효화, 인증토큰 삭제, 쿠키정보 삭제, 로그인 페이지로 리이렉트 시킵니다.
6. [UsernamePasswordAuthenticationFilter](/_posts/2022-09-22-inflearn_spring_security_learn_2#인증-api---usernamepasswordauthenticationfilter)
    - Filter는 사용자가 로그인 후의 인증처리를 담당하고, 관련 요청을 처리하는 필터입니다.
7. [concurrentSessionFilter](/posts/2022-09-25-inflearn_spring_security_learn_4#concurrentsessionfilter)
    - ConcurrentSessionFilter는 매 요청 마다 현재 사용자의 세션 만료 여부를 체크하고, 만약 세션이 만료되었을 경우 즉시 만료 처리를 합니다.
8. [RememberMeAuthenticationFilter](/posts/2022-09-24-inflearn_spring_security_learn_3#remembermeauthenticationfilter)
    - 필터는 인증을 받은 사용자가 세션이 만료되었거나, 사용하던 브라우저가 종료되어 세션이 끊겨 세션이 활성화되지 않아 인증 객체를 세션에서 찾지 못하는 경우 자동적으로 사용자의 인증을 유지하기 위해 필터가 인증을 시도해 다시 인증을 받게 만들어 그 사용자가 서버에 인증이 유지된채 접근이 가능하도록 합니다.
9. [AnaonyMousAuthenticationFilter](/posts/2022-09-25-inflearn_spring_security_learn_4#인증-api---anonymousauthenticationfilter)
    - 이 필터는 인증을 받지 않은 사용자를 null로 처리하는 것이 아니라 익명 사용자용 인증 객체를 만들어 처리합니다.
10. [SessionManagementFilter](/posts/2022-09-25-inflearn_spring_security_learn_4/#sessionmanagementfilter)
    - 세션 관리는 인증 시 사용자의 세션정보를 등록, 조회, 삭제 등의 세션 이력을 관리합니다.
11. [ExceptionTranslationFilter](/posts/2022-09-26-inflearn_spring_security_learn_5#인증인가-api---exceptiontranslationfilter)
    - 인증, 인가 예외 처리
12. [FilterSecurityInterceptor](#인가-필터의-이해---filtersecurityinterceptor)
    - 인증객체 없이 보호자원에 접근을 시도할 경우 AuthenticationException(인증 오류)을 발생시킵니다.

### 출처
1. [학습중인 강의](https://www.inflearn.com/course/%EC%BD%94%EC%96%B4-%EC%8A%A4%ED%94%84%EB%A7%81-%EC%8B%9C%ED%81%90%EB%A6%AC%ED%8B%B0)