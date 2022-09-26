---
# multilingual page pair id, this must pair with translations of this page. (This name must be unique)
lng_pair: springlearn
title: 인프런 스프링 시큐리티 강의 학습-6

# post specific
# if not specified, .name will be used from _data/owner/[language].yml
author: Othkkartho's Workshop
# multiple category is not supported
category: spring security learn
# multiple tag entries are possible
tags: [Spring, inflearn, spring security learn]
# thumbnail image for post
img: ":/inflearn_spring_security_learn/post_spring_security.png"
# disable comments on this page
#comments_disable: true

# publish date
date: 2022-09-27 10:00:00 +0900

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
위임 필터 및 필터 빈 초기화, 다중 보안 설정에 관해 포스팅 했습니다.
---------------------------------------------------------------

출처는 인프런의 스프링 시큐리티 - [Spring Boot 기반으로 개발하는 Spring Security](https://www.inflearn.com/course/%EC%BD%94%EC%96%B4-%EC%8A%A4%ED%94%84%EB%A7%81-%EC%8B%9C%ED%81%90%EB%A6%AC%ED%8B%B0)강의를 바탕으로 이 포스트를 작성하고 있습니다.<br>
강의의 세션 2의 1,2번 강의내용에 대한 정리입니다.

<!-- outline-end -->

### DelegatingFilterProxy, FilterChainProxy에 이해
#### DelegatingFilterProxy의 간단한 설명
Servlet Filter는 Servlet 2.3 부터 도입이 되었습니다.<br>
Filter가 하는 역할은 요청이 Servlet 자원으로 가기 전 필터가 먼저 요청을 받아 작업을 처리한 후 요청을 Servlet에 전달하고, Servlet에서 요청에 대한 처리가 끝난 후, 클라이언트에 응답을 전달하기 전에 필터가 받아 작업을 처리 후 필터가 클라이언트에 응답하게 됩니다.<br>
하지만 Filter는 Servlet 스팩을 지원하는 컨테이너에서 생성, 실행이 됩니다. 그래서 필터는 Spring Bean을 Injection하거나 스프링 기술들을 필터에서는 사용할 수 없습니다. 사용하는 위치가 서블릿 컨테이너와 스프링 컨테이너이기 때문입니다.<br>
하지만 세션 1에서 본 Spring Security는 모든 사용자에게 필터 기반으로 모든 요청에 대해 인증, 인가 처리를 하고 있습니다.<br>
스프링 빈은 서블릿 필터를 구현하지만 사용자의 요청에 관해서 Bean이 바로 받지 못합니다. 그 이유는 필터가 서블릿 컨테이너에서 동작하기 때문입니다. 그럼 사용자의 요청을 필터가 빈에게 전달하면 됩니다. 이 기술을 구현하기 위해 DelegatingFilterProxy입니다. 이 Proxy는 서블릿 필터입니다.<br>
서블릿 필터로 부터 받은 요청을 DelegatingFilterProxy는 Spring Bean에게 요청을 위임하고, 스프링 시큐리티는 필터 기반으로 보안처리를 하고, 스프링의 기술도 사용할 수 있게 됩니다.<br><br>
아래에 간단히 정리해보았습니다.
1. 서블릿 필터는 스프링에서 정의된 빈을 주입해 사용할 수 없습니다.
2. 특정한 이름을 가진 스프링 빈을 찾아 그 빈에게 요청을 위임합니다.
    - springSecurityFilterChain 이름으로 생성된 빈을 ApplicationContext에서 찾아 요청을 위임합니다.
    - Proxy는 실제 보안처리를 하지 않습니다.

#### FilterChainProxy 설명
1. springSecurityFilterChain의 이름으로 생성되는 필터 빈입니다.
2. DelegatingFilterProxy로 부터 요청을 위임받고, 실제 보안 처리를 합니다.
3. 스프링 시큐리티 초기화 시 생성되는 필터들을 관리하고 제어합니다.
    - 스프링 시큐리티가 기본적으로 생성하는 필터와
    - 설정 클래스에서 API 추가 시 생성되는 필터들을 관리, 제어합니다.
4. 사용자의 요청을 필터 순서대로(0번부터 아래로) 호출하여 전달합니다.
    - 마지막까지 요청에 대한 처리를 필터가 완료하고 나면 최종적으로 서블릿에 접근하게 됩니다.
5. 사용자정의 필터를 생성해 기존의 필터 전, 후로 추가가 가능합니다.
    - 이때 필터의 순서를 잘 정의해야 됩니다.
6. 마지막 필터까지 인증, 인가 예외가 발생하지 않으면 보안을 통과하는 것입니다.

#### 아키텍처 흐름
1. 유저가 서버에 요청을 보내면 Servlet Container가 처음 요청을 받고, DelegatingFilterProxy가 요청을 받으면 그 요청 객체를 springSecurityFilterChain이름을 가진 빈을 찾습니다.
2. FilterChainProxy가 빈을 등록할 때 이름을 springSecurityFilterChain으로 등록합니다. 그럼 DelegatingFilterProxy는 요청을 위임합니다.
3. FilterChainProxy는 본인이 가지고 있는 모든 필터들을 하나씩 보안처리를 하고, DispatcherServlet과 같은 Spring MVC로 요청을 보내 오청을 처리합니다.

### 필터 초기화와 다중 보안 설정
#### 필터 초기화와 다중 설정 클래스
- 설정클래스 별로 보안 기능이 각각 작동하도록 지원됩니다.
- 설정 클래스 별로 RequestMatcher 설정합니다.
    - SecurityConfig 1에서 http.antMatcher("/admin/**")를 설정했다면 사용자가 /admin으로 접근 하려 한다면 1번의 인가 정책에 따릅니다.
    - 만약 Config 2번에서 설정되 있는 인가 정책이 있다면 2번의 인가 정책을 따릅니다.
- 설정클래스 별로 필터가 생성되 독립적으로 운용됩니다.
- FilterChainProxy 가 각기 다른 SecurityConfg에서 생성된 각각의 SecurityFilterChain를 가지고 있습니다.
- 사용자의 요청에 따라 RequestMatcher와 매칭되는 SecurityFilterChain을 작동하도록 합니다.

#### 실제 코드
**SecurityConfig.java**{:data-align="center"}
```java
@Configuration
@EnableWebSecurity
@Order(0)
public class SecurityConfig {
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
                .antMatcher("/admin/**")
                .authorizeRequests()
                .anyRequest().authenticated()
                .and()
                .httpBasic();

        return http.build();
    }
}
@Configuration
@Order(1)
class SecurityConfig2 {
    @Bean
    public SecurityFilterChain filterChain2(HttpSecurity http) throws Exception {
        http
                .authorizeRequests()
                .anyRequest().permitAll()
                .and()
                .formLogin();

        return http.build();
    }
}
```
위의 코드에서처럼 Config를 2개 만들어 필터 값들의 작동을 확인해 보겠습니다.<br><br>
아래의 사진은 Config에서 설정한 2개의 보안 설정이 잘 적용되었는지 FilterChainProxy에 블록을 걸어 값을 확인해 보았습니다.
![필터 체인이 가지고 있는 값 확인](:/inflearn_spring_security_learn/2s/6/filter_chains.jpg){:data-align="center"}
사진을 보면 requestMatcher에 두 값들이 정상적으로 들어가 있는 것을 확인할 수 있고, 각각의 보안 정책의 차이에 따라 필터의 개수도 다른것을 확인할 수 있습니다.<br>
![13개의 filterchain 값](:/inflearn_spring_security_learn/2s/6/securityconfig1.jpg){:data-align="center"}
위는 처음 SecurityConfig의 필터들입니다. 베이직 인증을 도입해 베이직 필터가 있는 모습을 확인할 수 있습니다.<br>
![15개의 filterchain 값](:/inflearn_spring_security_learn/2s/6/securityconfig2.jpg){:data-align="center"}
위는 SecurityConfig2의 필터들입니다. formLogin 정책으로 인해 Basic 필터 대신 필터 6, 7, 8번에 formLogin관련 필터가 있는 것을 확인할 수 있습니다.<br>
![루트 페이지로의 이동](:/inflearn_spring_security_learn/2s/6/root_page.jpg){:data-align="center"}
루트 페이지로 이동 시 모든 사용자의 접근을 허용했기 때문에 자연스럽게 결과가 출력되는 것을 확인할 수 있습니다.<br>
![어드민 페이지 접근시 Basic 인증 출력](:/inflearn_spring_security_learn/2s/6/admin_page.jpg){:data-align="center"}
어드민 페이지 접근 시 인증된 사용자 접근만 허용했고, Basic 인증을 적용했기 때문에 Basic 로그인 화면이 출력되는 것을 확인할 수 있습니다.<br>
![어드민 페이지 정상 접근](:/inflearn_spring_security_learn/2s/6/admin_page.jpg){:data-align="center"}
인증을 완료하면 정상적으로 화면 출력이 됩니다.

### Authentication
#### Authentication이란?
사용자가 누구인지를 증명하는 것입니다.


### 참고
#### Order 순서 설정의 중요성
**SecurityConfig.java**{:data-align="center"}
```java
@Configuration
@EnableWebSecurity
@Order(1)
public class SecurityConfig {
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        // 위의 코드와 동일
    }
}
@Configuration
@Order(0)
class SecurityConfig2 {
    @Bean
    public SecurityFilterChain filterChain2(HttpSecurity http) throws Exception {
        // 위 코드와 동일
    }
}
```
필터 초기화와 다중 보안 설정의 실제 코드를 보시면 Config가 0순위로 설정 되있고, Config2가 1순위로 선택이 되어 있는 것을 확인하실 수 있습니다.<br>
하지만 이를 위 코드와 같이 변경할 경우 모든 경로에 관해 모든 요청을 수락한다는 코드가 /admin/** 보안 설정보다 앞에 있으므로 /admin/**에 설정된 인가 정책을 무시하고, 모든 경로에 대해 모든 요청을 수락하게 됩니다.<br>
이는 인가 정책 설정의 작은 범위에 경로를 더 위에 둬야 인가 정책이 정상적으로 처리 되는 것과 동일한 것입니다.


### 출처
1. [학습중인 강의](https://www.inflearn.com/course/%EC%BD%94%EC%96%B4-%EC%8A%A4%ED%94%84%EB%A7%81-%EC%8B%9C%ED%81%90%EB%A6%AC%ED%8B%B0)