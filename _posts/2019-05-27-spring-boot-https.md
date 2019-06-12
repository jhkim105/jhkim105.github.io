---
layout: post
title:  "Spring Booot Https"
date:   2019-05-27 20:00:00 +0900
categories: Backend
tag: SpringBoot
---

* content
{:toc}


# Overview
Local에서 개발하다면 https를 서비스해야 되는 경우가 있다. Spring Boot에서 https로 구성하는 방법에 대해 알아보자.

작업 순서는 다음과 같다
1. KeyStore 생성
1. 인중서 추출
1. TrustStore 생성
1. Spring Boot에 적용

# 인증서 만들기
## KeyStore 생성
```
keytool -genkey -alias <alias> -keyalg RSA -keystore <file name>
```
keytool -genkey -alias localKeyStore -keyalg RSA -keystore local.jks
![keystore]({{site.url}}/assets/images/2019-05/spring-boot-https-01.png) 

![keystore]({{site.url}}/assets/images/2019-05/spring-boot-https-02.png)  

## 인증서 추출
TrustStore를 생성하기 위해서는 KeyStore에서 인증서를 추출해야 함.
```
keytool -export -alias <keystore's alias> -keystore <kestore's file name> -rfc -file <cert file name>
```
keytool -export -alias localKeyStore -keystore local.jks -rfc -file local.cer
![cert]({{site.url}}/assets/images/2019-05/spring-boot-https-03.png)  

## TrustStore
TrustStore는 클라언트가 신뢰할 수 있는 인증서만을 가지고 있다.  
```
keytool -import -alias <truststore's alias> -file <cert file name> - keystore <truststore file name>
```
keytool -import -alias localTrustStore -file local.cer -keystore local.ts
![truststore]({{site.url}}/assets/images/2019-05/spring-boot-https-04.png)  

# Spring Boot 인증서 적용하기
위에서 생성한 KeyStore, TrustStore를 사용해서 SSL을 구성한다.
## application.properties
```
server.port=8443
server.ssl.enabled=true
server.ssl.key-store=@project.basedir@/src/main/resources/local.jks
server.ssl.key-store-password=dev#4430
server.ssl.key-password=dev#4430
server.ssl.key-alias=localKeyStore
server.ssl.trust-store=@project.basedir@/src/main/resources/local.ts
server.ssl.trust-store-password=dev#443
```
https 접속
![https]({{site.url}}/assets/images/2019-05/spring-boot-https-05.png)  

## HTTP 추가
Spring은 기본적으로 http, https 한가지만 설정가능하므로, 둘 다 서비스 하려면 다음과 같은 작업이 필요하다.
application.properties
```
server.http.port=8080
```

TomcatConfig.java
```
@Configuration
public class TomcatConfig {
@Value("${server.http.port}")
private int httpPort;
    @Bean
    public ServletWebServerFactory servletContainer() {
        TomcatServletWebServerFactory tomcat = new TomcatServletWebServerFactory();
        tomcat.addAdditionalTomcatConnectors(createHttpConnector());
        return tomcat;
    }
    private Connector createHttpConnector() {
        Connector httpConnector = new Connector(TomcatServletWebServerFactory.DEFAULT_PROTOCOL);
        httpConnector.setPort(httpPort);
        httpConnector.setSecure(false);
        httpConnector.setAllowTrace(false);
        httpConnector.setScheme("http");
        return httpConnector;
    }
}
```


