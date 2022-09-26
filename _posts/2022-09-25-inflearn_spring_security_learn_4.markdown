---
# multilingual page pair id, this must pair with translations of this page. (This name must be unique)
lng_pair: springlearn
title: 인프런 스프링 시큐리티 강의 학습-4

# post specific
# if not specified, .name will be used from _data/owner/[language].yml
author: Othkkartho's Workshop
# multiple category is not supported
category: Summary of Infron Spring Security Lectures
# multiple tag entries are possible
tags: [Spring, inflearn, spring security learn]
# thumbnail image for post
img: ":/inflearn_spring_security_learn/post_spring_security.png"
# disable comments on this page
#comments_disable: true

# publish date
date: 2022-09-25 20:00:00 +0900

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

익명사용자 인증 필터, 동시 세션제어, 세션 고정 보호, 세션 정책, 세션 제어 필터에 관해 포스트했습니다.
----------------------------------------------------------------------------------------------

출처는 인프런의 스프링 시큐리티 - [Spring Boot 기반으로 개발하는 Spring Security](https://www.inflearn.com/course/%EC%BD%94%EC%96%B4-%EC%8A%A4%ED%94%84%EB%A7%81-%EC%8B%9C%ED%81%90%EB%A6%AC%ED%8B%B0)강의를 바탕으로 이 포스트를 작성하고 있습니다.<br>
강의의 세션 1의 8,9,10번 강의내용에 대한 정리입니다.

<!-- outline-end -->

### 인증 API - AnonymousAuthenticationFilter
#### 사용 이유
앞에까지 우리는 사용자가 인증을 받아 Session에 유저 객체를 저장하고, 사용자가 자원에 접근하려 하면 세션에 저장된 유저 객체가 null인지 아닌지를 판단합니다.<br>
만약 null이면 인증을 받지 않았다고 판단해 자원에 접근하지 못하게 하고, null이 아니면 접근하게 합니다.<br>
하지만 이 필터는 인증을 받지 않은 사용자를 null로 처리하는 것이 아니라 익명 사용자용 인증 객체를 만들어 처리합니다.

#### 필터의 진행 사항
1. 필터가 사용자에게 요청을 받으면 인증 객체가 존재하는지 아닌지 여부를 판단합니다.
2. 인증을 받았다면 다른 필터로 넘어가고, 인증이 되지 않았다면 익명 사용자용 인증 토큰인 AnonymousAuthenticationToken을 생성합니다.
3. 생성 후 SecurityContext객체 안에 익명사용자용 인증 토큰을 저장합니다.

#### 필터 설명
- 익명사용자 인증 처리 필터입니다.
- 익명사용자와 인증 사용자를 구분해 처리하기 위한 용도로 사용됩니다.
- 화면에서 인증 여부를 구현할 때 isAnonymous()와 isAuthenticated()로 구분해 사용합니다.
- 인증객체를 세션에 저장하지 않습니다.

### 동시 세션 제어
#### 최대 세션 허용 개수 관리
만약 사용자가 최대 세션 허용 개수를 초과한다면 2가지의 방법으로 이를 처리할 수 있습니다.
1. 첫번째로 헌재 사용자가 요청한 세션이 아닌 그 전에 받았던 세션을 만료설정을 하는 방법이 있습니다.
2. 두번째는 사용자가 과거 생성된 세션이 있다면 현재 다른 세션을 생성하려고 할 때 인증 예외를 발생시켜 인증 실패를 방법이 있습니다.

#### 실행 코드
```java
http.sessionManagement()            // 1
    .maximumSessions(1)             // 2
    .maxSessionsPreventsLogin(ture) // 3
    .invalidSessionUrl("/invalid")  // 4
    .expiredUrl("/expired")         // 5
```
1. 세션 관리 기능을 작동시킵니다.
2. 최대 허용 가능 세션 수를 설정합니다. -1을 입력할 경우 무제한 로그인 세션을 허용합니다.
3. 동시 로그인 차단을 설정해 현재 사용자 인증 실패 전략을 사용합니다. Default는 false로 기존 세션을 만료시켜 이전 사용자 세션을 만료시키는 전략을 사용합니다.
4. 세션이 유효하지 않을 때 이동 할 페이지를 설정합니다.
5. 세션이 만료된 경우 이동할 페이지를 설정합니다.

#### 실제 실행 절차
서버에 사용자의 세션이 2개 생성된 것은 맞지만 첫번째 사용자가 서버 자원에 접근하려고 하면 서버는 사용자의 세션이 만료 되었는지 아닌지 확인하는 ConcurrentSessionFilter가 체크를 합니다. 만약 세션이 만료된것을 확인하면 사용자의 세션을 실제로 만료시키게 되고, 사용자 세션 만료를 응답을 보내게됩니다.

### 세션 고정 보호
#### 세션 고정 공격이란?
1. 공격자가 서버에 접속을 하면, 서버는 공격자에게 SessionId를 발급을 합니다.
2. 공격자는 사용자에게 자신이 발급받은 세션쿠키를 심어놓습니다.
3. 그럼 사용자는 공격자가 심어놓은 세션 쿠키를 가지고 서버에 로그인을 시도해 로그인에 성공합니다.
4. 그럼 사용자와 공격자 모두 공격자의 쿠키 값으로 인증되었기 때문에 서버는 둘 모두 세션을 공유하여 공격자는 인증을 하지 않고도 SessionId만 가진체 서버에 접근해 인증받은 사용자와 동일한 정보를 공유하게 됩니다.

#### 세션 고정 보호
위와 같은 공격의 위험을 막기위해 스프링 시큐리티는 세션 고정 보호를 제공합니다.
방법은 사용자가 어떤 세션쿠키로 로그인을 시도하던지에 관계 없이 로그인을 시도해 인증을 요청하게 되면 새로운 세션을 생성하고, 쿠키도 새롭게 생성됩니다.<br>
그럼 공격자는 본인이 가지고있는 세션 아이디를 가지고 사용자와 공유할 수 없게 됩니다.

#### 실행 코드
**SecurityConfig.java**{:data-align="center"}
```java
http.sessionManagement()
    .sessionFixation().changeSessionId()    // 1
```
1. 세션 고정 보호를 사용하는 기본값입니다.
    - changeSessionId는 Servlet 3.1 이상에서 기본값입니다. 사용자가 인증에 성공하면 세션은 그대로 두고, 세션 아이디만 변경됩니다.
    - migrateSession은 Servlet 3.1 이하에서 기본값입니다. 사용자가 인증에 성공하면 세션도 새로 만들고, 세션 아이디도 새로 만듭니다.
    - changeSessionId와 migrateSession은 이전의 세션에 설정한 속성 값들을 그대로 사용할 수 있도록 지원합니다.
    - newSession은 사용자가 인증에 성공하면 세션이 새롭게 생성되고, 세션 아이디도 새로 만들지만, 이전 세션에 설정한 속성 값들을 사용하지 못합니다.
    - none은 세션을 그대로 두고, 세션 아이디도 변경되지 않습니다.
- 위와 같은 코드로 따로 설정하지 않더라도 스프링 시큐리티가 기본적으로 초기화 되면서 기본값이 작동하도록 설정되어있습니다.

#### 실제 공격 시뮬레이션
**SecurityConfig.java**{:data-align="center"}
```java
http.sessionManagement()
    .sessionFixation().none();
```
실제 코드를 적용해 만약 세션 고정 보호가 되지 않는다면 어떤 일이 일어나는지 실제로 확인하도록 하겠습니다.<br>
먼저 각 브라우저의 세션 쿠키를 확인해보겠습니다.<br>
엣지의 세션 쿠키 값입니다.
![엣지 변경 전 세션 쿠키](:/inflearn_spring_security_learn/1s/4/edge_cookie.PNG){:data-align="center"}
크롬의 세션 쿠키 값입니다.
![크롬 변경 전 세션 쿠키](:/inflearn_spring_security_learn/1s/4/chrome_cookie.PNG){:data-align="center"}
두 브라우저의 세션 쿠키의 값이 다른 것을 확인할 수 있습니다.<br>
그 후 크롬 브라우저의 세션 쿠키로 엣지의 쿠키를 변경해 보겠습니다.
![엣지 변경 후 세션 쿠키](:/inflearn_spring_security_learn/1s/4/edge_cookie_2.PNG){:data-align="center"}
엣지의 세션 쿠키 값이 크롬의 쿠키 값과 같아진 것을 확인할 수 있습니다.<br>
그 후 엣지에서 서버에 로그인을 하고 크롬 브라우저에서 로그인을 하지 않은채 루트로 접근을 시도해 보았습니다.
![크롬의 서버 접근](:/inflearn_spring_security_learn/1s/4/chrome_direct.PNG){:data-align="center"}
![크롬의 서버 접근 완료](:/inflearn_spring_security_learn/1s/4/chrome_direct_2.PNG){:data-align="center"}
위 사진에서 보시면 알 수 있듯이 인증을 하지 않고도 서버에 접근이 된것을 알 수 있습니다.<br>
이 후 세션 옵션을 changeSessionId로 변경해 확인해보면 공격자가 쿠키를 같게 맞춰도 서버에 접근이 안되는 것을 확인할 수 있습니다.

### 세션 정책
#### 실행 코드
**SecurityConfig.java**{:data-align="center"}
```java
http.sessionManagement()
    .sessionCreationPolicy(SessionCreationPolicy.If_Required)   // 1
```
1. 스프링 시큐리티의 세션 정책옵션을 설정할 수 있습니다.
    - Always는 스프링 시큐리티가 항상 세션을 생성합니다.
    - If_Required는 스프링 시큐리티가 필요 시 세션을 생성합니다. 이 설정이 기본값입니다.
    - Never은 스프링 시큐리티가 세션을 생성하지 않지만, 이미 존재하면 사용합니다.
    - Stateless는 스프링 시큐리티가 생성하지 않고 존재해도 사용하지 않습니다. 이 방식은 JWT와 같은 세션을 사용하지 않는 인증 방식을 사용하고자 할 때 이 정책을 사용합니다.

### SessionManagementFilter
#### 전 포스트의 정리
- 세션 관리는 인증 시 사용자의 세션정보를 등록, 조회, 삭제 등의 세션 이력을 관리합니다.
- 동시적 세션 제어는 동일 계정으로 접속이 허용되는 최대 세션수를 제한합니다.
- 세션 고정 보호는 인증 할 때마다 세션 쿠키를 새로 발급해 공격자의 쿠키 조작을 방지합니다.
- 세션 생성 정책은 Always, If_Required, Never, Stateless 4가지가 있습니다.

### ConcurrentSessionFilter
#### 필터의 간단한 설명
ConcurrentSessionFilter는 매 요청 마다 현재 사용자의 세션 만료 여부를 체크합니다.<br>
만약 세션이 만료되었을 경우 즉시 만료 처리를 합니다.<br>
session.isExpired() == true라면 로그아웃 처리를 하고, 즉시 오류 페이지를 응답합니다.

#### 동시적 세션 처리를 할 때 처리 순서
1. 사용자가 로그인을 했지만 이미 세션이 생성이 되어 최대 세션 허용 개수를 초과했을 경우 session.expireNow()로 이전 사용자의 세션을 만료시킵니다.
2. 그 후 이전 사용자가 서버에 접근한다면 이미 이 사용자의 세션은 만료되도록 설정되어있는 상태입니다.
3. ConcurrentSessionFilter가 매 요청마다 사용자의 세션의 만료 여부를 체크합니다. 확인은 session.expireNow()를 참조합니다.
4. 참조 결과 만료가 설정이 되어있기 때문에 필터는 로그아웃을 하고, 오류 페이지를 응답합니다.

### 참고
#### 자바 서블릿(Java Servlet)에 관하여
세션 고정 보호에서 언급되었던 자바 서블릿에 관해 잠깐 무엇인지 확인해 보겠습니다.<br>
자바 서블릿에 관해 설명한 위키백과에 따르면 아래와 같이 나와있습니다.
자바 서블릿(Java Servlet)은 자바를 사용하여 웹페이지를 동적으로 생성하는 서버측 프로그램 혹은 그 사양을 말하며, 흔히 "서블릿"이라 불립니다. 자바 서블릿은 웹 서버의 성능을 향상하기 위해 사용되는 자바 클래스의 일종입니다. 서블릿은 JSP와 비슷한 점이 있지만, JSP가 HTML 문서 안에 Java 코드를 포함하고 있는 반면, 서블릿은 자바 코드 안에 HTML을 포함하고 있다는 차이점이 있습니다.<br>
자바 서블릿은 자바 EE 사양의 일부분으로, 주로 이 기능을 이용하여 쇼핑몰이나 온라인 뱅킹 등의 다양한 웹 시스템이 구현되고 있습니다.<br>
비슷한 기술로는 펄 등을 이용한 CGI, PHP를 아파치 웹 서버 프로세스에서 동작하게 하는 mod_php, 마이크로소프트사의 IIS에서 동작하는 ASP 등이 있습니다. CGI는 요청이 있을 때마다 새로운 프로세스가 생성되어 응답하는 데 비해, 자바 서블릿은 외부 요청마다 프로세스보다 가벼운 스레드로써 응답하므로 보다 가볍습니다. 또한, 자바 서블릿은 자바로 구현되므로 다양한 플랫폼에서 동작합니다.<br>
즉 간단히 정리하면 클라이언트의 요청을 처리하고, 그 결과를 반환하는 Servlet 클래스의 구현 규칙을 지킨 자바 웹 프로그래밍 기술을 의미합니다.<br>
서블릿은 자바를 사용해 웹을 만들기 위해 필요한 기술이라 할 수 있습니다.<br>
서블릿의 특징에 관해 다시 정리해 보면 아래와 같습니다.
- 클라이언트의 요청에 대해 동적으로 작동하는 웹 어플리케이션 컴포넌트입니다.
- html을 사용하여 요청에 응답합니다.
- Java Thread를 이용하여 동작합니다.
- MVC 패턴에서 Controller로 이용됩니다.
- HTTP 프로토콜 서비스를 지원하는 javax.servlet.http.HttpServlet 클래스를 상속받습니다.
- UDP보다 처리 속도가 느립니다.
- HTML 변경 시 Servlet을 재컴파일해야 하는 단점이 있습니다.
출처는 출처 스레드의 3번입니다.

#### JWT를 사용한다는 것
JWT를 사용하는 이유 중 하나는 세션을 사용하지 않기 위함입니다.<br>
예를 들어 서버를 여러대 운영할 경우 세션같은 경우 사용자의 동일한 세션이 서버간에 공유(클러스터)가 되어야 하는 문제가 생기는데 JWT는 토큰 자체에 인증기능을 구현할 수 있기 때문에 어떤 서버로 접속하더라도 동일한 토큰값이라면 서버개수와 상관없이 인증 시스템을 구현할 수 있습니다.

### 출처
1. [학습중인 강의](https://www.inflearn.com/course/%EC%BD%94%EC%96%B4-%EC%8A%A4%ED%94%84%EB%A7%81-%EC%8B%9C%ED%81%90%EB%A6%AC%ED%8B%B0)
2. [자바 서블릿 위키백과](https://ko.wikipedia.org/wiki/%EC%9E%90%EB%B0%94_%EC%84%9C%EB%B8%94%EB%A6%BF)
3. [[JSP] 서블릿(Servlet)이란?](https://mangkyu.tistory.com/14)
4. [인프런 - jwt 토큰방식에서의 세션 미사용 질문](https://www.inflearn.com/course/%EC%BD%94%EC%96%B4-%EC%8A%A4%ED%94%84%EB%A7%81-%EC%8B%9C%ED%81%90%EB%A6%AC%ED%8B%B0/unit/29834?tab=community&category=questionDetail&q=558844)