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
img: ":post_spring_security.jpg"
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

버츄얼 유튜버 커뮤니티 사이트 제작 과정 중 스프링에 대한 이해가 부족하다는 판단하에 인프런에서 관련 영상을 검색해 듣게되었습니다.
출처는 인프런의 스프링 시큐리티 - [Spring Boot 기반으로 개발하는 Spring Security](https://www.inflearn.com/course/%EC%BD%94%EC%96%B4-%EC%8A%A4%ED%94%84%EB%A7%81-%EC%8B%9C%ED%81%90%EB%A6%AC%ED%8B%B0)강의를 바탕으로 이 블로그를 제작하고 있습니다.

<!-- outline-end -->

### 제작 환경
이 강의를 들으며 테스트하는 환경은 Spring 2.7.3 버전을 사용하고, Java는 17버전을 사용합니다.
의존도는 Spring Security, Spring Thymeleaf, Spring Web 의 라이브러리를 참조했습니다.
제작 환경은 IntelliJ입니다.

### 인증 API - 사용자 정의 보안 기능 구현 설명
우선 스프링 시큐리티의 웹 보안 기능 초기화 및 설정을 하는 WebSecurityConfigurerAdapter라는 클레스를 보통 사용자 정의 보안 설정 클래스인 SecurityConfig에서 상속 받아 사용합니다.
이 클레스는 HttpSecurity라는 세부적인 보안 기능을 설정할 수 있는 API를 제공하는 클레스를 생성을 합니다.

### 인증 API - SecurityConfig 설정
현재 사람들이 가장 많이 사용하는 SecurityConfig의 사용 방법을 살펴보고, 스프링 2.7 이후의 SecurityConfig 제작에 관해 작성합니다.
#### 스프링 2.6 이전의 SecurityConfig
{% highlight Java Spring %}
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
위에서 설명했던 바와 같이 SecurityConfig가 WebSecurityConfigurerAdapter을 상속 받고, configure는 Override하고, HttpSecurity를 파라미터로 받는 메소드입니다.
이 안에서 인가나 인증 API를 작성해 사용합니다.
하지만 이는 [Spring 공식 문서](https://spring.io/blog/2022/02/21/spring-security-without-the-websecurityconfigureradapter)에도 나와있듯이 spring security 5.7이상과 SpringBoot 2.7 이상의 버전에서 사용을 권장하지 않고, 삭제 예약인 Deprecated가 걸려있습니다.
그 이유는 스프링 프레임워크 개발자들이 사용자들의 component-based security configuration로의 전환을 권하기 때문입니다.

#### 스프링 2.7 이후의 SecurityConfig


### 페이지 설계


### 데이터베이스 설계

### API 명세서

