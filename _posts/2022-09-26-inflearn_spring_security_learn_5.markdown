---
# multilingual page pair id, this must pair with translations of this page. (This name must be unique)
lng_pair: springlearn
title: 인프런 스프링 시큐리티 강의 학습-5

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
date: 2022-09-26 10:00:00 +0900

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

권한설정과 표현식, 예외 처리 및 요청 캐시 필터, 사이트 간 요청 위조에 관해 포스팅 했습니다.
---------------------------------------------------------------------------------------

출처는 인프런의 스프링 시큐리티 - [Spring Boot 기반으로 개발하는 Spring Security](https://www.inflearn.com/course/%EC%BD%94%EC%96%B4-%EC%8A%A4%ED%94%84%EB%A7%81-%EC%8B%9C%ED%81%90%EB%A6%AC%ED%8B%B0)강의를 바탕으로 이 포스트를 작성하고 있습니다.<br>
강의의 세션 1의 11,12,13번 강의내용에 대한 정리입니다.

<!-- outline-end -->

### 인가 API - 권한 설정
#### 권한 설정의 방식 2가지
- 선언적 방식
    - URL로 표현하면 http.anyMatchers("/users/**").hasRole("USER")로 표현할 수 있습니다.
    - Method 방식은 @PreAuthorize("hasRole('USER')") \n public void user() {System.out.println("user")} 로 표현할 수 있습니다.
- 동적 방식은 DB 연동 프로그래밍으로 구현합니다
    - 동적 방식 또한 URL과 Method 형태로 구현할 수 있습니다.
이 포스트에서는 선언적 방식의 URL 형태로 권한 설정해보겠습니다.

#### 권한 설정 실행 코드
```java
http.antMatcher("/shop/**")                                                         // 1
    .authorizeRequests()
    .antMatchers("/shop/login", "/shop/users/**").permitAll()                       // 2
    .antMatchers("/shop/mypage").hasRole("USER")
    .antMatchers("/shop/admin/pay").access("hasRole('ADMIN')");                     // 4
    .antMatchers("/shop/admin/**").access("hasRole('ADMIN') or hasRole('SYS')");    // 5
    .anyRequest().authenticated()                                                   // 3
```
1. 특정한 경로를 지정해 사용자의 요청이 설정한 경로로 접근할 때만 설정 클레스의 보안 기능이 작동합니다. 만약 아무 경로도 지정하지 않는다면 모든 경로에 관해 보안 검색을 합니다.
2. antMatchers는 설정한 경로나 설정한 경로에 포함된 모든(/**)경로들의 경우 뒤의 권한 심사를 하겠다는 코드입니다.
    - permitAll()은 유저 권한에 관계없이 허용하겠다는 의미입니다.
    - hasRole()은 경로에 접근하고자 하는 사용자는 해당 권한 심사를 하고, 사용자가 그 권한을 가지고 있어야 한다는 의미입니다.
    - access()를 통해 권한 정보를 보다 자세하게 표현식을 사용해 권한 설정을 할 수 있게 제공하고 있습니다.
3. 위 설정을 한 특정 경로들을 제외한 모든 요청에 관해서는 인증된 사용자만이 접근이 가능하다는 것을 의미합니다.
4. 5번 에서 의미하는 모든("/**") 경로는 위의 pay 경로(주석 4번)를 제외한 경로에 권한 설정을 의미합니다.
    - 주의 사항은 설정 시 구체적인 경로(주석 4번)가 먼저 오고 그것 보다 큰 범위(주석 5번)의 경로가 뒤에 오도록 해야합니다. 이유는 인증 처리를 위에서 아래로 설정하기 때문입니다.

#### 인증, 권한과 관련된 표현식

| 메소드 | 동작 |
| :------------------: | :--------------: |
| authenticated() | 인증된 사용자의 접근을 허용 |
| fullyAuthenticated() | 인증된 사용자의 접근을 허용, rememberMe 인증 제외 |
| permitAll() | 무조건 접근을 허용 |
| denyAll() | 무조건 접근을 허용하지 않음 |
| anonymous() | 익명사용자의 접근을 허용, 인증된 사용자는 접근이 허용되지 않음 |
| rememberMe() | rememberMe를 통해 인증된 사용자의 접근을 허용 |
| access(String) | 주어진 SpEL 표현식의 평가 결과가 true이면 접근을 허용 |
| hasRole(String) | 사용자가 주어진 역할이 있다면 접근을 허용, ROLE 프리비어스를 붙이지 않음 |
| hasAuthority(String) | 사용자가 주어진 권한이 있다면, ROLE 프리비어스를 붙임 |
| hasAnyRole(String...) | 사용자가 주어진 권한이 있다면 접근을 허용 |
| hasAnyAuthority(String...) | 사용자가 주어진 권한 중 어떤 것이라도 있다면 접근을 허용 |
| hasIpAddress(String) | 주어진 IP로부터 요청이 왔다면 접근을 허용 |
{:data-align="center"}

#### 실제 코드
**SecurityConfig.java**{:data-align="center"}
```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {
    @Bean
    public InMemoryUserDetailsManager userDetailsService() {    // 1
        UserDetails user = User.withUsername("user").password("{noop}1234").roles("USER").build();
        UserDetails sys = User.withUsername("sys").password("{noop}1234").roles("SYS", "USER").build();
        UserDetails admin = User.withUsername("admin").password("{noop}1234").roles("ADMIN", "SYS", "USER").build();
        return new InMemoryUserDetailsManager(user, sys, admin);
    }

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
                .authorizeRequests()
                .antMatchers("/user").hasRole("USER")
                .antMatchers("/admin/pay").hasRole("ADMIN")
                .antMatchers("/admin/**").access("hasRole('ADMIN') or hasRole('SYS')")
                .anyRequest().authenticated();
        http
                .formLogin();

        return http.build();
    }
}
```

**SecurityController.java**{:data-align="center"}
```java
@GetMapping("/user")
public String user() {
    return "user";
}

@GetMapping("/admin/pay")
public String adminPay() {
    return "adminPay";
}

@GetMapping("/admin/**")
public String admin() {
    return "admin";
}
```
위 코드의 주석 1번은 강의 중 아래 코드를 WebSecurityConfigurerAdapter의 Override를 사용하지 않는 방법을 찾아서 변경한 코드입니다. 출처는 출처 스레드 2번에 링크되어 있습니다.<br><br>
**SecurityConfig.java**{:data-align="center"}
```java
@Override
protected void configure(AuthenticationManagerBuilder auth) throws Exception {
    auth.inMemoryAutentication().withUser("user").password("{noop}1111").roles("USER");
    auth.inMemoryAutentication().withUser("sys").password("{noop}1111").roles("SYS","USER");
    auth.inMemoryAutentication().withUser("admin").password("{noop}1111").roles("ADMIN","SYS","USER");
}
```

실행은 같은 사진의 연속이기 때문에 굳이 모든 실행 화면을 다 첨부하진 않고, USER 권한을 가진 사용자만 첨부하겠습니다.<br>
![유저 정보를 가진 사용자 로그인](:/inflearn_spring_security_learn/1s/5/user_login.jpg){:data-align="center"}
![유저 경로에 접근 가능](:/inflearn_spring_security_learn/1s/5/user_auth.jpg){:data-align="center"}
![어드민 경로에 접근 불가능](:/inflearn_spring_security_learn/1s/5/user_auth_error.jpg){:data-align="center"}
차례대로 유저 정보를 가진 사용자로 로그인을 하고, /user 경로에 접근을 했습니다. 하지만 ADMIN이나 SYS 권한을 가져야 하는 /admin 경로에는 접근하지 못하는 것을 확인할 수 있습니다.<br>
SYS 권한을 가진 사용자로 로그인을 하게 되면 /admin은 접근이 가능하지만, /admin/pay는 접근이 불가능 한 것을 확인할 수 있고, ADMIN 권한을 가진 사용자는 모두 접근할 수 있는 것을 확인하실 수 있을 겁니다.<br>

### 인증/인가 API - ExceptionTranslationFilter
#### Filter의 처리
- AuthenticationException(인증 예외 처리)
    1. AuthenticationEntryPoint를 호출해 재인증을 위한 로그인 페이지 이동과 401 오류 코드 전달 등의 역할을 합니다.
    2. 인증 예외가 발생하기 전의 요청 정보를 저장합니다.(로그인과 인증이 성공했을 시 사용자가 요청한 경로로 자동으로 보내기 위한 것입니다.)
        - RequestCache는 사용자의 이전 요청 정보를 세션에 저장하고 이를 꺼내 오는 캐시 메카니즘입니다.
            - SaveRequest는 사용자가 요청했던 request 파라미터 값들, 그 당시의 헤더값들 등이 저장됩니다.
- AccessDeniedException(인가 예외 처리)
    - AccessDeniedHandler에서 예외 처리하도록 제공합니다.
위 2개의 예외를 던지는 필터는 FilterSecurityInterceptor입니다. 이 필터는 스프링 시큐리티가 관리하는 보안 필터 중 맨 마지막에 위치하게 되고, 이 필터 앞에 위치하는 필터가 ExceptionTranslationFilter입니다.<br>
Exception 필터가 사용자 요청을 받을 때 try~catch로 감싸서 Filter 인터셉터를 호출하고, 2개의 예외를 던지고, 그 예외를 처리합니다.

#### 필터의 실행 순서
사용자가 인증을 하지 않고 /user에 접근을 시도한다면
0. 익명 사용자의 접근이기 때문에 인가 예외가 발생됩니다. ExceptionTranslationFilter가 AccessDeniedException으로 보냅니다만, 익명 사용자나, RememberMe 인증 사용자일 경우 AccessDeniedHandler가 아닌 인증 예외를 처리하는 AuthenticationException으로 보냅니다.
1. 그렇게 인증 예외가 발생되고, AuthenticationExcption이 2가지 처리를 합니다.
    - SecurityContext를 null로 만들고, AuthenticationEntryPoint로 이동해 response.redirect(/login)을 통해 인증 처리를 하도록 합니다.
    - 사용자의 요청관련 정보를 DefaultSavedRequest 객체 안에 /user 정보를 저장하고, 그 객체를 Session에 저장합니다. 그리고 Session에 저장하는 역할을 HttpSessionRequestCache가 합니다.
***
0. 사용자가 인증은 했지만 권한이 맞지 않다면 인가 예외가 발생합니다.
1. ExceptionTranslationFilter가 AccessDeniedException으로 인가 예외를 처리합니다.
    - AccessDeniedHandler를 호출해 부속작업을 처리합니다. 보통은 response.redirect(/denied)을 호출해 인가 오류를 띄웁니다.

#### 실행 코드

```java
http.exceptionHandling()                                    // 1
    .authenticationEntryPoint(authenticationEntryPoint())   // 2
    .accessDeniedHandler(accessDeniedHandler())             // 3
```
1. 예외처리 기능이 작동하게 합니다.
2. API를 통해 인터페이스를 구현해 클래스를 만들어 설정하면 만든 클래스를 호출합니다.
    - commence 메소드 안에서 인증 예외가 발생했을 경우 로그인 페이지나, 인증오류 코드를 띄울 수 있습니다.
3. 인가 예외가 발생시 인터페이스를 구현해 클래스를 만들면 만든 클래스를 호출해 실패 처리를 진행합니다.
    - 보통 /denied로 가 인가 실패 메시지를 띄웁니다.

#### 실제 코드
**SecurityConfig.java**{:data-align="center"}
```java
http
    .formLogin()
    .successHandler(new AuthenticationSuccessHandler() {
        @Override
        public void onAuthenticationSuccess(HttpServletRequest request, HttpServletResponse response, Authentication authentication) throws IOException, ServletException { // 1
            RequestCache requestCache = new HttpSessionRequestCache();
            SavedRequest savedRequest = requestCache.getRequest(request, response);
            String redirectUrl = savedRequest.getRedirectUrl();
            response.sendRedirect(redirectUrl);
        }
    });

http
    .exceptionHandling()
    .authenticationEntryPoint(new AuthenticationEntryPoint() {
        @Override
        public void commence(HttpServletRequest request, HttpServletResponse response, AuthenticationException authException) throws IOException, ServletException {        // 2
            response.sendRedirect("/login");
        }
    })
    .accessDeniedHandler(new AccessDeniedHandler() {
        @Override
        public void handle(HttpServletRequest request, HttpServletResponse response, AccessDeniedException accessDeniedException) throws IOException, ServletException {    // 3
            response.sendRedirect("/denied");
        }
    });
```

**SecurityController.java**{:data-align="center"}
```java
@GetMapping("/denied")
public String denied() {
    return "access is denied";
}

@GetMapping("/login")
public String login() {
    return "login";
}
```
1. 사용자가 인증 전에 요청했던 경로를 저장했다가 로그인에 성공했을 때 그 경로로 이동하는 코드입니다.
2. 인증 예외가 발생했을 때 login 페이지로 이동시키는 코드입니다.
3. 인가 예외가 발생했을 때 denied 페이지로 이동시키는 코드입니다.
    - 주석 2, 3번의 경우 스프링 시큐리티가 기본으로 제공하는 페이지를 이용하지 못하기 때문에 Controller에 간단한 코드를 추가했습니다.
아래는 실제 실행 장면입니다.<br>
먼저 인증을 하지 않고 루트 페이지에 접속 해보았습니다.
![인증 예외 처리](:/inflearn_spring_security_learn/1s/5/authenticationentrypoint.jpg){:data-align="center"}
인증 예외에서 작성한 것과 같이 /login으로 이동했습니다.<br><br>
로그인을 하기 위해 인증 예외를 주석처리 하고, USER 권한 사용자로 로그인 후 /admin으로 접속했습니다.
![인가 예외 처리](:/inflearn_spring_security_learn/1s/5/accessDeniedHandler.jpg){:data-align="center"}
인가 예외 코드를 작성한 것과 같이 /denied 페이지로 이동해 정상적으로 출력 되는 것을 확인했습니다.<br><br>
페이지 자동 이동도 같이 확인하기 위해 /admin으로 요청을 진행했습니다.
![인증 없이 어드민 경로 접근](:/inflearn_spring_security_learn/1s/5/admin_access.jpg){:data-align="center"}
그 후 인증을 위해 로그인 페이지로 이동해 ADMIN 권한 사용자 게정으로 로그인을 하였습니다.
![인증을 위해 로그인 페이지 이동](:/inflearn_spring_security_learn/1s/5/admin_login.jpg){:data-align="center"}
![인증 완료 후 페이지 자동 이동](:/inflearn_spring_security_learn/1s/5/requestCache.jpg){:data-align="center"}
인증이 완료되고, 인증되기 전 접근을 요청한 경로로 이동하는 것을 확인 할 수 있습니다.

### 사이트 간 요청 위조 - CSRF, CsrfFilter
#### CSRF(사이트 간 요청 위조) 절차
사용자와 공격자, 쇼핑몰 서버가 있다고 가정하겠습니다.
1. 사용자가 쇼핑몰에 로그인을 하고, 서버로부터 세션 쿠키를 발급받습니다.
2. 공격자는 사용자에게 어떤(이메일, 쪽지 등) 방법을 통해 공격자의 웹페이지 링크를 전달합니다.
3. 사용자는 링크를 클릭해 공격자가 만든 공격용 웹페이지에 접속합니다.
    - 링크에는 어떤 링크, 또는 사진이 있고, 그 곳에는 쇼핑몰의 링크와 공격자의 주소가 (src="https://shop.com/address=공격자 주소")가 설정이 되있습니다.
4. 사용자가 공격용 페이지를 열면, 사용자의 브라우저는 이미지 파일을 받아오기 위해 공격용 URL을 엽니다.
5. 쇼핑몰은 요청에 발급된 쿠키가 있는 것을 확인하고, 서버에 사용자의 승인이나 인지 없이 배송지가 등록됨으로써 공격이 완료됩니다.

#### CsrfFilter
- 모든 요청에 랜덤하게 생성된 토큰을 클라이언트에 발급을 하고, 클라이언트가 서버에 접속하기 위해 토큰을 HTTP 파라미터로 요구합니다.
- 요청 시 전달되는 토큰 값과 서버에 저장된 실제 값과 비교한 후 만약 일치하지 않으면 요청은 실패합니다.
- Client
    - `<input type="hiden" name="${csrf.parameterName}" value="${_csrf.token}"/>`
        - ${csrf.parameterName}는 서버에서 발급한 토큰 명이고, ${_csrf.token}은 토큰 값입니다.
    - 서버에 자원에 HTTP 메소드(PATCH, POST, PUT, DELETE)로 접근할 때는 토큰 명과 토큰 값을 가지고 요청을 해야합니다.
        - 그렇지 않다면 사용자가 접근하지 못하도록 처리합니다.
- Spring Security
    - http.csrf() 기본 활성화 되어있고, 만약 비활성화 하고 싶다면 .disabled()를 붙이면 됩니다.
이렇게 하면 위에 예시에서 사용자와 쇼핑몰은 csrf 토큰 값으로 주고 받으며 정상적으로 요청을 처리하지만, 공격자의 링크에 경우 CsrfFilter가 csrf 값을 확인할 때 URL에 csrf 값이 없거나 다르기 때문어 쇼핑몰 서버는 요청을 수락하지 않으므로써 csrf 공격을 방지합니다.

#### 실제 실행 화면
먼저 localhost:8080에 접속해 로그인을 한 후 크롬 확장자를 통해 http://localhost:8080에 접속하겠습니다.<br>
해더에 아무 값도 넣지 않고, 접속한다면 actualToken은 null이 됩니다.
![actualToken Null](:/inflearn_spring_security_learn/1s/5/actualToken_null.PNG){:data-align="center"}
그럼 결국 인증에 실패하고, 에러가 발생하게 됩니다.
![403 Error](:/inflearn_spring_security_learn/1s/5/token_403.PNG){:data-align="center"}
이번에는 서버에서 주는 토큰 값을 확인하고, 해더에 값을 넣어 요청을 보내보겠습니다.
![토큰 값](:/inflearn_spring_security_learn/1s/5/token_value_1.PNG){:data-align="center"}
![해더에 입력](:/inflearn_spring_security_learn/1s/5/token_header.PNG){:data-align="center"}
그럼 아래 사진과 같이 acturalToken에 해더에 저장했던 값이 출력되고, 정상적으로 값이 출력됩니다.
![acutalToken 값](:/inflearn_spring_security_learn/1s/5/actualToken.PNG){:data-align="center"}
![요청에 대한 결과 정상 출력](:/inflearn_spring_security_learn/1s/5/auth_end.PNG){:data-align="center"}
위의 home은 SecurityController.java에 관련 코드를 추가해 나오는 화면입니다.<br>
**SecurityController.java**{:data-align="center"}
```java
@PostMapping("/")
public String goHome() {
    return "home";
}
```

그럼 이번에는 csrf 기능을 끈다면 어떻게 작동할 지 확인해 보도록 하겠습니다.<br>
**SecurityConfig.java**{:data-align="center"}
```java
http.csrf().disable();
```
![FilterChainProxy에 csrf 필터가 없음](:/inflearn_spring_security_learn/1s/5/csrf_disable.PNG){:data-align="center"}
위 사진은 FilterChainProxy입니다. 위 필더에서도 보면 csrf 필터가 체인에 없는 것을 확인할 수 있습니다. 그리고 실제로 csrf 값이 이상해도 검사를 하지 않기 때문에 요청에 대한 결과를 보여줍니다.<br>
결과는 따로 사진에 첨부하지 않겠습니다.

### 참고
#### 유저 권한 설정 순서의 중요성에 관해
유저 권한을 설정할 때 순서를 다르게 하면 어떻게 되는지 실제 구동을 통해 확인해 보겠습니다.
**SecurityConfig.java**{:data-align="center"}
```java
@Bean
public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
    http
            .authorizeRequests()
            .antMatchers("/user").hasRole("USER")
            .antMatchers("/admin/**").access("hasRole('ADMIN') or hasRole('SYS')")
            .antMatchers("/admin/pay").hasRole("ADMIN")
            .anyRequest().authenticated();
    http
            .formLogin();

    return http.build();
}
```
코드를 위와 같이 설정한다면 원래 /admin/pay 경로로는 SYS 권한을 가진 사용자가 접근할 수 없어야 하지만 접근이 가능한 것을 확인할 수 있습니다.
![SYS 사용자 로그인](:/inflearn_spring_security_learn/1s/5/sys_login.jpg){:data-align="center"}
![/admin/pay 경로에 SYS 사용자가 접근 가능](:/inflearn_spring_security_learn/1s/5/sys_adminPay.jpg){:data-align="center"}

### 출처
1. [학습중인 강의](https://www.inflearn.com/course/%EC%BD%94%EC%96%B4-%EC%8A%A4%ED%94%84%EB%A7%81-%EC%8B%9C%ED%81%90%EB%A6%AC%ED%8B%B0)
2. [Spring 공식 블로그 - Spring Security without the WebSecurityConfigurerAdapter, In-Memory Authentication 부분](https://spring.io/blog/2022/02/21/spring-security-without-the-websecurityconfigureradapter)