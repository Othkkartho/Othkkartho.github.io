---
# multilingual page pair id, this must pair with translations of this page. (This name must be unique)
lng_pair: springlearn
title: 인프런 스프링 시큐리티 강의 학습-16

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
date: 2022-10-10 22:00:00 +0900

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

인가 프로세스 DB 연동 구현 [인가 개요](#스프링-시큐리티-인가-개요), [웹 기반 인가처리 DB 연동](#웹-기반-인가처리-db-연동) 를 정리한 포스트입니다.
--------------------------------------

출처는 인프런의 스프링 시큐리티 - [Spring Boot 기반으로 개발하는 Spring Security](https://www.inflearn.com/course/%EC%BD%94%EC%96%B4-%EC%8A%A4%ED%94%84%EB%A7%81-%EC%8B%9C%ED%81%90%EB%A6%AC%ED%8B%B0)강의를 바탕으로 이 포스트를 작성하고 있습니다.<br>
강의의 세션 5의 2,3,4,5번 강의내용에 대한 정리입니다.<br><br>

<!-- outline-end -->

### 스프링 시큐리티 인가 개요
지금까지 여러 권한 설정을 SecurityConfig에 작성해 구현해 왔는데 그것을 DB와 연동해 자원 및 권한을 설정하고 제어함으로 동적 관리가 가능하도록 할 것입니다.<br>
- 관리자 시스템을 구축해 회원 관리, 권한 관리, 자원 관리가 가능하게 할 것입니다.
    - 회원 관리: 사용자의 권한 부여
    - 권한 관리: 사이트가 에서 사용자가 가질 수 있는 권한 생성 및 삭제
    - 자원 관리: 각 서버 자원의 생성, 삭제, 수정, 매핑
권한 계층 구현에는 URL 방식과 Method 방식이 있습니다. 2개 다 강의에는 나와있고, 코드는 다 작성해 보기는 했지만, 포스트에는 URL 위주로 작성하고, Method는 간단하게 살펴보겠습니다.<br><br>
그리고 포스트 14번까지와 15번의 코드가 많이 다릅니다. 그러니 이 코드는 제 [깃허브 주소](https://github.com/Othkkartho/SpringSecurityLearn/tree/ch5.2%2C3%2C4)를 참고하시면 됩니다.<br>
제작 과정에서 DB로 완전히 넘어가는 도중에 코드가 조금 꼬여 위 출처의 분기점은 제대로 동작이 안될 수 있습니다. 만약 동작하는 분기점을 확인하고 싶으시다면 [이 링크](https://github.com/Othkkartho/SpringSecurityLearn/tree/ch5.5%2C6)를 참고해 주시기 바랍니다.

### 웹 기반 인가처리 DB 연동
#### 주요 아키텍처 이해
스프링 시큐리티가 인가처리를 하기 위해 보통 코드를 `http.antMatchers("/user").access("hasRole('USER')")` 과 같이 프로그램을 작성합니다. <br>
위 코드가 나타내는 바는 사용자가 /user 경로로 접근하기 위해서는 USER라는 권한을 가지고 있어야 한다는 것을 의미합니다.<br>
그럼 인가 처리를 담당하는 필터인 FilterSecurityInterceptor 가 인가처리를 위임하는 클래스인 AccessDecisionManager에게 인증 정보, 요청 정보, 권한 정보를 전달 하고, 메니저가 인가처리를 합니다.<br><br>

그럼 어떻게 그 값을 얻어 전달할 지 확인해 보겠습니다.
1. 사용자가 인증을 요청하면, 시큐리티 보안 필터인 SecurityInterceptor가 요청을 받고, 인증, 요청, 권한 정보를 얻습니다.
    - 인증 정보는 SecurityContext안에 있으므로 필터가 SecurityContext를 참조해 정보를 얻습니다.
    - 요청 정보는 직접 FilterInvocation 객체를 생성해서 request 정보를 저장한 후 객체를 전달합니다.
    - 권한 정보는 설정한 정보를 읽어 경로를 Key로 권한 정보를 Value로 매핑되어 Map 객체에 저장되어 필터에 `List<configAttribute>` 로 hasRole("USER")같은 value를 저장해 전달합니다.
2. SecurityInterceptor가 얻은 정보를 AccessDecisionManager에게 전달합니다.

**ExpressionBasedFilterInvocationSecurityMetadataSource에서 저장된 Map 객체**{:data-align="center"}
![정상적으로 권한에 대한 Map 객체가 전달됨](:/inflearn_spring_security_learn/5s/15/requestMap.jpg){:data-align="center"}
위 사진은 FilterInvocationSecurityMetadataSource의 구현체 중 하나인 ExpressionBasedFilterInvocationSecurityMetadataSource 클레스에서 전달된 인가 정보 가 key와 value의 map 객체로 저장되 있는 것을 확인하실 수 있습니다.<br>
그 후 위 클래스에서 부모 클래스인 DefaultFilterInvocationSecurityMetadataSource에게 Map 객체를 전달해 그곳에서 requestMap 객체에 저장해 놓습니다.<br><br>

FilterSecurityInterceptor가 `invoke(new FilterInvocation(request, response, chain));` 를 이용해 FilterInvocation 객체를 생성해 request 객체를 담아 요청 정보를 저장합니다.<br>
그 후 AbstractSecurityInterceptor에서 `Collection<ConfigAttribute> attributes = this.obtainSecurityMetadataSource().getAttributes(object);` 를 통해 권한 정보를 추출합니다. DefaultFilterInvocationSecurityMetadataSource가 이미 초기화 시 requestMap을 저장해 놨기 때문에 관련 메소드를 불러 가져오는 것입니다. 그리고 코드를 확인해 보면 인가 처리가 정상적으로 되지 않는 것을 확인하실 수 있습니다. 그 이유는 이미 다 작성이 된 코드기 때문에 UrlFilterInvocationSecurityMetadataSource가 작동하고, DB 설정을 다 하지 않았기 때문에 등록한 `requestMap.put(new AntPathRequestMatcher("/mypage"), List.of(new SecurityConfig("ROLE_USER")));` 만 작동합니다.<br>
AbstractSecurityInterceptor에서 인증 정보인 authentication 또한 `Authentication authentication = SecurityContextHolder.getContext().getAuthentication();` 코드를 사용해 securitycontext에서 가져옵니다.<br>
결국 `this.accessDecisionManager.decide(authenticated, object, attributes);` 를 확인하면 인증, 요청, 권한 정보 모두를 가져와 accessDecisionManager에게 전달합니다.

#### FilterInvocationSecurityMetadataSource - 1
##### 설명
FilterInvocationSecurityMetadataSource는 SecurityMetadataSource를 상속한 Url 기반 인가처리 인터페이스입니다.
- 사용자가 접근을 원하는 URL 자원에 대해 권한 정보 추출합니다.
- AccessDecisionManager에게 추출한 권한 정보를 전달해 인가처리를 수행합니다.
- DB로부터 자원 및 권한 정보를 매핑해 맵으로 관리합니다.
- 사용자의 요청마다 요청정보에 매핑된 권한 정보를 확인합니다.

##### 인가 처리의 순서도
1. 사용자가 특정 자원에 접근하고자 하면 그 요청을 FilterSecurityInterceptor가 받습니다.
2. 그럼 Interceptor는 권한정보를 추출하기 위해 FilterInvocationSecurityMetadataSource 구현체를 호출하게 됩니다.
3. 그럼 이 구현체는 내부적으로 requestMap 객체를 관리합니다. 이 객체는 /admin - ROLE_ADMIN 과 같은 맵 형태의 Key(URL) - Value(필요 권한 정보) 형태로 인가 정보가 저장되어 있습니다.
    - 이 값들은 DB에서 얻어와 맵 형태로 저장하게 됩니다.
4. 만약 권한목록이 존재하면 그 값을 AccessDecisionManager에게 전달합니다.

##### 실제 코드
**UrlFilterInvocationSecurityMetadataSource.java**{:data-align="center"}
```java
public class UrlFilterInvocationSecurityMetadataSource implements FilterInvocationSecurityMetadataSource {
    private final LinkedHashMap<RequestMatcher, List<ConfigAttribute>> requestMap;
    private SecurityResourceService securityResourceService;

    @Override
    public Collection<ConfigAttribute> getAttributes(Object object) throws IllegalArgumentException {
        HttpServletRequest request = ((FilterInvocation) object).getRequest();  // 1

        requestMap.put(new AntPathRequestMatcher("/mypage"), List.of(new SecurityConfig("ROLE_USER"))); // 4

        if(requestMap != null) {
            for(Map.Entry<RequestMatcher, List<ConfigAttribute>> entry : requestMap.entrySet()) {
                RequestMatcher matcher = entry.getKey();    // 2
                if(matcher.matches(request)) {              // 2
                    return entry.getValue();                // 2
                }
            }
        }

        return null;
    }

    @Override
    public Collection<ConfigAttribute> getAllConfigAttributes() {
        Set<ConfigAttribute> allAttributes = new HashSet<>();

        for (Map.Entry<RequestMatcher, List<ConfigAttribute>> entry : requestMap.entrySet()) {
            allAttributes.addAll(entry.getValue());
        }

        return allAttributes;
    }

    @Override
    public boolean supports(Class<?> clazz) {
        return FilterInvocation.class.isAssignableFrom(clazz);      // 3
    }

    public void reload() {
        LinkedHashMap<RequestMatcher, List<ConfigAttribute>> reloadedMap = securityResourceService.getResourceList();
        Iterator<Map.Entry<RequestMatcher, List<ConfigAttribute>>> iterator = reloadedMap.entrySet().iterator();

        requestMap.clear();

        while (iterator.hasNext()) {
            Map.Entry<RequestMatcher, List<ConfigAttribute>> entry = iterator.next();
            requestMap.put(entry.getKey(), entry.getValue());
        }
    }
}
```
1. 사용자가 어떤 URL로 요청했는지 요청 정보를 가져옵니다.
2. DB의 요청 정보를 가져와 matcher에 저장하고, 그 값을 사용자의 요청 정보와 비교해 만약 일치한다면 해당 요청 정보에 대한 권한 정보를 Return합니다.
3. URL 방식인지 Method 방식인지 요청 타입을 확인하는 코드입니다.
4. 필터가 변경되었지만 DB 설정이 완료되지 않았기 때문에 인가 설정이 잘 되었는지 확인하기 위해 임의로 값을 넣었습니다.

**SecurityConfig.java**{:data-align="center"}
```java
@Bean
public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
    // 아래 코드까지의 코드는 동일합니다.
    http
            .authorizeRequests()    // 1
            .anyRequest().authenticated()
            .and()
            .formLogin()
            .loginPage("/login")
            .loginProcessingUrl("/login_proc")
            .authenticationDetailsSource(formWebAuthenticationDetailsSource)
            .successHandler(formAuthenticationSuccessHandler)
            .failureHandler(formAuthenticationFailureHandler)
            .permitAll()
            .and()
            .exceptionHandling()
            .authenticationEntryPoint(new LoginUrlAuthenticationEntryPoint("/login"))
            .accessDeniedPage("/denied")
            .accessDeniedHandler(accessDeniedHandler())
            .and()
            .addFilterBefore(customFilterSecurityInterceptor(), FilterSecurityInterceptor.class);

    http.csrf().disable();

    return http.build();
}

@Bean
public FilterSecurityInterceptor customFilterSecurityInterceptor() throws Exception {
    FilterSecurityInterceptor filterSecurityInterceptor = new FilterSecurityInterceptor();
    filterSecurityInterceptor.setSecurityMetadataSource(urlFilterInvocationSecurityMetadataSource());       // 2
    filterSecurityInterceptor.setAccessDecisionManager(affirmativeBased());                                 // 3
    filterSecurityInterceptor.setAuthenticationManager(authenticationManager(authenticationConfiguration)); // 4

    return filterSecurityInterceptor;
}

private AccessDecisionManager affirmativeBased() {  // 3
    return new AffirmativeBased(getAccessDecisionVoters());
}

private List<AccessDecisionVoter<?>> getAccessDecisionVoters() {    // 3
    return List.of(new RoleVoter());
}

@Bean
public FilterInvocationSecurityMetadataSource urlFilterInvocationSecurityMetadataSource() throws Exception {    // 5
    return new UrlFilterInvocationSecurityMetadataSource(urlResourcesMapFactoryBean().getObject(), securityResourceService);
}
```
1. 원래 지정했던 모든 인가 처리들은 필터를 추가함에 따라 이제 작동하지 않아 삭제했습니다.
2. 위에 만든 구현체가 정상적으로 등록되서 동작할 수 있도록 Interceptor에 등록해 줍니다.
3. 접근 결정 관리자 3개 중 가장 보편적으로 많이 사용하는 affirmativeBased를 사용하도록 설정해줍니다.
4. 인가 처리 전 인증 여부를 검사하기 때문에 인증 관리자를 설정해 줘야 합니다.
5. MetadataSource를 Bean에 추가합니다.

##### 실제 실행 화면
**FilterChainProxy에 추가된 FilterSecurityInterceptor**{:data-align="center"}
![Interceptor가 2개 있는 것을 보면 정상적으로 추가 된것을 알 수 있음](:/inflearn_spring_security_learn/5s/15/additionalFilters.jpg){:data-align="center"}
FilterChainProxy에 FilterSecurityInterceptor가 2개 있는 것을 확인할 수 있습니다. 원래 있던 필터와 추가한 필터입니다. 즉 정상적으로 필터 등록이 완료된 것을 확인할 수 있습니다.

**Metadata에 저장되어 있는 맵 객체**{:data-align="center"}
![저장한 맵 객체가 정상적으로 들어있는 것을 확인할 수 있음](:/inflearn_spring_security_learn/5s/15/securityMetadataSource.jpg){:data-align="center"}
생성한 FilterSecurityInterceptor 안에 생성한 메타 데이터인  UrlFilterInvocationSecurityMetadataSource가 정상적으로 등록이 되어 있고, 맵 객체의 값이 정상적으로 저장되어 있는 것을 확인할 수 있습니다.

**requestMap에 권한 목록을 추가한 뒤 조건문의 작동**{:data-align="center"}
![조건문이 정상적으로 작동함](:/inflearn_spring_security_learn/5s/15/result.jpg){:data-align="center"}
requestMap에 권한 정보를 추가한 후 실제 조건문이 작동함으로써 정상적으로 클라이언트 요청을 Match하는 것을 알 수 있습니다.

#### FilterInvocationSecurityMetadataSource - 2
DB에 저장된 권한/자원 정보를 얻어 requestMap를 빈으로 생성해 UrlFilterInvocationSecurityMetadataSource에 전달해야 하는데 그 역할을 하는 것이 **UrlResourcesMapFactoryBean** 입니다.

##### 실제 코드
**UrlResourcesMapFactoryBean.java**{:data-align="center"}
```java
public class UrlResourcesMapFactoryBean implements FactoryBean<LinkedHashMap<RequestMatcher, List<ConfigAttribute>>> {
    private SecurityResourceService securityResourceService;                    // 1
    private LinkedHashMap<RequestMatcher, List<ConfigAttribute>> resourceMap;   // 2

    public void setSecurityResourceService(SecurityResourceService securityResourceService) {
        this.securityResourceService = securityResourceService;
    }

    @Override
    public LinkedHashMap<RequestMatcher, List<ConfigAttribute>> getObject() {   // 3
        if (resourceMap == null) {
            init();
        }

        return resourceMap;
    }

    private void init() {                                       // 3
        resourceMap = securityResourceService.getResourceList();
    }

    @Override
    public Class<?> getObjectType() {
        return LinkedHashMap.class;
    }

    @Override
    public boolean isSingleton() {              // 4
        return FactoryBean.super.isSingleton();
    }
}
```
1. DB로 부터 가져온 정보를 Mapping하는 Service를 불러옵니다.
2. 권한/자원 정보를 Bean으로 만들 객체를 생성합니다.
3. DB로부터 가져온 정보가 Mapping된 Map을 이 Bean 클래스로 가져와 저장 합니다.
4. 메모리에 1개만 존재하도록 설정합니다.

**SecurityResourceService.java**{:data-align="center"}
```java
public class SecurityResourceService {
    private final ResourcesRepository resourcesRepository;              // 1

    public SecurityResourceService(ResourcesRepository resourcesRepository) {
        this.resourcesRepository = resourcesRepository;
    }

    public LinkedHashMap<RequestMatcher, List<ConfigAttribute>> getResourceList() {
        LinkedHashMap<RequestMatcher, List<ConfigAttribute>> result = new LinkedHashMap<>();
        List<Resources> resourcesList = resourcesRepository.findAllResources(); // 1

        resourcesList.forEach(re -> {       // 4
            List<ConfigAttribute> configAttributeList = new ArrayList<>();
            re.getRoleSet().forEach(role -> {
                configAttributeList.add(new SecurityConfig(role.getRoleName()));                // 2
            });
            result.put(new AntPathRequestMatcher(re.getResourceName()), configAttributeList);   // 3
        });

        return result;
    }
}
```
DB의 인가 정보를 Mapping하는 클래스입니다.
1. DB로 부터 권한/자원 정보를 가져와 저장합니다.
2. 자원 1개 마다 여러개의 권한 정보가 담깁니다.
3. 완성된 정보를 return하기 위해 result에 생성된 값을 담습니다.
4. 즉 요약하면 아래와 같이 됩니다.
    - DB에 모든 자원의 자원을 가지고 옵니다. 그럼 자원 1개마다 권한 정보가 여러개인 1:N 형태로 담겨서 가지고 옵니다.
    - 그럼 첫 forEach를 통해 자원 1개를 돌 때 마다 그 자원 정보와 매핑된 권한 정보를 리스트로 담아 자원과 권한이 1:N 형식으로 result에 들어가 모든 자원 정보의 처리가 끝나면 return 하게 됩니다.

**ResourcesRepository.java**{:data-align="center"}
```java
public interface ResourcesRepository extends JpaRepository<Resources, Long> {
    Resources findByResourceNameAndHttpMethod(String resourceName, String httpMethod);

    @Query("select r from Resources r join fetch r.roleSet where r.resourceType = 'url' order by r.orderNum desc")
    List<Resources> findAllResources();
}
```
jpa 문법을 사용해 권한/자원 정보를 가지고 옵니다. 주의해야 할 점은 이 매핑 작업도 작은게 앞으로 오고 큰 자원 정보가 뒤에 가야 정상적으로 작동하기 때문에 orderNum 순으로 정렬해 그 값을 받아와 저장할 때 순서대로 저장하지 않는 HashMap이 아닌 순서에 맞게 저장하는 LinkedHashMap으로 저장합니다.

**AppConfig.java**{:data-align="center"}
```java
@Configuration
class AppConfig {
    @Bean
    public SecurityResourceService securityResourceService(ResourcesRepository resourcesRepository) {   // 1
        return new SecurityResourceService(resourcesRepository);
    }
}
```
일반적으로 사용하는 설정 클래스입니다.
1. SecurityResourceService를 Bean에 등록하고, resourceRepository를 전달해 줍니다.

**SecurityConfig.java**{:data-align="center"}
```java
@Bean
public FilterInvocationSecurityMetadataSource urlFilterInvocationSecurityMetadataSource() throws Exception {
    return new UrlFilterInvocationSecurityMetadataSource(urlResourcesMapFactoryBean().getObject(), securityResourceService);
}

private UrlResourcesMapFactoryBean urlResourcesMapFactoryBean() {
    UrlResourcesMapFactoryBean urlResourcesMapFactoryBean = new UrlResourcesMapFactoryBean();

    urlResourcesMapFactoryBean.setSecurityResourceService(securityResourceService);

    return urlResourcesMapFactoryBean;
}
```
DB로부터 받은 ResourceMap을 FilterInvocationSecurityMetadataSource에게 전달합니다.

**UrlFilterInvocationSecurityMetadataSource.java**{:data-align="center"}
```java
public UrlFilterInvocationSecurityMetadataSource(LinkedHashMap<RequestMatcher, List<ConfigAttribute>> resourcesMap, SecurityResourceService securityResourceService) {
        this.requestMap = resourcesMap;
        this.securityResourceService = securityResourceService;
    }
```
위에서 전달한 데이터를 받기 위해 생성자를 추가했습니다.

##### 실제 실행 장면
**DB에 저장된 자원/권한 정보를 가저옴**{:data-align="center"}
![DB에 저장되어 있는 권한과 자원 정보가 정상적으로 저장됨](:/inflearn_spring_security_learn/5s/15/resourcesList.jpg){:data-align="center"}
DB에 저장되어 있는 자원/권한 정보가 List에 정상적으로 저장되었고, 아래에 보시면 result에도 정상적으로 매핑 된 것을 확인할 수 있습니다.

**가져온 정보를 매핑해 저장함**{:data-align="center"}
![DB에 있던 정보가 모두 매핑되어 있는 것을 확인할 수 있습니다.](:/inflearn_spring_security_learn/5s/15/resourcesList_result.jpg){:data-align="center"}

### 출처
1. [학습중인 강의](https://www.inflearn.com/course/%EC%BD%94%EC%96%B4-%EC%8A%A4%ED%94%84%EB%A7%81-%EC%8B%9C%ED%81%90%EB%A6%AC%ED%8B%B0)