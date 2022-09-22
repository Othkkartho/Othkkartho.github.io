---
# multilingual page pair id, this must pair with translations of this page. (This name must be unique)
lng_pair: springlearn
title: 인프런 스프링 시큐리티 강의 학습-1

# post specific
# if not specified, .name will be used from _data/owner/[language].yml
author: Othkkartho's Workshop
# multiple category is not supported
category: Spring
# multiple tag entries are possible
tags: [Spring, inflearn, spring security class]
# thumbnail image for post
img: ":/inflearn_spring_security_learn/post_spring_security.PNG"
# disable comments on this page
#comments_disable: true

# publish date
date: 2022-09-20 10:00:00 +0900

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

버츄얼 유튜버 커뮤니티 사이트 제작 과정 중 스프링에 대한 이해가 부족하다는 판단하에 인프런에서 관련 영상을 검색해 듣게되었습니다.<br>
출처는 인프런의 스프링 시큐리티 - [Spring Boot 기반으로 개발하는 Spring Security](https://www.inflearn.com/course/%EC%BD%94%EC%96%B4-%EC%8A%A4%ED%94%84%EB%A7%81-%EC%8B%9C%ED%81%90%EB%A6%AC%ED%8B%B0)강의를 바탕으로 이 블로그를 제작하고 있습니다.<br>
강의의 세션 1의 1, 2번 강의내용에 대한 정리입니다.

<!-- outline-end -->

### 제작 환경
이 강의를 들으며 테스트하는 환경은 Spring 2.7.3 버전을 사용하고, Java는 17버전을 사용합니다.<br>
의존도는 Spring Security, Spring Thymeleaf, Spring Web 의 라이브러리를 참조했습니다.<br>
제작 환경은 IntelliJ와 Gradle을 사용합니다.

## 프로젝트 구성 및 의존성 추가
### 프로젝트 구성
우선 간단하게 루트에 접속할 수 있는 컨트롤러를 만들어 보겠습니다.<br>
의존성을 Spring Web만 추가하고, 사이트에 들어갈 수 있는 간단한 컨트롤러를 만든다면.
{% highlight Spring %}
@RestController
public class SecurityController {
    @GetMapping("/")
    public String index() {
        return "home";
    }
}
{% endhighlight %}
위와 같이 만들 수 있습니다. 그럼 localhost:8080으로 접속해본다면 루트에 리턴한 문자값인 home이 나오게 됩니다.<br>
여기에 Spring Security의존성을 추가해 보겠습니다.<br>
의존성을 추가하고, 홈페이지에 접속을 하면 아래의 사진과 같이 로그인 입력창이 출력됩니다.<br>
![로그인 페이지](:/inflearn_spring_security_learn/1s/1/login.PNG){:data-align="center"}
그럼 기본으로 제공해주는 ID인 user와 무작위 생성되는 비밀번호를 치면
![비밀번호](:/inflearn_spring_security_learn/1s/1/password.PNG){:data-align="center"}
처음 접속했을 때 보여지는 home 화면이 보여지게 됩니다.
![홈 화면](:/inflearn_spring_security_learn/1s/1/home.PNG){:data-align="center"}

## 사용자 정의 보안 기능 구현
### Security 의존성의 동작
위의 사례를 보면 알 수 있듯 개발자가 의존성을 추가하게 된다면 다음과 같은 일들이 일어나게 됩니다.
- 서버를 작동하면 Spring Security의 초기화 작업과 보안 설정이 이루어집니다.
- 별도의 설정이나 구현을 하지 않아도 기본적인 웹 보안 기능이 작동하게 됩니다.
    - 모든 자원을 로그인과 같은 인증을 통해 접근이 가능해집니다.
    - 인증방식은 Form Login 방식과, httpBasic 방식이 가능합니다.
    - 기본 로그인 페이지를 제공합니다.
    - 기본 계정 1개를 제공해 줍니다. ID: user, Password: 콘솔에 출력되는 무작위 문자열

하지만 위의 방식에는 문제점 또한 존재합니다.
- 계정 추가, 권한 부여 등의 추가적으로 보다 높은 수준의 작업을 실행할 수 없습니다.
- 기본적인 보안 기능은 보안에 취약하며, 시스템이 원하는 보안기능을 제공하지 않아 사용자의 편의를 해칠 가능성이 있습니다.

### 인증 API - 사용자 정의 보안 기능 구현 설명
우선 스프링 시큐리티의 웹 보안 기능 초기화 및 설정을 하는 WebSecurityConfigurerAdapter라는 클레스를 보통 사용자 정의 보안 설정 클래스인 SecurityConfig에서 상속 받아 사용합니다.<br>
이 클레스는 HttpSecurity라는 세부적인 보안 기능을 설정할 수 있는 API를 제공하는 클레스를 생성을 합니다.

### 인증 API - SecurityConfig 설정
현재 사람들이 가장 많이 사용하는 SecurityConfig의 사용 방법을 살펴보고, 스프링 2.7 이후의 SecurityConfig 제작에 관해 작성합니다.

#### 스프링 2.6 이전의 SecurityConfig
{% highlight Spring %}
@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
        .authorizeRequests()
        .anyRequest().authenticated()
        .and()
        .formLogin();
    }
}
{% endhighlight %}
위에서 설명했던 바와 같이 SecurityConfig가 WebSecurityConfigurerAdapter을 상속 받고, configure는 Override하고, HttpSecurity를 파라미터로 받는 메소드입니다.<br>
이 안에서 인가나 인증 API를 작성해 사용합니다.<br>
하지만 이는 [Spring 공식 문서](https://spring.io/blog/2022/02/21/spring-security-without-the-websecurityconfigureradapter)에도 나와있듯이 spring security 5.7이상과 SpringBoot 2.7 이상의 버전에서 사용을 권장하지 않고, 삭제 예약인 Deprecated가 걸려있습니다.<br>
그 이유는 스프링 프레임워크 개발자들이 사용자들의 component-based security configuration로의 전환을 권하기 때문입니다.

#### 스프링 2.7 이후의 SecurityConfig
좀 더 자세히 살펴본 후 어떻게 변환할 수 있는지 알아보도록 하겠습니다.<br>
우선 현재 제가 사용하는 Spring 버전 2.7.3에서 WebSecurityConfigurerAdapter을 호출하면
![WebSecurityConfigurerAdapter Extend](:/inflearn_spring_security_learn/1s/1/home.PNG){:data-align="center"}
위의 그림과 같이 취소선 표시가 뜨며 더이상 사용되지 않는 파일임을 알려줍니다.
![WebSecurityConfigurerAdapter Deprecated](:/inflearn_spring_security_learn/1s/1/deprecated_security.PNG){:data-align="center"}
조금 더 자세히 살펴보면 아래의 사진과 같이 Deprecated가 작성되어 있는 것을 확인하실 수 있습니다.

이제 그럼 어떻게 위 클래스를 상속받지 않고, Security사용을 할 수 있는지를 설명드리도록 하겠습니다.<br>
기존 방식
{% highlight Spring %}
@Override
protected void configure(HttpSecurity http) throws Exception {
    http
    .authorizeRequests()
    .anyRequest().authenticated()
    .and()
    .formLogin();
}
{% endhighlight %}

변경된 방식
{% highlight Spring %}
@Bean
public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
    http
    .authorizeRequests()
    .anyRequest().authenticated()
    .and()
    .formLogin();

    return http.build();
}
{% endhighlight %}
으로 작성할 수 있습니다.

