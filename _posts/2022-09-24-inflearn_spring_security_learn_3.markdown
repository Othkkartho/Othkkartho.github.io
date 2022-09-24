---
# multilingual page pair id, this must pair with translations of this page. (This name must be unique)
lng_pair: springlearn
title: 인프런 스프링 시큐리티 강의 학습-3

# post specific
# if not specified, .name will be used from _data/owner/[language].yml
author: Othkkartho's Workshop
# multiple category is not supported
category: spring security learn
# multiple tag entries are possible
tags: [Spring, inflearn, spring security learn]
# thumbnail image for post
img: ":/inflearn_spring_security_learn/post_spring_security.PNG"
# disable comments on this page
#comments_disable: true

# publish date
date: 2022-09-24 10:00:00 +0900

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

출처는 인프런의 스프링 시큐리티 - [Spring Boot 기반으로 개발하는 Spring Security](https://www.inflearn.com/course/%EC%BD%94%EC%96%B4-%EC%8A%A4%ED%94%84%EB%A7%81-%EC%8B%9C%ED%81%90%EB%A6%AC%ED%8B%B0)강의를 바탕으로 이 포스트를 작성하고 있습니다.<br>
강의의 세션 1의 5,6,7번 강의내용에 대한 정리입니다.

<!-- outline-end -->

### 인증 API - Logout, LogoutFilter
#### Logout
1. 클라이언트가 /logout으로 로그아웃 요청을 합니다.
2. 서버는 세션 무효화, 인증토큰 삭제, 쿠키정보 삭제, 로그인 페이지로 리이렉트 시킵니다.

##### 로그아웃 API 설정
{% highlight Spring %}
http.logout()                                       // 1
    .logoutUrl("/logout")                           // 2
    .logoutSuccessUrl("/login")                     // 3
    .deleteCookies("JESSIONID", "remember-me")      // 4
    .addLogoutHandler("logoutHandler()")            // 5
    .logoutSuccessHandler("logoutSuccessHandler")   // 6
{% endhighlight %}
1. 로그아웃 처리를 진행합니다.
2. 로그아웃 처리를 할 때 로그아웃을 요청할 URL을 설정합니다.
3. 로그아웃 성공 후 어디로 이동할 지 이동페이지를 설정합니다.
4. 쿠키를 발급 했을 때 삭제할 쿠키를 적으면 로그아웃이 되며 쿠키가 삭제됩니다.
5. 시큐리티가 제공하는 기본적인 로그아웃 핸들러가 아닌 사용자가 설정한 방식으로 로그아웃이 될 수 있도록 사용자가 만든 핸들러를 입력하면 그 핸들러 동작을 실행하게 됩니다.
6. 로그아웃 성공 후 작동하는 핸들러를 작성해 적으면 로그아웃이 된 후 그 핸들러 동작을 실행하게 됩니다.

##### 실제 코드
우선 로그아웃은 POST 방식으로 요청을 보냅니다. 따라서 GET 방식으로 로그아웃 할 수도 있지만, 원칙적으로는 POST 방식으로 logout을 설정해야합니다.
{% highlight Spring %}
http
    .logout()
    .logoutUrl("/logout")
    .logoutSuccessUrl("login")
    .addLogoutHandler(new LogoutHandler() {
        @Override
        public void logout(HttpServletRequest request, HttpServletResponse response, Authentication authentication) {
            HttpSession session = request.getSession();
            session.invalidate();
        }
    })
    .logoutSuccessHandler(new LogoutSuccessHandler() {
        @Override
        public void onLogoutSuccess(HttpServletRequest request, HttpServletResponse response, Authentication authentication) throws IOException, ServletException {
            response.sendRedirect("/login");
        }
    })
    .deleteCookies("remember-me");  // 1
{% endhighlight %}
위 코드는 실제 코드입니다. 받는 파라미터는 request, response, authentication을 받습니다.

- 1번은 추후 공부할 remember-me 인증을 발급하는데 이 때 서버는 쿠키를 발급합니다. 하지만 로그아웃 할 때 이 쿠키를 삭제하고 싶다면 작성하면 삭제가 됩니다.
실제 로그아웃 버튼을 클릭하고 나면 아래 이미지와 같이 브레이크 포인트가 걸리며 정상적으로 실행되고 있음을 보여줍니다.
![logout 실제 브레이크 포인트 장면](:/inflearn_spring_security_learn/1s/3/logout.PNG)

### LogoutFilter
1. 로그아웃 요청을 사용자가 하면 로그아웃 필터가 처리합니다.
2. AntPathRequestMatcher에서 /logout으로 요청이 들어왔는지 검사합니다.
3. 위에서 매치가 되면 Filter가 SecurityContext로 부터 Authentication를 꺼냅니다.
4. 인증 객체를 SecurityContextLogoutHandler에게 보냅니다.
5. 핸들러는 세션 무효화, 쿠키 삭제, SecurityContexHolder.clearContext()에서 SecurityContext를 삭제하고, 인증 객체도 null로 초기화합니다.
6. 로그아웃 핸들러가 정상적으로 종료가 되면 SimpleUrlLogoutSuccessHandler를 호출해 로그인 페이지로 이동시킵니다.

### 인증 API - Remember Me 인증
#### 인증의 이해 및 실행내용
Remember Me는 세션이 만료되거나, 웹 브라우저가 종료된 후에도 어플리케이션이 사용자를 기억하게 하는 기능입니다.<br>
설정을 활성화 시키면 아래와 같은 상황이 일어납니다.
1. 로그인 폼에서 시큐리티가 Remember me 체크박스를 만들어 줍니다.
2. 체크박스를 하고, 로그인 인증을 하게되면 Remember-me 쿠키를 만들어줍니다.
3. 그럼 이후부터 Remember-Me 쿠키에 대한 Http 요청을 확인한 후 토큰 기반 인증을 사용해 유효성을 검사하고, 토큰이 검증되면 사용자는 로그인이 됩니다.

#### 사용자 라이프 사이클
- 인증을 성공하면 Remember-Me 쿠키가 설정됩니다.
- 인증에 실패하면 쿠키가 존재한다면 쿠키를 무효화 시킵니다.
- 로그아웃이 일어날 때 쿠키가 존재한다면 쿠키를 무효화 시킵니다.

#### Remember Me 인증 API
{% highlight Spring %}
http.rememberMe()                               // 1
    .rememberMeParameter("remember")            // 2
    .tokenValiditySeconds(3600)                 // 3
    .alwaysRemember(true)                       // 4
    .userDetails Service(userDetailsService)    // 5
{% endhighlight %}
위에 작성된 코드가 RememberMe 설정 코드입니다.
1. Remember Me의 처리를 진행합니다.
2. Remember Me의 파라미터명을 지정합니다. 기본명은 remember-me입니다. 사용자 화면에 체크박스 파라미터명도 동일하게  맞춰줘야합니다.
3. Remember Me 쿠키의 만료시간을 초 단위로 설정할 수 있습니다. 기본값은 14일입니다.
4. Remember Me 체크박스에 체크를 하지 않아도 Remember Me 설정을 허용할지를 설정할 수 있습니다.
5. Remember Me 기능을 수행할 때 시스템에 있는 사용자 계정을 조회하는데 필요한 클레스입니다.

#### 실제 코드
{% highlight Spring %}
@Bean
public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
    private final UserDetailsService userDetailsService;

    public SecurityConfig(UserDetailsService userDetailsService) {
        this.userDetailsService = userDetailsService;
    }

    ~~ // 위 코드는 로그인 코드와 동일
            .and()
            .rememberMe()
            .rememberMeParameter("remember")
            .tokenValiditySeconds(3600)
            .userDetailsService(userDetailsService);

    return http.build();
}
{% endhighlight %}
간단히 코드를 살펴보면 먼저 Remember Me를 선언해 주고, 파라미터명을 remember로 변경시켰습니다.<br>
쿠키의 유지 기간은 3600초 = 1시간 이고, UserDetailService를 Security에 선언된 것을 받아와 사용했습니다.

