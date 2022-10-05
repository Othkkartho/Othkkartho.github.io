---
# multilingual page pair id, this must pair with translations of this page. (This name must be unique)
lng_pair: springlearn
title: 인프런 스프링 시큐리티 강의 학습-13

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
date: 2022-10-05 22:00:00 +0900

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

Ajax 인증의 [흐름 및 개요](#ajax-인증의-흐름-및-개요), [인증 필터](#인증-필터---ajaxauthenticationfilter), [인증 처리자](#인증-처리자---ajaxauthenticationprovider)를 정리한 포스트입니다.
--------------------------------------

출처는 인프런의 스프링 시큐리티 - [Spring Boot 기반으로 개발하는 Spring Security](https://www.inflearn.com/course/%EC%BD%94%EC%96%B4-%EC%8A%A4%ED%94%84%EB%A7%81-%EC%8B%9C%ED%81%90%EB%A6%AC%ED%8B%B0)강의를 바탕으로 이 포스트를 작성하고 있습니다.<br>
강의의 세션 4의 1,2,3번 강의내용에 대한 정리입니다.<br><br>

<!-- outline-end -->

### Ajax 인증의 흐름 및 개요
#### Ajax 인증의 흐름
1. 사용자가 POST /ajaxLogin 으로 요청 시 AjaxAuthenticationFilter가 받아 각 클래스로 인증처리를 맡깁니다.
2. Filter는 AjaxAuthenticationToke 안에 사용자가 인증 처리를 위해 보낸 정보를 담고 인증 처리를 하게 됩니다.
3. 처리를 위해 Filter는 AuthenticationManager에게 Token을 전달해 주고, 메니저가 AjaxAuthenticationProvider에게 인증처리를 위임합니다.
4. Provider는 실질적인 인증을 진행합니다.
5. 만약 인증이 실패하거나 성공하면 Filter는 각 Handler를 호출해 각 결과 이후의 처리를 하게 됩니다.
    - 인증 성공 시 AjaxAuthenticationSuccessHandler가 호출됩니다.
    - 인증 실패 시 AjaxAuthenticationFailureHandler가 호출됩니다.
6. 인증 성공시 서버는 사용자가 자원에 접근할 때 인가 처리를 하게 됩니다.
7. FilterSecurityInterceptor가 관련 인가 처리를 담당합니다.
8. 자격 검사 중 인증 또는 인가 예외가 발생했을 때 ExceptionTranslationFilter에게 예외를 전달합니다.
    - 인증이 실패할 경우 AjaxUrlAuthenticationEntryPoint 로 처리합니다.
    - 자원 접근이 거부되었을 경우 AjaxAccessDeniedHandler로 처리합니다.

### 인증 필터 - AjaxAuthenticationFilter
사용자가 Ajax로 인증 요청시 그 요청을 받아 인증 처리를 담당합니다. 
#### 개요
- *AbstractAuthenticationProcessingFilter* 라는 추상 클래스를 상속합니다.
    - 대부분의 인증 처리의 기능을 추상 클래스가 하고 있습니다.
- 필터 작동 조건은 AntPathRequestMather("/api/login") 으로 오청정보와 매칭하고, 요청 방식이 Ajax 이면 필터가 작동하도록 구성할 예정입니다.
- AjaxAuthenticationToken을 생성해 AuthenticationManager에게 전달하여 인증 처리를 맡깁니다.

#### 실제 코드
**AjaxLoginProcessingFilter.java**{:data-align="center"}
```java
public class AjaxLoginProcessingFilter extends AbstractAuthenticationProcessingFilter {
    private final ObjectMapper objectMapper = new ObjectMapper(); // 3

    public AjaxLoginProcessingFilter() {                    // 1
        super(new AntPathRequestMatcher("/api/login"));
    }

    @Override
    public Authentication attemptAuthentication(HttpServletRequest request, HttpServletResponse response) throws AuthenticationException, IOException, ServletException {
        if (!isAjax(request)) {                             // 2
            throw new IllegalStateException("Authentication is not support");
        }
        AccountDto accountDto = objectMapper.readValue(request.getReader(), AccountDto.class);  // 3

        if (!StringUtils.hasText(accountDto.getUsername()) || !StringUtils.hasText(accountDto.getPassword())) { // 4
            throw new IllegalArgumentException("username or Password is empty");
        }

        AjaxAuthenticationToken ajaxAuthenticationToken = new AjaxAuthenticationToken(accountDto.getUsername(), accountDto.getPassword());  // 5

        return getAuthenticationManager().authenticate(ajaxAuthenticationToken);    // 6
    }

    private boolean isAjax(HttpServletRequest request) {    // 2
        return "XMLHttpRequest".equals(request.getHeader("X-Requested-With"));
    }
}
```
1. 필터 작동 조건을 정의합니다.
2. ajax인지 아닌지를 확인해 아니라면 예외를 던집니다.
    - http X-Requested-With의 header 명의 값이 위 값과 동일 하다면 ajax 방식으로 생각해 true를 return 하고, 아니면 false를 return합니다.
3.  objectMapper를 통해 url 값을 읽고, 그 값을 AccountDto 클래스 타입으로 담아 받도록 합니다.
4. 가져온 아이디나 비밀번호의 값이 null이면 예외를 발생시킵니다.
    - 강의에서는 `if (StringUtils.isEmpty(accountDto.getUsername()) || StringUtils.isEmpty(accountDto.getPassword()))` 로 조건문이 작성되었지만 제가 제작하는 버전에서는 Deprecated 되었기 때문에 변경해 제작하였습니다.
    - hasText는 text 값이 있으면 true를 반환하고, 없으면 false를 반환해 !(not) 을 작성해줘야 정상작동합니다.
5. 아래 만든 토큰 생성자를 통해 토큰을 생성합니다.
6. authenticationManager에게 토큰을 전달해 인증 처리를 시킵니다.

**AjaxAuthenticationToken.java**{:data-align="center"}
```java
public class AjaxAuthenticationToken extends AbstractAuthenticationToken {
    @Serial         // 1
    private static final long serialVersionUID = SpringSecurityCoreVersion.SERIAL_VERSION_UID;

    private final Object principal;

    private Object credentials;

    public AjaxAuthenticationToken(Object principal, Object credentials) {      // 2
        super(null);
        this.principal = principal;
        this.credentials = credentials;
        setAuthenticated(false);
    }

    public AjaxAuthenticationToken(Object principal, Object credentials,
                                               Collection<? extends GrantedAuthority> authorities) {    // 2
        super(authorities);
        this.principal = principal;
        this.credentials = credentials;
        super.setAuthenticated(true);
    }

    public static UsernamePasswordAuthenticationToken unauthenticated(Object principal, Object credentials) {
        return new UsernamePasswordAuthenticationToken(principal, credentials);
    }

    public static UsernamePasswordAuthenticationToken authenticated(Object principal, Object credentials,
                                                                    Collection<? extends GrantedAuthority> authorities) {
        return new UsernamePasswordAuthenticationToken(principal, credentials, authorities);
    }

    @Override
    public Object getCredentials() {
        return this.credentials;
    }

    @Override
    public Object getPrincipal() {
        return this.principal;
    }

    @Override
    public void setAuthenticated(boolean isAuthenticated) throws IllegalArgumentException {
        Assert.isTrue(!isAuthenticated,
                "Cannot set this token to trusted - use constructor which takes a GrantedAuthority list instead");
        super.setAuthenticated(false);
    }

    @Override
    public void eraseCredentials() {
        super.eraseCredentials();
        this.credentials = null;
    }
}
```
form인증에 사용되는 usernamePasswordAuthenticationToken.java를 참조해 제작되었습니다. 따라서 모든 기능은 동일합니다.
1. 
2. 마찬가지로 위에 있는 생성자가 실제 인증을 받기 전 사용자가 입력한 로그인 정보를 담는 생성자고, 두번째 생성자가 인증 후 인증 결과를 담는 생성자입니다.

**SecurityConfig.java**{:data-align="center"}
```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {
    private AuthenticationConfiguration authenticationConfiguration;
    private FormAuthenticationDetailsSource authenticationDetailsSource;
    private AuthenticationSuccessHandler customAuthenticationSuccessHandler;
    private AuthenticationFailureHandler customAuthenticationFailureHandler;

    @Autowired
    private void setSecurityConfig(AuthenticationConfiguration authenticationConfiguration, FormAuthenticationDetailsSource authenticationDetailsSource, AuthenticationSuccessHandler customAuthenticationSuccessHandler, AuthenticationFailureHandler customAuthenticationFailureHandler) {
        this.authenticationConfiguration = authenticationConfiguration;
        this.authenticationDetailsSource = authenticationDetailsSource;
        this.customAuthenticationSuccessHandler = customAuthenticationSuccessHandler;
        this.customAuthenticationFailureHandler = customAuthenticationFailureHandler;
    }

    @Bean
    public AuthenticationManager authenticationManager(AuthenticationConfiguration authenticationConfiguration) throws Exception {  // 4
        return authenticationConfiguration.getAuthenticationManager();
    }

    @Bean
    public AjaxLoginProcessingFilter ajaxLoginProcessingFilter() throws Exception { // 3
        AjaxLoginProcessingFilter ajaxLoginProcessingFilter = new AjaxLoginProcessingFilter();
        ajaxLoginProcessingFilter.setAuthenticationManager(authenticationManager(authenticationConfiguration));
        return ajaxLoginProcessingFilter;
    }

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        // 이전 코드와 동일
                .exceptionHandling()
                .accessDeniedHandler(accessDeniedHandler())
                .and()
                .addFilterBefore(ajaxLoginProcessingFilter(), UsernamePasswordAuthenticationFilter.class);  // 1

        http.csrf().disable();  // 2

        return http.build();
    }
}
```
1. 총 4개의 addFilter API가 있습니다.
    - addFilterBefor는 실제 추가하고자 하는 필터가 기존 필터 앞에 위치할 때 사용합니다.
    - addFilter는 추가하는 필터가 가장 마지막에 위치할 때 사용합니다.
    - addFilterAfter은 추가하는 필터가 기존 필터 뒤에 위치할 때 사용합니다.
    - addFilterAt은 기존 필터의 위치를 추가하는 필터가 대체하고자 할 때 사용합니다.
2. 기본은 csrf가 작동을 하도록 되어 있는데 http로 전달하는 post에 csrf가 없기 때문에 임시로 비활성화 시켜놨습니다. 
3. ajaxLoginProcessingFilter를 사용하기 위해 Bean 설정을 합니다.
4. Override authenticationManagerBean을 대체하는 클래스 설정입니다.

**ajax.http**{:data-align="center"}
```
POST http://localhost:8080/api/login
Content-Type: application/json
X-Requested-With: XMLHttpRequest

{
  "username": "user",
  "password": "1234"
}
```

### 인증 처리자 - AjaxAuthenticationProvider


### 참고
#### Ajax 인증과 Form 인증의 차이점
Form 인증 방식은 동기적인 방식의 인증이고,
Ajax 인증은 비동기적인 방식이 차이점입니다. (자세한 내용 찾아서 넣어놓고, 없으면 위로 올려놓을것.) 

### 출처
1. [학습중인 강의](https://www.inflearn.com/course/%EC%BD%94%EC%96%B4-%EC%8A%A4%ED%94%84%EB%A7%81-%EC%8B%9C%ED%81%90%EB%A6%AC%ED%8B%B0)
