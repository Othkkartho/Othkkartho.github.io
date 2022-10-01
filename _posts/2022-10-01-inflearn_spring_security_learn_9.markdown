---
# multilingual page pair id, this must pair with translations of this page. (This name must be unique)
lng_pair: springlearn
title: 인프런 스프링 시큐리티 강의 학습-9

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
date: 2022-10-01 22:00:00 +0900

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

실전 프로젝트 구현전 구상, 정적 자원 관리 - WebIgnore 설정 및 의존성 설정
----------------------------------------------------------------------------

출처는 인프런의 스프링 시큐리티 - [Spring Boot 기반으로 개발하는 Spring Security](https://www.inflearn.com/course/%EC%BD%94%EC%96%B4-%EC%8A%A4%ED%94%84%EB%A7%81-%EC%8B%9C%ED%81%90%EB%A6%AC%ED%8B%B0)강의를 바탕으로 이 포스트를 작성하고 있습니다.<br>
강의의 세션 3의 1,2번 강의내용에 대한 정리입니다.

<!-- outline-end -->

### 실전 프로젝트 구성
#### 프로젝트 기본 구성
- 의존성 설정, 환경설정, UI 화면 구성, 기본 CRUD 기능을 구현합니다.
- 스프링 시큐리티 보안 기능을 점진적으로 구현 및 완성할 계획입니다.
- 코드 중 resource에 들어가는 코드는 강의의 코드를 그대로 가져왔습니다. 각 프론트앤드 부분 코드들은 강의에서 자세히 다루지 않을경우 코드를 확인해 포스트를 작성하겠습니다.
- 강의에서는 각각의 강의에 따른 코드를 다른 브랜치로 관리했지만 저는 포스트에 코드를 조금 더 자세히 작성하는 것으로 대체하도록 하겠습니다.

#### 실제 코드
**build.gradle**{:data-align="center"}
```java
implementation 'org.springframework.boot:spring-boot-starter-security'
implementation 'org.springframework.boot:spring-boot-starter-thymeleaf'
implementation 'org.springframework.boot:spring-boot-starter-web'
implementation 'org.thymeleaf.extras:thymeleaf-extras-springsecurity5'
testImplementation 'org.springframework.boot:spring-boot-starter-test'
testImplementation 'org.springframework.security:spring-security-test'
```
위 코드를 보시면 어떤 의존성을 추가했는지 아실 수 있습니다.<br>
추가한 의존성은 Spring Security, Thymeleaf, Spring Web입니다. 강의에서 JPA 사용한다고 먼저 [JPA 의존성을 추가하시면 안됩니다](#failed-to-configure-a-datasource-에러).

**ConfigController.java**{:data-align="center"}
```java
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;

@Controller
public class ConfigController {
    @GetMapping("/config")
    public String config() {
        return "/admin/config";
    }
}
```
**MessageController.java**{:data-align="center"}
```java
// 위 코드와 import가 같음

@Controller
public class MessageController {
    @GetMapping(value = "messages")
    public String messages() {
        return "/user/messages";
    }
}
```
**UserController.java**{:data-align="center"}
```java
// 위 코드와 import가 같음

@Controller
public class UserController {
    @GetMapping(value = "/mypage")
    public String myPage() {
        return "/user/mypage";
    }
}
```
**HomeController.java**{:data-align="center"}
```java
// 위 코드와 import가 같음

@Controller
public class HomeController {
    @GetMapping("/")
    public String home() {
        return "/home";
    }
}
```
각각의 Controller들은 매핑 이외의 처리는 하지 않아서 따로 설명을 추가하지는 않겠습니다.<br><br>

**SecurityConfig.java**{:data-align="center"}
```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.core.userdetails.User;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.crypto.factory.PasswordEncoderFactories;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.security.provisioning.InMemoryUserDetailsManager;
import org.springframework.security.web.SecurityFilterChain;

@Configuration
@EnableWebSecurity
public class SecurityConfig {
    @Bean
    public InMemoryUserDetailsManager userDetailsService() {                        // 1
        String password = passwordEncoder().encode("1234");

        UserDetails user = User.withUsername("user").password(password).roles("USER").build();
        UserDetails sys = User.withUsername("manager").password(password).roles("MANAGER").build();
        UserDetails admin = User.withUsername("admin").password(password).roles("ADMIN").build();
        return new InMemoryUserDetailsManager(user, sys, admin);
    }

    @Bean
    public PasswordEncoder passwordEncoder() {                                      // 2
        return PasswordEncoderFactories.createDelegatingPasswordEncoder();
    }

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {    // 3
        http
                .authorizeRequests()
                .antMatchers("/").permitAll()
                .antMatchers("/mypage").hasRole("USER")
                .antMatchers("/messages").hasRole("MANAGER")
                .antMatchers("/config").hasRole("ADMIN")
                .anyRequest().authenticated()
                .and()
                .formLogin();

        return http.build();
    }
}
```
혹시 WebSecurityConfigurerAdapter의 Deprecated의 대처에 관해 궁금하신 분들은 [이곳](/posts/2022-09-20-inflearn_spring_security_learn_1#스프링 2.7-이후의-SecurityConfig)을 클릭해주시면 좀 더 자세히 설명해 놓았습니다.<br>
1. 아직 DB연동을 하지 않았기 때문에 InMemory에 유저 정보를 저장했습니다.
2. 비밀번호를 암호화 하는 passwordEncoder를 만드는 클래스입니다.
3. [FilterChainProxy](/posts/2022-09-20-inflearn_spring_security_learn_6#FilterChainProxy-설명)에 필터를 추가하기 위한 클래스입니다.
    - 루트 경로에 관해서는 모두 접속하게 하고 각 관련 경로로 요청이 들어왔을 때 모든 사용자를 허용하고, 각 설정된 경로에는 설정된 권한을 지닌 사용자만 들어올 수 있게 했습니다.
따로 결과 화면은 사진을 첨부하지 않았지만 로그인은 잘 되고, **user** 는 /mypage 경로에만, **manager** 는 /messages 경로에만, **admin** 은 /config 경로에만 접근할 수 있게 되 정상적으로 동작이 됩니다.

### 정적 자원 관리 - WebIgnore 설정
#### WebIgnore의 개념
Spring Security는 js / css / image 등과 같은 보안이 적용될 필요가 없는 파일들도 보안을 적용시킵니다.<br><br>
그래서 위와 같이 보안 필터를 적용할 필요가 없는 리소스를 설정해 보안 필터가 적용되지 않도록 해주는 설정입니다.

#### properties 설정 및 추가된 의존성 추가 코드
**build.gradle**{:data-align="center"}
```java
implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
implementation 'org.postgresql:postgresql:42.5.0'
implementation 'org.modelmapper:modelmapper:3.1.0'
implementation 'org.springframework.boot:spring-boot-devtools:2.7.4'
compileOnly 'org.projectlombok:lombok:1.18.24'
```
위는 추가한 의존성입니다. 의존성은 따로 설명 프로젝트에서 설명하겠습니다.

**application.properties**{:data-align="center"}
~~~java
spring.datasource.url=jdbc:postgresql://localhost:5432/springlearndb    // 1
spring.datasource.username=username                                     // 1
spring.datasource.password=password                                     // 1

spring.jpa.hibernate.ddl-auto=create                                    // 2
spring.jpa.database-platform=org.hibernate.dialect.PostgreSQLDialect    // 3
spring.jpa.properties.hibernate.format_sql=true                         // 4
spring.jpa.properties.hibernate.show_sql=true                           // 4
spring.jpa.properties.hibernate.jdbc.lob.non_contextual_creation=true   // 4

spring.thymeleaf.cache=false                                            // 5

spring.devtools.livereload.enabled=true                                 // 6
spring.devtools.restart.enabled=true                                    // 6

spring.main.allow-bean-definition-overriding=true                       // 7
~~~
1. Spring의 DB연결 접속 url과 설정한 유저 이름, 패스워드를 입력해 DB 접속을 합니다.
2. domain을 만들면 그 내용을 자동으로 DB에 연결되게 해주는 설정입니다. create, update, none, validate, create-drop 등의 설정이 있습니다.
3. 각 database의 플랫폼 설정을 작성합니다.
4. 각 properties 설정합니다.
    - format_sql는 콘솔에 보여지는 sql 문법을 더 보기 좋게 변경합니다.
    - show_sql는 콘솔에 실행되는 sql 문법이 보여지게 합니다.
    - postgresSQL을 사용하면 발생하는 Caused by: java.sql.SQLFeatureNotSupportedException: Method org.postgresql.jdbc.PgConnection.createClob() is not yet implemented.라는 createClob()메서드를 구현하지 않았다는 하이버네이트의 에러로그를 보여주지 않는 설정입니다.
5. 스프링의 Thymeleaf 템플릿 결과는 캐싱하는 것이 Default 값이기 때문에 개발 과정에서 Thymeleaf를 수정하고 브라우저를 새로고침하면 바로 반영되지 않습니다. 그래서 이 값을 false로 해 줘 재시작 없이 새로고침만으로 반영되게 합니다.
6. spring devtools에서 지원하는 설정으로 개발을 더 편하게 지원해줍니다.
    - livereload.enabled는 템플릿 엔진이나, js, css등의 리소스 데이터들이 수정 될때마다 새로 빌드하는 것은 매우 번거롭기 때문에 새로고침 만으로 실시간 반영해주는 설정입니다.
    - restart.enabled는 classpath에 속해있는 파일들의 수정을 감지하고 자동으로 재시작해주는 기능입니다.
7. 스프링 프레임워크에서 빈을 등록하는 과정에서 발생하는 에러를 비활성화할 수 있는 기능을 제공합니다. 하지만 설정 이후에는 중복되는 빈을 찾아 중복이 발생하지 않도록 해줘야 합니다.

#### 실제 코드
```java
@Bean
public WebSecurityCustomizer webSecurityCustomizer() {
    return (web) -> web.ignoring().requestMatchers(PathRequest.toStaticResources().atCommonLocations());
}
```
위 코드는 StaticResourceLocation.java에 설정되어 있는 경로의 파일들을 보안 필터가 작동하지 않게 하는 방식입니다. 만약 프로그래머가 직접 경로를 설정하고 싶다면 이 [포스트](/posts/2022-09-20-inflearn_spring_security_learn_1#스프링 2.7-이후의-SecurityConfig)을 참고하시면 됩니다. <br><br>
`.antMatchers("/").permitAll()` 에 링크를 추가해도 같은 효과를 내지만 저렇게 따로 설정하는 이유는 만약 permitAll()로 설정하게 되면 Filter의 검사를 거칩니다. 하지만 WebSecurityCustomizer()에서는 검사를 거치지 않는 차이가 있습니다.

### 참고
#### Failed to configure a DataSource 에러
전체 에러 코드를 보면 아래와 같이 되어있습니다.
```
Failed to configure a DataSource: 'url' attribute is not specified and no embedded datasource could be configured.

Reason: Failed to determine a suitable driver class
```
에러가 DataSource를 설정해주지 않아서 발생한 문제임을 위 코드를 통해 알 수 있습니다.<br>
스프링 라이브러리를 소개한 프로젝트에서 자세히 소개하겠지만 JPA는 매핑된 관계를 이용해 SQL을 생성, 실행합니다.<br>
즉, DB를 건드리기 때문에 DataSource 설정이 필요한것입니다. 하지만 세션 3의 1번 강의에서는 DBMS연결을 하지 않아 datasource 설정을 하지 않았고, 그로 인해 오류가 발생한 것입니다.

#### Gradle와 Maven의 차이와 각각의 설명
- Gradle은 빌드 자동화 및 다국어 개발 지원에 중점을 둔 빌드 도구입니다.
    - Gradle의 장점
        1. 사용자 정의를 고도화 되게 할 수 있습니다. 다양한 프로젝트를 위해 다양한 기술로 수정할 수 있습니다.
        2. 성능이 매우 빠르고 효율적입니다.
        3. 플러그인을 만드는 데 사용되는 도구이기 때문에 유연합니다.
        4. 향상된 사용자 경험을 위해 다양한 IDE(통합 개발 환경)를 제공합니다.
    - Gradle의 단점
        1. Grandle로 작업을 구축하기 위해선 훌륭한 기술 전문 지식이 필요합니다.
        2. 내장된 아파치 ant 프로젝트 구조는 제공하지 않습니다.
        3. 문서가 다소 광범위합니다.
        4. Ant 빌드 스크립트는 XML의 도움으로 작성해야 합니다. 또한 어려운 프로젝트를 자동하 하기 위해선 많은 논리를 XML 파일로 작성해야합니다.
- Maven은 Java 프로젝트들을 위한 빌드 자동화 도구입니다.
    - Maven의 장점
        1. 프로젝트 구축 과정이 단순화되고 잘 조직되어 있습니다.
        2. Jar 파일 및 기타 종속성을 다운로드하는 작업을 자동으로 실행합니다.
        3. POM 파일에 종속성 코드를 공식화해 새로운 종속성을 쉽게 통합 할 수 있습니다.
        4. 모든 필수 정보에 쉽게 액세스 할 수 있습니다.
        5. 확장 가능하며 플러그인을 스크립팅 언어 또는 Java를 이용해 쉽게 작성할 수 있습니다.
    - Maven의 단점
        1. 작업 시스템에 설치해야 합니다.
        2. 기존 종속성에 대한 Maven코드를 찾을 수 없을 경우 Maven을 사용해 종속성을 구현할 수 없습니다.
        3. 프로젝트 실행 측면에서 Maven은 매우 느립니다.

#### Postgresql의 설명
##### PostgreSQL의 개요
PostgreSQL은 확장 가능성 및 표준 준수를 강조하는 [객체-관계형 데이터베이스 관리 시스템](https://ko.wikipedia.org/wiki/%EA%B0%9D%EC%B2%B4_%EA%B4%80%EA%B3%84_%EB%8D%B0%EC%9D%B4%ED%84%B0%EB%B2%A0%EC%9D%B4%EC%8A%A4)입니다.<br>
소규모의 단일 머신 애플리케이션에서부터 수많은 동시 접속 사용자가 있는 대형의 인터넷 애플리케이션(또는 [데이터 웨어하우스용](https://ko.wikipedia.org/wiki/%EB%8D%B0%EC%9D%B4%ED%84%B0_%EC%9B%A8%EC%96%B4%ED%95%98%EC%9A%B0%EC%8A%A4))에 이르기까지 여러 부하를 관리할 수 있으며 macOS 서버의 경우 PostgreSQL은 기본 데이터베이스입니다. 마이크로소프트 윈도우, 리눅스(대부분의 배포판에서 제공됨)용으로도 이용 가능합니다.
##### 장점
- **유연한 객체 생성**
    - 다른 관계형 데이터베이스 시스템과 달리, 연산자, 복합 자료형, 집계 함수, 자료형 변환자, 확장 기능 등 다양한 데이터베이스 객체를 사용자가 임의로 만들 수 있는 기능을 SQL 차원에서 제공합니다.
    - 이런 특징은 단순한 자료 저장소로써의 기능을 넘어 마치 하나의 새로운 프로그래밍 언어처럼 개발자의 창의성에 따라 무한한 기능을 손쉽게 구현할 수 있도록 합니다.
- **상속**
    - java 또는 C++ 프로그래밍 언어와 같이 테이블을 만들어 그 테이블 상속 기능을 이용해 하위 테이블을 만들 수 있습니다.
    - 테이블에 저장된 자료는 상위 테이블을 조회하면, 해당 테이블의 하위 테이블에 포함된 모든 자료를 조회할 수 있으며, 하위 테이블을 만들 때, 상위 테이블의 칼럼을 그대로 상속 받으면서, 하위 테이블에만 속하는 칼럼을 추가로 만들 수 있습니다.
- **함수**
    - 때때로, '저장 프로시저'라고 불리는 SQL문으로 작성된 함수를 서버환경에서 사용할 수 있습니다. 비록 다른 언어와는 달리 제어문과 반복문을 사용하지는 못하지만, 다른 언어와 결합시킬 수 있습니다. 일부 언어에서는 심지어 트리거 내부에서 실행시킬 수 있습니다. 그 언어 중 하나가 Java입니다.
    - PostgreSQL은 테이블에 대한 질의 결과를 반환하기 위한 '행 반환 함수'를 지원합니다. 실행권한은 함수 작성자 및, 실행자 모두에게 있습니다.

##### 설치 방법 및 필수 sql 설명
1. [PostgreSQL의 다운로드 링크](https://www.postgresql.org/download/)로 들어갑니다.
2. 들어가서 본인이 사용하는 운영체제를 고르면 Interactive installer by EDB 밑에 Download the installer이라는 하이퍼텍스트가 있습니다. 그걸 클릭해 들어갑니다.
3. 그곳에서 다운을 받고, MySQL이나, MairaDB와 같은 DB와 같이 저장위치, 루트 비밀번호, 포트, Locale을 설정하고, 이외에 열심히 설치 파일에서 하라는 데로 하다보면 설치가 완료됩니다.
4. 설치 검사 겸 접속은 윈도우 검색 창같은 프로그램 검색창에 SQL Shell (psql)을 찾아 실행합니다.
5. 서버, 데이터베이스, 포트, 사용자 이름과 같은 정보를 입력할 수 있지만, Enter를 치면 기본 정보가 입력됩니다. 마지막으로 비밀번호만 제대로 입력하면 들어가집니다.
6. `SELECT version();` 을 하면 본인이 설치한 버전이 나오면 잘 설치가 진행 된것입니다.
7. 각종 명령어는 cmd 창에 직접 치거나, [pgAdmin](https://www.pgadmin.org/download/)을 다운로드 받으면 MySQL의 workbench처럼 GUI의 형태로 DB를 관리할 수 있습니다.
    - 사용자, DB 생성 및 권한 부여는 [출처](#출처) 7번에 남겨놨습니다.

#### thymeleaf 설명
thymeleaf는 웹 및 독립 실행형 환경 모두를 위한 서버측 Java 템플릿 엔진입니다.<br>
조금 다르게 설명하면  Java XML/XHTML/HTML5 템플릿 엔진으로 웹과 웹 이외의 환경에서 모두 작동할 수 있다고도 할 수 있습니다.
Tymeleaf의 주요 목표는 개발 워크플로우에 우아한 자연 템플릿을 가져오는 것입니다.<br>
HTML은 브라우저에 올바르게 표시될 수 있고 정적 프로토타입으로 작동하여 개발 팀에서 더 강력한 협업을 가능하게 합니다.<br>
Tymeleaf는 또한 완전한 스프링 프레임워크 통합을 제공합니다.

### 출처
1. [학습중인 강의](https://www.inflearn.com/course/%EC%BD%94%EC%96%B4-%EC%8A%A4%ED%94%84%EB%A7%81-%EC%8B%9C%ED%81%90%EB%A6%AC%ED%8B%B0)
2. [스프링 부트로 시작하는 테스트(JPA). part 2](https://medium.com/msolo021015/%EC%8A%A4%ED%94%84%EB%A7%81-%EB%B6%80%ED%8A%B8%EB%A1%9C-%EC%8B%9C%EC%9E%91%ED%95%98%EB%8A%94-%ED%85%8C%EC%8A%A4%ED%8A%B8-jpa-part-2-ef3c302cff52)
3. [Thymeleaf 템플릿 캐시 설정](https://countryxide.tistory.com/8)
4. [Spring Devtools restart로 자동 재시작하기](https://velog.io/@gidskql6671/Spring-Devtools-restart%EB%A1%9C-%EC%9E%90%EB%8F%99-%EC%9E%AC%EC%8B%9C%EC%9E%91%ED%95%98%EA%B8%B0)
5. [BeanDefinitionOverrideException 예외 발생시](https://java.ihoney.pe.kr/530)
6. [위키백과 - PostgreSQL](https://ko.wikipedia.org/wiki/PostgreSQL)
7. [PostgreSQL USER, DB 생성 및 권한 부여](https://velog.io/@moonpiderman/PostgreSQL-USER-DB-%EC%83%9D%EC%84%B1-%EB%B0%8F-%EA%B6%8C%ED%95%9C-%EB%B6%80%EC%97%AC)
8. [위키백과 - thymeleaf](https://en.wikipedia.org/wiki/Thymeleaf)
9. [Thymeleaf 공식 홈페이지](https://www.thymeleaf.org/)
10. [Github - gradle](https://github.com/gradle/gradle)
11. [위키백과 - 아파치 메이븐](https://ko.wikipedia.org/wiki/%EC%95%84%ED%8C%8C%EC%B9%98_%EB%A9%94%EC%9D%B4%EB%B8%90)
12. [GeekforGeeks - Difference between Gradle and Maven](https://www.geeksforgeeks.org/difference-between-gradle-and-maven/)