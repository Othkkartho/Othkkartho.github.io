---
# multilingual page pair id, this must pair with translations of this page. (This name must be unique)
lng_pair: springlearn
title: 인프런 스프링 시큐리티 강의 학습-13

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
하지만 사진에서 보시다시피 DaoProvider밖에 없기 때문에 인증을 찾지 못하고, 예외를 보냅니다.

**인증이 등록되있지 않기 때문에 오류 발생**{:data-align="center"}
![인증이 등록되있지 않기 때문에 오류 발생](:/inflearn_spring_security_learn/3s/13/request.JPG){:data-align="center"}
인증이 되지 않고, 오류가 난 모습입니다.

### 인증 처리자 - AjaxAuthenticationProvider
#### 실제 코드
**AjaxAuthenticationProvider.java**{:data-align="center"}
```java
@Component
public class AjaxAuthenticationProvider implements AuthenticationProvider {
    private UserDetailsService userDetailsService;
    private PasswordEncoder passwordEncoder;

    @Autowired
    private void setAjaxAuthenticationProvider(UserDetailsService userDetailsService, PasswordEncoder passwordEncoder) {
        this.userDetailsService = userDetailsService;
        this.passwordEncoder = passwordEncoder;
    }

    @Override
    @Transactional
    public Authentication authenticate(Authentication authentication) throws AuthenticationException {
        AccountContext accountContext = FormAuthenticationProvider.authenticationIf(authentication, userDetailsService, passwordEncoder);   // 1

        return new AjaxAuthenticationToken(accountContext.getAccount(), null, accountContext.getAuthorities()); // 1
    }

    @Override
    public boolean supports(Class<?> authentication) {
        return authentication.equals(AjaxAuthenticationToken.class);        // 2
    }
}

```
기본적으로 FormAuthenticationProvider과 하는 것은 모두 같습니다.
1. authenticate 메소드가 사용하는 코드가 비슷하기 때문에 아래 나오는 authenticationIf 메소드로 만들어 값을 전달하고, AccountContext를 반환받아 사용하도록 했습니다.
2. authentication에서 받은 객체의 클래스 타입이 AjaxAuthenticationToken와 같다면 이 Provider를 실행하게 합니다.

**FormAuthenticationProvider.java**{:data-align="center"}
```java
static AccountContext authenticationIf(Authentication authentication, UserDetailsService userDetailsService, PasswordEncoder passwordEncoder) {
    String username = authentication.getName();
    String password = (String) authentication.getCredentials();

    AccountContext accountContext = (AccountContext) userDetailsService.loadUserByUsername(username);

    if (!passwordEncoder.matches(password, accountContext.getAccount().getPassword())) {
        throw new BadCredentialsException("BadCredentialsException");
    }

    FormWebAuthenticationDetails formWebAuthenticationDetails = (FormWebAuthenticationDetails) authentication.getDetails();
    String secretKey = formWebAuthenticationDetails.getSecretKey();
    if (!"secret".equals(secretKey)) {
        throw new InsufficientAuthenticationException("InsufficientAuthenticationException");
    }

    return accountContext;
}
```
CustomAuthenticationProvider를 FormAuthenticationProvider로 이름을 바꾸고, AjaxAuthenticationProvider도 사용하는 위 코드를 메소드로 묶어 사용했습니다.

**AjaxSecurityConfig.java**{:data-align="center"}
```java
@Configuration
@Order(0)                           // 1
public class AjaxSecurityConfig {
    private AuthenticationConfiguration authenticationConfiguration;

    @Autowired
    private void setAjaxSecurityConfig(AuthenticationConfiguration authenticationConfiguration) {
        this.authenticationConfiguration = authenticationConfiguration;
    }

    @Bean
    public AuthenticationProvider ajaxAuthenticationProvider() {                                    // 2
        return new AjaxAuthenticationProvider();
    }

    @Bean
    public AuthenticationManager authenticationManager(AuthenticationConfiguration authenticationConfiguration) throws Exception {
        return authenticationConfiguration.getAuthenticationManager();
    }

    @Bean
    public SecurityFilterChain ajaxFilterChain(HttpSecurity http) throws Exception {
        AuthenticationManagerBuilder authenticationManagerBuilder = http.getSharedObject(AuthenticationManagerBuilder.class);
        authenticationManagerBuilder.authenticationProvider(ajaxAuthenticationProvider());          // 2

        http
                .antMatcher("/api/**")                                                              // 3
                .authorizeRequests()
                .anyRequest().authenticated()
                .and()
                .addFilterBefore(ajaxLoginProcessingFilter(), UsernamePasswordAuthenticationFilter.class);

        http.csrf().disable();

        return http.build();
    }

    @Bean
    public AjaxLoginProcessingFilter ajaxLoginProcessingFilter() throws Exception {
        AjaxLoginProcessingFilter ajaxLoginProcessingFilter = new AjaxLoginProcessingFilter();
        ajaxLoginProcessingFilter.setAuthenticationManager(authenticationManager(authenticationConfiguration));
        return ajaxLoginProcessingFilter;
    }
}
```
SecurityConfig가 너무 설정하는 것들이 많아져 유지보수와 가독성이 떨어져 AjaxSecurityConfig를 다시 만들었습니다.
1. SecurityConfig가 2개이기 때문에 어노케이션을 이용해 우선순위를 지정해 줘야 합니다. AjaxSecurityConfig가 0으로 먼저 실행되고, SecurityConfig가 1로 두번째 실행되게 만들었습니다.
2. AjaxAuthenticationProvider를 Bean에 등록하고, Provider를 추가합니다.
3. /api 로 시작하는 모든 경로에 아래 설정을 적용합니다.

