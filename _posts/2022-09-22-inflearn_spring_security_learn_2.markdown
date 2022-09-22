---
# multilingual page pair id, this must pair with translations of this page. (This name must be unique)
lng_pair: springlearn
title: 인프런 스프링 시큐리티 강의 학습-2

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
date: 2022-09-22 10:00:00 +0900

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
강의의 세션 1의 3,4번 강의내용에 대한 정리입니다.

<!-- outline-end -->

### Form Login 인증
#### 인증 API - Form 인증
우선 전 포스트에서 참조에 작성되었던 내용이지만 강의에 있기 때문에 다시 작성하겠습니다.<br>
1. 클라이언트가 서버에 GET 형태로 /home에 접근을 시도한다면
2. 서버는 자신의 자원에 접근하기 위해서는 인증된 사용자만이 접근 가능하다고 보안 정책에 설정 되어있습니다.
3. 서버는 인증이 안되었다면 로그인 페이지로 리다이렉트합니다.
4. 클라이언트는 POST form data로 username(ID)와 password를 서버에 넘겨 인증을 시도하게됩니다.
5. 서버에서는 Security가 Session을 생성하게 되고, Session에 최종 성공한 인증 결과를 담은 인증 토큰(객체)을 생성하고, Security context에 생성하고, Session에 저장합니다.
6. 다시 GET /home 세션에 접근하게 되면 서버는 인증 토큰의 존재 여부를 판단하고, 사용자는 지속적으로 그 인증 토큰으로 접근하고/인증을 유지하는 방식으로 처리합니다.

#### Form Login API
다음으론 폼 로그인이 가지고 있는 API에 관해 설명하겠습니다.
{% highlight Spring %}
http.formLogin()
    .loginPage("/login.html")               // 1
    .defaultSuccessUrl("/home)              // 2
    .failureUrl("/login.html?error=true")   // 3
    .usernameParameter("username")          // 4
    .passwordParameter("passowrd")          // 5
    .loginProcessingUrl("/login")           // 6
    .successHandler(loginSuccessHandler())  // 7
    .failureHandler(loginFailureHandler())  // 8
{% endhighlight %}

1. 기본적으로 제공하는 로그인 페이지가 아닌 사용자 정의 로그인 페이지를 넣습니다.
2. 로그인 성공 후 이동할 페이지를 넣습니다.
3. 로그인 실패 후 이동할 페이지를 넣습니다.
4. 아이디의 파라미터명을 변경할 수 있습니다. 이걸 바꾼다면 폼 테그의 이름 또한 변경됩니다. 디폴트는 username입니다.
5. 비밀번호의 파라미터명을 변경할 수 있습니다. 디폴트는 password입니다.
6. 로그인할 때 Form Action url을 변경할 수 있습니다.
    - 따라서 4, 5, 6번은 UI에 적은 이름 그대로 변경하면 됩니다.
7. 로그인을 성공했을 때의 핸들러입니다. 로그인을 성공했을 때의 동작입니다.
8. 로그인을 실패했을 때의 핸들러입니다. 로그인을 실패했을 때의 동작입니다.

#### 실제 코드 및 결과 화면
{% highlight Spring %}
@Configuration
@EnableWebSecurity
public class SecurityConfig {
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
                .authorizeRequests()
                .anyRequest().authenticated(); // 1
        http
                .formLogin()
                .loginPage("/loginPage")
                .defaultSuccessUrl("/")
                .failureUrl("/login")
                .usernameParameter("userId")        // 5
                .passwordParameter("pw")            // 5
                .loginProcessingUrl("/login_proc")  // 5
                .successHandler(new AuthenticationSuccessHandler() { // 2
                    @Override
                    public void onAuthenticationSuccess(HttpServletRequest request, HttpServletResponse response, Authentication authentication) throws IOException, ServletException {
                        System.out.println("authentication: " + authentication.getName());
                        response.sendRedirect("/");
                    }
                })
                .failureHandler(new AuthenticationFailureHandler() { // 3
                    @Override
                    public void onAuthenticationFailure(HttpServletRequest request, HttpServletResponse response, AuthenticationException exception) throws IOException, ServletException {
                        System.out.println("exception: " + exception.getMessage());
                        response.sendRedirect("/login");
                    }
                })
                .permitAll(); // 4

        return http.build();
    }
}
{% endhighlight %}
위 코드는 SecuritConfig의 전문입니다.

1. 서버에 접속하는 모든 요청에 관해 인증을 요청합니다.
2. 성공했을 때 작동하는 핸들러입니다. 받는 변수는 이고, 성공시 루트 페이지로 이동하고, 사용자 이름이 콘솔에 출력되도록 했습니다.
3. 실패했을 때 작동하는 핸들러입니다. 받는 변수는 이고, 실패시 로그인 페이지로 이동하고, 실패 메시지를 콘솔에 출력되도록 했습니다.
4. 1번에서 인증 요청을 했기 때문에 사용자 지정 로그인 페이지에 접속할 때 인증하지 않도록 예외처리를 진행했습니다.