#### 결과
다음은 결과 캡쳐들입니다.<br>
![remember me 체크박스](:/inflearn_spring_security_learn/1s/3/remember_checkbox.PNG){:data-align="center"}
위 사진을 보면 알 수 있듯이 원래 로그인 폼에 Remember me 체크박스가 생긴것을 알 수 있습니다.
![remember me 체크박스 체크 완료](:/inflearn_spring_security_learn/1s/3/remember_checkbox_check.PNG){:data-align="center"}
위 사진은 로그인을 하고 Remember Me 체크박스에 체크한 모습입니다.
![JsessionID 쿠키 확인](:/inflearn_spring_security_learn/1s/3/Jsessionid_cookie.PNG){:data-align="center"}
위 사진은 로그인이 완료되고, 로그인 쿠키인 JsessionID 쿠키가 생성된 사진입니다.
![remember me 쿠키 확인](:/inflearn_spring_security_learn/1s/3/remember_me_cookie.PNG){:data-align="center"}
위 사진은 로그인이 완료되고, 위 코드덕분에 remember me 쿠키가 생성된 모습입니다.
![JsessionID 쿠키 삭제](:/inflearn_spring_security_learn/1s/3/jsessionid_delet.PNG){:data-align="center"}
위 사진은 로그인 정보가 담겨있는 JsessionID 쿠키를 삭제한 모습입니다.
![JsessionID 쿠키 재발급](:/inflearn_spring_security_learn/1s/3/Jsessionid_cookie_re.PNG){:data-align="center"}
위 사진은 JsessionID 쿠키를 삭제한 후 새로고침 후 JsessionID가 정상적으로 재생성된 모습입니다.

