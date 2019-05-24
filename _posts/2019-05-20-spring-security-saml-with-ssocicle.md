---
layout: post
title:  "Spring Security Saml Sample with SSOCicle"
date:   2019-05-20 20:00:00 +0900
categories: Backend
tag: SAML
---

* content
{:toc}

# Overview
Spring Security SAML Extension을 사용해서 SAML Service Provider sample 실행해보기 

# spring-security-saml
현재 정식버전은 1.0.9.RELEASE이다. 이 버전에서 제공하는 샘플은 XML Configuration으로 작성되어 있다.  
 Java Configuration sample은 https://github.com/vdenotaris/spring-boot-security-saml-sample를 참고하라고 되어 있다. ([레퍼런스](https://docs.spring.io/spring-security-saml/docs/1.0.x/reference/htmlsingle/#configuration-java))

 현재(2018/05/20) develop 브랜치 버전은 2.0.0.BUILD-SNAPSHOT이고 Sample은 Java Configuration으로 개발되어 있다.  
 
 ![idp,sp sample]({{site.url}}/assets/images/2019-05/ssocicle-01.png)  

# Spring Boot SAML Sample
 [Github](https://github.com/vdenotaris/spring-boot-security-saml-sample)  
 dependency
 * Spring Boot 2.1.3.RELEASE
 * spring-security-saml2-core 1.0.3.RELEASE
 * IdP - [ssocicle.com](ssocicle.com)

## SSOCicle 계정 생성
 [SSOCicle](ssocicle.com) 사이트에서 계정 등록한다.  

## SP 등록
### 메타데이터 등록
![idp,sp sample]({{site.url}}/assets/images/2019-05/ssocicle-03.png)    
SP에서 entityId를 수정한 후 메타데이터 조회(http://localhost:8080/saml/metadata)후 붙여넣기  

![idp,sp sample]({{site.url}}/assets/images/2019-05/ssocicle-04.png)  

## SP에 KeyStore 생성
### 인증서 다운로드
![idp,sp sample]({{site.url}}/assets/images/2019-05/ssocicle-05.png)   
인증서를 다운로드하여 SP project의 src/main/resources/saml에 복사 

### update-certificate.cmd
테스트 환경이 Windows라서 [update-certficate.sh]( https://github.com/vdenotaris/spring-boot-security-saml-sample/blob/master/src/main/resources/saml/update-certifcate.sh)를 참조하여 cmd로 생성 후 실행함  
![idp,sp sample]({{site.url}}/assets/images/2019-05/ssocicle-06.png)    

정상적으로 실행된 화면
![idp,sp sample]({{site.url}}/assets/images/2019-05/ssocicle-07.png)   
![idp,sp sample]({{site.url}}/assets/images/2019-05/ssocicle-08.png)   
![idp,sp sample]({{site.url}}/assets/images/2019-05/ssocicle-09.png)   
