---
# multilingual page pair id, this must pair with translations of this page. (This name must be unique)
lng_pair: springlearn
title: 인프런 스프링 시큐리티 강의 학습-17

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
date: 2022-10-30 22:00:00 +0900

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

[웹 기반 인가처리 실시간 반영](#웹-기반-인가처리-실시간-반영), [인가처리 허용 필터](#인가처리-허용-필터---permitallfilter), [계층 권한 적용하기](#계층-권한-적용하기---rolehierachy), [아이피 접속 제한하기](#아이피-접속-제한하기---customipaddressvoter)를 정리한 포스트입니다.
--------------------------------------

출처는 인프런의 스프링 시큐리티 - [Spring Boot 기반으로 개발하는 Spring Security](https://www.inflearn.com/course/%EC%BD%94%EC%96%B4-%EC%8A%A4%ED%94%84%EB%A7%81-%EC%8B%9C%ED%81%90%EB%A6%AC%ED%8B%B0)강의를 바탕으로 이 포스트를 작성하고 있습니다.<br>
강의의 세션 5의 6,7,8,9번 강의내용에 대한 정리입니다.<br><br>

<!-- outline-end -->

### 웹 기반 인가처리 실시간 반영
관리자가 권한/자원 정보를 업데이트 하면 그 정보가 페이지에 실시간으로 반영되도록 코드를 작성합니다.
#### 실제 코드
**UrlFilterInvocationSecurityMetadataSource.java**{:data-align="center"}
```java
private final LinkedHashMap<RequestMatcher, List<ConfigAttribute>> requestMap;
private SecurityResourceService securityResourceService;

public UrlFilterInvocationSecurityMetadataSource(LinkedHashMap<RequestMatcher, List<ConfigAttribute>> resourcesMap, SecurityResourceService securityResourceService) {
    this.requestMap = resourcesMap;
    this.securityResourceService = securityResourceService;
}

public void reload() {      // 1
    LinkedHashMap<RequestMatcher, List<ConfigAttribute>> reloadedMap = securityResourceService.getResourceList();
    Iterator<Map.Entry<RequestMatcher, List<ConfigAttribute>>> iterator = reloadedMap.entrySet().iterator();

    requestMap.clear();     // 2

    while (iterator.hasNext()) {    // 3
        Map.Entry<RequestMatcher, List<ConfigAttribute>> entry = iterator.next();
        requestMap.put(entry.getKey(), entry.getValue());
    }
}
```
1. 권한 관련 정보를 업데이트 하게 되면 DB에 저장되고, 그때 reload를 호출해 requestMap 그 정보를 다시 매핑해 저장합니다.
2. 이전에 저장되어 있던 정보를 삭제합니다.
3. 권한 정보를 다시 저장합니다.

**ResourcesController.java**{:data-align="center"}
```java
private UrlFilterInvocationSecurityMetadataSource urlFilterInvocationSecurityMetadataSource;

@Autowired
private void setResourceController(ResourcesService resourcesService, RoleRepository roleRepository, RoleService roleService, UrlFilterInvocationSecurityMetadataSource urlFilterInvocationSecurityMetadataSource) {
    this.resourcesService = resourcesService;
    this.roleRepository = roleRepository;
    this.roleService = roleService;
    this.urlFilterInvocationSecurityMetadataSource = urlFilterInvocationSecurityMetadataSource;
}

@PostMapping(value="/admin/resources")
public String createResources(ResourcesDto resourcesDto) {
    ModelMapper modelMapper = new ModelMapper();
    Role role = roleRepository.findByRoleName(resourcesDto.getRoleName());
    Set<Role> roles = new HashSet<>();
    roles.add(role);
    Resources resources = modelMapper.map(resourcesDto, Resources.class);
    resources.setRoleSet(roles);

    resourcesService.createResources(resources);        // 1
    urlFilterInvocationSecurityMetadataSource.reload();

    return "redirect:/admin/resources";
}

@GetMapping(value="/admin/resources/delete/{id}")
public String removeResources(@PathVariable String id, Model model) {
    Resources resources = resourcesService.getResources(Long.parseLong(id));
    resourcesService.deleteResources(Long.parseLong(id));
    urlFilterInvocationSecurityMetadataSource.reload();     // 2

    return "redirect:/admin/resources";
}
```
1. 생성된 정보가 실시간으로 반영되게 합니다.
2. 삭제된 정보가 실시간으로 반영되게 합니다.

**SecurityConfig.java**{:data-align="center"}
```java
@Bean
public FilterInvocationSecurityMetadataSource urlFilterInvocationSecurityMetadataSource() throws Exception {
    return new UrlFilterInvocationSecurityMetadataSource(urlResourcesMapFactoryBean().getObject(), securityResourceService);
}
```
UrlFilterInvocationSecurityMetadataSource의 초기화시 입력 변수값이 추가되었기 때문에 securityResourceService를 추가했습니다.

#### 실제 실행화면
**인가 정보 수정시 Map에 저장된 정보**{:data-align="center"}
![정상적으로 권한에 대한 Map 객체가 변경됨](:/inflearn_spring_security_learn/5s/17/mapping_data.jpg){:data-align="center"}
위 사진은 /messages의 값을 Manager 권한에서 User 권한으로 내렸을 때, UrlFilterInvocationSecurityMetadataSource 클래스에서 발생한 Map 정보입니다.   
우선 원래 저장되어 있는 값인 requestMap에는 정상적으로 ROLE_MANAGER가 등록이 되어 있고, 새로운 DB 값인 reloadedMap에는 ROLE_USER가 등록되어 있는 것을 알 수 있습니다.   <br>
실제로 실행해 보면 User가 정상적으로 /messages 자원에 접근하는 것을 확인할 수 있습니다.

### 인가처리 허용 필터 - PermitAllFilter
인증 및 권한심사가 필요없는 자원(/, /login)들을 미리 설정해 사용자가 바로 리소스에 접근할 수 있도록 하는 필터입니다.   <br>
권한 심사의 내부 동작의 경우 아래와 같이 진행됩니다.
1. 유저가 서버의 자원에 접근을 요청하면 FilterSecurityInterceptor가 그 요청을 받고, AbstractSecurityInterceptor에게 인가처리를 맡깁니다.
2. 인가처리가 끝나면 List<ConfigAttribute> 타입의 권한 목록을 확인합니다. 만약 사용자가 접근하고자 하는 자원의 권한 정보가 Null일 경우 권한 심사 없이 통과하고, Null이 아닐 경우 AccessDecisionManager을 통해 권한을 심사합니다.
이를 응용 동작으로 구현할 방향 아래와 같습니다.
1. 유저가 서버 자원에 접근하면 FilterSecurityInterceptor를 상속받은 PermitAllFilter가 List<RequestMatcher>에 저장한 정보들과 사용자가 요청한 정보를 확인합니다.
2. 만약 매치되는 정보가 있으면, Null을 return해 권한 심사 없이 바로 통과하고, 아니라면 인가처리를 위해 AbstractSecurityInterceptor를 호출합니다.
위와 같이 변경하면 리스트의 정보만을 확인해 인가 처리가 필요하지 않을 경우 인가 처리가 진행되지 않기 때문에 서버의 자원을 아낄 수 있습니다.

#### 실제 코드
**PermitAllFilter.java**{:data-align="center"}
```java
public class PermitAllFilter extends FilterSecurityInterceptor {
    private static final String FILTER_APPLIED = "__spring_security_filterSecurityInterceptor_filterApplied";
    private boolean observeOncePerRequest = true;
    private final List<RequestMatcher> permitAllRequestMatchers = new ArrayList<>();    // 1

    public PermitAllFilter(String...permitAllResources) {   // 2
        for (String resource : permitAllResources) {
            permitAllRequestMatchers.add(new AntPathRequestMatcher(resource));
        }
    }

    @Override
    protected InterceptorStatusToken beforeInvocation(Object object) {
        boolean permitAll = false;
        HttpServletRequest request = ((FilterInvocation) object).getRequest();  // 3

        for (RequestMatcher requestMatcher : permitAllRequestMatchers) {        // 4
            if (requestMatcher.matches(request)) {
                permitAll = true;
                break;
            }
        }

        if (permitAll) {        // 5
            return null;
        }

        return super.beforeInvocation(object);  // 6
    }

    public void invoke(FilterInvocation filterInvocation) throws IOException, ServletException {
        if (isApplied(filterInvocation) && this.observeOncePerRequest) {
            filterInvocation.getChain().doFilter(filterInvocation.getRequest(), filterInvocation.getResponse());
            return;
        }
        if (filterInvocation.getRequest() != null && this.observeOncePerRequest) {
            filterInvocation.getRequest().setAttribute(FILTER_APPLIED, Boolean.TRUE);
        }
        InterceptorStatusToken token = beforeInvocation(filterInvocation);
        try {
            filterInvocation.getChain().doFilter(filterInvocation.getRequest(), filterInvocation.getResponse());
        }
        finally {
            super.finallyInvocation(token);
        }
        super.afterInvocation(token, null);
    }

    private boolean isApplied(FilterInvocation filterInvocation) {
        return (filterInvocation.getRequest() != null)
                && (filterInvocation.getRequest().getAttribute(FILTER_APPLIED) != null);
    }
}
```
위 클래스는 이미 구현되어 있는 FilterSecurityInterceptor 클래스를 가져와 변경한 것입니다.
1. 인증이나 인가가 필요없는 자원들을 저장할 리스트 변수입니다.
2. 인증이나 인가가 필요없는 자원들을 저장하는 메서드입니다.
3. 사용자 요청 정보를 가지고 옵니다.
4. 저장된 변수와 사용자의 요청 정보가 일치하면 더 이상 검사할 필요가 없으므로 permitAll을 true로 두고 반복문을 빠져나옵니다.
5. permitAll이 true라면 즉 주석 4번의 조건과 맞았다면 null을 반환해 권한 심사가 되지 않도록 합니다.
6. 만약 조건을 만족하지 않았다면 AbstractSecurityInterceptor에 인가 처리를 요청합니다.

**SecurityConfig.java**{:data-align="center"}
```java
private final String[] permitAllResources = {"/", "/login", "/user/login/**"};

@Bean
public PermitAllFilter customFilterSecurityInterceptor() throws Exception {
    PermitAllFilter filterSecurityInterceptor = new PermitAllFilter(permitAllResources);    // 1
    filterSecurityInterceptor.setSecurityMetadataSource(urlFilterInvocationSecurityMetadataSource());
    filterSecurityInterceptor.setAccessDecisionManager(affirmativeBased());
    filterSecurityInterceptor.setAuthenticationManager(authenticationManager(authenticationConfiguration));

    return filterSecurityInterceptor;
}
```
생성한 PermitAllFilter를 추가합니다.

#### 실제 실행 화면
**permitAllRequestMatcher에 저장되있는 정보들**{:data-align="center"}
![permitAllRequestMatcher에 정상적으로 저장되어 있음](:/inflearn_spring_security_learn/5s/17/permitAllRequestMatcher.jpg){:data-align="center"}
permitAllRequestMatcher에 인가, 권한 처리가 필요 없는 리스트들이 정상적으로 저장되어 있는 모습입니다.

**/에 접근시 permitAll**{:data-align="center"}
![permitAll이 true가 되 null을 리턴함](:/inflearn_spring_security_learn/5s/17/permitAll.jpg){:data-align="center"}
/에 접근시 permitAll이 true가 되 null을 리턴하는 모습입니다.

### 계층 권한 적용하기 - RoleHierachy
데이터 베이스에 각 상하관계를 구현합니다.   
**RoleHierarchy**는 하위 계층 Role이 하위 계층 Role의 자원에 접근할 수 있도록 합니다.   
**RoleHierarchyVoter**는 RoleHierarchy를 생성자로 받으면서 이 클래스에서 설정한 규칙이 적용되어 심사합니다.

**RoleHierarchy.java**{:data-align="center"}
```java
@Entity
@Table(name="ROLE_HIERARCHY")
@AllArgsConstructor
@NoArgsConstructor
@Getter
@Setter
@Builder
public class RoleHierarchy implements Serializable {
    @Id
    @GeneratedValue
    private Long id;

    @Column(name = "child_name")
    private String childName;

    @ManyToOne(cascade = {CascadeType.ALL},fetch = FetchType.LAZY)
    @JoinColumn(name = "parent_name", referencedColumnName = "child_name")
    private RoleHierarchy parentName;

    @OneToMany(mappedBy = "parentName", cascade={CascadeType.ALL})
    private Set<RoleHierarchy> roleHierarchy = new HashSet<RoleHierarchy>();
}
```
권한 정보들의 상하관계를 나타내는 정보를 저장하기 위해 DB에 릴레이션을 만듭니다.

**RoleHierarchyRepository.java**{:data-align="center"}
```java
public interface RoleHierarchyRepository extends JpaRepository<RoleHierarchy, Long> {
    RoleHierarchy findByChildName(String roleName);
}
```
Role 이름으로 데이터를 얻습니다.

**RoleHierarchyServiceImpl.java**{:data-align="center"}
```java
@Service
public class RoleHierarchyServiceImpl implements RoleHierarchyService {
    private RoleHierarchyRepository roleHierarchyRepository;

    @Autowired
    private void setRoleHierarchyServiceImpl(RoleHierarchyRepository roleHierarchyRepository) {
        this.roleHierarchyRepository = roleHierarchyRepository;
    }

    @Transactional
    @Override
    public String findAllHierarchy() {
        List<RoleHierarchy> rolesHierarchy = roleHierarchyRepository.findAll();

        Iterator<RoleHierarchy> itr = rolesHierarchy.iterator();
        StringBuilder concatedRoles = new StringBuilder();
        while (itr.hasNext()) {
            RoleHierarchy roleHierarchy = itr.next();
            if (roleHierarchy.getParentName() != null) {
                concatedRoles.append(roleHierarchy.getParentName().getChildName());
                concatedRoles.append(" > ");
                concatedRoles.append(roleHierarchy.getChildName());
                concatedRoles.append("\n");
            }
        }

        return concatedRoles.toString();
    }
}
```
DB로 부터 Role 정보를 가져와 규칙대로 포맷팅 하고, 그 정보를 문자열로 리턴하는 클래스입니다.

**SecurityConfig.java**{:data-align="center"}
```java
@Transactional
public void createRoleHierarchyIfNotFound(Role childRole, Role parentRole) {
    RoleHierarchy roleHierarchy = roleHierarchyRepository.findByChildName(parentRole.getRoleName());

    if (roleHierarchy == null) {
        roleHierarchy = RoleHierarchy.builder()
                .childName(parentRole.getRoleName())
                .build();
    }
    RoleHierarchy parentRoleHierarchy = roleHierarchyRepository.save(roleHierarchy);

    roleHierarchy = roleHierarchyRepository.findByChildName(childRole.getRoleName());

    if (roleHierarchy == null) {
        roleHierarchy = RoleHierarchy.builder()
                .childName(childRole.getRoleName())
                .build();
    }

    RoleHierarchy childRoleHierarchy = roleHierarchyRepository.save(roleHierarchy);
    childRoleHierarchy.setParentName(parentRoleHierarchy);
}
```
초기화 할 때 각 상하관계를 DB에 저장하는 역할을 하는 메서드입니다.

**SetupDataLoader.java**{:data-align="center"}
```java
private List<AccessDecisionVoter<?>> getAccessDecisionVoters() {
    List<AccessDecisionVoter<? extends Object>> accessDecisionVoters = new ArrayList<>();
    accessDecisionVoters.add(roleVoter());

    return accessDecisionVoters;
}

@Bean
public AccessDecisionVoter<? extends Object> roleVoter() {
    return new RoleHierarchyVoter(roleHierarchy());
}

@Bean
public RoleHierarchyImpl roleHierarchy() {
    return new RoleHierarchyImpl();
}
```
RoleHierarchyImpl의 값을 저장하는 설정 클래스의 메소드입니다.

**SecurityInitializer.java**{:data-align="center"}
```java
@Component
public class SecurityInitializer implements ApplicationRunner {
    private RoleHierarchyService roleHierarchyService;
    private RoleHierarchyImpl roleHierarchy;

    @Autowired
    private void setSecurityInitializer(RoleHierarchyService roleHierarchyService, RoleHierarchyImpl roleHierarchy) {
        this.roleHierarchyService = roleHierarchyService;
        this.roleHierarchy = roleHierarchy;
    }

    @Override
    public void run(ApplicationArguments args) throws Exception {
        String allHierarchy = roleHierarchyService.findAllHierarchy();
        roleHierarchy.setHierarchy(allHierarchy);
    }
}
```
DB로 부터 Role 정보를 가져와 포맷팅 된 결과값을 RoleHierarchyImpl에 넣어 주는 클래스입니다.

#### 실제 실행 화면
**SecurityInitializer에서 DB에 저장된 정보를 불러옴**{:data-align="center"}
![rolesHierarchy에 정상적으로 3개의 권한과 그 하위 권한이 저장되어 있음](:/inflearn_spring_security_learn/5s/17/rolesHierarchy.jpg){:data-align="center"}
rolesHierarchy에 DB에 저장되어 있는 정보들이 정상적으로 각각의 권한과 그 하위 권한으로 저장되어 있습니다.

**role_hierarchy 테이블**{:data-align="center"}
![미리 설정했던 값들이 정상적으로 들어가 있음](:/inflearn_spring_security_learn/5s/17/role_hierarchy.jpg){:data-align="center"}
위 rolesHierarchy에 저장되어 있는 값과 동일한 정보가 DB에 저장되어 있습니다.

**모든 처리가 끝나고, concatedRoles**{:data-align="center"}
![모두 설정한대로 저장됨](:/inflearn_spring_security_learn/5s/17/concatedRoles.jpg){:data-align="center"}
concatedRoles에 위에 설정했던 권한 상하위 정보들이 정해진 규칙대로 저장되었습니다.

### 아이피 접속 제한하기 - CustomIpAddressVoter
IpAddressVoter의 심의 기준은 다음과 같습니다.
1. 특정한 IP만 접근이 가능하도록 심의하는 Voter을 추가합니다.
2. Voter 중 가장 먼저 심사하도록 해 허용된 IP 일 경우 최종 승인 및 거부 결정을 하도록 합니다.
3. 허용된 IP면 ACCESS_GRANTED가 아닌 ACCESS_ABSTAIN을 리턴해 이후 남은 심의를 계속 진행하도록 합니다.
    - AffirmativeBased는 하나라도 승인이 되면 자원 접근이 허용되기 때문에 승인이 아닌 보류를 리턴해야 합니다.
4. 허용된 IP가 아니면 ACCESS_DENIED를 리턴하지 않고 즉시 예외를 발생시켜 최종 자원에 접근을 거부합니다.
    - 거부되었다고 해도 바로 접근이 거부되지 않고, 다른 Voter로 심의하다가 하나라도 승인이 되면 접근이 승인되기 때문에 예외를 발생시켜 접근하지 못하게 해야 합니다.

#### 실제 코드
**AccessIp.java**{:data-align="center"}
```java
@Entity
@Table(name = "ACCESS_IP")
@Getter
@Setter
@ToString
@RequiredArgsConstructor
@Builder
@AllArgsConstructor
public class AccessIp implements Serializable {
    @Id
    @GeneratedValue
    @Column(name = "IP_ID", unique = true, nullable = false)
    private Long id;

    @Column(name = "IP_ADDRESS", nullable = false)
    private String ipAddress;

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || Hibernate.getClass(this) != Hibernate.getClass(o)) return false;
        AccessIp accessIp = (AccessIp) o;
        return id != null && Objects.equals(id, accessIp.id);
    }

    @Override
    public int hashCode() {
        return getClass().hashCode();
    }
}
```
허용할 IP를 담을 테이블을 만듭니다.

**AccessIpRepository.java**{:data-align="center"}
```java
public interface AccessIpRepository extends JpaRepository<AccessIp, Long> {
    AccessIp findByIpAddress(String IpAddress);
}
```
IP 주소를 가지고 데이터를 찾는 클래스입니다.

**AccessIpRepository.java**{:data-align="center"}
```java
@Override
@Transactional
public void onApplicationEvent(final ContextRefreshedEvent event) {
    if (alreadySetup) {
        return;
    }

    setupSecurityResources();
    setupAccessIpData();
    alreadySetup = true;
}

private void setupSecurityResources() {
    Set<Role> roles = new HashSet<>();
    Role adminRole = createRoleIfNotFound("ROLE_ADMIN", "관리자");
    roles.add(adminRole);
    createUserIfNotFound("admin", "admin@admin.com", "pass", roles);
    Role managerRole = createRoleIfNotFound("ROLE_MANAGER", "매니저권한");
    Role userRole = createRoleIfNotFound("ROLE_USER", "사용자권한");
    createRoleHierarchyIfNotFound(managerRole, adminRole);
    createRoleHierarchyIfNotFound(userRole, managerRole);
}

private void setupAccessIpData() {
    AccessIp byIpAddress = accessIpRepository.findByIpAddress("0:0:0:0:0:0:0:1");

    if (byIpAddress == null) {
        AccessIp accessIp = AccessIp.builder()
                .ipAddress("0:0:0:0:0:0:0:1")
                .build();
        accessIpRepository.save(accessIp);
    }
}
```
기본적인 IP정보를 저장하는 메서드입니다.

**IpAddressVoter.java**{:data-align="center"}
```java
public class IpAddressVoter implements AccessDecisionVoter<Object> {
    private final SecurityResourceService securityResourceService;

    public IpAddressVoter(SecurityResourceService securityResourceService) {
        this.securityResourceService = securityResourceService;
    }

    @Override
    public boolean supports(ConfigAttribute attribute) {
        return true;
    }

    @Override
    public boolean supports(Class<?> clazz) {
        return true;
    }

    @Override
    public int vote(Authentication authentication, Object object, Collection<ConfigAttribute> attributes) { // 1
        WebAuthenticationDetails details = (WebAuthenticationDetails)authentication.getDetails();
        String remoteAddress = details.getRemoteAddress();  // 2

        List<String> accessIpList = securityResourceService.getAccessIpList();

        int result = ACCESS_DENIED;

        for (String ipAddress : accessIpList){  // 3
            if (remoteAddress.equals(ipAddress)){
                return ACCESS_ABSTAIN;
            }
        }

        if (result == ACCESS_DENIED){
            throw new AccessDeniedException("Invalid IpAddress");
        }

        return result;
    }
}
```
1. 실질적인 심의 로직입니다. 각각 인증 정보, 요청 정보, 자원에 접근할 때 필요한 권한정보를 받습니다.
2. 사용자의 IP 주소 정보를 가지고 옵니다.
3. 각 IP 주소를 확인해 만약 정보가 같다면 보류를 return해 이후 심의를 이어나가고, 만약 정보가 없다면 에러를 던져 심의를 끝냅니다.

**SecurityResourceService.java**{:data-align="center"}
```java
private final ResourcesRepository resourcesRepository;
private final AccessIpRepository accessIpRepository;

public SecurityResourceService(ResourcesRepository resourcesRepository, AccessIpRepository accessIpRepository) {
    this.resourcesRepository = resourcesRepository;
    this.accessIpRepository = accessIpRepository;
}

public List<String> getAccessIpList() {
    return accessIpRepository.findAll().stream().map(AccessIp::getIpAddress).collect(Collectors.toList());
}
```
허용하는 IP 주소를 DB에서 가지고옵니다.

**SecurityConfig.java**{:data-align="center"}
```java
private List<AccessDecisionVoter<?>> getAccessDecisionVoters() {
    List<AccessDecisionVoter<? extends Object>> accessDecisionVoters = new ArrayList<>();
    accessDecisionVoters.add(new IpAddressVoter(securityResourceService));
    accessDecisionVoters.add(roleVoter());

    return accessDecisionVoters;
}
```
IP가 허용되지 않으면 바로 오류를 던져 심의를 종료해야 하기 때문에 심의 중 가장 먼저 실행되야 합니다.

#### 실제 실행 화면
**저장되어 있는 심의 정보**{:data-align="center"}
![IpAddressVoter와 RoleHierachyImpl가 모두 정상적으로 저장되어 있음](:/inflearn_spring_security_learn/5s/17/voter.jpg){:data-align="center"}
SecurityConfig에서 저장한 IpAddressVoter와 RoleHierachyImpl가 모두 정상적으로 저장되어 있음을 볼 수 있습니다.

**remoteAddress와 accessIpList**{:data-align="center"}
![둘 모두 정상적으로 값이 저장되어 있습니다](:/inflearn_spring_security_learn/5s/17/remoteAddress.jpg){:data-align="center"}
DB에 저장되어 있는 허용 주소는 accessIpList에, 접근 주소는 remoteAddress에 정상적으로 저장되어 있습니다.

**서로 비교**{:data-align="center"}
![정상적으로 비교되 보류를 return함](:/inflearn_spring_security_learn/5s/17/abstain.jpg){:data-align="center"}
둘 모두 같은 값이기 때문에 보류를 정상적으로 return 함을 알 수 있습니다.

### 완료
이로써 위의 4개의 기능들을 모두 만들어 보았습니다. 사진으로 따로 찍지는 않았지만 각 화면들이 원하는대로 출력되었습니다. 이외의 결과는 제 [깃허브](https://github.com/Othkkartho/SpringSecurityLearn/tree/ch5.7%2C8%2C9)에 가셔서 코드를 실행해 확인하실 수 있습니다.

### 출처
1. [학습중인 강의](https://www.inflearn.com/course/%EC%BD%94%EC%96%B4-%EC%8A%A4%ED%94%84%EB%A7%81-%EC%8B%9C%ED%81%90%EB%A6%AC%ED%8B%B0)