### RememberMeAuthenticationFilter
#### RememberMeAuthenticationFilter의 작동 방식
필터는 인증을 받은 사용자가 세션이 만료되었거나, 사용하던 브라우저가 종료되어 세션이 끊겨 세션이 활성화되지 않아 인증 객체를 세션에서 찾지 못하는 경우 자동적으로 사용자의 인증을 유지하기 위해 필터가 인증을 시도해 다시 인증을 받게 만들어 그 사용자가 서버에 인증이 유지된채 접근이 가능하도록 합니다.
1. 필터는 사용자의 요청시 조건에 맞으면 동작하게 됩니다.
    - 첫번째 조건은 Authentication(인증 객체)가 null일 경우입니다. 인증받은 사용자는 인증 객체가 Security Context에 늘 존재합니다. 하지만 인증 객체가 S.C에 존재하지 않는 경우 Filter가 작동합니다. 인증 객체가 있다는 의미는 이미 인증이 되어 있다는 것을 의미하기에 Rememberme필터가 작동할 이유가 없기 때문입니다.
        - S.C에 인증 객체가 존재하지 않는 경우로는 사용자의 세션이 만료되었거나, 세션이 끊겨 더이상 세션안에서 S.C를 찾지 못면 인증 객체도 없는 것이 되는 경우입니다.
    - 두번째 조건은 사용자가 Remember-me 쿠키를 가지고 있어야 합니다.
2. 그럼 RememberMeServices 인터페이스에 TokenBasedRememberMeServices와 PersistentTokenBasedRememberMeServices 2개의 구현체가 있습니다. 이 두 구현체가 실제로 Remember Me 인증 처리를 하는 Class입니다.
    - TokenBasedRememberMeServices는 메모리에서 실제로 저장한 토큰과 사용자가 요청에 보낸 토큰과 비교해 인증처리를 합니다. 기능적으로는 14일 만료기간이 있습니다.
    - PersistentTokenBasedRememberMeServices는 DB에 발급한 토큰을 저장하고, 사용자가 요청에 보낸 토큰의 값과 DB에 저장된 값을 비교해 인증처리를 합니다. 만료기간은 무한으로 가능합니다.
3. RememberMeServices는 Token Cookie를 추출합니다.
4. Token중 RememberMe Token이 존재하는지 확인합니다.
5. 존재할 경우 Decode Token에서 Remember Me 토큰이 정상적인 규칙을 지키고 있는지 정상 유무를 판단합니다.
6. 그 후 서버에 저장된 토큰과 사용자의 토큰이 일치하는지를 확인하고, 일치하면 그 토큰의 User 계정이 서버에 존재하는지를 확인합니다. 존재한다면 새로운 Authentication을 생성하고, 이 인증 객체를 AuthenticationManager에게 보내 실질적인 인증처리를 하게됩니다.
    - 5, 6번에서 오류가 난다면 Exception 처리를 합니다.

### 참고
#### rememberMe 코드에서 private final 선언에 관해
원래 강의에서는 Autowired 어노테이션을 사용했습니다. 하지만 Field injection is not recommended 오류가 확인되었습니다. 사실 Error 메시지가 아닌 Warning 메시지라 무시해도 되지만, 해결할 수 있는 것은 해결하고 싶었기 때문에 private final로 변경해 코드를 작성했습니다.<br>

