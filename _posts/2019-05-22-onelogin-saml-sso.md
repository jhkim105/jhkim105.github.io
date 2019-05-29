---
layout: post
title:  "OneLogin SAML SSO"
date:   2019-05-22 20:00:00 +0900
categories: Backend
tag: SAML
---

* content
{:toc}



# Overview
[OneLogin](https://www.onelogin.com/)에서 제공하는 SDK를 사용하여 OneLogin SAML SSO 연동하는 방법에 대해 알아보자. 

# OneLogin SDK
Java SDK를 제공한다.  
[https://developers.onelogin.com/saml/java](https://developers.onelogin.com/saml/java)  
프로젝트를 살펴보면 core와 sample 코드로 구성되어 있다.  
![onelogin sdk]({{site.url}}/assets/images/2019-05/onelogin-saml-01.png) 
samples는 jsp로 구현되어 있음.

# Create Account
테스트를 위해서 계정을 만들어야 한다. 일반 계정은 30일 Trial인데, 개발자 계정을 생성할 수 있다.  
## Developer account
[https://www.onelogin.com/developer-signup](https://www.onelogin.com/developer-signup)
![onelogin-account]({{site.url}}/assets/images/2019-05/onelogin-saml-02.png) 

이메일 인증 후 로그인
![onelogin login]({{site.url}}/assets/images/2019-05/onelogin-saml-03.png) 

설정 화면
![onelogin login]({{site.url}}/assets/images/2019-05/onelogin-saml-04.png) 

최종 화면
![onelogin console]({{site.url}}/assets/images/2019-05/onelogin-saml-05.png) 


# Service Provider 만들기
onelogin sdk를 사용하여 SP Sample project를 만든다.
```
        <dependency>
            <groupId>com.onelogin</groupId>
            <artifactId>java-saml</artifactId>
            <version>2.4.0</version>
        </dependency>
```
SP로 등록하기 위해서는 다음과 같은 페이지들이 필요하다. OneLogin SDK에서 제공하는 sample code를 보고 구현하였다.
## metadata
[sample jsp](https://github.com/onelogin/java-saml/blob/v2.4.0/samples/java-saml-tookit-jspsample/src/main/webapp/metadata.jsp)

```
 @RequestMapping("/metadata")
  @ResponseBody
  public String metadata() {
    try {
      Auth auth = new Auth();
      Saml2Settings settings = auth.getSettings();
      settings.setSPValidationOnly(true);
      String metadata = settings.getSPMetadata();
      List<String> errors = Saml2Settings.validateMetadata(metadata);
      if (errors.isEmpty()) {
        return metadata;
      } else {
        return StringUtils.join(errors, System.lineSeparator()); // TODO
      }
    } catch (Exception e) {
      throw new RuntimeException(e);
    }
  }
```
## login
[sample jsp](https://github.com/onelogin/java-saml/blob/v2.4.0/samples/java-saml-tookit-jspsample/src/main/webapp/dologin.jsp)
```
  @RequestMapping("/login")
  public ResponseEntity login(HttpServletRequest request, HttpServletResponse response) {
    try {
      Auth auth = new Auth(request, response);
      auth.login();
      return new ResponseEntity(HttpStatus.OK);
    } catch (Exception e) {
      throw new RuntimeException(e);
    }
  }
```

## acs(Assertion Consumer Service)
[sample jsp]((https://github.com/onelogin/java-saml/blob/v2.4.0/samples/java-saml-tookit-jspsample/src/main/webapp/acs.jsp))
```
  @RequestMapping("/acs")
  public String acs(HttpServletRequest request, HttpServletResponse response, Model model) {
    try {
      Auth auth = new Auth(request, response);
      auth.processResponse();
      if (!auth.isAuthenticated()) {
        throw new RuntimeException("Not Authenticated");
      }
      List<String> errors = auth.getErrors();
      if (errors.isEmpty()) {
        String relayState = request.getParameter("RelayState");
        if (relayState != null && !relayState.isEmpty() && !relayState.equals(ServletUtils.getSelfRoutedURLNoQuery(request)) &&
            !relayState.contains("/onelogin/login")) { // We don't want to be redirected to login neither
          log.debug("redirect to login, relayState:{}", relayState);
          response.sendRedirect(request.getParameter("RelayState"));
        } else {
          Map<String, List<String>> attributes = auth.getAttributes();
          model.addAttribute("attributes", attributes);
          log.debug("attributes -> {}", attributes);
          printAttribute(attributes);
        }
      } else {
        model.addAttribute("errors", errors);
        log.debug("errors -> {}", errors);
      }
      return "onelogin/acs";
    } catch(Exception e) {
      throw new RuntimeException(e);
    }
  }
  ```

## logout
[sample jsp](https://github.com/onelogin/java-saml/blob/v2.4.0/samples/java-saml-tookit-jspsample/src/main/webapp/dologout.jsp)
```
  @RequestMapping("/logout")
  @ResponseBody
  public ResponseEntity logout(HttpServletRequest request, HttpServletResponse response, HttpSession session) {
    try {
      Auth auth = new Auth(request, response);
      String nameId = null;
      if (session.getAttribute("nameId") != null) {
        nameId = session.getAttribute("nameId").toString();
      }
      String nameIdFormat = null;
      if (session.getAttribute("nameIdFormat") != null) {
        nameIdFormat = session.getAttribute("nameIdFormat").toString();
      }
      String nameidNameQualifier = null;
      if (session.getAttribute("nameidNameQualifier") != null) {
        nameIdFormat = session.getAttribute("nameidNameQualifier").toString();
      }
      String nameidSPNameQualifier = null;
      if (session.getAttribute("nameidSPNameQualifier") != null) {
        nameidSPNameQualifier = session.getAttribute("nameidSPNameQualifier").toString();
      }
      String sessionIndex = null;
      if (session.getAttribute("sessionIndex") != null) {
        sessionIndex = session.getAttribute("sessionIndex").toString();
      }
      auth.logout(null, nameId, sessionIndex, nameIdFormat, nameidNameQualifier, nameidSPNameQualifier);
      return new ResponseEntity(HttpStatus.OK);
    } catch (Exception e) {
      throw new RuntimeException(e);
    }
  }
  ```

## sls(Singl Logout Service)
[sample jsp](https://github.com/onelogin/java-saml/blob/v2.4.0/samples/java-saml-tookit-jspsample/src/main/webapp/sls.jsp)
```
  @RequestMapping("/sls")
  public void sls(HttpServletRequest request, HttpServletResponse response) {
    try {
      Auth auth = new Auth(request, response);
      auth.processSLO();
      PrintWriter out = response.getWriter();
      List<String> errors = auth.getErrors();
      if (errors.isEmpty()) {
        out.println("<p>Sucessfully logged out</p>");
        out.println("<a href=\"dologin.jsp\" class=\"btn btn-primary\">Login</a>");
      } else {
        out.println("<p>");
        for (String error : errors) {
          out.println(" " + error + ".");
        }
        out.println("</p>");
      }
    } catch(Exception e) {
      throw new RuntimeException(e);
    }
  }
```

## Idp 정보 SP에 Setting하기
OneLogin Console에서 SSO정보를 조회해서 property에 적용
![onelogin sso]({{site.url}}/assets/images/2019-05/onelogin-saml-06.png) 

onelogin.saml.properties
```
# IDP
#onelogin.saml2.idp.entityid=https://app.onelogin.com/saml/metadata/
8a41fc95-40d0-4eff-be5f-5573580e70c7
#onelogin.saml2.idp.single_sign_on_service.url=https://rsupport-dev.onelogin.com/trust/saml2/http-post/sso/936070
#onelogin.saml2.idp.single_logout_service.url=https://rsupport-dev.onelogin.com/trust/saml2/http-redirect/slo/936070
#onelogin.saml2.idp.x509cert=-----BEGIN CERTIFICATE-----MIID3zCCAsegAwIBAgIUEdjrgxCKuxks/HL5wSfb8omgCZcwDQYJKoZIhvcNAQEFBQAwRjERMA8GA1UECgwIUnN1cHBvcnQxFTATBgNVBAsMDE9uZUxvZ2luIElkUDEaMBgGA1UEAwwRT25lTG9naW4gQWNjb3VudCAwHhcNMTkwNTIxMDgyODM1WhcNMjQwNTIxMDgyODM1WjBGMREwDwYDVQQKDAhSc3VwcG9ydDEVMBMGA1UECwwMT25lTG9naW4gSWRQMRowGAYDVQQDDBFPbmVMb2dpbiBBY2NvdW50IDCCASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEBANe0HfD5PuT0hW+XFQ/+CuBSxB5I1pYMr8DeS0PbiDRbLmgn7dqoyY9C9eLsb73iOLxsPRp9a8TSI4R/HCGKDumr00BiX7pr1tBu9Nf99BC4aBE3aV0+QzWDfLTY4zbaU+/uZUBhTqIehAFY8xmQ/99CgE+zCFsu2OOPfQ9ErsPYLdFy4XlSsao4DcdycZqAwg53DaR9j+zCT7UoT+dxZ33ai3qcqUKOfM+ygf4TzE4Z0H6EG5UxFEzoPCTRbwpYmoOyP0R2pChGkvKTkL6XAES9S2BLKRyLcOVhOu2mogMQQzTZx8BdK2EDNzqDZQs/2vrmCXf6SGN8T3hWY+U/ze0CAwEAAaOBxDCBwTAMBgNVHRMBAf8EAjAAMB0GA1UdDgQWBBSa+L8CeXbiwKQrLi3ukhlT097J+TCBgQYDVR0jBHoweIAUmvi/Anl24sCkKy4t7pIZU9PeyfmhSqRIMEYxETAPBgNVBAoMCFJzdXBwb3J0MRUwEwYDVQQLDAxPbmVMb2dpbiBJZFAxGjAYBgNVBAMMEU9uZUxvZ2luIEFjY291bnQgghQR2OuDEIq7GSz8cvnBJ9vyiaAJlzAOBgNVHQ8BAf8EBAMCB4AwDQYJKoZIhvcNAQEFBQADggEBANLrPh1Z1Aa/v5hep9zPoOs7uoWjJBgawlpT+uNGMl16uh3FSQov35ZMGYCGoQjtpaEH5ysSLrTf1UIDsKT5HQHfGtWurxJ9v1S7VUyc6L1fCETVhBBqHLGXWYpPgRtNTjtKWFBopFTdtVAd/bW2h5XT9bDMGHJQ69tCf7xTkYcnVkspiraO1gfiO8CsqFFjG0hnUOTyIHENoPbV3BK8rnVl76BnDT3w6Zu19PvLklOHzQdt7iujl3t/aCTFhVrZg+RtWZPjBzL/ZsOsRBZmHTUfK+hIlM4/xT6EPaXAtHrpj8QJmwhA8fbTyp138gQ0AK5aEA7IlQ6X/8BXo4l1XMM=-----END CERTIFICATE-----
#SP values
#onelogin.saml2.sp.entityid=http://localhost:8080/onelogin/metadata
#onelogin.saml2.sp.assertion_consumer_service.url=http://localhost:8080/onelogin/acs
#onelogin.saml2.sp.single_logout_service.url=http://localhost:8000/onelogin/sls
```


# SP 등록하기
OneLogin에 SP 정보를 등록한다
![onelogin setting]({{site.url}}/assets/images/2019-05/onelogin-saml-07.png) 

localhost:8080/onelogin/login 접속
![sp login]({{site.url}}/assets/images/2019-05/onelogin-saml-08.png)  
![sp login]({{site.url}}/assets/images/2019-05/onelogin-saml-09.png) 

# 사용자 마다 SSO 정보 설정하기
SAML sample들이 SSO 정보를 하나만 설정할 수 있도록 되어 있는데, 실제 서비스에 적용하려면 고객사(유저) 별로 설정이 가능해야 한다. 코드를 수정해보자.

## Model 추가
### SamlSetting.java
```
@Entity
@Table(name = "tu_saml_setting")
@Getter @Setter
@ToString
@EqualsAndHashCode(of = "id")
public class SamlSetting implements Serializable {
 
  private static final long serialVersionUID = -7169474063609963047L;
 
  @Id
  @GeneratedValue(generator = "system-uuid")
  @GenericGenerator(name = "system-uuid", strategy = "uuid2")
  @Column(length = 50)
  private String id;

  @Column(name = "entity_id")
  private String entityId;

  @Column(name = "sso_url")
  private String ssoUrl;

  @Column(name = "slo_url")
  private String sloUrl;

  @Lob
  private String cert;

  @OneToOne
  private Company company;

}
```
프로퍼티에 설정한 값을 Table에 저장한다.  
data.sql
```
INSERT INTO tu_saml_setting (id, company_id, entity_id, sso_url, slo_url, cert) VALUES ('cert-uuid-01', 'company-uuid-01', 'https://app.onelogin.com/saml/metadata/8a41fc95-40d0-4eff-be5f-5573580e70c7', 'https://rsupport-dev.onelogin.com/trust/saml2/http-post/sso/936070', 'https://rsupport-dev.onelogin.com/trust/saml2/http-redirect/slo/936070', '-----BEGIN CERTIFICATE-----MIID3zCCAsegAwIBAgIUEdjrgxCKuxks/HL5wSfb8omgCZcwDQYJKoZIhvcNAQEFBQAwRjERMA8GA1UECgwIUnN1cHBvcnQxFTATBgNVBAsMDE9uZUxvZ2luIElkUDEaMBgGA1UEAwwRT25lTG9naW4gQWNjb3VudCAwHhcNMTkwNTIxMDgyODM1WhcNMjQwNTIxMDgyODM1WjBGMREwDwYDVQQKDAhSc3VwcG9ydDEVMBMGA1UECwwMT25lTG9naW4gSWRQMRowGAYDVQQDDBFPbmVMb2dpbiBBY2NvdW50IDCCASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEBANe0HfD5PuT0hW+XFQ/+CuBSxB5I1pYMr8DeS0PbiDRbLmgn7dqoyY9C9eLsb73iOLxsPRp9a8TSI4R/HCGKDumr00BiX7pr1tBu9Nf99BC4aBE3aV0+QzWDfLTY4zbaU+/uZUBhTqIehAFY8xmQ/99CgE+zCFsu2OOPfQ9ErsPYLdFy4XlSsao4DcdycZqAwg53DaR9j+zCT7UoT+dxZ33ai3qcqUKOfM+ygf4TzE4Z0H6EG5UxFEzoPCTRbwpYmoOyP0R2pChGkvKTkL6XAES9S2BLKRyLcOVhOu2mogMQQzTZx8BdK2EDNzqDZQs/2vrmCXf6SGN8T3hWY+U/ze0CAwEAAaOBxDCBwTAMBgNVHRMBAf8EAjAAMB0GA1UdDgQWBBSa+L8CeXbiwKQrLi3ukhlT097J+TCBgQYDVR0jBHoweIAUmvi/Anl24sCkKy4t7pIZU9PeyfmhSqRIMEYxETAPBgNVBAoMCFJzdXBwb3J0MRUwEwYDVQQLDAxPbmVMb2dpbiBJZFAxGjAYBgNVBAMMEU9uZUxvZ2luIEFjY291bnQgghQR2OuDEIq7GSz8cvnBJ9vyiaAJlzAOBgNVHQ8BAf8EBAMCB4AwDQYJKoZIhvcNAQEFBQADggEBANLrPh1Z1Aa/v5hep9zPoOs7uoWjJBgawlpT+uNGMl16uh3FSQov35ZMGYCGoQjtpaEH5ysSLrTf1UIDsKT5HQHfGtWurxJ9v1S7VUyc6L1fCETVhBBqHLGXWYpPgRtNTjtKWFBopFTdtVAd/bW2h5XT9bDMGHJQ69tCf7xTkYcnVkspiraO1gfiO8CsqFFjG0hnUOTyIHENoPbV3BK8rnVl76BnDT3w6Zu19PvLklOHzQdt7iujl3t/aCTFhVrZg+RtWZPjBzL/ZsOsRBZmHTUfK+hIlM4/xT6EPaXAtHrpj8QJmwhA8fbTyp138gQ0AK5aEA7IlQ6X/8BXo4l1XMM=-----END CERTIFICATE-----');
```
## Controller 수정
Login시 parameter로 username을 받아야 한다. SP Url들도 username을 추가해야 함.
### OneLoginController.java
```
  @RequestMapping("/login")
  @ResponseBody
  public ResponseEntity login(@RequestParam String username,  HttpServletRequest request, HttpServletResponse response) {
    try {
      Saml2Settings saml2Settings = loadSaml2Settings(username);
      Auth auth = new Auth(saml2Settings, request, response);
      auth.login();
      return new ResponseEntity(HttpStatus.OK);
    } catch (Exception e) {
      throw new RuntimeException(e);
    }
  }
  private Saml2Settings loadSaml2Settings(String username) {
    User user = userRepository.findByUsername(username);
    Properties prop = new Properties();
    loadSp(prop, username);
    loadIdp(prop, user);
    return new SettingsBuilder().fromProperties(prop).build();
  }
  private void loadSp(Properties prop, String username) {
    prop.setProperty(SettingsBuilder.SP_ENTITYID_PROPERTY_KEY, "http://localhost:8080/onelogin/metadata?username=" + username);
    prop.setProperty(SettingsBuilder.SP_ASSERTION_CONSUMER_SERVICE_URL_PROPERTY_KEY, "http://localhost:8080/onelogin/acs?username=" + username);
    prop.setProperty(SettingsBuilder.SP_SINGLE_LOGOUT_SERVICE_URL_PROPERTY_KEY, "http://localhost:8080/onelogin/sls?username=" + username");
  }
  private void loadIdp(Properties prop, User user) {
    SamlSetting samlSetting = user.getSamlSetting();
    prop.setProperty(SettingsBuilder.IDP_ENTITYID_PROPERTY_KEY, samlSetting.getEntityId());
    prop.setProperty(SettingsBuilder.IDP_SINGLE_SIGN_ON_SERVICE_URL_PROPERTY_KEY, samlSetting.getSsoUrl());
    prop.setProperty(SettingsBuilder.IDP_SINGLE_LOGOUT_SERVICE_URL_PROPERTY_KEY, samlSetting.getSloUrl());
    prop.setProperty(SettingsBuilder.IDP_X509CERT_PROPERTY_KEY, samlSetting.getCert());
  }
```
다른 메소드들도 다음 코드를 변경해야 함.
```
Auth auth = new Auth();
```
->
```
Saml2Settings saml2Settings = loadSaml2Settings(username);
Auth auth = new Auth(saml2Settings, request, response);
```

## OneLogin Configuration
![onelogin sdk]({{site.url}}/assets/images/2019-05/onelogin-saml-2-01.png)   
Required로 된 부분외엔 설정 제거. 코드로 전달하면 됨  
Single Logout URL에 값이 설정되어 있으면 코드로 전달한 값이 적용 안됨
ACS URL에 username 추가 -> 이 설정은 OneLogin  App에서 사용됨  
![onelogin sdk]({{site.url}}/assets/images/2019-05/onelogin-saml-2-02.png)  


# References
[OneLogin](https://developers.onelogin.com/saml/java)