아직 강의에는 나오지 않았지만, 예외를 처리하는 WebSecurity를 변수로 받는 configure의 경우<br>
기존 방식
{% highlight Spring %}
@Override
protected void configure(WebSecurity web) throws Exception {
    web.ignoring().antMatchers("/images/**", "/js/**", "/css/**"); 
}
{% endhighlight %}

변경된 방식
{% highlight Spring %}
@Bean
public WebSecurityCustomizer webSecurityCustomizer() {
    return (web) -> web.ignoring().antMatchers("/images/**", "/js/**", "/css/**");
}
{% endhighlight %}
로 변경할 수 있습니다.

아직 삭제된 것은 아니기 때문에 각 방식의 간단한 코드와 결과 또한 아래 사진에 첨부하겠습니다.<br>
기존 방식 코드
![Override code](:/inflearn_spring_security_learn/1s/1/override_code.PNG){:data-align="center"}

기존 방식 결과
![Override result](:/inflearn_spring_security_learn/1s/1/override_result.PNG){:data-align="center"}

변경된 방식 코드
![Bean code](:/inflearn_spring_security_learn/1s/1/Bean_code.PNG){:data-align="center"}

변경된 방식 결과
![Bean code](:/inflearn_spring_security_learn/1s/1/Bean_result.PNG){:data-align="center"}

