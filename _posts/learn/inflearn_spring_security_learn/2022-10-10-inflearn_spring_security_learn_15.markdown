---
# multilingual page pair id, this must pair with translations of this page. (This name must be unique)
lng_pair: springlearn
title: 인프런 스프링 시큐리티 강의 학습-15

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
date: 2022-10-10 10:00:00 +0900

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

[Ajax Custom DSLs 구현하기](#ajax-custom-dsls-구현하기), [Ajax 로그인 구현 & CSRF 설정](#ajax-custom-dsls-구현하기-ajax-로그인-구현--csrf-설정를-정리한-포스트입니다)를 정리한 포스트입니다.
--------------------------------------

출처는 인프런의 스프링 시큐리티 - [Spring Boot 기반으로 개발하는 Spring Security](https://www.inflearn.com/course/%EC%BD%94%EC%96%B4-%EC%8A%A4%ED%94%84%EB%A7%81-%EC%8B%9C%ED%81%90%EB%A6%AC%ED%8B%B0)강의를 바탕으로 이 포스트를 작성하고 있습니다.<br>
강의의 세션 4의 6,7번 강의내용에 대한 정리입니다.<br><br>

<!-- outline-end -->

### Ajax Custom DSLs 구현하기
Custom DSLs를 구현하기 위해서 1개의 클래스와 1개의 메서드를 사용합니다.
1. AbstractHttpConfigurer은 스프링 시큐리티 초기화 설정 클래스로써 필터, 핸들러, 메서드, 속성 등을 한 곳에 정의해 처리할 수 있도록 합니다.
2. HttpSecurity의 apply(C configurer) 메서드를 사용해 구현한 구현체를 설정해 DSLs를 작동시킵니다.

#### 실제 코드
**AjaxLoginConfigurer.java**{:data-align="center"}
```java
public final class AjaxLoginConfigurer<H extends HttpSecurityBuilder<H>> extends AbstractAuthenticationFilterConfigurer<H, AjaxLoginConfigurer<H>, AjaxLoginProcessingFilter> {
    private AuthenticationSuccessHandler successHandler;
    private AuthenticationFailureHandler failureHandler;
    private AuthenticationManager authenticationManager;

    public AjaxLoginConfigurer() {
        super(new AjaxLoginProcessingFilter(), null);
    }

    @Override
    public void init(H http) throws Exception {
        super.init(http);
    }

    @Override
    public void configure(H http) {
        if (authenticationManager == null) {
            authenticationManager = http.getSharedObject(AuthenticationManager.class);
        }

        getAuthenticationFilter().setAuthenticationManager(authenticationManager);  // 1
        getAuthenticationFilter().setAuthenticationSuccessHandler(successHandler);  // 1
        getAuthenticationFilter().setAuthenticationFailureHandler(failureHandler);  // 1

        SessionAuthenticationStrategy sessionAuthenticationStrategy = http.getSharedObject(SessionAuthenticationStrategy.class);

        if (sessionAuthenticationStrategy != null) {
            getAuthenticationFilter().setSessionAuthenticationStrategy(sessionAuthenticationStrategy);
        }
        RememberMeServices rememberMeServices = http.getSharedObject(RememberMeServices.class);

        if (rememberMeServices != null) {
            getAuthenticationFilter().setRememberMeServices(rememberMeServices);
        }

        http.setSharedObject(AjaxLoginProcessingFilter.class, getAuthenticationFilter());
        http.addFilterBefore(getAuthenticationFilter(), UsernamePasswordAuthenticationFilter.class);
    }
    public AjaxLoginConfigurer<H> successHandlerAjax(AuthenticationSuccessHandler successHandler) {
        this.successHandler = successHandler;
        return this;
    }
    public AjaxLoginConfigurer<H> failureHandlerAjax(AuthenticationFailureHandler authenticationFailureHandler) {
        this.failureHandler = authenticationFailureHandler;
        return this;
    }
    public AjaxLoginConfigurer<H> setAuthenticationManager(AuthenticationManager authenticationManager) {
        this.authenticationManager = authenticationManager;
        return this;
    }

    @Override
    protected RequestMatcher createLoginProcessingUrlMatcher(String loginProcessingUrl) {
        return new AntPathRequestMatcher(loginProcessingUrl, "POST");
    }
}
```
상속받은 AbstractAuthenticationFilterConfigurer 클래스의 코드를 가져와 적절하게 변경하였습니다.   
1. 원래는 AjaxSecurityConfig에서 ajaxLoginProcessingFilter 메서드로 설정했지만 getAuthenticationFilter를 통해 메니저와 핸들러를 Filter에 저장하고 있습니다.

**AjaxSecurityConfig.java**{:data-align="center"}
```java
@Bean
public SecurityFilterChain FilterChain(HttpSecurity http) throws Exception {
    http
            .antMatcher("/api/**")
            .authorizeRequests()
            .antMatchers("/api/message").hasRole("MANAGER")
            .anyRequest().authenticated();

    http
            .exceptionHandling()
            .authenticationEntryPoint(new AjaxLoginAuthenticationEntryPoint())
            .accessDeniedHandler(ajaxAccessDeniedHandler());

    http.csrf().disable();

    customConfigurerAjax(http);     // 1

    return http.build();
}

public void customConfigurerAjax(HttpSecurity http) throws Exception {
    http                            // 2
            .apply(new AjaxLoginConfigurer<>())
            .successHandlerAjax(ajaxAuthenticationSuccessHandler())
            .failureHandlerAjax(ajaxAuthenticationFailureHandler())
            .setAuthenticationManager(authenticationManager(authenticationConfiguration))
            .loginProcessingUrl("/api/login");
}

//    @Bean
//    public AjaxLoginProcessingFilter ajaxLoginProcessingFilter() throws Exception {
//        AjaxLoginProcessingFilter ajaxLoginProcessingFilter = new AjaxLoginProcessingFilter();
//        ajaxLoginProcessingFilter.setAuthenticationManager(authenticationManager(authenticationConfiguration));
//        ajaxLoginProcessingFilter.setAuthenticationSuccessHandler(ajaxAuthenticationSuccessHandler());
//        ajaxLoginProcessingFilter.setAuthenticationFailureHandler(ajaxAuthenticationFailureHandler());
//        return ajaxLoginProcessingFilter;
//    }
```
1. Custom DSLs를 사용하기 위한 메서드를 호출합니다.
2. AjaxLoginConfigurer를 불러오고, success, failure, Manager를 설정합니다.

#### 실제 실행 화면
**configurers에 저장된 AjaxLogin**{:data-align="center"}
![정상적으로 리스트에 들어가 있는 모습](:/inflearn_spring_security_learn/3s/15/configurer.jpg){:data-align="center"}
위에서 제작한 AjaxLoginConfigurer가 정상적으로 configurers에 등록이 되었습니다.

**configure의 실행**{:data-align="center"}
![정상적으로 만든 메서드가 실행됨](:/inflearn_spring_security_learn/3s/15/configure.jpg){:data-align="center"}
이 사진은 AjaxLoginConfigurer의 configure 메서드가 정상적으로 실행되 Filter 저장까지 진행된 것을 확인할 수 있습니다.
   <br>
이후에 ajax.http를 실행한 결과 다 정상적으로 실행이 되는 것을 확인할 수 있었습니다.   <br>
해당 코드는 [깃허브](https://github.com/Othkkartho/SpringSecurityLearn/tree/ch4.6%2C7)에 올려놨습니다.

### Ajax 로그인 구현 & CSRF 설정
위 구현을 위해서 **전송 방식이 Ajax인지 여부를 확인하기 위한 헤더**를 설정하고, **CSRF 헤더**를 설정해야 합니다.

**login.html**{:data-align="center"}
```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">

<meta name="_csrf" th:content="${_csrf?.token}" th:if="${_csrf} ne null">               <!-- 1 -->
<meta name="_csrf_header" th:content="${_csrf?.headerName}" th:if="${_csrf} ne null">   <!-- 1 -->

<head th:replace="layout/header::userHead"></head>
<script>
  function formLogin(e) {

    let username = $("input[name='username']").val().trim();
    let password = $("input[name='password']").val().trim();
    let data = {"username" : username, "password" : password};

    let csrfHeader = $('meta[name="_csrf_header"]').attr('content') // 2
    let csrfToken = $('meta[name="_csrf"]').attr('content')         // 2

    $.ajax({
      type: "post",
      url: "/api/login",
      data: JSON.stringify(data),
      dataType: "json",
      beforeSend : function(xhr){
        xhr.setRequestHeader(csrfHeader, csrfToken);
        xhr.setRequestHeader("X-Requested-With", "XMLHttpRequest");
        xhr.setRequestHeader("Content-type","application/json");
      },
      success: function (data) {
        console.log(data);
        window.location = '/';

      },
      error : function(xhr, status, error) {
        console.log(error);
        window.location = '/login?error=true&exception=' + xhr.responseText;
      }
    });
  }
</script>
<body>
<div th:replace="layout/top::header"></div>
<div class="container text-center">
  <div class="login-form d-flex justify-content-center">
    <div class="col-sm-5" style="margin-top: 30px;">
      <div class="panel">
        <p>아이디와 비밀번호를 입력해주세요</p>
      </div>
      <div th:if="${param.error}" class="form-group">
        <span th:text="${exception}" class="alert alert-danger">잘못된 아이디나 암호입니다</span>
      </div>
      <form th:action="@{/login_proc}" class="form-signin" method="post">
        <input type="hidden" th:value="secret" name="secret_key" />
        <div class="form-group">
          <input type="text" class="form-control" name="username" placeholder="아이디" required="required" autofocus="autofocus">
        </div>
        <div class="form-group">
          <input type="password" class="form-control" name="password" placeholder="비밀번호" required="required">
        </div>
        <button type="button" onclick="formLogin()" id="formbtn" class="btn btn-lg btn-primary btn-block">로그인</button>
        <!--<button type="submit" class="btn btn-lg btn-primary btn-block">로그인</button>-->
      </form>
    </div>
  </div>
</div>
</body>
</html>
```
1. 전송 방식이 Ajax인지 여부를 확인하기 위한 헤더를 설정합니다.
2. CSRF 헤더를 설정합니다.

**login.html**{:data-align="center"}
```html
<!DOCTYPE html>
<html lang="ko" xmlns:th="http://www.thymeleaf.org">
<head th:replace="layout/header::userHead"></head>
<html xmlns:th="http://www.thymeleaf.org">

<meta name="_csrf" th:content="${_csrf?.token}" th:if="${_csrf} ne null">
<meta name="_csrf_header" th:content="${_csrf?.headerName}" th:if="${_csrf} ne null">

<head th:replace="layout/header::userHead"></head>
<script>
    function messages() {

        let csrfHeader = $('meta[name="_csrf_header"]').attr('content')
        let csrfToken = $('meta[name="_csrf"]').attr('content')

        $.ajax({
            type: "post",
            url: "/api/messages",
            //dataType: "json",
            beforeSend : function(xhr){
                xhr.setRequestHeader(csrfHeader, csrfToken);
                xhr.setRequestHeader("X-Requested-With", "XMLHttpRequest");
                xhr.setRequestHeader("Content-type","application/json");
            },
            success: function (data) {
                console.log(data);
                window.location = '/messages';
            },
            error : function(xhr, status, error) {
                console.log(error);
                if(xhr.responseJSON.status === '401'){
                    window.location = '/api/login?error=true&exception=' + xhr.responseJSON.message;
                }else if(xhr.responseJSON.status === '403'){
                    window.location = '/api/denied?exception=' + xhr.responseJSON.message;
                }
            }
        });
    }
</script>

<a href="#" onclick="messages()" style="margin:5px;" class="nav-link text-primary">메시지</a>
```

**LoginController.java**{:data-align="center"}
```java
@Controller
public class LoginController {
    @GetMapping(value = {"/login", "/api/login"})   // 1
    public String login(@RequestParam(value = "error", required = false) String error,
                        @RequestParam(value = "exception", required = false) String exception,
                        Model model) {
        // 코드는 이전과 동일
    }

    @GetMapping(value={"/denied","/api/denied"})    // 1
    public String accessDenied(@RequestParam(value = "exception", required = false) String exception, Principal principal, Model model) throws Exception {
        // 코드는 이전과 동일
    }
}
```
1. 각 매핑 값들에 api 값들 또한 넣어 정상적으로 url이 동작하도록 설정했습니다.


**AjaxLoginAuthenticationEntryPoint.java**{:data-align="center"}
```java
public class AjaxLoginAuthenticationEntryPoint implements AuthenticationEntryPoint {
    @Override
    public void commence(HttpServletRequest request, HttpServletResponse response, AuthenticationException authException) throws IOException {
        response.sendError(HttpServletResponse.SC_UNAUTHORIZED, "UnAuthorized");
    }
}
```

### 출처
1. [학습중인 강의](https://www.inflearn.com/course/%EC%BD%94%EC%96%B4-%EC%8A%A4%ED%94%84%EB%A7%81-%EC%8B%9C%ED%81%90%EB%A6%AC%ED%8B%B0)
2. [Spring Security 공식 문서](https://docs.spring.io/spring-security/site/docs/4.2.3.RELEASE/reference/htmlsingle/#jc-custom-dsls)