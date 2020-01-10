---
layout: post
title:  "Logging 정리 - logback"
date:   2019-05-14 19:21:00 +0900
categories: Backend
tag: SpringBoot
---

* content
{:toc}


# Overview
WebRTC API서버에 적용된 logback 설정을 검토하고 정리하는 작업을 진행

# As-Is
log 파일이 두개 생성됨 - api.log, spring.log  
spring.log는 롤링될때 사이즈가 줄어들지만, api.log는 계속 증가함  
![Logging File]({{site.url}}/assets/images/2019-05/logback-01.png)  

## appplication.yml  
![application.yml]({{site.url}}/assets/images/2019-05/logback-02.png)  

## logback-spring.xml
![application.yml]({{site.url}}/assets/images/2019-05/logback-03.png)  
spring.log 파일이 생성되는 이유는 base.xml을 include에서 그렇다. [base.xml](https://github.com/spring-projects/spring-boot/blob/master/spring-boot-project/spring-boot/src/main/resources/org/springframework/boot/logging/logback/base.xml)을 살펴보면 INFO level에 spring.log 파일로 생성되도록 설정되어 있다. 
base.xml
```
<?xml version="1.0" encoding="UTF-8"?>
<!--
Base logback configuration provided for compatibility with Spring Boot 1.1
-->
<included>
    <include resource="org/springframework/boot/logging/logback/defaults.xml" />
    <property name="LOG_FILE" value="${LOG_FILE:-${LOG_PATH:-${LOG_TEMP:-${java.io.tmpdir:-/tmp}}}/spring.log}"/>
    <include resource="org/springframework/boot/logging/logback/console-appender.xml" />
    <include resource="org/springframework/boot/logging/logback/file-appender.xml" />
    <root level="INFO">
        <appender-ref ref="CONSOLE" />
        <appender-ref ref="FILE" />
    </root>
</included>
```
시스템에 적용된 logback-spring.xml에는 dailyRollingFileAppender로 DEBUG레벨을 출력하도록 설정되어 있다.이 부분을 file-appender.xml에 설정된 Appender를 사용해서 파일을 남기도록 설정해보자

# logback-spring.xml 수정
base.xml을 include하는 것을 defaults.xml을 include하도록 변경하고, 파일명, level등을 변경 적용한다. 그리고 불필요한 로그를 남기지 않도록 root level을 WARN으로 변경하고 실제 작성한 코드에 대한 로그레벨을 지정하도록 한다. 실제 운영시에는 이 부분도 WARN으로 변경한다.
```
<?xml version="1.0" encoding="UTF-8"?>
<configuration scan="true" scanPeriod="5 seconds">
  <include resource="org/springframework/boot/logging/logback/defaults.xml" />
  <property name="LOG_PATH" value="/DATA/WEB/webrtc/log"/>
  <property name="LOG_FILE" value="${LOG_PATH}/api.log}"/>
  <include resource="org/springframework/boot/logging/logback/console-appender.xml" />
  <include resource="org/springframework/boot/logging/logback/file-appender.xml" />
  <logger name="com.rsupport" level="DEBUG"/>
  <root level="WARN">
    <appender-ref ref="CONSOLE" />
    <appender-ref ref="FILE" />
  </root>
</configuration>
```
log file 지정
application.properties
```
logging.file=/DATA/WEB/webrtc/log/api.log
```  
logging.path로 directory만 지정하면 지정한 디렉토리에 spring.log로 로그파일이 생긴다.

이렇게 서비스에 적용해보니 spring.log는 더이상 기록되지 않고, api.log가 롤링되고, 파일 사이즈가 초기화 되었다. logrotate를 적용해서 롤링을 하지 않아도 되겠다.
![application.yml]({{site.url}}/assets/images/2019-05/logback-02.png)  

그런데 TRACE로그들이 출력된다. logback에는 root level을 WARN으로 셋팅했는데!!  
![application.yml]({{site.url}}/assets/images/2019-05/logback-06.png)  

이유는 application.yml에 설정된 logging.level 때문이다
![application.yml]({{site.url}}/assets/images/2019-05/logback-05.png)  

이것을 다음과 같이 지정해주니 원하는대로 더이상 TRACE로그가 출력되지 않았다.
![application.yml]({{site.url}}/assets/images/2019-05/logback-07.png)  

사실 Query를 출력하기 위한 용도의 설정은 application.yml보다는 동적으로 로딩을 지원하는 logback.xml에 설정하는 것이 더 좋겠다.
```
<logger name="org.hibernate.SQL" level="DEBUG"/>
<logger name="org.hibernate.type.descriptor.sql.BasicBinder" level="TRACE"/>
<logger name="org.hibernate.type.EnumType" level="TRACE"/>
```

처음에는 application.yml에 해당 설정을 지웠는데, classpath에 존재하는 application.yml의 설정이 적용되어, TRACE 로그가 사라지지 않았다. configuration 적용이 ./config에 있는 파일이 적용될 줄 알았는데, classpath 파일이 적용 된 후 ./config/에 있는 설정은 오버라이드 된다.
이것은 [Reference](https://docs.spring.io/spring-boot/docs/2.1.4.RELEASE/reference/htmlsingle/#boot-features-external-config-application-property-files)에 잘 설명되어 있다.  

## spring confing 적용 순서
config 적용 순서를 정리하면  
* default:  
 classpath:/ -> classpath:/config/ -> file:./ -> file:./config/

* spring.config.location=classpath:/custom-config/,file:./custom-config/ 를 지정:  
 classpath:/custom-config/ -> file:./custom-config/

* spring.config.addtional-location=classpath:/custom-config/,file:./custom-config/ 를 지정:  
 classpath:/ -> classpath:/config/ -> file:./ -> file:./config/ -> classpath:/custom-config/ -> file:./custom-config/

spring.config.location을 사용하는 경우에는 지정한 프로퍼티만 적용되고, spring.config.addtional-location을 사용하는 경우에는 default가 적용되고 지정한 프로퍼티가 적용된다.


## log 파일명 또는 디렉토리 지정하기
logging.path 또는logging.file을 사용해서 파일명 또는 디렉토리를 지정할 수 있다.  
이 설정이 적용되게 하려면 base.xml에서 처럼 로그파일 관련하여 다음 설정을 추가한다.
```
<property name="LOG_FILE" value="${LOG_FILE:-${LOG_PATH:-${LOG_TEMP:-${java.io.tmpdir:-/tmp}}}/spring.log}"/>
```
logback-spring.xml
```
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
  <include resource="org/springframework/boot/logging/logback/defaults.xml" />
  <property name="LOG_FILE" value="${LOG_FILE:-${LOG_PATH:-${LOG_TEMP:-${java.io.tmpdir:-/tmp}}}/spring.log}"/>
  <include resource="org/springframework/boot/logging/logback/console-appender.xml" />
  <include resource="org/springframework/boot/logging/logback/file-appender.xml" />
  <logger name="org.springframework" level="INFO"/>
  <root level="DEBUG">
    <appender-ref ref="CONSOLE" />
    <appender-ref ref="FILE" />
  </root>
</configuration>
```
application.properties
```
logging.file=/DATA/WEB/remotemeeting/log/document.log
```
logging.path로 directory만 지정하면 spring.log로 로그파일이 생긴다.



logging.file을 지정해도, 기본적으로 java.io.tmpdir에 spring.log를 접근한다. 로그를 기록하지는 않음. 이것이 문제가 된다면 다음과 같이 직접 지정.  
```
<property name="LOG_FILE" value="${LOG_PATH}/api.log"/>
```














