---
layout: post
title:  "Admin Sample with Spring Boot and SBAdmin2"
date:   2019-05-16 18:05:00 +0900
categories: Backend
tag: SpringBoot
---

* content
{:toc}



# Overview
Spring Boot을 사용하여 Admin Sample Site를 작성해보았다.  
개발 환경은 
* Spring Boot 2.1.4.RELEASE
* Thymeleaf
  - Fragments
  - [Layout Dialect](https://ultraq.github.io/)
* [SBAdmin2](https://startbootstrap.com/themes/sb-admin-2/) 4.0.2
* Spring Security form login


# SBAdmin2 Template
[Template 다운로드](https://startbootstrap.com/themes/sb-admin-2/)  
![SBAdmin2]({{site.url}}/assets/images/2019-05/demo-sbadmin2-01.png)    

압축을 풀면  
![SBAdmin2]({{site.url}}/assets/images/2019-05/demo-sbadmin2-02.png)   

# login.html
Copy to src/main/resources/templates/ 
[webjars](https://www.webjars.org/) 적용
```
<script src="/webjars/bootstrap/4.3.1/js/bootstrap.bundle.min.js"></script>
```
localhost:8080 접속
![login]({{site.url}}/assets/images/2019-05/demo-sbadmin2-03.png)   

# Spring Security
## maven dependency
```
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-security</artifactId>
    </dependency>
    
    <dependency>
      <groupId>org.springframework.security</groupId>
      <artifactId>spring-security-test</artifactId>
      <scope>test</scope>
    </dependency>    
```
## Domain, Repository
### User.java
```
@Entity
@Table(name = "tu_user")
@Getter @Setter
@ToString
@EqualsAndHashCode(of = "id")
public class User implements Serializable {

  private static final long serialVersionUID = 7535937185214543104L;

  @Id
  @GeneratedValue
  private Long id;
  private String username;

  private String password;

  @ElementCollection(targetClass = Authority.class)
  @JoinTable(name = "tu_user_authorities", joinColumns = {@JoinColumn(name = "user_id")})
  @Column(name = "authority", nullable = false)
  @Enumerated(EnumType.STRING)
  private Set<Authority> authorities = new HashSet<>();
}
```

### Authority.java
```
public enum Authority {
  ADMIN, USER
}
```
Authority를 Entity(Table)로 한적도 있으나, 이 값은 변경되거나 추가될 일이 없으므로 Enum으로 처리  
역할에 따른 권한관리를 해야 할 경우 Role Entity를 추가하고 User - Role -Authority 구조를 만들면 된다.  
간단한 샘플이므로 여기에서는 Role을 두지 않았다.  

### UserRepository.java
```
public interface UserRepository extends JpaRepository<User, Long> {
  User findByUsername(String username);
}
```
## SpringSecurity에 필요한 UserDetails, UserDetailsService

User가 UserDetails를 implements 하지 않고 별도로 분리  
 -> 세션데이터와 DB 데이터 불일치 이슈 및 불필요한 데이터가 세션에 담기는 문제를 해결하기 위해  
 JWT Token Authentication을 구현할 경우 Token 생성시 사용된다.

### AuthUser.java
```
@Getter
@ToString
@NoArgsConstructor
@RequiredArgsConstructor
public class AuthUser implements UserDetails {

  public static final String AUTHORITY_SEPERATOR = ",";
  
  @NonNull
  private Long id;
  
  @NonNull
  private String authority;
  
  private String username;
  private String password;
  protected boolean enabled;
  private Set<GrantedAuthority> authorities;
  
  public AuthUser(User user) {
    this.id = user.getId();
    this.username = user.getUsername();
    this.password = user.getPassword();
    this.authorities = new HashSet<>();
    user.getAuthorities().stream().forEach(authority -> authorities.add((GrantedAuthority) authority::name));
    this.authority = authorities.stream().map(GrantedAuthority::getAuthority).collect(Collectors.joining(","));
  }
  
  @Override
  public boolean isAccountNonExpired() {
    return true;
  }
  
  @Override
  public boolean isAccountNonLocked() {
    return true;
  }
  
  @Override
  public boolean isCredentialsNonExpired() {
    return true;
  }
  
  @Override
  public boolean isEnabled() {
    return true;
  }
}

```

### UserDetailsServiceImpl.java
```
@Slf4j
public class UserDetailsServiceImpl implements UserDetailsService {
  @Autowired
  private UserRepository userRepository;

  @Override
  @Transactional(readOnly = true)
  public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
    User user = userRepository.findByUsername(username);
    return new AuthUser(user);
  }
}
```

### SecurityConfig.java
```
@Configuration
public class SecurityConfig extends WebSecurityConfigurerAdapter {
  @Override
  public void configure(WebSecurity web) {
    web.ignoring().antMatchers("/js/**", "/css/**", "/webjars/**", "/error","/favicon.ico");
  }

  protected void configure(HttpSecurity http) throws Exception {
    // @formatter:off
    http
      .authorizeRequests()
        .antMatchers("/login").permitAll()
        .anyRequest().authenticated()
        .and()
        .formLogin()
          .loginPage("/login")
        .and()
      .csrf().disable();
    // @formatter:on
  }

  @Bean
  public UserDetailsServiceImpl userDetailsService() {
    return new UserDetailsServiceImpl();
  }

  @Bean
  public PasswordEncoder passwordEncoder() {
    return NoOpPasswordEncoder.getInstance();
  }

}
```
passwordEncoder는 테스트 데이터(data.sql) 이용시 편하게 하기 위해 평문을 사용

## 초기데이터
### data.sql
src/main/resources/data.sql
```
insert into tu_user (id, username, password) values(1, 'admin', 'admin');
insert into tu_user_authorities(user_id, authority) values (1, 'ADMIN');
```

### application.properties
src/main/resources/application.properties
```
# Logging
logging.level.com.example.demo=trace
logging.level.org.hibernate.SQL=debug
logging.level.org.hibernate.type.descriptor.sql.BasicBinder=trace
logging.level.org.hibernate.type.EnumType=trace
# H2
spring.h2.console.enabled=true
spring.h2.console.path=/h2-console
#spring.datasource.url=jdbc:h2:mem:demo
#spring.datasource.username=sa
#spring.datasource.password=
# DATASOURCE - data
spring.datasource.sqlScriptEncoding=UTF-8
#spring.datasource.data=classpath:/data.sql
spring.datasource.initialization-mode=always
#spring.datasource.continue-on-error=false
# JPA
spring.jpa.properties.hibernate.hbm2ddl.auto=create
spring.jpa.properties.hibernate.id.new_generator_mappings=false
```
## login page
로그인 처리를 위해서 수정  
[https://spring.io/guides/gs/securing-web/](https://spring.io/guides/gs/securing-web/)  

### login.html
```
...
                  <form id="login" class="user" th:action="@{/login}" method="post">
                    <div class="form-group">
                      <input type="email" class="form-control form-control-user" id="username" name="username" aria-describedby="emailHelp" placeholder="Enter Email Address...">
                    </div>
                    <div class="form-group">
                      <input type="password" class="form-control form-control-user" id="password" name="password" placeholder="Password">
                    </div>
...
                    <a href="#" class="btn btn-login btn-primary btn-user btn-block">
                      Login
                    </a>
...
  <script src="./js/login.js"></script>
```

### login.js
```
$(function (){
  // login
  $('.btn-login').on('click', function (event){
    event.preventDefault();
    console.log("login");
    $('#login').submit();
  });
});
```

지금까지 SBAdmin2 bootstrap template를 사용해서 어드민 로그인 기능을 구현해보았다. 
Thymeleaf Fragments/Layout Dialect를 활용해서 page를 구성하는 방법은 [여기]({{site.url}}/2019/05/16/demo-thymeleaf-layout/)에서 다룰 예정이다.

[Github](https://github.com/jhkim105/demo-sbadmin)