##### DI
Autowired를 살펴보기 전 DI에 대해 알아보고 가겠습니다.<br>
DI(의존성 종속, Dependency Injection)란, 클래스간의 의존관계를 스프링 컨테이너가 자동으로 연결해주는 것을 말합니다.<br>
DI를 필요로 하는 이유는 아래 Factory 인터페이스를 상속받는 ConsoleFactory, UserFactory가 있습니다.
SW를 사용하는 고객은 Factory 클래스만을 호출해야하며, 그것이 ConsoleFactory인지 UserFactory인지 몰라야합니다. 
고객마다 전용 Factory를 생성할 경우 코드 생산성이 떨어지며, 고객이 몰라도 되는 코드가 노출되기 때문입니다. 
때문에 스프링은 Factory가 ConsoleFactory인지 UserFactory인지를 프레임워크가 자동으로 객체간 의존성을 주입해줍니다.

##### Autowired
오류를 살펴보기에 앞서 Autowired를 먼저 살펴보겠습니다.<br>
Autowired는 스프링 DI에서 사용되는 어노테이션입니다. 스프링에서 빈 인스턴스가 생성된 후 @Autowired를 설정한 메소드가 자동으로 호출되고, 인스턴스가 자동으로 주입됩니다.<br>즉, 해당 변수 및 메서드에 스프링이 관리하는 Bean을 자동으로 매핑해주는 개념입니다. @Autowired 는 변수, Setter메서드, 생성자, 일반 메서드에 적용이 가능하며 <property>, <constructor-arg>태그와 동일한 역할을 합니다.

##### Field injection is not recommended 오류
{% highlight Spring %}
@Autowired
UserDetailsService userDetailsService
{% endhighlight %}
위의 방식을 Field Injection 방식이라고 합니다. 이 방식은 setter기반과 마찬가지로 빈 생성이 완료된 이후 주입되며, final로 선언할 수 없습니다.<br>
보기에는 매우 간결합니다. 하지만 IDE에서의 워닝과 같이 몇 가지 단점이 있습니다.<br>
1. 주입된 객체는 Immutable한 상태를 만들 수 없습니다.
    - 오직 Constructor Injection 만이 final 선언이 가능합니다. 나머지 방법은 주입되는 필드에 대해 mutable한 상태를 만들기 때문에 가급적이면 생성자 주입을 적용하는게 좋습니다.
2. DI 컨테이너에 강한 결합 발생
    - Field Injection은 생성자를 통해서도, setter를 통해서도 주입을 받는 방식이 아닙니다. 그렇기 때문에 Spring이 아니면 해당 필드에 Injection을 할 수 있는 방법이 없습니다. (리플렉션 제외) 즉, Spring DI 컨테이너 밖에서 작동할 수 없는 코드를 만듭니다. 이는 흔히 소프트웨어 공학에서 추구하는 loose coupling (결합도를 낮추고 응집도를 높이는 행위)와 반대됩니다.

Spring 5.2.3 Reference에서는
필수적인 의존성에서는 Constructor Injection을,
선택적인 의존성에서는 Setter Injection을 사용하라고 합니다.

##### 강한 결합 & 느슨한 결합
- 강한 결합은 객체 내부에서 다른 객체를 생성하는 것은 강한 결합도를 가지는 구조입니다. A 클래스 내부에서 B 라는 객체를 직접 생성하고 있다면, B 객체를 C 객체로 바꾸고 싶은 경우에 A 클래스도 수정해야 하는 방식이기 때문에 강한 결합입니다.
- 느슨한 결합은 객체를 주입 받는다는 것은 외부에서 생성된 객체를 인터페이스를 통해서 넘겨받는 것입니다. 이렇게 하면 결합도를 낮출 수 있고, 런타임시에 의존관계가 결정되기 때문에 유연한 구조를 가집니다.

### 출처
1. [학습중인 강의](https://www.inflearn.com/course/%EC%BD%94%EC%96%B4-%EC%8A%A4%ED%94%84%EB%A7%81-%EC%8B%9C%ED%81%90%EB%A6%AC%ED%8B%B0)
2. [DI와 Autowired 설명부분 출처](https://life-with-coding.tistory.com/433) 
3. [Field injection is not recommended 오류에 관한 설명의 출처](https://sightstudio.tistory.com/20)
4. [강한 결합과 약한 결합의 설명 출처](https://yaboong.github.io/spring/2019/08/29/why-field-injection-is-bad/)