### 참고
#### Ajax 인증과 Form 인증의 차이점
Form 인증 방식은 동기적인 방식의 인증이고,
Ajax 인증은 비동기적인 방식이 차이점입니다. (자세한 내용 찾아서 넣어놓고, 없으면 위로 올려놓을것.) 

#### 직렬화
- 자바 시스템 내부에서 사용되는 Object나 Data를 외부의 자바 시스템에서도 사용할 수 있도록 byte 형태로 데이터를 변환하는 기술이면서
- JVM의 메모리에 상주되어 있는 객체 데이터를 바이트 형태로 변환하는 기술입니다.

#### AjaxAuthenticationProvider 객체가 등록이 되지 않는 오류
##### 오류 상황
AjaxAuthenticationFilter는 정상적으로 FilterChainProxy에 등록이 됩니다. Config가 정상 작동 한다는 의미입니다.<br>
하지만 Filter가 ProviderManager로 이동하니 ProviderManager에 등록되어 있는 providers가 DaoAuthenticationProvider 1개밖에 존재하지 않습니다.<br><br>

##### 오류 해결
AuthenticationManager 는 초기화 때 생성되어 기본적으로 DaoAuthenticationProvider 와 같은 객체를 가지고 있습니다. 그리고 UsernamePasswordAuthenticationFilter 와 같은 클래스에서 참조하고 있습니다.<br><br>

그렇다면 ajaxAuthenticationProvider 도 초기화때 생성된 AuthenticationManager 에서 추가해 주어야 합니다.
```java
AuthenticationManagerBuilder authenticationManagerBuilder = http.getSharedObject(AuthenticationManagerBuilder.class);
authenticationManagerBuilder.authenticationProvider(ajaxAuthenticationProvider());
```

위 코드가 초기화 때 생성된 AuthenticationManager 입니다. 그런데 실제 AjaxLoginProcessingFilter 를 생성하는 코드를 보면 초기화 때 생성된 동일한 AuthenticationManager 가 아닌 다른 AuthenticationManager 를 참조하고 있습니다. <br>

즉 authenticationConfiguration.getAuthenticationManager(); 에서 참조하고 있는 AuthenticationManager 와 authenticationManagerBuilder.authenticationProvider(ajaxAuthenticationProvider()); 를 통해 참조되는 AuthenticationManager 는 동일한 객체가 아닙니다.<br><br>

그렇기 때문에 AjaxLoginProcessingFilter 에서 참조하고 있는 AuthenticationManager 에 ajaxAuthenticationProvider 를 추가해 주어야 정상동작하게 됩니다. 위 코드로 수정한 후에 디버깅해 보시면 AuthenticationManager 가 서로 다른 두개의 객체가 생성되어 있음을 알게 됩니다.<br><br>

출처는 [인프런 질문 게시판](https://www.inflearn.com/questions/667022)입니다.

***

제가 이해한 내용을 정리하면 다음과 같습니다.<br><br>

AuthenticationManager는 초기화 때 생성됩니다. 그래서 ajaxAuthenticationProvider도 초기화때 생성된 AuthenticationManager에서 추가되어야 작동을 합니다.<br>
하지만 위 코드의 `authenticationManagerBuilder.authenticationProvider(ajaxAuthenticationProvider());` 와 `ajaxLoginProcessingFilter.setAuthenticationManager(authenticationManager(authenticationConfiguration));` 는 다른 AuthenticationManager를 참조하고 있으므로, 실행을 해 봤을 때는 기본 생성된 DaoAuthenticationProvider만 가지고 검사됩니다.<br>
이 참조되는 AuthenticationManager을 같게 하기 위해 ajaxLoginProcessingFilter가 AuthenticationManager를 set하기 위해 가져가는 authenticationManager(authenticationConfiguration) 의 안에서 Provider를 추가해야 서로 같은 Manager 객체가 되어 Provider가 추가되게 됩니다.<br><br>
오류 수정을 진행할 때 그냥 SecurityConfig에서 provider가 정상적으로 생성되는 이유는 AuthenticationManager가 filterChain에서만 생성되기 때문이었습니다.<br>

**오류 시 ProviderManager가 확인한 Provider**{:data-align="center"}
![Dao Provider만 있는 모습](:/inflearn_spring_security_learn/3s/13/error_providers.jpg){:data-align="center"}

**오류 해결 흐 ProviderManager가 확인한 Provider**{:data-align="center"}
![Ajax Provider과 Dao Provider이 있는 모습](:/inflearn_spring_security_learn/3s/13/after_providers){:data-align="center"}

### 출처
1. [학습중인 강의](https://www.inflearn.com/course/%EC%BD%94%EC%96%B4-%EC%8A%A4%ED%94%84%EB%A7%81-%EC%8B%9C%ED%81%90%EB%A6%AC%ED%8B%B0)
2. [고코딩 - Java의 직렬화(Serialize)란?](https://go-coding.tistory.com/101)