다음은 결과 화면입니다.<br>
loginPage()에 관한 결과 화면입니다.
![loginPage](:/inflearn_spring_security_learn/1s/2/loginPage.PNG){:data-align="center"}
실제 로그인 화면인 /login이 아닌 /loginPage가 출력이 됩니다. 화면에는 아래의 코드와 같이 loginPage 문자를 출력합니다.
{% highlight Spring %}
@GetMapping("loginPage")
public String loginPage() {
    return "loginPage";
}
{% endhighlight %}

다음은 loginPage를 주석처리 하고, 로그인 시도를 했습니다.<br>
- 2번의 결과 화면입니다.
![successHandler](:/inflearn_spring_security_learn/1s/2/authentication.PNG){:data-align="center"}
먼저 로그인 페이지가 출력이 되고, 로그인이 완료가 되면 루트 페이지로 돌아갑니다.<br>
위 결과 화면은 로그인 한 유저의 username입니다.<br>

- 3번의 결과 화면입니다.
![failureHandler](:/inflearn_spring_security_learn/1s/2/exception.PNG){:data-align="center"}
로그인 페이지에서 비밀번호를 다르게 입력하면, 설정했던 대로 위의 사진과 같이 오류가 로그에 출력되고, 로그인 화면으로 돌아갑니다.<br>

- 5번의 결과 화면입니다.
![parameter](:/inflearn_spring_security_learn/1s/2/parameter.PNG){:data-align="center"}
위 코드에서 설정했던 대로 action에 /login_proc, username input 메소드에 이름이 userId, password input 메소드에 이름이 pw가 되어있는 것을 확인할 수 있습니다.

### 인증 API - UsernamePasswordAuthenticationFilter
#### UsernamePasswordAuthenticationFilter의 흐름
UsernamePasswordAuthenticationFilter은 사용자가 로그인 후의 인증처리를 담당하고, 관련 요청을 처리하는 필터입니다.
1. 먼저 사용자가 인증을 시도하면, 필터가 요청을 받습니다.
2. AntPathRequestMatcher가 요청 정보의 Url이 매칭되는지 확인합니다. 디폴트는 /login이고, 이 값은 loginProcessingUrl에서 변경이 가능합니다. 매칭이 되면 실제 인증처리를 하고, 매칭이 되지 않으면 인증처리를 하지 않고 다른 필터로 이동합니다.
3. 매칭 후 필터는 Authentication객체를 만들어 사용자가 로그인할 때 입력한 Username과 Password 값을 인증 객체에 저장해 인증을 맡깁니다.
4. AuthenticationManager가 Authentication 객체를 받고, 실제 인증 처리를 합니다.
    - 이 Manager은 AuthenticationProvider 객체가 있고, 이 객체들 중에게 인증을 위임합니다.
    - 인증이 성공, 실패 결과를 리턴하는데 실패하면 필터에게 예외를 보냅니다.
    - 인증에 성공하면 Authentication 객체를 만들어 인증 성공 결과를 객체에 저장하고, Manager에게 Return합니다.
5. Manager는 Provider에게 받은 객체를 필터에게 반환합니다.
    - 그 객체에는 유저 정보와, 유저의 권한 정보가 담겨 있습니다.
6. Filter은 인증 객체를 SecurityContext에 저장합니다.
    - 이 Context 객체가 Session에도 저장되 사용자가 Context 안에서 인증 객체를 참조할 수 있도록 처리해주기도 합니다.
7. 인증에 성공한 이후에 SuccessHandler가 성공 이후의 작업들을 처리하기도 합니다.

### 참고
#### Handler의 입력 메소드 설명
- HttpServletRequest
    - Http프로토콜의 request 정보를 Servlet에게 전달하기 위한 목적으로 사용합니다.
    - Header정보, Parameter, Cookie, URI, URL 등의 정보를 읽어드리는 메소드를 가진 클래스입니다.
    - Body의 Stream을 읽어들이는 메소드를 가지고 있습니다.

- HttpServletResponse
    - WAS는 어떤 클라이언트가 요청을 보냈는지 알고 있고, 해당 클라이언트에게 응답을 보내기 위한 HttpServleResponse 객체를 생성하여 Servlet에게 전달합니다.
    - Servlet은 HttpServletResponse 객체에 Content Type, 응답코드, 응답 메시지등을 담아 전송합니다.

- Authentication
    - 인증에 성공했을 시 그 인증결과를 담은 인정 에셋 파라미터 인터페이스입니다.

- AuthenticationException
    - 인증 오류시 그 인증예외를 파라미터로 전달하는 클래스입니다.

#### FilterChainProxy
Filter들을 관리하는 빈입니다. additionalFilters에 각 실행할 필터들이 있고 이 필터들은 스프링을 실행할 때 만들어 지는 필터들도 있고, 사용자가 등록한 필터들도 등록이 되어있습니다. 그래서 0번부터 끝까지 실행됩니다.

### 출처
1. [학습중인 강의](https://www.inflearn.com/course/%EC%BD%94%EC%96%B4-%EC%8A%A4%ED%94%84%EB%A7%81-%EC%8B%9C%ED%81%90%EB%A6%AC%ED%8B%B0)
2. [HttpServletRequest, HttpServletResponse에 대한 설명](https://www.boostcourse.org/web326/lecture/258511/?isDesc=false)