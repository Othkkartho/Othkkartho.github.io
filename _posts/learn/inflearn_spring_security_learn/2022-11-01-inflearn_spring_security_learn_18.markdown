---
# multilingual page pair id, this must pair with translations of this page. (This name must be unique)
lng_pair: springlearn
title: 인프런 스프링 시큐리티 강의 학습-18

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
date: 2022-11-01 22:00:00 +0900

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

[Method 보안 방식 개요](#method-방식-개요), [어노테이션 권한 설정](#어노테이션-권한-설정---preauthorize-postauthorize-secured-rolesallowed)을 정리한 포스트입니다.
--------------------------------------

출처는 인프런의 스프링 시큐리티 - [Spring Boot 기반으로 개발하는 Spring Security](https://www.inflearn.com/course/%EC%BD%94%EC%96%B4-%EC%8A%A4%ED%94%84%EB%A7%81-%EC%8B%9C%ED%81%90%EB%A6%AC%ED%8B%B0)강의를 바탕으로 이 포스트를 작성하고 있습니다.<br>
강의의 세션 6의 1,3번 강의내용에 대한 정리입니다.<br><br>
코드는 [깃허브](https://github.com/Othkkartho/SpringSecurityLearn/tree/ch6.3)에 있습니다.

<!-- outline-end -->

### Method 방식 개요
서비스 계층의 인가처리 방식은
- 화면 메뉴 단위가 아닌 기능 단위로 인가처리 합니다.
- 메소드 처리 전.후로 보안 검사 수행하여 인가처리를 합니다.
- URL 방식에 비해 세부적이고 구체적으로 인가처리가 되도록 합니다.
- URL 방식은 Filter 기반인데 반해 Method 방식은 [AOP 기반](#aop)으로 프록시와 어드바이스로 메소드 인가처리를 수행합니다.
- 보안 설정 방식은 어노테이션 권한 설정 방식과 맵 기반 권한 설정 방식으로 나뉩니다.

### 어노테이션 권한 설정 - @PreAuthorize, @PostAuthorize, @Secured, @RolesAllowed
- @PreAuthorize, @PostAuthorize는 [SpEL](#spel)을 지원해 어떤 조건에 따라서 권한을 처리할 수 있게 합니다. PrePostAnnotationSecurityMetadataSource가 담당합니다.
- @Secured, @RolesAllowed는 [SpEL](#spel)을 지원하지 않으며 각각 SecuredAnnotationSecurityMetadataSource, Jsr250MethodSecurityMetadataSource가 담당합니다.
- EnableGlobalMethodSecurity(prePostEnabled = true, securedEnabled = true) 를 선언해야 어노테이션 방식들이 사용됩니다.

#### 실제 코드
**AopSecurityController.java**{:data-align="center"}
```java
@Controller
public class AopSecurityController {
    @GetMapping("/preAuthorize")
    @PreAuthorize("hasRole('ROLE_USER') and #account.username == principal.username")   // 1
    public String preAuthorize(AccountDto account, Model model, Principal principal) {
        model.addAttribute("method", "Success @PreAuthorize");

        return "/aop/method";
    }
}
```
1. User 권한과, 접속한 username과 로그인 후 인증된 사용자의 username이 동일하다면 접근이 가능하다는 의미입니다.

**home.html**{:data-align="center"}
```html
<a th:href="@{/preAuthorize(username='user')}" style="margin:5px;" class="nav-link text-primary">@메소드보안</a>
```

**method.html**{:data-align="center"}
```html
<!DOCTYPE html>
<html lang="ko" xmlns:th="http://www.thymeleaf.org">
<head th:replace="layout/header::userHead">
    <title>method</title>
</head>
<body>
<div th:replace="layout/top::header"></div>
<div class="container">
    <div class="row align-items-start">
        <nav class="col-md-2 d-none d-md-block bg-light sidebar">
            <div class="sidebar-sticky">
                <ul class="nav flex-column">
                    <li class="nav-item">
                        <div style="padding-top:10px;" class="nav flex-column nav-pills" aria-orientation="vertical">
                            <a th:href="@{/}" style="margin:5px;" class="nav-link text-primary">대시보드</a>
                            <a th:href="@{/mypage}" style="margin:5px;" class="nav-link text-primary">마이페이지</a>
                            <a th:href="@{/messages}" style="margin:5px;" class="nav-link text-primary">메시지</a>
                            <a th:href="@{/config}" style="margin:5px;" class="nav-link text-primary">환경설정</a>
                            <a th:href="@{/preAuthorize(username='user')}" style="margin:5px;" class="nav-link text-primary">@메소드보안</a>
                        </div>
                    </li>
                </ul>
            </div>
        </nav>
        <div style="padding-top:50px;"  class="col">
            <div class="container text-center">
                <h1 class="text-primary" th:text="${method}">Method</h1>
            </div>
        </div>
    </div>
    <div th:replace="layout/footer::footer"></div>
</div>
</body>
</html>
```

**SecurityConfig.java**{:data-align="center"}
```java
@EnableGlobalMethodSecurity(securedEnabled = true, prePostEnabled = true)
```
각 어노테이션들을 정상적으로 실행하기 위해 사용할 설정들을 true로 만들어 줍니다.

#### 실제 실행 화면
**GlobalMethodSecurityConfiguration.java의 설정 boolean 값들**{:data-align="center"}
![SecurityConfig에서 true로 변경한 값들이 정상적으로 적용됨](:/inflearn_spring_security_learn/5s/18/boolean.JPG){:data-align="center"}
SecurityConfig에서 true로 변경한 설정들이 정상적으로 true로 설정되고 설정을 하지 않은 값은 false가 되는 것을 확인할 수 있습니다.

**GlobalMethodSecurityConfiguration.java의 저장된 MetadataSource**{:data-align="center"}
![SecurityConfig에서 true로 설정한 값들이 MetadataSource에 저장됨](:/inflearn_spring_security_learn/5s/18/sources.JPG){:data-align="center"}
위에서 true로 변경된 설정들의 MetadataSource가 sources에 정상적으로 저장된 것을 확인할 수 있습니다.   <br>

그 후 DelegatingMethodSecurityMetadataSource 클래스에서 `this.attributeCache.put(cacheKey, attributes);` 로 해당 메서드와 클래스를 키로 해서 권한 정보를 저장합니다.

**AbstractSecurityInterceptor에서 MetadataSource 구현체 저장**{:data-align="center"}
![정상적으로 위 두개의 구현체가 저장되어 있음](:/inflearn_spring_security_learn/5s/18/metadatasource.JPG){:data-align="center"}
위에서 설정한 2개의 구현체 클래스를 가지고 현재 호출하고자 하는 메서드의 권한을 가져옵니다. 사진은 저장되어있는 사진입니다.

**가져온 권한 정보**{:data-align="center"}
![정상적으로 설정한 권한을 가져 옴](:/inflearn_spring_security_learn/5s/18/attributes.JPG){:data-align="center"}
정상적으로 설정했던 권한 설정을 가져온 것을 확인할 수 있습니다.   <br>

이후로 확인해 보면 모든 기능들이 정상적으로 작동함을 확인할 수 있습니다.

### 참고
#### AOP
AOP는 관점 지향 프로그래밍(aspect-oriented programming)의 약어로써 다른 관심사에 영향을 미치는 프로그램 에스펙트인 횡단 관심사(cross-cutting concerns)의 분리를 허용함으로써 모듈성을 증가시키는 것이 목적인 프로그래밍 패러다임입니다.   
이는 로깅, 트랜잭션 등 여러 모듈에서 공통적으로 사용하는 기능을 분리해 관리할 수 있음을 의미합니다.   
이러한 스프링의 특징들 중 하나이기도 합니다. 스프링에서는 반복할당을 줄이기 위해 포인터를 대신해 스프링 어노테이션을 사용해 여러 기능들을 구현하는 것을 AOP의 일환이라고 볼 수 있습니다.
[참고할 만한 자료](https://yadon079.github.io/2021/spring/spring-aop-core)

#### SpEL
Spring Expression Language, 스프링 표현식 언어라고 번역되며 런타임에 객체 그래프를 쿼리하고 조작할 수 있도록 지원하는 강력한 표현 언어입니다.SpEL은 API를 기반으로 다른 표현 언어 구현들이 필요할 때 통합될 수 있습니다. SpEL은 Spring 내에서 표현 평가의 기초 역할을 하지만 Spring과 직접 연계되지 않고 독립적으로 사용될 수 있습니다.   
일반적인 용도로는 Bean 정의를 위한 표현 지원 섹션에 나와있는 XML이나, 주석이 달린 Bean 정의를 만들기 위해 SpEL을 통합하는 것이 있습니다.

#### 받는 값 확인 위치
public String preAuthorize(AccountDto account, Model model, Principal principal) 이 받는 값이 들어오는 위치는   <br>
AbstractSecurityInterceptor의 `this.accessDecisionManager.decide(authenticated, object, attributes);` 입니다.

### 출처
1. [학습중인 강의](https://www.inflearn.com/course/%EC%BD%94%EC%96%B4-%EC%8A%A4%ED%94%84%EB%A7%81-%EC%8B%9C%ED%81%90%EB%A6%AC%ED%8B%B0)
2. [관점 지향 프로그래밍 - 위키백과](https://ko.wikipedia.org/wiki/%EA%B4%80%EC%A0%90_%EC%A7%80%ED%96%A5_%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D)
3. [스프링 공식 문서 - 8. Spring Expression Language (SpEL)](https://docs.spring.io/spring-framework/docs/3.2.x/spring-framework-reference/html/expressions.html)