---
# multilingual page pair id, this must pair with translations of this page. (This name must be unique)
lng_pair: springlearn
title: 인프런 스프링 시큐리티 강의 학습-11

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
date: 2022-10-03 10:00:00 +0900

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

[로그인 페이지 생성](#커스텀-로그인-페이지-생성하기), [로그아웃 인증](#로그아웃-및-인증에-따른-화면-보안-처리), [인증 부가 기능](#인증-부가-기능---webauthenticationdetails-authenticationdetailssource)을 정리한 포스트입니다.
--------------------------------------

출처는 인프런의 스프링 시큐리티 - [Spring Boot 기반으로 개발하는 Spring Security](https://www.inflearn.com/course/%EC%BD%94%EC%96%B4-%EC%8A%A4%ED%94%84%EB%A7%81-%EC%8B%9C%ED%81%90%EB%A6%AC%ED%8B%B0)강의를 바탕으로 이 포스트를 작성하고 있습니다.<br>
강의의 세션 3의 6,7,8번 강의내용에 대한 정리입니다.<br><br>

<!-- outline-end -->

### 커스텀 로그인 페이지 생성하기
#### 프론트 앤드 실제 코드
**top.html**{:data-align="center"}
```html
<!DOCTYPE html>
<html lang="ko" xmlns:th="http://www.thymeleaf.org">
<div th:fragment="header">
    <nav class="navbar navbar-dark sticky-top bg-dark ">
        <div class="container">
            <a class="text-light" href="#"><h4>Core Spring Security</h4></a>
            <ul class="nav justify-content-end">
                <li class="nav-item"><a class="nav-link text-light" th:href="@{/login}">로그인</a></li>     <!--1--> 
                <li class="nav-item"><a class="nav-link text-light" th:href="@{/users}">회원가입</a></li>
                <li class="nav-item" ><a class="nav-link text-light" href="/">HOME</a></li>
            </ul>
        </div>
    </nav>
</div>
</html>
```
1. 회원가입과 HOME만 있던 html에 로그인을 위한 경로를 추가합니다.

**login.html**{:data-align="center"}
```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head th:replace="layout/header::userHead"></head>
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
      <form th:action="@{/login_proc}" class="form-signin" method="post">                       <!--1-->
        <input type="hidden" th:value="secret" name="secret_key" />
        <div class="form-group">
          <input type="text" class="form-control" name="username" placeholder="아이디" required="required" autofocus="autofocus">
        </div>
        <div class="form-group">
          <input type="password" class="form-control" name="password" placeholder="비밀번호" required="required">
        </div>
        <button type="submit" class="btn btn-lg btn-primary btn-block">로그인</button>
      </form>
    </div>
  </div>
</div>
</body>
</html>
```
로그인 html의 출처는 강의의 github입니다.
1. 로그인을 하기 위한 Action 경로입니다. loginProcessingUrl과 같은 값이어야 합니다.

#### Spring 실제 코드
**SecurityConfig.java**{:data-align="center"}
```java
@Bean
public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
    AuthenticationManagerBuilder authenticationManagerBuilder = http.getSharedObject(AuthenticationManagerBuilder.class);
    authenticationManagerBuilder.authenticationProvider(authenticationProvider());

    http
            .authorizeRequests()
            .antMatchers("/", "/users", "/user/login/**", "/error").permitAll() // 1
            .antMatchers("/mypage").hasRole("USER")
            .antMatchers("/messages").hasRole("MANAGER")
            .antMatchers("/config").hasRole("ADMIN")
            .anyRequest().authenticated()
            .and()
            .formLogin()
            .loginPage("/login")                                                // 2
            .loginProcessingUrl("/login_proc")                                  // 3
            .defaultSuccessUrl("/")                                             // 4
            .permitAll();                                                       // 5

    return http.build();
}
```
1. 로그인 중 `{"timestamp":"2022-01-01T20:11:13.890+00:00","status":999,"error":"None"}` 오류를 해결하기 위해 /error 경로를 예외처리 하였습니다.
    - 확인 결과 인터넷 웹 브라우저의 주소창에 표시되는 아이콘인 favicon의 에러 코드가 문제를 일으키는 것으로 예상됩니다. 출처는 [출처 2번](#출처)입니다.
2. 로그인을 진행할 경로를 입력하여 로그인 페이지에 사용자가 접근할 수 있도록 합니다.
3. 로그인 Form의 Action 경로입니다. 실제 로그인이 진행되는 메소드입니다.
4. 로그인에 성공하게 되면 이동할 경로를 지정합니다.
5. 위 설정으로 인해 로그인 페이지가 생성된 페이지로 이동하기 때문에 permitAll로 권한을 통과 처리를 합니다.

**LoginController.java**{:data-align="center"}
```java
@Controller
public class LoginController {
    @GetMapping("/login")
    public String login() {
        return "/user/login/login";
    }
}
```
로그인 페이지로 이동하게 하기 위해 경로와 페이지를 매핑 처리합니다.

### 로그아웃 및 인증에 따른 화면 보안 처리
#### 로그아웃 인증의 설명
- **로그아웃 방법**
    - <form> 태그를 사용해서 POST로 요청하면 LogoutFilter가 작동하고, 실제 로그아웃이 작동합니다.
    - <a> 태그를 사용해서 GET으로 요청하면 LogoutFilter가 로그아웃을 진행하지 않고, SecurityContextLogoutHandler 활용해 직접 로그아웃 메소드를 만들어야 합니다.
- **인증 여부에 따른 로그인/로그아웃 표현**
    - `sec:authorize="isAnonymous()` 는 로그인을 하지 않았을 경우 화면에 메뉴를 띄웁니다.
    - `sec:authorize=isAuthenticated()` 는 로그인을 했을 경우 화면에 메뉴를 띄웁니다.

#### 프론트 앤드 실제 코드
**top.html**{:data-align="center"}
```html
<!DOCTYPE html>
<html lang="ko" xmlns:th="http://www.thymeleaf.org"
                xmlns:sec="http://www.thymeleaf.org/thymeleaf-extras-springsecurity5">                                                          <!--1-->
<div th:fragment="header">
    <nav class="navbar navbar-dark sticky-top bg-dark ">
        <div class="container">
            <a class="text-light" href="#"><h4>Core Spring Security</h4></a>
            <ul class="nav justify-content-end">
                <li class="nav-item" sec:authorize="isAnonymous()"><a class="nav-link text-light" th:href="@{/login}">로그인</a></li>           <!--2-->
                <li class="nav-item" sec:authorize="isAnonymous()"><a class="nav-link text-light" th:href="@{/users}">회원가입</a></li>         <!--2-->
                <li class="nav-item" sec:authorize="isAuthenticated()"><a class="nav-link text-light" th:href="@{/logout}">로그아웃</a></li>    <!--3-->
                <li class="nav-item" ><a class="nav-link text-light" href="/">HOME</a></li>
            </ul>
        </div>
    </nav>
</div>
</html>
```
따로 사진 첨부는 하지 않았지만 로그인/로그아웃시 정상적으로 메뉴를 띄우는 것을 확인했습니다.
1. 로그인/로그아웃 관련 스프링 시큐리티와 연결된 타임리프를 사용하기 위해 설정해야하는 namespace입니다.
2. 로그인을 하지 않았을 경우 로그인과 회원가입을 화면에 띄웁니다.
3. 로그인을 했을 경우 로그아웃을 화면에 띄웁니다.

#### Spring 실제 코드
**LoginController.java**{:data-align="center"}
```java
@GetMapping("/logout")
public String logout(HttpServletRequest request, HttpServletResponse response) {
    Authentication authentication = SecurityContextHolder.getContext().getAuthentication(); // 1

    if (authentication != null) {
        new SecurityContextLogoutHandler().logout(request, response, authentication);       // 2
    }

    return "redirect:/";
}
```
따로 사진 첨부는 안했지만 로그아웃이 정상적으로 작동하는 것을 확인했습니다.
1. 로그아웃을 한다는 의미는 이미 로그인을 했다는 것을 의미하기에 SecurityContext안에 인증 객체가 담겨 있다는 것을 의미합니다. 그래서 인증객체를 가져와 authentication에 저장합니다.
2. 로그인을 진행하는 핸들러를 불러와 실제 로그아웃 처리를 진행합니다.

### 인증 부가 기능 - WebAuthenticationDetails, AuthenticationDetailsSource
#### 클레스 설명 및 로그인 순서도
- **WebAuthenticationDetails**
    - 1번 과정(인증 과정) 중 전달된 데이터를 저장해 인증 과정 속에서 활용하거나 인증 이후에 계속적으로 정보들을 참조해 어떤 처리를 할 때 사용할 수 있게 하는 클레스입니다,
- **AuthenticationDetailsSource**
    - WebAuthenticationDetails 객체를 생성하는 클레스입니다.

다음은 로그인 처리 순서입니다.
0. 사용자가 Username과 passoword 정보를 입력해 로그인을 요청하면 AuthenticationFilter가 요청을 처리합니다. 하지만 사용자가 이 정보 이외의 추가적인 파라미터를 정의해 데이터를 전달 한다고 가정하겠습니다.
1. 인증 필터가 인증을 처리하면 가장 먼저 인증 객체를 생성합니다. 인증 객체에는 ID와 PW가 저장이 됩니다. 그리고 인증 객체는 내부적으로 details라는 object 타입의 속성을 가지고 있습니다.
    - details라는 속성을 가지고 있고, details 안에는 AuthenticationDetailsSource가 생성한 WebAuthenticationDetails라는 클레스 객체가 저장되어 있습니다.
    - details는 object 타입이기 때문에 어떤 타입의 정보라도 담을 수 있습니다.
2. 사용자가 추가적으로 전달한 파라미터들을 얻어 클레스에 저장하는 역할을 합니다.
    - remoteAddress와 SessionId는 스프링 시큐리티가 처리를 하고, 추가 파라미터들은 Details에게 전달되는 request 객체를 사용해 `request.getParameter("")` 형식으로 추가적으로 전달되는 파라미터를 저장합니다.

#### 실제 코드
**FormWebAuthenticationDetails.java**{:data-align="center"}
```java
public class FormWebAuthenticationDetails extends WebAuthenticationDetails {
    private String secretKey;

    public FormWebAuthenticationDetails(HttpServletRequest request) {
        super(request);
        secretKey = request.getParameter("secret_key");                     // 1
    }

    public String getSecretKey() {
        return secretKey;                                                   // 1
    }
}
```
사용자가 전달하는 ID와 PW외에 추가적인 파라미터들을 저장하는 클레스입니다.
1. 전달받은 데이터 중에서 추가적인 정보인 secret_key를 저장하는 클레스입니다.

**FormAuthenticationDetailsSource.java**{:data-align="center"}
```java
@Component                  // 1
public class FormAuthenticationDetailsSource implements AuthenticationDetailsSource<HttpServletRequest, WebAuthenticationDetails> {
    @Override
    public WebAuthenticationDetails buildDetails(HttpServletRequest context) {
        return new FormWebAuthenticationDetails(context);
    }
}
```
FormWebAuthenticationDetails 클레스를 생성하는 클레스입니다.
1. 설정 클레스에서 추가를 해야하기 때문에 빈으로 등록하였습니다.

**SecurityConfig.java**{:data-align="center"}
```java
private AuthenticationDetailsSource authenticationDetailsSource;

@Autowired
private void setSecurityConfig(AuthenticationDetailsSource authenticationDetailsSource) {
    this.authenticationDetailsSource = authenticationDetailsSource;
}

@Bean
public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
    AuthenticationManagerBuilder authenticationManagerBuilder = http.getSharedObject(AuthenticationManagerBuilder.class);
    authenticationManagerBuilder.authenticationProvider(authenticationProvider());

    http
            // 전 코드와 동일
            .and()
            .formLogin()
            .loginPage("/login")
            .loginProcessingUrl("/login_proc")
            .authenticationDetailsSource(authenticationDetailsSource)       // 1
            .defaultSuccessUrl("/")
            .permitAll();

    return http.build();
}
```
1. 만든 2개의 클레스를 사용하도록 설정하였습니다.

**login.html**{:data-align="center"}
```html
<form th:action="@{/login_proc}" class="form-signin" method="post">
    <input th:type="hidden" th:value="secret" name="secret_key" >        <!--1-->
    <div class="form-group">
        <input type="text" class="form-control" name="username" placeholder="아이디" required="required" autofocus="autofocus">
    </div>
    <div class="form-group">
        <input type="password" class="form-control" name="password" placeholder="비밀번호" required="required">
    </div>
    <button type="submit" class="btn btn-lg btn-primary btn-block">로그인</button>
    </form>
```
1. 추가적인 파라미터인 secret 키를 로그인시 전달해 백앤드에서 전달받을 수 있도록 해줍니다.

**FormAuthenticationProvider.java**{:data-align="center"}
```java
@Override
@Transactional
public Authentication authenticate(Authentication authentication) throws AuthenticationException {
    String username = authentication.getName();
    String password = (String) authentication.getCredentials();

    AccountContext accountContext = (AccountContext)userDetailsService.loadUserByUsername(username);

    if (!passwordEncoder.matches(password, accountContext.getAccount().getPassword())) {
        throw new BadCredentialsException("BadCredentialsException");
    }

    FormWebAuthenticationDetails formWebAuthenticationDetails = (FormWebAuthenticationDetails) authentication.getDetails();     // 1
    String secretKey = formWebAuthenticationDetails.getSecretKey();                                                             // 2
    if (!"secret".equals(secretKey)) {                                                                                          // 3
        throw new InsufficientAuthenticationException("InsufficientAuthenticationException");                                   // 4
    }

    return new UsernamePasswordAuthenticationToken(accountContext.getAccount(), null, accountContext.getAuthorities());
}
```
1. ID, PW외 파라미터를 저장하는 formWebAuthenticationDetails 형태로 인증 객체에게 Details 객체를 받습니다
2. 저장된 객체 중 SecretKey를 꺼내 저장합니다.
3. 
4. 만약 secret가 다르다면 오류를 던집니다.

##### 오류발생
secretKey가 같다면 정상적으로 오류를 던지지 않고, 이동하지만 secretKey가 다르면 FormAuthenticationProvider.java의 주석 4번의 오류를 정상적으로 던지지만 오류 처리가 되지 않고, 로그인이 정상적으로 되는 오류가 확인되었습니다.<br><br>
![회원가입 화면](:/inflearn_spring_security_learn/3s/11/customauthenticationprovider.jpg){:data-align="center"}
FormAuthenticationProvider 클레스 주석 1번에 BreakPoint를 걸고 진행사항을 확인한 것입니다. 정상적으로 오류가 던져지는 것을 확인할 수 있습니다.
![회원가입 화면](:/inflearn_spring_security_learn/3s/11/providermanager.jpg){:data-align="center"}
구현 클레스가 아닌 스프링 시큐리티 authentication 패키지의 providermanager 클레스입니다.<br>
보시면 원래 result에 null이 저장이 되야하지만 result에 값이 저장되고, parentException이 null이고, lastException만 값이 저장되있는 것을 확인할 수 있습니다.<br><br>
그래서 throw lastException로 넘어가지 않고, if (result != null) 조건에 걸려 이후 절차를 수행하는 것을 확인했습니다. 그 후 정상적으로 로그인이 됩니다. 이 오류에 관해 해결하지 못해, 일단 세션 3의 남은 강의들을 진행하면서 오류를 지속적으로 고쳐보도록 하겠습니다.

### 참고
#### 오류 해결 시 작성 예정

### 출처
1. [학습중인 강의](https://www.inflearn.com/course/%EC%BD%94%EC%96%B4-%EC%8A%A4%ED%94%84%EB%A7%81-%EC%8B%9C%ED%81%90%EB%A6%AC%ED%8B%B0)
2. [stackoverflow - Spring security redirects to page with status code 999](https://stackoverflow.com/questions/61029340/spring-security-redirects-to-page-with-status-code-999)