확인해 보시면 두 방식 모두 SecurityConfig에 직접 작성한 코드 부분의 Breakpoint에서 걸려 작성한 코드가 정상적으로 실행되었다는 것을 알 수 있습니다.<br>
결과 사진 또한 모두 동일하게 출력되어 정상적으로 실행 되었음을 알려줍니다.<br>
현재 예외 사항 처리가 필요하지 않아 이 포스트에서는 작성하지 않았지만 예외 처리가 필요할 때 예외처리 코드의 변경 또한 작성하도록 하겠습니다. 이상으로 세션 1의 1, 2번 강의의 정리 포스트를 마치겠습니다. 감사합니다.

## 참고
### Form 로그인 방식
Form 로그인 방식의 경우 
- 사용자가 서버에 특정 자원의 접근을 요청하면 그 요청의 인증 여부를 판단해 인증이 되지 않았다면 로그인 페이지를 사용자에게 return하게 됩니다.
- 그럼 그 로그인 페이지에 유저ID와 비밀번호를 입력해 로그인을요청하면 PostMapping으로 해당 데이터가 서버에 전송됩니다.
- 해당 정보가 일치할 경우 인증을 완료하고, 접속을 허용하게 됩니다.

### HttpBasic 방식
Http Basic은 Http 프로토콜에서 정의한 기본 인증입니다.
- 사용자가 인증을 받지 않은 상태로 요청을 하게 되면 서버에서는 사용자에게 401 Unauthorized 오류 응답과 함께 WWW-Authenticate 헤더를 기술해 인증을 어떤 방식으로 해야하는지에 대한 설명을 동봉하여 보냅니다.
- 사용자는 아이디와 패스워드를 Base64로 인코딩한 문자열을 Authorization 헤더에 담아 요청합니다.
- 서버에서 인증이 수락되면 정상적인 상태 코드를 반환합니다.

### Form 로그인 VS HttpBasic
차이점은 Http Basic은 세션방식의 인증이 아닌 서버로부터 요청받은 인증방식대로 구성한 다음 헤더에 기술해서 서버로 보내는 방식을 취합니다.
반면 Form 인증방식은 서버에 해당 사용자의 Session 상태가 유효한지를 판단해 인증처리를 합니다.

### Spring Security 기존 아이디 비밀번호 변경
기존 무작위로 생성되는 비밀번호나, 기존 ID인 user를 변경하고 싶다면 application.properties에서 다음과 같은 코드를 작성하면 변경됩니다.

{% highlight Spring %}
spring.security.user.name=user
spring.security.user.password=1234
{% endhighlight %}

## 출처
1. [학습중인 강의](https://www.inflearn.com/course/%EC%BD%94%EC%96%B4-%EC%8A%A4%ED%94%84%EB%A7%81-%EC%8B%9C%ED%81%90%EB%A6%AC%ED%8B%B0)  
2. [Http Basic 인증과 Form 인증의 차이](https://www.inflearn.com/questions/250472)
3. [WebSecurityConfigurerAdapter의 Deprected 처리의 해결책](https://yooooonnf.tistory.com/3)