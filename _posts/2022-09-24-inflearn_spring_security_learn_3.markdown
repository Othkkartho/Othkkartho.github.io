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
강의의 세션 1의 5번 강의내용에 대한 정리입니다.

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

### 참고
#### 


### 출처
1. [학습중인 강의](https://www.inflearn.com/course/%EC%BD%94%EC%96%B4-%EC%8A%A4%ED%94%84%EB%A7%81-%EC%8B%9C%ED%81%90%EB%A6%AC%ED%8B%B0)
2. 