---
layout: post
title:  "property 암호화 하기"
date:   2018-08-06 20:00:00 +0900
categories: Backend
tag: SpringBoot
---

* content
{:toc}


프로퍼티에 설정된 DB password를 암호화할 필요가 있을 경우 DataSource를 customize에서 암호화를 적용했는데, spring-boot-jasypt를 사용하면 프로퍼티 암호화를 쉽게 할 수 있다.


------------------------
## 방법1 - Customize DataSource
DB Password를 암호화하기 위해 DataSource를 custoimze
CryptoDataSource.java
```
public class CryptoDataSource extends BasicDataSource {
  @Override
  public void setPassword(String password) {
    super.setPassword(decrypt(password));
  }
  private String decrypt(String str) {
    if (str == null)
      return null;
    try {
      return RsCrypto.getInstance().decrypt(str);
    } catch (SystemException e) {
      return str;
    }
  }
}
```

JpaConfig.java
```
 @Bean
  public DataSource dataSource() {
    CryptoDataSource dataSource = new CryptoDataSource();
    dataSource.setDriverClassName(env.getProperty("spring.datasource.driver-class-name"));
    dataSource.setUrl(env.getProperty("spring.datasource.url"));
    dataSource.setUsername(env.getProperty("spring.datasource.username"));
    dataSource.setPassword(env.getProperty("spring.datasource.password"));
    dataSource.setMaxTotal(Integer.parseInt(env.getProperty("spring.datasource.dbcp2.max-total", "100")));
    dataSource.setMaxWaitMillis(Integer.parseInt(env.getProperty("spring.datasource.dbcp2.max-wait-millis", "3000")));
    dataSource.setInitialSize(Integer.parseInt(env.getProperty("spring.datasource.dbcp2.initial-size")));
    dataSource.setPoolPreparedStatements(true);
    dataSource.setDefaultAutoCommit(true);
    dataSource.setValidationQuery(env.getProperty("spring.datasource.dbcp2.validation-query"));
    dataSource.setTestOnBorrow(true);
    return dataSource;
  }
```

# 방법2 - spring-boot-jasypt
spring-boot-jasypt를 사용하면 코드 변경 없이 원하는 프로퍼티를 암호화할 수 있다.  

jasypt-spring-boot dependency 추가
```
    <dependency>
      <groupId>com.github.ulisesbocchio</groupId>
      <artifactId>jasypt-spring-boot</artifactId>
      <version>1.17</version>
    </dependency>
```

PropertyEncryptConfig.java    
```
@EnableEncryptableProperties
@ConditionalOnProperty(value = "jasypt.encryptor.enabled", havingValue = "true")
public class PropertyEncryptConfig {

  public final static String A = "PBEWithMD5AndDES";

  public final static String B = "mpp123!";

  @Bean(name = "stringEncryptor")
  public StringEncryptor stringEncryptor() {
    PooledPBEStringEncryptor encryptor = new PooledPBEStringEncryptor();
    SimpleStringPBEConfig config = new SimpleStringPBEConfig();
    config.setAlgorithm(A);
    config.setPassword(B);
    config.setKeyObtentionIterations("1000");
    config.setPoolSize("1");
    config.setProviderName("SunJCE");
    config.setSaltGeneratorClassName("org.jasypt.salt.RandomSaltGenerator");
    config.setStringOutputType("base64");
    encryptor.setConfig(config);
    return encryptor;
  }
}

```
property에서 jasypt를 사용하도록 설정
```
jasypt.encryptor.enabled=true
jasypt.encryptor.bean=stringEncryptor
```

encrypted property 사용은 다음과 같은 방법으로 한다. 
ENC(암호화된 문자열)  
```
spring.datasource.password=ENC(59yxIc2Xt+Nv4vhdEJLrGUlMjX/fkX1K)
```

## Spring, Java Config 환경에서 적용하기 (2020/01/15)
리모트 미팅은 Spring Boot환경이 아니어서 위 방법으로는 적용이 안된다.  

Maven Dependency  
```
    <dependency>
      <groupId>org.jasypt</groupId>
      <artifactId>jasypt-spring31</artifactId>
      <version>1.9.2</version>
    </dependency>
```

