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
date: 2022-10-02 10:00:00 +0900

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

사용자 DB 등록 및 PasswordEncoder
--------------------------------------

출처는 인프런의 스프링 시큐리티 - [Spring Boot 기반으로 개발하는 Spring Security](https://www.inflearn.com/course/%EC%BD%94%EC%96%B4-%EC%8A%A4%ED%94%84%EB%A7%81-%EC%8B%9C%ED%81%90%EB%A6%AC%ED%8B%B0)강의를 바탕으로 이 포스트를 작성하고 있습니다.<br>
강의의 세션 3의 3번 강의내용에 대한 정리입니다.<br><br>

github에 브렌치를 추가했으므로, import와 같은 코드는 포스트에서 추가하지 않겠습니다.

<!-- outline-end -->

### 사용자 DB 등록 및 PasswordEncoder
#### PasswordEncoder
- 비밀번호를 안전하게 암호화 해 저장할 수 있도록 해줍니다.
- Spring Security 5.0 이전에는 기본 PasswordEncoder가 평문을 지원하는 NoOpPasswordEncoder였습니다. 현재는 Deprecated 되었습니다.
- **생성**
    - `PasswordEncoder passwordEncoder = PasswordEncoderFactories.createDelegatingPasswordEncoder()`
    - 여러개의 PasswordEncoder 유형을 선언한 뒤, 상황에 맞게 선택해 사용할 수 있도록 지원하는 Encoder입니다.
- **암호화 포맷: {id}encodedPassword**
    - 기본 포맷은 Bcrypt: {bcrypt}암호화된 비밀번호 입니다.
    - 알고리즘의 종류는 bcrypt, noop, pbkdf2, scrypt, sha256 등이 있고, 암호화 방식과 방식별 설명들은 [참고](#암호화-방식의-종류와-설명)에 작성했습니다.
- *인터페이스*
    - encode(password): 패스워드 암호화를 담당합니다.
    - matches(rawPassword, encodedPassword): 패스워드 비교에 사용됩니다.

#### 실제 DB 등록 코드 및 비밀번호 암호화 코드
**Usercontroller.java**{:data-align="center"}
```java
private final UserService userService;                                              // 1
private final PasswordEncoder passwordEncoder;                                      // 1

public UserController(UserService userService, PasswordEncoder passwordEncoder) {   // 2
    this.userService = userService;
    this.passwordEncoder = passwordEncoder;
}

@GetMapping("/users")
public String createUser() {                                                        // 3
    return "/user/login/register";
}

@PostMapping("/users")
public String createUser(AccountDto accountDto) {                                   // 4
    ModelMapper modelMapper = new ModelMapper();
    Account account = modelMapper.map(accountDto, Account.class);
    account.setPassword(passwordEncoder.encode(account.getPassword()));
    userService.createUser(account);

    return "redirect:/";
}
```
유저와 관련된 전반적인 기능과, 사이트 매핑을 컨트롤하는 클레스입니다.<br>
1. 유저 저장과 비밀번호 암호화를 위한 클래스 선언입니다.
2. Autowired 사용시 발생하는 [warning](/posts/2022-09-20-inflearn_spring_security_learn_3#rememberMe-코드에서-private-final-선언에-관해)을 없애기 위해 작성된 Constructor입니다.
3. 유저 회원가입을 위한 Mapping 메소드입니다.
4. 유저 저장 메소드로서, Account 엔터티가 클라이언트로 부터 전달되는 값을 바로 받는 역할을 하지 않도록 하고 그 역할은 DTO 에 일임하고 이후 DTO 에 저장된 값을 엔터티로 복사해서 이후 처리를 하고 있습니다.

**Account.java**{:data-align="center"}
```java
@Entity                         // 1
@Setter                         // 2
@Getter                         // 3
public class Account {
    @Id                         // 4
    @GeneratedValue             // 5
    private Long Id;
    private String username;
    private String password;
    private String email;
    private String age;
    private String role;
}
```
DB와의 매핑에 사용되는 역할입니다.<br>
2, 3번 주석은 원래 강의에서는 Data로 작성되었으나 원래 강의에서는 Data로 작성되었지만, `Using @Data for JPA entities is not recommended. It can cause severe performance and memory consumption issues.` warning으로 인해 위 어노테이션으로 변경하였습니다.
1. JPA가 관리하게 하며, 테이블과 매핑시키는 어노테이션입니다.
2. setId나 SetUsername과 같이 값을 설정하게 해주는 setter을 사용자가 직접 작성하지 않고 자동으로 생성하도록 하는 어노테이션입니다.
3. getId나 getUsername과 같은 값을 가져오게 해주는 getter을 사용자가 직접 작성하지 않고 자동으로 생성하도록 하는 어노테이션입니다.
4. 해당 데이터가 DB에서 ID역할을 한다는 것을 나타내는 어노테이션입니다.
5. 컴퓨터가 자동으로 값을 집어 넣으라는 것을 나타내는 어노테이션입니다.

**AccountDto.java**{:data-align="center"}
```java
@Data                           // 1
public class AccountDto {
    private String username;
    private String password;
    private String email;
    private String age;
    private String role;
}
```
클라이언트로 부터 전달되는 값을 받는 역할을 하는 클레스입니다.
1. POJO(Plain Old Java Objects)와 bean과 관련된 모든 보일러플레이트(재사용 가능한 코드)를 생성합니다.
    - Getter @Setter @RequiredArgsConstructor @ToString @EqualsAndHashCode를 합쳐놓은 어노테이션입니다.

**UserRepository.java**{:data-align="center"}
```java
public interface UserRepository extends JpaRepository<Account, Long> {}
```
UserService에서 user 정보를 저장하기 위해 JPA에서 제공하는 Repository를 상속받은 인터페이스입니다.

**UserService.java**{:data-align="center"}
```java
public interface UserService {
    void createUser(Account account);
}
```
UserService를 구현과 캡슐화를 위해 생성한 인터페이스입니다.

**UserServiceImpl.java**{:data-align="center"}
```java
@Service("userService")
public class UserServiceImpl implements UserService {
    private final UserRepository userRepository;

    public UserServiceImpl(UserRepository userRepository) {
        this.userRepository = userRepository;
    }

    @Override
    @Transactional
    public void createUser(Account account) {
        userRepository.save(account);
    }
}
```
유저 저장을 구현한 클래스입니다.<br>

**SecurityConfig.java**{:data-align="center"}
```java
@Bean
public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
    http
            .authorizeRequests()
            .antMatchers("/", "/users").permitAll()
            .antMatchers("/mypage").hasRole("USER")
            .antMatchers("/messages").hasRole("MANAGER")
            .antMatchers("/config").hasRole("ADMIN")
            .anyRequest().authenticated()
            .and()
            .formLogin();

    return http.build();
}
```
/users 경로를 permitAll 해주지 않으면 회원가입을 로그인 해야 할 수 있는 참사가 발생하기 때문에 Config에서는 그 부분만 수정하였습니다. 

### 참고
#### 암호화 방식의 종류와 설명
PasswordEncoder의 구현체인 createDelegatingPasswordEncoder() 메소드에 있는 암호화 방식의 종류와 설명입니다. 각 암호 알고리즘의 자세한 내용은 출처를 확인해 주시면 됩니다.

| 종류 | 설명 |
| :---: | :---: |
| bcrypt | [블로피시](https://ko.wikipedia.org/wiki/%EB%B8%94%EB%A1%9C%ED%94%BC%EC%8B%9C) 암호에 기반을 둔 암호화 해시 함수입니다. [Rainbow table attack](https://ko.wikipedia.org/wiki/%EB%A0%88%EC%9D%B8%EB%B3%B4_%ED%85%8C%EC%9D%B4%EB%B8%94) 공격 방지를 위해 [Salt](https://ko.wikipedia.org/wiki/%EC%86%94%ED%8A%B8_(%EC%95%94%ED%98%B8%ED%95%99))를 통합한 적응형 함수의 하나 입니다. |
| ldap | 경량 디렉터리 액세스 프로토콜(LDAP)는 TCP/IP 위에서 디렉터리 서비스를 조회하고 수정하는 응용 프로토콜입니다. 디렉터리는 논리, 계급 방식 속에서 조직화된, 비슷한 특성을 가진 객체들의 모임입니다. 이런 기본 설계 때문에 LDAP는 인증을 위한 다른 서비스에 의해 자주 사용됩니다. |
| MD4 | 다이제스트 길이는 128bit 입니다. 1995년 최초의 완전 충돌 공격이 발표되었으며. 2007년에는 2개 미만의 MD4 해시 작업에서 충돌을 생성할 수 있습니다. 2011년 RFC 6150은 MD4가 역사(시대에 뒤떨어짐)가 되었다고 했습니다. |
| MD5 | 임의의 길이의 메시지를 입력받아, 128비트짜리 고정 길이의 출력값을 내는 암호화 해시 함수입니다. 1996년 설계상 결함이 발견되었습니다. 2006년에는 노트북 컴퓨터 한 대의 계산 능력으로 1분내에 해시 충돌을 찾은 알고리즘이 발표되었고, 2008년에는 결함을 이용해 SSL 인증서를 변조하는 것이 가능하다는 것을 발표했습니다. 현재는 보안 관련 용도로 쓰는것을 권장하지 않습니다. |
| noop | 암호화를 하지 않는다는 알고리즘입니다. 위에서 설명한 NoOpPasswordEncoder 알고리즘이 사용됩니다. |
| pbkdf2 | Salt 값과 함께 입력 암호 또는 암호에 [해시 기반 메시지 인증 코드](https://en.wikipedia.org/wiki/HMAC)와 같은 [의사 랜덤 함수](https://en.wikipedia.org/wiki/Pseudorandom_function_family)를 적용하는 프로세스를 여러번 반복해 파생 키를 생성하는 슬라이딩 계산 비용을 가진 주요 파생 함수입니다. |
| scrypt | 암호 기반의 [키 파생 함수](https://en.wikipedia.org/wiki/Key_derivation_function)입니다. 계산 집약적으로 설계되 계산하는데 비교적 오랜 시간이 걸립니다. 알고리즘의 리소스 요구를 높여 대규모 병렬 공격을하는 무차별 대입 공격의 시도를 방해하도록 설계되었습니다. |
| SHA-1 | 메시지 다이제스트라는 160비트 해시값을 만드는 암호화 해시 함수로 보통 16진수 40자리로 렌더링 됩니다. 2020년 SHA-1에 대항하는 선택 접두어 공격이 현실화 되 SHA-1을 제거하고 SHA-2 또는 SHA-3을 사용할 것이 권고되었습니다. 마이크로소프트는 2020년 8월 7일 윈도우 업데이트의 SHA-1 코드 서명 지원을 중단했습니다. |
| SHA-256 | SHA-2의 암호화 해시 함수들의 집합 중 하나이며, 암호 해시 함수는 디지털 데이터 상에서 수학적으로 동작하며 알려져 있고 예측된 해시값에 대해 계산된 해시(알고리즘의 실행 출력)를 비교함으로써 사람이 데이터의 무결성을 파악할 수 있게 됩니다. |
| argon2 | 암호를 해싱하는데 걸리는 시간이나 소요되는 메모리 양을 설정할 수 있게 설계되었습니다. Argon2d, Argon2i, Argon2id로 3가지 종류가 있습니다. |
{:data-align="center"}

여기서 고민해볼만한 암호화 방식은 PBKDF2, Scrypt, Bcrypt, SHA-256입니다. 설명은 [출처 10](#출처)을 참고 밑 인용했습니다

##### PBKDF2
- 해시 함수의 컨테이너인 PBKDF2는 Salt를 적용한 후 해시 함수의 반복 횟수를 임의로 선택할 수 있습니다.
- PBKDF2는 아주 가볍고 구현하기 쉬우며, SHA와 같이 검증된 해시 함수만을 사용합니다.
    - 2017년에 게시된 RFC 8018은 암호 해싱을 위해 PBKDF2를 권장합니다. 표준이 작성된 2000년에는 최소 반복 횟수가 1,000개였지만, 2021년 OWASP는 PBKDF2-HMAC-SHA256에 대해 310,000회 반복을, PBKDF2-HMAC-SHA512에 120,000회 반복을 사용할 것을 권장했습니다. Salt의 경우 미국 국립 표준 기술 연구소는 128 비트의 Salt 길이를 권장합니다.
- PBKDF2의 기본 파라미터는 다음과 같은 5개 파라미터로 구성되어 있습니다.
    - `DIGEST = PBKDF2(PRF, Password, Salt, c, DLen)`
    - PRF: 난수(예: HMAC)
    - Password: 패스워드
    - Salt: 데이터, 비밀번호, 통과암호를 해시 처리하는 단방향 함수의 추가 입력으로 사용되는 랜덤 데이터입니다.
    - c: 원하는 iteration 반복 수
    - DLen: 원하는 다이제스트 길이

##### Scrypt
- 오프라인 [무차별 대입 공격](https://ko.wikipedia.org/wiki/%EB%AC%B4%EC%B0%A8%EB%B3%84_%EB%8C%80%EC%9E%85_%EA%B3%B5%EA%B2%A9)에 대해 더 강력하지만, 많은 메모리와 CPU를 사용합니다.
- 하드웨어 구현을 하는데 크기와 비용이 훨씬 더 비싸기 때문에, 주어진 자원에서 공격자가 사용할 수 있는 병렬처리의 양이 한정적입니다.
- OpenSSL 1.1 이상을 제공하는 시스템에서만 작동합니다.
- scrypt의 파라미터는 다음과 같은 6개 파라미터로 구성되어 있습니다.
    - `DIGEST = scrypt(Password, Salt, N, r, p, DLen)`
    -   Password: 패스워드
    - Salt: 암호학 솔트
    - N: CPU 비용
    - r: 메모리 비용
    - p: 병렬화(parallelization)
    - DLen: 원하는 다이제스트 길이

##### Bcrypt
- gensalt()의 "work factor"를 조정하는 것만으로 간단하게 시스템의 보안성을 높일 수 있습니다.
- Salting과 Key stretching으로 Rainbow table attack, 무차별 대입 공격에 대비할 수 있습니다.
- 다만 PBKDF2나 scrypt와는 달리 bcrypt는 입력 값으로 72 bytes character를 사용해야 하는 제약이 있습니다.

##### SHA-256
- SHA-256은 256비트로 구성되며 64자리 문자열을 반환합니다.
- SHA-256은 단방향 암호화 방식이기 때문에 복호화가 불가능 하다는 것이 큰 특징이며, 복호화를 하지 않아도 되기 때문에 속도가 빠른 장점이 있습니다.

##### 사용처
- 서드파티의 라이브러리에 의존하지 않으면서 사용자 패스워드의 다이제스트를 생성하려면 PBKDF2을 사용하면 됩니다.
    - 단, 최소 반복 횟수를 사이트의 필요 보안 필요 레벨에 맞춰 잘 설정해 줘야 속도와 보안 모두를 잡을 수 있습니다.
- 구현하려는 시스템이 매우 민감한 정보를 다루고, 보안 시스템을 구현하는 데 많은 비용을 투자할 수 있다면 scrypt를 사용하면 됩니다.
- 강력한 패스워드 다이제스트를 생성하는 시스템을 쉽게 구현하고 싶다면 bcrypt를 사용하는 것이 좋습니다.
- SHA-2에 대한 공격은 2008년부터 발생하기 시작했고, SHA-2에 대한 공격에 점점 더 박차를 가하며 SHA-2마저 약화하고 있습니다. 일부 공격은 SHA-2의 유효 보호 수준을 237비트까지 낮췄고 2016년에 발표된 일부 최근 공격을 보면 SHA-2 공격은 이미 "실용" 단계에 있다고 할 수 있습니다.

### 출처
1. [학습중인 강의](https://www.inflearn.com/course/%EC%BD%94%EC%96%B4-%EC%8A%A4%ED%94%84%EB%A7%81-%EC%8B%9C%ED%81%90%EB%A6%AC%ED%8B%B0)
2. [위키백과 - bcrypt](https://ko.wikipedia.org/wiki/Bcrypt)
3. [위키백과 - LDAP](https://ko.wikipedia.org/wiki/LDAP)
4. [Wikipedia - MD4](https://en.wikipedia.org/wiki/MD4)
6. [위키백과 - MD5](https://ko.wikipedia.org/wiki/MD5)
7. [위키백과 - SHA-1](https://ko.wikipedia.org/wiki/SHA-1)
8. [Wikipedia - PBKDF2](https://en.wikipedia.org/wiki/PBKDF2)
9. [Wikipedia - Scrypt](https://en.wikipedia.org/wiki/Scrypt)
10. [개팔자 블로그 - 패스워드 암호화 - PBKDF2, scrypt, bcrypt, argon2](https://velog.io/@palza4dev/%ED%8C%A8%EC%8A%A4%EC%9B%8C%EB%93%9C-%EC%95%94%ED%98%B8%ED%99%94-PBKDF2-scrypt-bcrypt-argon2)
11. [위키백과 - 솔트 (암호학)](https://ko.wikipedia.org/wiki/%EC%86%94%ED%8A%B8_(%EC%95%94%ED%98%B8%ED%95%99))