---
layout: post
title:  "Azure Active Directory SAML SSO"
date:   2019-05-27 20:00:00 +0900
categories: Backend
tag: SAML
---

* content
{:toc}



# Overview
OneLogin SDK를 사용하여 OneLogin SAML SSO SP Sample을 구현해보았다. 여기에 Azure Active Directory SSO를 적용해보자.

# SP 수정
Azure를 처리할 수 있도록 하려면 Saml Setting시 IdP 별로 설정값이 관리되어야 한다.  
URL에 Idp 구분자가 추가되어야 한다.  
  - /saml/onelogin/login  
  - /saml/azure/login  


## SamlSetting.java
Idp enum field 추가  
unique index 추가  
![SamlSetting]({{site.url}}/assets/images/2019-05/azure-saml-sso-01.png)  

## Company.java
Company에서 IdP 별로 SamlSetting을 가질 수 있다. (OneToOne -> OneToMany)  
![Company]({{site.url}}/assets/images/2019-05/azure-saml-sso-02.png)  

## SamlController.java
![SamlController]({{site.url}}/assets/images/2019-05/azure-saml-sso-03.png)   

## OneLogin Config 적용 후 테스트
http://localhost:8080/saml/onelogin/login?username=admin  
![OneLogin Test]({{site.url}}/assets/images/2019-05/azure-saml-sso-04.png)   

# AZure에서 SP 등록
## 비 갤러리 응용 프로그램 만들기
![AZure]({{site.url}}/assets/images/2019-05/azure-saml-sso-05.png)   
![AZure 응용프로그램 만들기]({{site.url}}/assets/images/2019-05/azure-saml-sso-06.png)   

## SSO 설정하기
![SamlSetting]({{site.url}}/assets/images/2019-05/azure-saml-sso-07.png)   
![SamlSetting]({{site.url}}/assets/images/2019-05/azure-saml-sso-08.png) 
ACS URL은 https여야 함  
![SamlSetting]({{site.url}}/assets/images/2019-05/azure-saml-sso-09.png)   

## 인증서 다운로드 및 SP에 등록
![SamlSetting]({{site.url}}/assets/images/2019-05/azure-saml-sso-10.png)   

## 사용자 등록
![SamlSetting]({{site.url}}/assets/images/2019-05/azure-saml-sso-11.png)   
![SamlSetting]({{site.url}}/assets/images/2019-05/azure-saml-sso-12.png)   

## SSO Test
![SamlSetting]({{site.url}}/assets/images/2019-05/azure-saml-sso-13.png)   
  
현재 사용자로 로그인  
![SamlSetting]({{site.url}}/assets/images/2019-05/azure-saml-sso-14.png)    
  
acs로 리다이렉트 됨 
![SamlSetting]({{site.url}}/assets/images/2019-05/azure-saml-sso-15.png)   