PropertyEncryptConfig.java
```
@Configuration
public class PropertyEncryptConfig {

  private final static String P = "woori123!";
  private final static String ALG = "PBEWithMD5AndDES";


  @Bean
  public EnvironmentStringPBEConfig environmentVariablesConfiguration() {
    EnvironmentStringPBEConfig encryptorConfig = new EnvironmentStringPBEConfig();
    encryptorConfig.setAlgorithm(ALG);
    encryptorConfig.setPassword(P);
    return encryptorConfig;
  }

  @Bean
  public StandardPBEStringEncryptor stringEncryptor() {
    StandardPBEStringEncryptor encryptor = new StandardPBEStringEncryptor();
    encryptor.setConfig(environmentVariablesConfiguration());
    return encryptor;
  }

  @Bean
  public EncryptablePropertyPlaceholderConfigurer encryptablePropertyPlaceholderConfigurer() {
    EncryptablePropertyPlaceholderConfigurer encryptablePropertyPlaceholderConfigurer
        = new EncryptablePropertyPlaceholderConfigurer(stringEncryptor());
    Resource resource1 = new ClassPathResource("jdbc.properties");
    Resource resource2 = new FileSystemResource("${rm_home}/config/jdbc.properties");
    encryptablePropertyPlaceholderConfigurer.setLocations(resource1, resource2);
    encryptablePropertyPlaceholderConfigurer.setIgnoreResourceNotFound(true);
    encryptablePropertyPlaceholderConfigurer.setIgnoreUnresolvablePlaceholders(true);
    return encryptablePropertyPlaceholderConfigurer;
  }
```

복호화가 되지 않음. 기존 Placeholder 사용하는 부분이 문제 인듯 함.

다음과 같은 방법으로 필요한 프로퍼티만 적용

JpaConfig.java
```
@Autowired
  StringEncryptor stringEncryptor;


  @Bean
  public DataSource dataSource() {
    String username = getUsername(stringEncryptor);
    String password = getPassword(stringEncryptor);
    log.debug("username:{}", username);
    log.debug("password:{}", password);
    BasicDataSource dataSource = new BasicDataSource();
    dataSource.setDriverClassName(env.getProperty("hibernate.connection.driver_class"));
    dataSource.setUrl(env.getProperty("hibernate.connection.url"));
    dataSource.setUsername(username);
    dataSource.setPassword(password);
    dataSource.setMaxTotal(Integer.parseInt(env.getProperty("jdbc.maxActive")));
    dataSource.setMaxWaitMillis(Integer.parseInt(env.getProperty("jdbc.maxWait")));
    dataSource.setInitialSize(Integer.parseInt(env.getProperty("jdbc.initialSize")));
    dataSource.setPoolPreparedStatements(true);
    dataSource.setDefaultAutoCommit(true);
    dataSource.setRemoveAbandonedOnBorrow(BooleanUtils.toBoolean(env.getProperty("jdbc.removeAbandoned")));
    dataSource.setRemoveAbandonedOnMaintenance(BooleanUtils.toBoolean(env.getProperty("jdbc.removeAbandoned")));
    dataSource.setRemoveAbandonedTimeout(Integer.parseInt(env.getProperty("jdbc.removeAbandonedTimeout")));
    dataSource.setValidationQuery(env.getProperty("db.validationQuery"));
    dataSource.setTestOnBorrow(true);
    return dataSource;
  }


  private String getUsername(StringEncryptor stringEncryptor) {
    String username = env.getProperty("hibernate.connection.username");
    try {
      username = stringEncryptor.decrypt(username);
    } catch(Exception ex) {
      // ignored
      stringEncryptor.encrypt(username);
      log.warn("Use encryption property. orignal:[{}], encrypted:[{}]", username, stringEncryptor.encrypt(username));
    }
    return username;
  }


  private String getPassword(StringEncryptor stringEncryptor) {
    String password = env.getProperty("hibernate.connection.password");
    try {
      password = stringEncryptor.decrypt(password);
    } catch(Exception ex) {
      // ignored
      stringEncryptor.encrypt(password);
      log.warn("Use encryption property. orignal:[{}], encrypted:[{}]", password, stringEncryptor.encrypt(password));


    }
    return password;
  }




  @Bean(name = "stringEncryptor")
  public StringEncryptor stringEncryptor() {
    PooledPBEStringEncryptor encryptor = new PooledPBEStringEncryptor();
    SimpleStringPBEConfig config = new SimpleStringPBEConfig();
    config.setPassword(P);
    config.setAlgorithm(ALG);
    config.setKeyObtentionIterations("1000");
    config.setPoolSize("1");
    config.setProviderName("SunJCE");
    config.setSaltGeneratorClassName("org.jasypt.salt.RandomSaltGenerator");
    config.setStringOutputType("base64");
    encryptor.setConfig(config);
    return encryptor;
  }

```

Test Code
```
@Slf4j
public class JpaConfigTest extends BaseTestCase {

  @Autowired
  StringEncryptor stringEncryptor;

  @Test
  public void stringEncryptor() {
    log.debug("{}", stringEncryptor.encrypt("gosemffj#4430")); //Xwy/K15WzgnMGzzaUatkVIX9Pxx+kodY
    log.debug("{}", stringEncryptor.encrypt("rsup#4430")); //loWHRjvYGEblcSCuXk4H9OstrfXCjM97
    log.debug("{}", stringEncryptor.encrypt("root")); //c2QN97tylg4Xy4A2FCfUHA==
    log.debug("{}", stringEncryptor.decrypt("SqeUGVbuuC5iAQtHkj3XO92dan+TMu/M")); //rsup#4430
  }

}
```
