---
# multilingual page pair id, this must pair with translations of this page. (This name must be unique)
lng_pair: springlearn
title: 인프런 스프링 시큐리티 강의 학습-14

# post specific
# if not specified, .name will be used from _data/owner/[language].yml
author: Othkkartho's Workshop
# multiple category is not supported
category: Spring 공부
# multiple tag entries are possible
tags: [Spring, Spring 공부, 강의 정리, Spring Security]
# thumbnail image for post
img: ":/inflearn_spring_security_learn/post_spring_security.png"
# disable comments on this page
#comments_disable: true

# publish date
date: 2022-10-09 22:00:00 +0900

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

[인증 핸들러](#인증-핸들러---ajaxauthenticationsuccesshandler-failurehandler), [인증 및 인가 예외 처리](#인증-및-인가-예외-처리---ajaxloginurlauthenticatoinentrypoint-ajaxaccessdeniedhandler)를 정리한 포스트입니다.
--------------------------------------

출처는 인프런의 스프링 시큐리티 - [Spring Boot 기반으로 개발하는 Spring Security](https://www.inflearn.com/course/%EC%BD%94%EC%96%B4-%EC%8A%A4%ED%94%84%EB%A7%81-%EC%8B%9C%ED%81%90%EB%A6%AC%ED%8B%B0)강의를 바탕으로 이 포스트를 작성하고 있습니다.<br>
강의의 세션 4의 4,5번 강의내용에 대한 정리입니다.<br><br>

<!-- outline-end -->

### 인증 핸들러 - AjaxAuthenticationSuccessHandler, FailureHandler
#### 실제 코드
**AjaxAuthenticationSuccessHandler**{:data-align="center"}
```java
public class AjaxAuthenticationSuccessHandler implements AuthenticationSuccessHandler {
    private final ObjectMapper objectMapper = new ObjectMapper();

    @Override
    public void onAuthenticationSuccess(HttpServletRequest request, HttpServletResponse response, Authentication authentication) throws IOException {
        Account account = (Account) authentication.getPrincipal();      // 1

        response.setStatus(HttpStatus.OK.value());                      // 2
        response.setContentType(MediaType.APPLICATION_JSON_VALUE);      // 3

        objectMapper.writeValue(response.getWriter(), account);         // 4
    }
}
```
1. AjaxAuthenticationProvider에서 AjaxAuthenticationToken을 통해 전달된 인증에 성공한 Account 객체를 저장합니다.
2. HTTP 상태 코드를 전달합니다.
    - ok는 정상 성공을 의미하는 200번을 전달합니다. 다른 값들은 [참고](#참고) 2번에 있습니다.
3. HTTP에서 정의된 품질 매개 변수에 대한 지원을 추가하는 MimeType의 하위 클래스입니다. 다른 값들은 [참고](#참고) 3번에 있습니다.
4. Mapper가 JSON 형식으로 account 객체를 변환해 클라이언트로 전달해 주게 하기 위한 코드입니다.

**AjaxAuthenticationFailureHandler**{:data-align="center"}
```java
public class AjaxAuthenticationFailureHandler implements AuthenticationFailureHandler {
    private final ObjectMapper objectMapper = new ObjectMapper();

    @Override
    public void onAuthenticationFailure(HttpServletRequest request, HttpServletResponse response, AuthenticationException exception) throws IOException {
        String errorMsg = "Invalid Username or Password";

        response.setStatus(HttpStatus.UNAUTHORIZED.value());
        response.setContentType(MediaType.APPLICATION_JSON_VALUE);

        if (exception instanceof BadCredentialsException) {
            errorMsg = "Invalid Username or Password";
        }
        else if (exception instanceof DisabledException) {
            errorMsg = "Locked";
        }
        else if (exception instanceof InsufficientAuthenticationException) {
            errorMsg = "Invalid Secret Key";
        }

        objectMapper.writeValue(response.getWriter(), errorMsg);
    }
}
```
Url 방식과 마찬가지로 실패의 예외 코드를 확인하고, 그 값을 JSON 형태로 클라이언트로 전달해 줍니다.

***SecurityConfig.java**{:data-align="center"}
```java
@Bean
public AuthenticationSuccessHandler ajaxAuthenticationSuccessHandler() {
    return new AjaxAuthenticationSuccessHandler();
}

@Bean
public AuthenticationFailureHandler ajaxAuthenticationFailureHandler() {
    return new AjaxAuthenticationFailureHandler();
}

@Bean
public AjaxLoginProcessingFilter ajaxLoginProcessingFilter() throws Exception {
    AjaxLoginProcessingFilter ajaxLoginProcessingFilter = new AjaxLoginProcessingFilter();
    ajaxLoginProcessingFilter.setAuthenticationManager(authenticationManager(authenticationConfiguration));
    ajaxLoginProcessingFilter.setAuthenticationSuccessHandler(ajaxAuthenticationSuccessHandler());
    ajaxLoginProcessingFilter.setAuthenticationFailureHandler(ajaxAuthenticationFailureHandler());
    return ajaxLoginProcessingFilter;
}
```
실제 성공, 실패 핸들러를 필터에 등록해 실행될 수 있게 합니다.

#### 실제 실행 장면
**ProviderManager가 받은 authentication 객체**{:data-align="center"}
![정상적으로 authentication 객체가 전달됨](:/inflearn_spring_security_learn/3s/14/success_account.jpg){:data-align="center"}
ProviderManager가 Ajax 로 전달한 데이터를 authentication 객체에 정상적으로 받아 저장하고 있습니다.

**모든 인증을 마치고, 클라이언트에 응답이 전달됨**{:data-align="center"}
![SuccessHandler가 정상적으로 작동되고 값이 전달됨](:/inflearn_spring_security_learn/3s/14/http_request.jpg){:data-align="center"}
SuccessHandler의 모든 처리를 마치고, 설정했던 유저 값이 JSON 형식으로 클라이언트에 정상 전달되는 것을 확인할 수 있습니다.

**AbstractAuthenticationProcessingFilter에 ex 값이 전달**{:data-align="center"}
![전달된 ex 값이 설정한 값으로 잘 전달됨](:/inflearn_spring_security_learn/3s/14/ex.jpg){:data-align="center"}
AjaxLoginProcessingFilter에 설정한 `throw new IllegalArgumentException("username or Password is empty");` 가 정상적으로 AbstractAuthenticationProcessingFilter의 ex 객체에 저장된 것을 확인할 수 있습니다.

**예외 처리 후 클라이언트에 응답이 전달됨**{:data-align="center"}
![예외 처리 후 클라이언트에 정상적으로 값이 전달됨](:/inflearn_spring_security_learn/3s/14/http_request_error.jpg){:data-align="center"}
예외 처리 값이 FailureHandler를 타고 정상적으로 값이 전달된 것을 확인할 수 있습니다.

### 인증 및 인가 예외 처리 - AjaxLoginUrlAuthenticatoinEntryPoint, AjaxAccessDeniedHandler
#### 실제 코드
**AjaxLoginUrlAuthenticationEntryPoint.java**{:data-align="center"}
```java
public class AjaxLoginUrlAuthenticationEntryPoint implements AuthenticationEntryPoint {
    @Override
    public void commence(HttpServletRequest request, HttpServletResponse response, AuthenticationException authException) throws IOException { 
        response.sendError(HttpServletResponse.SC_UNAUTHORIZED, "UnAuthorized");
    }
}
```
익명 사용자가 인증이 필요한 자원에 접근할 경우 인증을 받지 않은 사용자에게 401 코드와 예외 메시지를 전달합니다.

**AjaxAccessDeniedHandler.java**{:data-align="center"}
```java
public class AjaxAccessDeniedHandler implements AccessDeniedHandler {
    @Override
    public void handle(HttpServletRequest request, HttpServletResponse response, AccessDeniedException accessDeniedException) throws IOException {
        response.sendError(HttpServletResponse.SC_FORBIDDEN, "Access is denied");
    }
}
```
인증을 받은 사용자가 인가가 허용되지 않은 자원에 접근할 경우 인가가 되지 않은 사용자에게 403 코드와 예외 메시지를 전달합니다.

**AjaxSecurityConfig.java**{:data-align="center"}
```java
@Bean
public SecurityFilterChain FilterChain(HttpSecurity http) throws Exception {
    http
        .antMatcher("/api/**")
        .authorizeRequests()
        .antMatchers("/api/messages").hasRole("MANAGER")         // 4
        .anyRequest().authenticated()
        .and()
        .addFilterBefore(ajaxLoginProcessingFilter(), UsernamePasswordAuthenticationFilter.class);

    http
        .exceptionHandling()                                                // 1
        .authenticationEntryPoint(new AjaxLoginAuthenticationEntryPoint())  // 2
        .accessDeniedHandler(ajaxAccessDeniedHandler());                    // 3
}

@Bean
public AccessDeniedHandler ajaxAccessDeniedHandler() {                      // 3
    return new AjaxAccessDeniedHandler();
}
```
1. 인가 예외 처리에 필요한 설정을 하기 위해 필요한 API입니다.
2. 익명 사용자의 인증 예외 처리 API 설정입니다.
3. 인증 사용자의 인가 예외 처리 API 설정입니다.
4. ajax 인가 테스트를 위해 경로에 MANAGER 권한 설정을 추가했습니다.

**ajax.http**
```
### Send POST request with body as parameters
POST http://localhost:8080/api/login
Content-Type: application/json
X-Requested-With: XMLHttpRequest

{
  "username": "user",
  "password": "1234"
}

### Send POST request with body as parameters
POST http://localhost:8080/api/login
Content-Type: application/json
X-Requested-With: XMLHttpRequest

{
"username": "manager",
"password": "1234"
}

### Send GET request with body as parameters
GET http://localhost:8080/api/messages
Content-Type: application/json
X-Requested-With: XMLHttpRequest
```
인가 처리 확인을 위해 작성한 코드입니다.

**MessageController.java**{:data-align="center"}
```java
@GetMapping("/api/messages")
@ResponseBody
public String apiMessage() {
    return "messages ok";
}
```
ajax로 인증을 성공한 사용자를 위한 controller 설정입니다.

#### 실제 실행 화면
**인증을 하지 않은 사용자가 접근할 때 예외 전달**{:data-align="center"}
![정상적으로 예외가 전달되 클라이언트에 출력됨](:/inflearn_spring_security_learn/3s/14/AjaxLoginAuthenticationEntryPoint_request.jpg){:data-align="center"}
인증을 하지 않은 익명 사용자로 /api/messages 경로에 접근을 했을 때 설정한 AjaxLoginAuthenticationEntryPoint가 실행되 예외 코드와 메시지를 전달하고, 정상적으로 클라이언트에 출력되는 것을 확인하실 수 있습니다.

**인증을 한 사용자가 인가 권한이 없을 때 예외 전달**{:data-align="center"}
![정상적으로 예외가 전달되 클라이언트에 출력됨](:/inflearn_spring_security_learn/3s/14/AjaxAccessDeniedHandler_request.jpg){:data-align="center"}
인증을 받은 user 권한을 가진 사용자가 /api/messages에 접근을 했을 때 설정한 권한은 manager 권한이기 때문에 인가 오류가 발생해 설정한 AjaxAccessDeniedHandler가 실행되 예외 코드와 메시지를 전달하고, 정상적으로 클라이언트에 출력 되는것을 확인하실 수 있습니다.

**인증된 사용자가 인가 권한이 있을 경우 성공 메세지 전달**{:data-align="center"}
![정상적으로 성공 메세지가 전달됨](:/inflearn_spring_security_learn/3s/14/success_account2.jpg){:data-align="center"}
인증을 받은 manager 권한을 가진 사용자가 /api/messages에 접근을 하면 인증, 인가 확인을 모두 통과하고, 정상적으로 성공 메시지가 클라이언트에 출력됩니다.

### 출처
1. [학습중인 강의](https://www.inflearn.com/course/%EC%BD%94%EC%96%B4-%EC%8A%A4%ED%94%84%EB%A7%81-%EC%8B%9C%ED%81%90%EB%A6%AC%ED%8B%B0)
2. [Spring Framework - HttpStatus](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/http/HttpStatus.html)
3. [Spring Framework - MediaType](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/http/MediaType.html)