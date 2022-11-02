---
# multilingual page pair id, this must pair with translations of this page. (This name must be unique)
lng_pair: springlearn
title: 인프런 스프링 시큐리티 강의 학습-19

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
date: 2022-11-02 22:00:00 +0900

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

[주요 아키텍처 이해](#aop-method-기반-db-연동---주요-아키텍처-이해) [MapBasedSecurityMetadataSource](#aop-method-기반-db-연동---mapbasedsecuritymetadatasource), [ProtectPointcutPostProcessor](#aop-method-기반-db-연동---protectpointcutpostprocessor)를 정리한 포스트입니다.
--------------------------------------

출처는 인프런의 스프링 시큐리티 - [Spring Boot 기반으로 개발하는 Spring Security](https://www.inflearn.com/course/%EC%BD%94%EC%96%B4-%EC%8A%A4%ED%94%84%EB%A7%81-%EC%8B%9C%ED%81%90%EB%A6%AC%ED%8B%B0)강의를 바탕으로 이 포스트를 작성하고 있습니다.<br>
강의의 세션 6의 2,4,5,6,7번 강의내용에 대한 정리입니다.<br><br>

<!-- outline-end -->

### AOP Method 기반 DB 연동 - 주요 아키텍처 이해
Spring이 인가 처리를 위해 초기화 과정은 먼저 DefaultAdvisorAutoProxyCreator가 MethodSecurityMetadataSourceAdvisor가 MethodSeurityMetadataSourcePointcut으로 메소드에 보안이 설정되어 있는지 빈을 검사하며 조건에 맞는지를 MethodSecurityMetadataSource에 클래스 정보와 메서드 정보를 전달해 Source 클래스가 클래스와 메서드 정보를 Parse()합니다. 
1. Source가 보안 메서드가 설정된 빈일 경우 AutoProxyCreator가 그 빈의 프록시 객체를 생성합니다.
2. 해당 메서드를 MethodSecurityInterceptor에 등록합니다.
초기화 과정이 끝나 프록시 객체가 생성되고, 인가처리를 위한 Advice가 등록되있으며, MethodMap이 만들어 집니다. 이후 인가처리를 진행한다면 다음과 같습니다.
1. 사용자가 메서드를 호출하면 OrderServiceProxy 객체가 해당 메서드를 호출합니다.
2. 그 후 Advcie 등록 여부를 확인하고, 되어 있다면 인가처리의 부가 기능을 담고 있는 Advice인 MethodSecurityInterceptor를 호출합니다.
3. Interceptor가 인가처리를 진행합니다.
    - 만약 권한 심사가 승인되면 프록시 객체가 아닌 실제 객체를 호출합니다.
    - 승인되지 않으면 AccessDeniedException을 던저 메서드 접근을 차단합니다.

### AOP Method 기반 DB 연동 - MapBasedSecurityMetadataSource
#### 개요
User가 메서드 접근을 요청하면 MethodSecurityInterceptor가 MethodSecurityMetadataSource라는 구현체에게 요청해 권한 정보를 반환받고, 구현체는 DB로 부터 자원과 권한들을 매핑해 맵 객체를 저장하고 있다가 그 요청에 해당하는 키 값의 자원 정보를 반한합니다. 그럼 Interceptor는 AccessDecisionManager에게 권한 정보를 전달합니다.   <br>

MapBasedSecurityMetadataSource는 MethodSecurityMetadataSource를 구현하고, SecurityMetadataSource 최상위 클래스를 상속합니다.
1. 어노테이션 설정 방식이 아닌 맵 기반으로 권한을 설정합니다.
2. 기본적인 구현이 완성되어 있고 DB로 부터 자원과 권한 정보를 매핑한 데이터를 전달하면 메소드 방식의 인가처리가 이루어지는 클래스입니다.

#### 실제 실행
**DelegatingMethodSecurityMetadataSource가 가지고 있는 권한 정보**{:data-align="center"}
![정상적으로 설정한 권한을 가져 옴](:/inflearn_spring_security_learn/5s/18/abstract_attrs.JPG){:data-align="center"}
```java
@Override
@Secured("ROLE_USER")
public void order() {
    System.out.println("order");
}
```
위 코드에 설정되어 있는 권한 정보를 정상적으로 가져와 저장되어 있는 것을 확인할 수 있습니다.   <br>

다 실행을 해 보면 Url 방식과 Method 방식의 처리 방식은 Method의 경우 마지막에 메소드를 호출해 주는 부분을 제외하곤 모두 동일합니다.

#### Map 기반 DB 연동의 진행 방식
1. 유저가 권한이 설정되어 있는 메소드를 요청하면 요청한 메소드의 어드바이스인 MethodSecurityInterceptor를 호출하며 인가처리를 시작합니다.
2. Interceptor는 MapBasedMethodSecurityMetadataSource에게 권한 정보를 요청합니다.
3. MetadataSource 자체적으로 MethodMap을 가지고 있고, 이를 통해 메소드 정보와 매칭되는 권한정보를 추출합니다.
4. 권한목록이 존재하면 decide(Authentication, MethodInvocation, List<ConfigAttribute>) 를 통해 AccessDecisionManager에 정보를 전달해 실질적인 인가처리를 합니다.

#### 실제 코드
**MethodSecurityConfig.java**{:data-align="center"}
```java
@Configuration
@EnableGlobalMethodSecurity // 1
public class MethodSecurityConfig extends GlobalMethodSecurityConfiguration {   // 2
    private SecurityResourceService securityResourceService;

    @Autowired
    private void setMethodSecurityConfig(SecurityResourceService securityResourceService) {
        this.securityResourceService = securityResourceService;
    }

    @Override
    protected MethodSecurityMetadataSource customMethodSecurityMetadataSource() {   // 3
        return mapBasedMethodSecurityMetadataSource();
    }

    @Bean
    public MapBasedMethodSecurityMetadataSource mapBasedMethodSecurityMetadataSource() {
        return new MapBasedMethodSecurityMetadataSource(Objects.requireNonNull(methodResourcesMapFactoryBean().getObject()));
    }

    @Bean
    public MethodResourcesFactoryBean methodResourcesMapFactoryBean() {
        MethodResourcesFactoryBean methodResourcesFactoryBean = new MethodResourcesFactoryBean();
        methodResourcesFactoryBean.setSecurityResourceService(securityResourceService);

        return methodResourcesFactoryBean;
    }
}
```
1. 이제는 맵 방식으로 진행을 하기 때문에 따로 값을 지정하지는 않습니다.
2. 메서드 보안이 활성화되고, 관련 초기화 작업들이 이루어 지는 클래스를 상속받습니다.
3. 맵 기반으로 메서드 인가처리를 할 수 있는 클래스를 생성해 return합니다.

**MethodResourcesFactoryBean.java**{:data-align="center"}
```java
public class MethodResourcesFactoryBean implements FactoryBean<LinkedHashMap<String, List<ConfigAttribute>>> {
    private SecurityResourceService securityResourceService;
    private LinkedHashMap<String, List<ConfigAttribute>> resourceMap;

    public void setSecurityResourceService(SecurityResourceService securityResourceService) {
        this.securityResourceService = securityResourceService;
    }

    @Override
    public LinkedHashMap<String, List<ConfigAttribute>> getObject() {
        if (resourceMap == null) {
            init();
        }

        return resourceMap;
    }

    private void init() {
        resourceMap = securityResourceService.getMethodResourceList();
    }

    @Override
    public Class<?> getObjectType() {
        return LinkedHashMap.class;
    }

    @Override
    public boolean isSingleton() {
        return FactoryBean.super.isSingleton();
    }
}
```

**ResourcesRepository.java**{:data-align="center"}
```java
@Query("select r from Resources r join fetch r.roleSet where r.resourceType = 'method' order by r.orderNum desc")
    List<Resources> findAllMethodResources();
```
DB로 부터 메서드 방식으로 저장한 방식을 가져오는 Query문입니다.

**SecurityResourceService.java**{:data-align="center"}
```java
public LinkedHashMap<String, List<ConfigAttribute>> getMethodResourceList() {
    LinkedHashMap<String, List<ConfigAttribute>> result = new LinkedHashMap<>();
    List<Resources> resourcesList = resourcesRepository.findAllMethodResources();

    resourcesList.forEach(re -> {
        List<ConfigAttribute> configAttributeList = new ArrayList<>();
        re.getRoleSet().forEach(role -> {
            configAttributeList.add(new SecurityConfig(role.getRoleName()));
        });
        result.put(re.getResourceName(), configAttributeList);
    });

    return result;
}
```

### AOP Method 기반 DB 연동 - ProtectPointcutPostProcessor


### 참고
#### customMethodSecurityMetadataSource()
**GlobalMethodSecurityConfiguration.java**{:data-align="center"}
```java
protected MethodSecurityMetadataSource customMethodSecurityMetadataSource() {
    return null;
}
```

**GlobalMethodSecurityConfiguration.java**{:data-align="center"}
```java
if (customMethodSecurityMetadataSource != null) {
    sources.add(customMethodSecurityMetadataSource);
}
```

**MethodSecurityConfig.java**{:data-align="center"}
```java
@Override
protected MethodSecurityMetadataSource customMethodSecurityMetadataSource() {
    return mapBasedMethodSecurityMetadataSource();
}
```
원래는 null을 return하기 때문에 2번째 코드가 실행되지 않지만, MethodSecurityConfig에서 customMethodSecurityMetadataSource를 Override해 null이 아닌 값을 return함으로 조건문이 실행됩니다.

#### JPQA


### 출처
1. [학습중인 강의](https://www.inflearn.com/course/%EC%BD%94%EC%96%B4-%EC%8A%A4%ED%94%84%EB%A7%81-%EC%8B%9C%ED%81%90%EB%A6%AC%ED%8B%B0)