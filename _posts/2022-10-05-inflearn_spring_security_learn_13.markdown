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
1. 주석이 달린 필드 또는 메소드가 Java Object Serialization Specification에 의해 정의된 직렬화 메커니즘의 일부임을 나타냅니다. 이 어노테이션은 직렬화 관련 선언의 컴파일 시간 검사를 허용하기 위한 것입니다. 직렬화 가능 클래스는 컴파일러가 잘못 선언된 직렬화 관련 필드 및 메서드나 감지가 어려울 수 있는 잘못된 선언을 포착하는데 도움이 되도록 이 어노테이션 사용이 권장됩니다.
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
4. Override authenticationManagerBean을 대체하는 클래스 설정입니다. 아래에 강의 코드를 남기겠습니다.

**강의 코드**{:data-align="center"}
```java
@Override
public AuthenticationManager authenticationManagerBean() throws Exception {  // 4
    return super.authenticationMangerBean();
}

@Bean
public AjaxLoginProcessingFilter ajaxLoginProcessingFilter() throws Exception { // 3
    AjaxLoginProcessingFilter ajaxLoginProcessingFilter = new AjaxLoginProcessingFilter();
    ajaxLoginProcessingFilter.setAuthenticationManager(authenticationManagerBean());
    return ajaxLoginProcessingFilter;
}
```

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

#### 실제 실행 화면
**Header에 값을 정상적으로 가져옴**{:data-align="center"}
![Header에 값을 정상적으로 가져옴](:/inflearn_spring_security_learn/3s/13/isAjax_request.JPG){:data-align="center"}
Header에 X-Requested-With 이름으로 있는 값을 정상적으로 꺼내오는 보여줍니다.

**유저의 로그인 입력 정보를 저장**{:data-align="center"}
![유저의 로그인 입력 정보를 저장](:/inflearn_spring_security_learn/3s/13/accountDto.JPG){:data-align="center"}
유저가 로그인시 입력했던 정보를 가져와 저장한 결과입니다. 정상적으로 저장되어 있는것을 확인할 수 있습니다.

**값을 보내기 위해 Token을 생성함**{:data-align="center"}
![값을 보내기 위해 Token을 생성함](:/inflearn_spring_security_learn/3s/13/ajaxAuthenticationToken.JPG){:data-align="center"}
인증을 위임하며 인증에 필요한 값을 보내기 위해 Token을 생성하고, 정상적으로 로그인이 된 것을 확인할 수 있습니다.

**ProviderManager에 있는 Provider**{:data-align="center"}
![ProviderManager에 있는 Provider](:/inflearn_spring_security_learn/3s/13/prividerManager.JPG){:data-align="center"}
ProviderManager가 본인이 가지고 있는 Provider를 반복해서 확인하며 인증에 사용할 수 있는 Provider를 찾습니다.<br>
하지만 사진에서 보시다시피 DaoProvider밖에 없기 때문에 모든 if문에 들어가지 못하고, 예외를 보냅니다.

**인증이 등록되있지 않기 때문에 오류 발생**{:data-align="center"}
![인증이 등록되있지 않기 때문에 오류 발생](:/inflearn_spring_security_learn/3s/13/request.JPG){:data-align="center"}
인증이 되지 않고, 오류가 난 모습입니다.

### 인증 처리자 - AjaxAuthenticationProvider


### 참고
#### Ajax 인증과 Form 인증의 차이점
Form 인증 방식은 동기적인 방식의 인증이고,
Ajax 인증은 비동기적인 방식이 차이점입니다. (자세한 내용 찾아서 넣어놓고, 없으면 위로 올려놓을것.) 

#### 직렬화
- 자바 시스템 내부에서 사용되는 Object나 Data를 외부의 자바 시스템에서도 사용할 수 있도록 byte 형태로 데이터를 변환하는 기술이면서
- JVM의 메모리에 상주되어 있는 객체 데이터를 바이트 형태로 변환하는 기술입니다.

### 출처
1. [학습중인 강의](https://www.inflearn.com/course/%EC%BD%94%EC%96%B4-%EC%8A%A4%ED%94%84%EB%A7%81-%EC%8B%9C%ED%81%90%EB%A6%AC%ED%8B%B0)
2. [고코딩 - Java의 직렬화(Serialize)란?](https://go-coding.tistory.com/101)