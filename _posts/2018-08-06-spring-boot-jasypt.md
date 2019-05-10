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
# 방법1 - Customize DataSource
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