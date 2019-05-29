---
layout: post
title:  "OneLogin SAML SSO #2"
date:   2019-05-23 20:00:00 +0900
categories: Backend
tag: SAML
---

* content
{:toc}



# Overview
OneLogin SSO 설정을 사용자마다 가능하도록 지난번에 작성한 코드를 수정해보았다.

# Model 추가
## SamlSetting.java
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
# Controller 수정
Login시 parameter로 username을 받아야 한다. SP Url들도 username을 추가해야 함.
## OneLoginController.java
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

# OneLogin Configuration
![onelogin sdk]({{site.url}}/assets/images/2019-05/onelogin-saml-2-01.png)   
Required로 된 부분외엔 설정 제거. 코드로 전달하면 됨  
Single Logout URL에 값이 설정되어 있으면 코드로 전달한 값이 적용 안됨
ACS URL에 username 추가 -> 이 설정은 OneLogin  App에서 사용됨  
![onelogin sdk]({{site.url}}/assets/images/2019-05/onelogin-saml-2-02.png)  