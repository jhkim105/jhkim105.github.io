---
layout: post
title:  "Migrating from Spring to Spring Boot"
date:   2019-04-22 10:55:00 +0900
categories: Backend
tag: SpringBoot
---

* content
{:toc}


Migrating from Spring to Spring Boot(Remote Meeting)


------------

# Overview
RemoteMeeting Project를 Spring Applicaton을 Spring Boot으로 마이그레이션  

작업 순서는 
1. Spring Boot(sprint-boot-starter) pom 적용
1. Spring Boot Application 적용

# As-Is
1. project는 core, api, web, admin으로 구성되어 있다. 
1. core를 참조하는 web, api에서 dependency를 중복 정의하고 있다.
1. Version
    ```
    spring 4.3.16.RELEASE
    spring-security 4.2.5.RELEASE
    spring-session 1.3.2.RELEASE
    hibernate 4.3.11.Final
    ```
    Spring 버전이 4.3이므로 Spring Boot Version은 1.5.20을 사용한다.



# Spring Boot(spring-boot-starter) pom 적용

## core/pom.xml
### spring-boot-starter 적용 전
```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <parent>
    <groupId>com.rsupport</groupId>
    <artifactId>seminar</artifactId>
    <version>1.0-SNAPSHOT</version>
    <relativePath>../pom.xml</relativePath>
  </parent>
  <groupId>com.rsupport.seminar</groupId>
  <artifactId>core</artifactId>
  <packaging>jar</packaging>
  <name>core</name>
  <properties>
    <version.filename>${project.build.directory}/classes/version.txt</version.filename>
    <dbunit.dataxml.path>target/test-classes/sample-data.xml</dbunit.dataxml.path>
  </properties>
  <dependencies>
    <!-- dependency 중복 오류로 일단 치환하지 않도록 변경 -->
    <dependency>
      <groupId>mysql</groupId>
      <artifactId>mysql-connector-java</artifactId>
    </dependency>
    <dependency>
      <groupId>com.tmax.tibero</groupId>
      <artifactId>tibero6-jdbc</artifactId>
    </dependency>
    <dependency>
      <groupId>commons-digester</groupId>
      <artifactId>commons-digester</artifactId>
    </dependency>
    <!-- junit -->
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
    </dependency>
    <dependency>
      <groupId>org.jmock</groupId>
      <artifactId>jmock</artifactId>
    </dependency>
    <dependency>
      <groupId>org.jmock</groupId>
      <artifactId>jmock-junit4</artifactId>
    </dependency>
    <!-- commons -->
    <dependency>
      <groupId>org.apache.commons</groupId>
      <artifactId>commons-dbcp2</artifactId>
    </dependency>
    <dependency>
      <groupId>commons-lang</groupId>
      <artifactId>commons-lang</artifactId>
    </dependency>
    <dependency>
      <groupId>org.apache.commons</groupId>
      <artifactId>commons-lang3</artifactId>
    </dependency>
    <dependency>
      <groupId>commons-beanutils</groupId>
      <artifactId>commons-beanutils</artifactId>
    </dependency>
    <dependency>
      <groupId>commons-collections</groupId>
      <artifactId>commons-collections</artifactId>
    </dependency>
    <dependency>
      <groupId>commons-codec</groupId>
      <artifactId>commons-codec</artifactId>
    </dependency>
    <dependency>
      <groupId>commons-fileupload</groupId>
      <artifactId>commons-fileupload</artifactId>
    </dependency>
    <dependency>
      <groupId>commons-io</groupId>
      <artifactId>commons-io</artifactId>
    </dependency>
    <dependency>
      <groupId>commons-configuration</groupId>
      <artifactId>commons-configuration</artifactId>
    </dependency>
    <dependency>
      <groupId>commons-pool</groupId>
      <artifactId>commons-pool</artifactId>
    </dependency>
    <dependency>
      <groupId>commons-net</groupId>
      <artifactId>commons-net</artifactId>
    </dependency>
    <dependency>
      <groupId>org.apache.commons</groupId>
      <artifactId>commons-exec</artifactId>
    </dependency>
    <!-- hibernate -->
    <dependency>
      <groupId>org.hibernate</groupId>
      <artifactId>hibernate-entitymanager</artifactId>
    </dependency>
    <dependency>
      <groupId>org.hibernate</groupId>
      <artifactId>hibernate-core</artifactId>
    </dependency>
    <dependency>
      <groupId>org.hibernate</groupId>
      <artifactId>hibernate-ehcache</artifactId>
    </dependency>
    <dependency>
      <groupId>org.hibernate</groupId>
      <artifactId>hibernate-envers</artifactId>
    </dependency>
    <dependency>
      <groupId>com.mysema.querydsl</groupId>
      <artifactId>querydsl-apt</artifactId>
    </dependency>
    <dependency>
      <groupId>com.mysema.querydsl</groupId>
      <artifactId>querydsl-jpa</artifactId>
    </dependency>
    <dependency>
      <groupId>com.mysema.querydsl</groupId>
      <artifactId>querydsl-sql</artifactId>
    </dependency>
    <dependency>
      <groupId>com.mysema.querydsl</groupId>
      <artifactId>querydsl-sql-codegen</artifactId>
      <scope>provided</scope>
    </dependency>
    <dependency>
      <groupId>net.sf.ehcache</groupId>
      <artifactId>ehcache-core</artifactId>
    </dependency>
    <dependency>
      <groupId>com.googlecode.ehcache-spring-annotations</groupId>
      <artifactId>ehcache-spring-annotations</artifactId>
    </dependency>
    <!-- spring -->
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-test</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-context-support</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-aop</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-orm</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework.security</groupId>
      <artifactId>spring-security-config</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework.security</groupId>
      <artifactId>spring-security-taglibs</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-webmvc</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springmodules</groupId>
      <artifactId>spring-modules-validation</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-expression</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework.session</groupId>
      <artifactId>spring-session-data-redis</artifactId>
    </dependency>
    <dependency>
      <groupId>org.slf4j</groupId>
      <artifactId>slf4j-api</artifactId>
    </dependency>
    <dependency>
      <groupId>org.slf4j</groupId>
      <artifactId>jcl-over-slf4j</artifactId>
      <scope>runtime</scope>
    </dependency>
    <dependency>
      <groupId>org.slf4j</groupId>
      <artifactId>log4j-over-slf4j</artifactId>
    </dependency>
    <dependency>
      <groupId>ch.qos.logback</groupId>
      <artifactId>logback-access</artifactId>
    </dependency>
    <dependency>
      <groupId>ch.qos.logback</groupId>
      <artifactId>logback-core</artifactId>
    </dependency>
    <dependency>
      <groupId>ch.qos.logback</groupId>
      <artifactId>logback-classic</artifactId>
    </dependency>
    <!-- https://mvnrepository.com/artifact/janino/janino -->
    <dependency>
      <groupId>janino</groupId>
      <artifactId>janino</artifactId>
    </dependency>
    <!-- Logging End -->
    <dependency>
      <groupId>org.sitemesh</groupId>
      <artifactId>sitemesh</artifactId>
    </dependency>
    <dependency>
      <groupId>javax.servlet</groupId>
      <artifactId>javax.servlet-api</artifactId>
      <scope>provided</scope>
    </dependency>
    <dependency>
      <groupId>javax.servlet.jsp</groupId>
      <artifactId>jsp-api</artifactId>
    </dependency>
    <dependency>
      <groupId>javax.servlet</groupId>
      <artifactId>jstl</artifactId>
    </dependency>
    <dependency>
      <groupId>org.apache.poi</groupId>
      <artifactId>poi</artifactId>
    </dependency>
    <dependency>
      <groupId>org.apache.poi</groupId>
      <artifactId>poi-examples</artifactId>
    </dependency>
    <dependency>
      <groupId>org.apache.poi</groupId>
      <artifactId>poi-ooxml</artifactId>
    </dependency>
    <dependency>
      <groupId>org.aspectj</groupId>
      <artifactId>aspectjweaver</artifactId>
    </dependency>
    <dependency>
      <groupId>org.aspectj</groupId>
      <artifactId>aspectjrt</artifactId>
    </dependency>
    <dependency>
      <groupId>org.apache.httpcomponents</groupId>
      <artifactId>httpclient</artifactId>
    </dependency>
    <dependency>
      <groupId>javax.xml</groupId>
      <artifactId>jaxb-impl</artifactId>
    </dependency>
    <dependency>
      <groupId>org.projectlombok</groupId>
      <artifactId>lombok</artifactId>
    </dependency>
    <dependency>
      <groupId>com.maxmind.geoip2</groupId>
      <artifactId>geoip2</artifactId>
    </dependency>
    <dependency>
      <groupId>com.fasterxml.jackson.core</groupId>
      <artifactId>jackson-core</artifactId>
    </dependency>
    <dependency>
      <groupId>com.fasterxml.jackson.core</groupId>
      <artifactId>jackson-annotations</artifactId>
    </dependency>
    <dependency>
      <groupId>com.fasterxml.jackson.core</groupId>
      <artifactId>jackson-databind</artifactId>
    </dependency>
    <dependency>
      <groupId>com.fasterxml.jackson.dataformat</groupId>
      <artifactId>jackson-dataformat-smile</artifactId>
    </dependency>
    <dependency>
      <groupId>com.fasterxml.jackson.jaxrs</groupId>
      <artifactId>jackson-jaxrs-json-provider</artifactId>
    </dependency>
    <dependency>
      <groupId>com.fasterxml.jackson.module</groupId>
      <artifactId>jackson-module-jaxb-annotations</artifactId>
    </dependency>
    <dependency>
      <groupId>javax.mail</groupId>
      <artifactId>mail</artifactId>
    </dependency>
    <dependency>
      <groupId>org.apache.tiles</groupId>
      <artifactId>tiles-extras</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework.data</groupId>
      <artifactId>spring-data-jpa</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework.data</groupId>
      <artifactId>spring-data-commons</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-aspects</artifactId>
    </dependency>
    <dependency>
      <groupId>org.eclipse.paho</groupId>
      <artifactId>org.eclipse.paho.client.mqttv3</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework.data</groupId>
      <artifactId>spring-data-redis</artifactId>
    </dependency>
    <dependency>
      <groupId>redis.clients</groupId>
      <artifactId>jedis</artifactId>
    </dependency>
    <!-- https://mvnrepository.com/artifact/com.vmlens/concurrent-junit -->
    <dependency>
      <groupId>com.vmlens</groupId>
      <artifactId>concurrent-junit</artifactId>
      <version>1.0.0</version>
      <scope>test</scope>
    </dependency>
    <dependency>
      <groupId>com.amazonaws</groupId>
      <artifactId>aws-java-sdk-dynamodb</artifactId>
    </dependency>
    <dependency>
      <groupId>com.amazonaws</groupId>
      <artifactId>aws-java-sdk-autoscaling</artifactId>
    </dependency>
    <dependency>
      <groupId>com.amazonaws</groupId>
      <artifactId>aws-java-sdk-sqs</artifactId>
    </dependency>
    <dependency>
      <groupId>joda-time</groupId>
      <artifactId>joda-time</artifactId>
    </dependency>
    <dependency>
      <groupId>com.github.fernandospr</groupId>
      <artifactId>javapns-jdk16</artifactId>
    </dependency>
    <dependency>
      <groupId>io.springfox</groupId>
      <artifactId>springfox-swagger2</artifactId>
    </dependency>
    <dependency>
      <groupId>io.springfox</groupId>
      <artifactId>springfox-swagger-ui</artifactId>
    </dependency>
    <dependency>
      <groupId>com.fasterxml</groupId>
      <artifactId>classmate</artifactId>
    </dependency>
    <!-- AMQP -->
    <dependency>
      <groupId>org.springframework.amqp</groupId>
      <artifactId>spring-rabbit</artifactId>
      <exclusions>
        <exclusion>
          <groupId>org.springframework</groupId>
          <artifactId>spring-context</artifactId>
        </exclusion>
        <exclusion>
          <groupId>org.springframework</groupId>
          <artifactId>spring-web</artifactId>
        </exclusion>
        <exclusion>
          <groupId>org.springframework</groupId>
          <artifactId>spring-tx</artifactId>
        </exclusion>
        <exclusion>
          <groupId>org.springframework</groupId>
          <artifactId>spring-test</artifactId>
        </exclusion>
      </exclusions>
    </dependency>
    <!-- Spring Integration -->
    <dependency>
      <groupId>org.springframework.integration</groupId>
      <artifactId>spring-integration-mqtt</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework.integration</groupId>
      <artifactId>spring-integration-xml</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework.integration</groupId>
      <artifactId>spring-integration-jms</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework.integration</groupId>
      <artifactId>spring-integration-amqp</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework.integration</groupId>
      <artifactId>spring-integration-java-dsl</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework.integration</groupId>
      <artifactId>spring-integration-core</artifactId>
    </dependency>
    <!-- Spring Integration End -->
    <dependency>
      <groupId>com.auth0</groupId>
      <artifactId>java-jwt</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework.data</groupId>
      <artifactId>spring-data-mongodb</artifactId>
    </dependency>
  </dependencies>
  <build>
    <plugins>
      <plugin>
        <groupId>de.juplo</groupId>
        <artifactId>hibernate4-maven-plugin</artifactId>
      </plugin>
      <plugin>
        <groupId>org.codehaus.mojo</groupId>
        <artifactId>sql-maven-plugin</artifactId>
      </plugin>
      <plugin>
        <groupId>org.codehaus.mojo</groupId>
        <artifactId>dbunit-maven-plugin</artifactId>
      </plugin>
      <plugin>
        <groupId>com.mysema.maven</groupId>
        <artifactId>apt-maven-plugin</artifactId>
        <version>1.1.1</version>
      </plugin>
    </plugins>
    <resources>
      <resource>
        <directory>src/main/resources</directory>
        <filtering>true</filtering>
      </resource>
    </resources>
    <testResources>
      <testResource>
        <directory>src/test/resources</directory>
        <filtering>true</filtering>
      </testResource>
    </testResources>
  </build>
</project>
```


### spring-boot-starter 적용 후
```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <parent>
    <groupId>com.rsupport</groupId>
    <artifactId>seminar</artifactId>
    <version>1.0-SNAPSHOT</version>
    <relativePath>../pom.xml</relativePath>
  </parent>
  <groupId>com.rsupport.seminar</groupId>
  <artifactId>core</artifactId>
  <packaging>jar</packaging>
  <name>core</name>
  <properties>
    <version.filename>${project.build.directory}/classes/version.txt</version.filename>
    <dbunit.dataxml.path>target/test-classes/sample-data.xml</dbunit.dataxml.path>
  </properties>
  <dependencies>
    <!-- spring boot -->
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-test</artifactId>
      <scope>test</scope>
    </dependency>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-aop</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-security</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-amqp</artifactId>
    </dependency>
    <!-- //spring boot -->
    <!-- spring -->
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-context-support</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework.security</groupId>
      <artifactId>spring-security-taglibs</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springmodules</groupId>
      <artifactId>spring-modules-validation</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-expression</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework.session</groupId>
      <artifactId>spring-session-data-redis</artifactId>
    </dependency>
    <!-- //spring -->
    <!-- data -->
    <dependency>
      <groupId>mysql</groupId>
      <artifactId>mysql-connector-java</artifactId>
    </dependency>
    <dependency>
      <groupId>com.tmax.tibero</groupId>
      <artifactId>tibero6-jdbc</artifactId>
    </dependency>
    <dependency>
      <groupId>org.hibernate</groupId>
      <artifactId>hibernate-ehcache</artifactId>
    </dependency>
    <dependency>
      <groupId>org.hibernate</groupId>
      <artifactId>hibernate-envers</artifactId>
    </dependency>
    <dependency>
      <groupId>com.mysema.querydsl</groupId>
      <artifactId>querydsl-apt</artifactId>
    </dependency>
    <dependency>
      <groupId>com.mysema.querydsl</groupId>
      <artifactId>querydsl-jpa</artifactId>
    </dependency>
    <dependency>
      <groupId>com.mysema.querydsl</groupId>
      <artifactId>querydsl-sql</artifactId>
    </dependency>
    <dependency>
      <groupId>com.mysema.querydsl</groupId>
      <artifactId>querydsl-sql-codegen</artifactId>
      <scope>provided</scope>
    </dependency>
    <dependency>
      <groupId>net.sf.ehcache</groupId>
      <artifactId>ehcache-core</artifactId>
    </dependency>
    <dependency>
      <groupId>com.googlecode.ehcache-spring-annotations</groupId>
      <artifactId>ehcache-spring-annotations</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework.data</groupId>
      <artifactId>spring-data-redis</artifactId>
    </dependency>
    <dependency>
      <groupId>redis.clients</groupId>
      <artifactId>jedis</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework.data</groupId>
      <artifactId>spring-data-mongodb</artifactId>
    </dependency>
    <!-- //data -->
    <!-- web -->
    <dependency>
      <groupId>javax.servlet</groupId>
      <artifactId>javax.servlet-api</artifactId>
      <scope>provided</scope>
    </dependency>
    <dependency>
      <groupId>javax.servlet.jsp</groupId>
      <artifactId>jsp-api</artifactId>
    </dependency>
    <dependency>
      <groupId>javax.servlet</groupId>
      <artifactId>jstl</artifactId>
    </dependency>
    <dependency>
      <groupId>org.sitemesh</groupId>
      <artifactId>sitemesh</artifactId>
    </dependency>
    <!-- //web -->
    <!-- commons -->
    <dependency>
      <groupId>commons-digester</groupId>
      <artifactId>commons-digester</artifactId>
    </dependency>
    <dependency>
      <groupId>org.apache.commons</groupId>
      <artifactId>commons-dbcp2</artifactId>
    </dependency>
    <dependency>
      <groupId>commons-lang</groupId>
      <artifactId>commons-lang</artifactId>
    </dependency>
    <dependency>
      <groupId>org.apache.commons</groupId>
      <artifactId>commons-lang3</artifactId>
    </dependency>
    <dependency>
      <groupId>commons-beanutils</groupId>
      <artifactId>commons-beanutils</artifactId>
    </dependency>
    <dependency>
      <groupId>commons-collections</groupId>
      <artifactId>commons-collections</artifactId>
    </dependency>
    <dependency>
      <groupId>commons-codec</groupId>
      <artifactId>commons-codec</artifactId>
    </dependency>
    <dependency>
      <groupId>commons-fileupload</groupId>
      <artifactId>commons-fileupload</artifactId>
    </dependency>
    <dependency>
      <groupId>commons-io</groupId>
      <artifactId>commons-io</artifactId>
    </dependency>
    <dependency>
      <groupId>commons-configuration</groupId>
      <artifactId>commons-configuration</artifactId>
    </dependency>
    <dependency>
      <groupId>commons-pool</groupId>
      <artifactId>commons-pool</artifactId>
    </dependency>
    <dependency>
      <groupId>commons-net</groupId>
      <artifactId>commons-net</artifactId>
    </dependency>
    <dependency>
      <groupId>org.apache.commons</groupId>
      <artifactId>commons-exec</artifactId>
    </dependency>
    <!-- //commons -->
    <dependency>
      <groupId>org.apache.poi</groupId>
      <artifactId>poi</artifactId>
    </dependency>
    <dependency>
      <groupId>org.apache.poi</groupId>
      <artifactId>poi-examples</artifactId>
    </dependency>
    <dependency>
      <groupId>org.apache.poi</groupId>
      <artifactId>poi-ooxml</artifactId>
    </dependency>
    <dependency>
      <groupId>org.apache.httpcomponents</groupId>
      <artifactId>httpclient</artifactId>
    </dependency>
    <dependency>
      <groupId>javax.xml</groupId>
      <artifactId>jaxb-impl</artifactId>
    </dependency>
    <dependency>
      <groupId>org.projectlombok</groupId>
      <artifactId>lombok</artifactId>
    </dependency>
    <dependency>
      <groupId>com.maxmind.geoip2</groupId>
      <artifactId>geoip2</artifactId>
    </dependency>
    <dependency>
      <groupId>javax.mail</groupId>
      <artifactId>mail</artifactId>
    </dependency>
    <dependency>
      <groupId>org.apache.tiles</groupId>
      <artifactId>tiles-extras</artifactId>
    </dependency>
    <dependency>
      <groupId>org.eclipse.paho</groupId>
      <artifactId>org.eclipse.paho.client.mqttv3</artifactId>
    </dependency>
    <dependency>
      <groupId>com.vmlens</groupId>
      <artifactId>concurrent-junit</artifactId>
      <version>1.0.0</version>
      <scope>test</scope>
    </dependency>
    <!-- aws -->
    <dependency>
      <groupId>com.amazonaws</groupId>
      <artifactId>aws-java-sdk-dynamodb</artifactId>
    </dependency>
    <dependency>
      <groupId>com.amazonaws</groupId>
      <artifactId>aws-java-sdk-autoscaling</artifactId>
    </dependency>
    <dependency>
      <groupId>com.amazonaws</groupId>
      <artifactId>aws-java-sdk-sqs</artifactId>
    </dependency>
    <dependency>
      <groupId>joda-time</groupId>
      <artifactId>joda-time</artifactId>
    </dependency>
    <!-- //aws -->
    <dependency>
      <groupId>com.github.fernandospr</groupId>
      <artifactId>javapns-jdk16</artifactId>
    </dependency>
    <!-- swagger -->
    <dependency>
      <groupId>io.springfox</groupId>
      <artifactId>springfox-swagger2</artifactId>
    </dependency>
    <dependency>
      <groupId>io.springfox</groupId>
      <artifactId>springfox-swagger-ui</artifactId>
    </dependency>
    <!-- //swagger -->
    <dependency>
      <groupId>com.fasterxml</groupId>
      <artifactId>classmate</artifactId>
    </dependency>
    <!-- Spring Integration -->
    <dependency>
      <groupId>org.springframework.integration</groupId>
      <artifactId>spring-integration-mqtt</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework.integration</groupId>
      <artifactId>spring-integration-xml</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework.integration</groupId>
      <artifactId>spring-integration-jms</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework.integration</groupId>
      <artifactId>spring-integration-amqp</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework.integration</groupId>
      <artifactId>spring-integration-java-dsl</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework.integration</groupId>
      <artifactId>spring-integration-core</artifactId>
    </dependency>
    <!-- //Spring Integration -->
    <dependency>
      <groupId>com.auth0</groupId>
      <artifactId>java-jwt</artifactId>
    </dependency>
  </dependencies>
  <build>
    <plugins>
      <plugin>
        <groupId>de.juplo</groupId>
        <artifactId>hibernate4-maven-plugin</artifactId>
      </plugin>
      <plugin>
        <groupId>org.codehaus.mojo</groupId>
        <artifactId>sql-maven-plugin</artifactId>
      </plugin>
      <plugin>
        <groupId>org.codehaus.mojo</groupId>
        <artifactId>dbunit-maven-plugin</artifactId>
      </plugin>
      <plugin>
        <groupId>com.mysema.maven</groupId>
        <artifactId>apt-maven-plugin</artifactId>
      </plugin>
    </plugins>
    <resources>
      <resource>
        <directory>src/main/resources</directory>
        <filtering>true</filtering>
      </resource>
    </resources>
    <testResources>
      <testResource>
        <directory>src/test/resources</directory>
        <filtering>true</filtering>
      </testResource>
    </testResources>
  </build>
</project>
```
## core/web.xml
### spring-boot-starter 적용 전
```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <parent>
    <groupId>com.rsupport</groupId>
    <artifactId>seminar</artifactId>
    <version>1.0-SNAPSHOT</version>
    <relativePath>../pom.xml</relativePath>
  </parent>
  <groupId>com.rsupport.seminar</groupId>
  <artifactId>web</artifactId>
  <name>web</name>
  <packaging>war</packaging>
  <properties>
    <version.filename>src/main/webapp/version.txt</version.filename>
  </properties>
  <dependencies>
    <dependency>
      <groupId>${project.groupId}</groupId>
      <artifactId>core</artifactId>
      <version>${project.parent.version}</version>
    </dependency>
    <!-- dependency 중복 오류로 일단 치환하지 않도록 변경 -->
    <dependency>
      <groupId>mysql</groupId>
      <artifactId>mysql-connector-java</artifactId>
    </dependency>
    <dependency>
      <groupId>com.tmax.tibero</groupId>
      <artifactId>tibero6-jdbc</artifactId>
    </dependency>
    <dependency>
      <groupId>commons-digester</groupId>
      <artifactId>commons-digester</artifactId>
    </dependency>
    <dependency>
      <groupId>com.github.fernandospr</groupId>
      <artifactId>javapns-jdk16</artifactId>
    </dependency>


    <!-- junit -->
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <scope>test</scope>
    </dependency>
    <dependency>
      <groupId>org.jmock</groupId>
      <artifactId>jmock</artifactId>
      <scope>test</scope>
    </dependency>
    <dependency>
      <groupId>org.jmock</groupId>
      <artifactId>jmock-junit4</artifactId>
      <scope>test</scope>
    </dependency>
    <dependency>
      <groupId>com.jayway.jsonpath</groupId>
      <artifactId>json-path</artifactId>
      <scope>test</scope>
    </dependency>
    <dependency>
      <groupId>com.jayway.jsonpath</groupId>
      <artifactId>json-path-assert</artifactId>
      <scope>test</scope>
    </dependency>
    <!-- commons -->
    <dependency>
      <groupId>org.apache.commons</groupId>
      <artifactId>commons-dbcp2</artifactId>
    </dependency>
    <dependency>
      <groupId>commons-lang</groupId>
      <artifactId>commons-lang</artifactId>
    </dependency>
    <dependency>
      <groupId>org.apache.commons</groupId>
      <artifactId>commons-lang3</artifactId>
    </dependency>
    <dependency>
      <groupId>commons-beanutils</groupId>
      <artifactId>commons-beanutils</artifactId>
    </dependency>
    <dependency>
      <groupId>commons-collections</groupId>
      <artifactId>commons-collections</artifactId>
    </dependency>
    <dependency>
      <groupId>commons-codec</groupId>
      <artifactId>commons-codec</artifactId>
    </dependency>
    <dependency>
      <groupId>commons-fileupload</groupId>
      <artifactId>commons-fileupload</artifactId>
    </dependency>
    <dependency>
      <groupId>commons-io</groupId>
      <artifactId>commons-io</artifactId>
    </dependency>
    <dependency>
      <groupId>commons-configuration</groupId>
      <artifactId>commons-configuration</artifactId>
    </dependency>
    <dependency>
      <groupId>org.apache.commons</groupId>
      <artifactId>commons-exec</artifactId>
    </dependency>
    <!-- hibernate -->
    <dependency>
      <groupId>org.hibernate</groupId>
      <artifactId>hibernate-entitymanager</artifactId>
    </dependency>
    <dependency>
      <groupId>org.hibernate</groupId>
      <artifactId>hibernate-core</artifactId>
    </dependency>
    <dependency>
      <groupId>org.hibernate</groupId>
      <artifactId>hibernate-ehcache</artifactId>
    </dependency>
    <dependency>
      <groupId>org.hibernate</groupId>
      <artifactId>hibernate-envers</artifactId>
    </dependency>
    <dependency>
      <groupId>org.hibernate</groupId>
      <artifactId>hibernate-validator</artifactId>
    </dependency>
    <dependency>
      <groupId>com.mysema.querydsl</groupId>
      <artifactId>querydsl-apt</artifactId>
    </dependency>
    <dependency>
      <groupId>com.mysema.querydsl</groupId>
      <artifactId>querydsl-jpa</artifactId>
    </dependency>
    <dependency>
      <groupId>net.sf.ehcache</groupId>
      <artifactId>ehcache-core</artifactId>
    </dependency>
    <dependency>
      <groupId>com.googlecode.ehcache-spring-annotations</groupId>
      <artifactId>ehcache-spring-annotations</artifactId>
    </dependency>
    <!-- spring -->
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-test</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-context-support</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-aop</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-orm</artifactId>
    </dependency>
    <!--<dependency>-->
    <!--<groupId>org.springframework.security</groupId>-->
    <!--<artifactId>spring-security-web</artifactId>-->
    <!--</dependency>-->
    <dependency>
      <groupId>org.springframework.security</groupId>
      <artifactId>spring-security-config</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework.security</groupId>
      <artifactId>spring-security-taglibs</artifactId>
    </dependency>
    <!-- https://mvnrepository.com/artifact/com.onelogin/java-saml -->
    <dependency>
      <groupId>com.onelogin</groupId>
      <artifactId>java-saml</artifactId>
      <version>2.3.0</version>
    </dependency>


    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-webmvc</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-oxm</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springmodules</groupId>
      <artifactId>spring-modules-validation</artifactId>
    </dependency>
    <dependency>
      <groupId>javax.xml.bind</groupId>
      <artifactId>jaxb-api</artifactId>
    </dependency>
    <!-- AspectJ -->
    <dependency>
      <groupId>org.aspectj</groupId>
      <artifactId>aspectjweaver</artifactId>
    </dependency>
    <dependency>
      <groupId>org.aspectj</groupId>
      <artifactId>aspectjrt</artifactId>
    </dependency>


    <!-- Logging -->
    <dependency>
      <groupId>org.slf4j</groupId>
      <artifactId>slf4j-api</artifactId>
    </dependency>
    <dependency>
      <groupId>org.slf4j</groupId>
      <artifactId>jcl-over-slf4j</artifactId>
      <scope>runtime</scope>
    </dependency>
    <dependency>
      <groupId>org.slf4j</groupId>
      <artifactId>log4j-over-slf4j</artifactId>
    </dependency>
    <dependency>
      <groupId>ch.qos.logback</groupId>
      <artifactId>logback-access</artifactId>
    </dependency>
    <dependency>
      <groupId>ch.qos.logback</groupId>
      <artifactId>logback-core</artifactId>
    </dependency>
    <dependency>
      <groupId>ch.qos.logback</groupId>
      <artifactId>logback-classic</artifactId>
    </dependency>
    <!-- Logging End -->


    <!-- @Inject -->
    <dependency>
      <groupId>javax.inject</groupId>
      <artifactId>javax.inject</artifactId>
    </dependency>
    <!-- Servlet -->
    <dependency>
      <groupId>javax.servlet</groupId>
      <artifactId>javax.servlet-api</artifactId>
    </dependency>
    <dependency>
      <groupId>javax.servlet.jsp</groupId>
      <artifactId>jsp-api</artifactId>
    </dependency>
    <dependency>
      <groupId>javax.servlet</groupId>
      <artifactId>jstl</artifactId>
    </dependency>
    <dependency>
      <groupId>org.apache.httpcomponents</groupId>
      <artifactId>httpclient</artifactId>
    </dependency>
    <dependency>
      <groupId>org.projectlombok</groupId>
      <artifactId>lombok</artifactId>
    </dependency>
    <dependency>
      <groupId>javax.mail</groupId>
      <artifactId>mail</artifactId>
    </dependency>
    <dependency>
      <groupId>org.sitemesh</groupId>
      <artifactId>sitemesh</artifactId>
    </dependency>
    <dependency>
      <groupId>com.maxmind.geoip2</groupId>
      <artifactId>geoip2</artifactId>
    </dependency>
    <dependency>
      <groupId>com.fasterxml.jackson.core</groupId>
      <artifactId>jackson-core</artifactId>
    </dependency>
    <dependency>
      <groupId>com.fasterxml.jackson.core</groupId>
      <artifactId>jackson-annotations</artifactId>
    </dependency>
    <dependency>
      <groupId>com.fasterxml.jackson.core</groupId>
      <artifactId>jackson-databind</artifactId>
    </dependency>
    <dependency>
      <groupId>com.fasterxml.jackson.dataformat</groupId>
      <artifactId>jackson-dataformat-smile</artifactId>
    </dependency>
    <dependency>
      <groupId>com.fasterxml.jackson.jaxrs</groupId>
      <artifactId>jackson-jaxrs-json-provider</artifactId>
    </dependency>
    <dependency>
      <groupId>com.fasterxml.jackson.module</groupId>
      <artifactId>jackson-module-jaxb-annotations</artifactId>
    </dependency>
    <dependency>
      <groupId>com.google.api-client</groupId>
      <artifactId>google-api-client</artifactId>
    </dependency>
    <dependency>
      <groupId>org.apache.poi</groupId>
      <artifactId>poi</artifactId>
    </dependency>
    <dependency>
      <groupId>org.apache.poi</groupId>
      <artifactId>poi-examples</artifactId>
    </dependency>
    <dependency>
      <groupId>org.apache.poi</groupId>
      <artifactId>poi-ooxml</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework.data</groupId>
      <artifactId>spring-data-jpa</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework.data</groupId>
      <artifactId>spring-data-commons</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-aspects</artifactId>
    </dependency>
    <dependency>
      <groupId>org.eclipse.paho</groupId>
      <artifactId>org.eclipse.paho.client.mqttv3</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework.data</groupId>
      <artifactId>spring-data-redis</artifactId>
    </dependency>
    <dependency>
      <groupId>redis.clients</groupId>
      <artifactId>jedis</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework.session</groupId>
      <artifactId>spring-session-data-redis</artifactId>
    </dependency>
    <!-- https://mvnrepository.com/artifact/io.springfox/springfox-swagger2 -->
    <!-- springfox related -->
    <dependency>
      <groupId>io.springfox</groupId>
      <artifactId>springfox-swagger2</artifactId>
    </dependency>
    <dependency>
      <groupId>io.springfox</groupId>
      <artifactId>springfox-swagger-ui</artifactId>
    </dependency>
    <dependency>
      <groupId>com.fasterxml</groupId>
      <artifactId>classmate</artifactId>
    </dependency>
    <!-- springfox related end-->


    <!-- https://mvnrepository.com/artifact/org.springframework.mobile/spring-mobile-device -->
    <dependency>
      <groupId>org.springframework.mobile</groupId>
      <artifactId>spring-mobile-device</artifactId>
    </dependency>
    <!-- AMQP -->
    <dependency>
      <groupId>org.springframework.amqp</groupId>
      <artifactId>spring-rabbit</artifactId>
      <exclusions>
        <exclusion>
          <groupId>org.springframework</groupId>
          <artifactId>spring-context</artifactId>
        </exclusion>
        <exclusion>
          <groupId>org.springframework</groupId>
          <artifactId>spring-web</artifactId>
        </exclusion>
        <exclusion>
          <groupId>org.springframework</groupId>
          <artifactId>spring-tx</artifactId>
        </exclusion>
      </exclusions>
    </dependency>
    <dependency>
      <groupId>com.auth0</groupId>
      <artifactId>java-jwt</artifactId>
    </dependency>


  </dependencies>
  <build>
    <plugins>
      <plugin>
        <groupId>org.codehaus.mojo</groupId>
        <artifactId>exec-maven-plugin</artifactId>
      </plugin>
      <plugin>
        <groupId>org.eclipse.jetty</groupId>
        <artifactId>jetty-maven-plugin</artifactId>
        <version>${jetty.version}</version>
        <configuration>
          <systemProperties>
            <systemProperty>
              <name>org.eclipse.jetty.server.Request.maxFormContentSize</name>
              <value>-1</value>
            </systemProperty>
          </systemProperties>
          <jettyXml>
            target/test-classes/jetty/jetty.xml,target/test-classes/jetty/jetty-http.xml,target/test-classes/jetty/jetty-ssl.xml,target/test-classes/jetty/jetty-https.xml
          </jettyXml>
          <scanIntervalSeconds>${jetty.scanIntervalSeconds}</scanIntervalSeconds>
          <webAppConfig>
            <contextPath>/</contextPath>
            <defaultsDescriptor>src/test/resources/jetty/webdefault.xml</defaultsDescriptor>
          </webAppConfig>
          <reload>${jetty.reload}</reload>
          <webAppSourceDirectory>${basedir}/src/main/webapp</webAppSourceDirectory>
          <classesDirectory>target/classes</classesDirectory>
          <contextHandlers>
            <contextHandler implementation="org.eclipse.jetty.webapp.WebAppContext">
              <war>${basedir}/../public/src/main/webapp</war>
              <!-- minify 파일 참조시에 사용 : public install 후에 사용한다. install시에 minify 함 -->
              <!-- <war>${basedir}/../public/target/public</war> -->
              <contextPath>/public</contextPath>
              <defaultsDescriptor>src/test/resources/jetty/webdefault.xml</defaultsDescriptor>
            </contextHandler>
            <contextHandler implementation="org.eclipse.jetty.webapp.WebAppContext">
              <war>${basedir}/src/main/resources/swagger</war>
              <contextPath>/swagger</contextPath>
              <defaultsDescriptor>src/test/resources/jetty/webdefault.xml</defaultsDescriptor>
            </contextHandler>
          </contextHandlers>
          <scanTargetPatterns>
            <scanTargetPattern>
              <directory>src/main/webapp/WEB-INF</directory>
              <excludes>
                <exclude>**/*.jsp</exclude>
                <exclude>**/*.js</exclude>
              </excludes>
              <includes>
                <include>**/*.properties</include>
                <include>**/*.xml</include>
              </includes>
            </scanTargetPattern>
          </scanTargetPatterns>
        </configuration>
        <dependencies>
          <dependency>
            <groupId>net.sf.ehcache</groupId>
            <artifactId>ehcache-web</artifactId>
            <version>2.0.4</version>
          </dependency>
          <dependency>
            <groupId>com.samaxes.filter</groupId>
            <artifactId>cachefilter</artifactId>
            <version>2.3.1</version>
          </dependency>
        </dependencies>
      </plugin>
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-war-plugin</artifactId>
      </plugin>
    </plugins>
    <resources>
      <resource>
        <directory>src/main/resources</directory>
        <filtering>true</filtering>
      </resource>
    </resources>
    <testResources>
      <testResource>
        <directory>src/test/resources</directory>
        <filtering>true</filtering>
      </testResource>
      <testResource>
        <directory>src/main/webapp</directory>
        <filtering>true</filtering>
        <includes>
          <include>**/*.xml</include>
        </includes>
      </testResource>
    </testResources>
  </build>
</project>
```
### spring-boot-starter 적용 후
core와 중복 설정된 디펜던시 모두 제거함
```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <parent>
    <groupId>com.rsupport</groupId>
    <artifactId>seminar</artifactId>
    <version>1.0-SNAPSHOT</version>
    <relativePath>../pom.xml</relativePath>
  </parent>
  <groupId>com.rsupport.seminar</groupId>
  <artifactId>web</artifactId>
  <name>web</name>
  <packaging>war</packaging>
  <properties>
    <version.filename>src/main/webapp/version.txt</version.filename>
  </properties>
  <dependencies>
    <dependency>
      <groupId>${project.groupId}</groupId>
      <artifactId>core</artifactId>
      <version>${project.parent.version}</version>
    </dependency>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-test</artifactId>
      <scope>test</scope>
    </dependency>
    <dependency>
      <groupId>com.onelogin</groupId>
      <artifactId>java-saml</artifactId>
      <version>2.3.0</version>
    </dependency>
    <dependency>
      <groupId>javax.inject</groupId>
      <artifactId>javax.inject</artifactId>
    </dependency>
    <dependency>
      <groupId>org.projectlombok</groupId>
      <artifactId>lombok</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework.mobile</groupId>
      <artifactId>spring-mobile-device</artifactId>
    </dependency>
  </dependencies>
  <build>
    <plugins>
      <plugin>
        <groupId>org.eclipse.jetty</groupId>
        <artifactId>jetty-maven-plugin</artifactId>
        <version>${jetty.version}</version>
        <configuration>
          <systemProperties>
            <systemProperty>
              <name>org.eclipse.jetty.server.Request.maxFormContentSize</name>
              <value>-1</value>
            </systemProperty>
          </systemProperties>
          <jettyXml>
            target/test-classes/jetty/jetty.xml,target/test-classes/jetty/jetty-http.xml,target/test-classes/jetty/jetty-ssl.xml,target/test-classes/jetty/jetty-https.xml
          </jettyXml>
          <scanIntervalSeconds>${jetty.scanIntervalSeconds}</scanIntervalSeconds>
          <webAppConfig>
            <contextPath>/</contextPath>
            <defaultsDescriptor>src/test/resources/jetty/webdefault.xml</defaultsDescriptor>
          </webAppConfig>
          <reload>${jetty.reload}</reload>
          <webAppSourceDirectory>${basedir}/src/main/webapp</webAppSourceDirectory>
          <classesDirectory>target/classes</classesDirectory>
          <contextHandlers>
            <contextHandler implementation="org.eclipse.jetty.webapp.WebAppContext">
              <war>${basedir}/../public/src/main/webapp</war>
              <!-- minify 파일 참조시에 사용 : public install 후에 사용한다. install시에 minify 함 -->
              <!-- <war>${basedir}/../public/target/public</war> -->
              <contextPath>/public</contextPath>
              <defaultsDescriptor>src/test/resources/jetty/webdefault.xml</defaultsDescriptor>
            </contextHandler>
            <contextHandler implementation="org.eclipse.jetty.webapp.WebAppContext">
              <war>${basedir}/src/main/resources/swagger</war>
              <contextPath>/swagger</contextPath>
              <defaultsDescriptor>src/test/resources/jetty/webdefault.xml</defaultsDescriptor>
            </contextHandler>
          </contextHandlers>
          <scanTargetPatterns>
            <scanTargetPattern>
              <directory>src/main/webapp/WEB-INF</directory>
              <excludes>
                <exclude>**/*.jsp</exclude>
                <exclude>**/*.js</exclude>
              </excludes>
              <includes>
                <include>**/*.properties</include>
                <include>**/*.xml</include>
              </includes>
            </scanTargetPattern>
          </scanTargetPatterns>
        </configuration>
        <dependencies>
          <dependency>
            <groupId>net.sf.ehcache</groupId>
            <artifactId>ehcache-web</artifactId>
            <version>2.0.4</version>
          </dependency>
          <dependency>
            <groupId>com.samaxes.filter</groupId>
            <artifactId>cachefilter</artifactId>
            <version>2.3.1</version>
          </dependency>
        </dependencies>
      </plugin>
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-war-plugin</artifactId>
      </plugin>
    </plugins>
    <resources>
      <resource>
        <directory>src/main/resources</directory>
        <filtering>true</filtering>
      </resource>
    </resources>
    <testResources>
      <testResource>
        <directory>src/test/resources</directory>
        <filtering>true</filtering>
      </testResource>
    </testResources>
  </build>
</project>

```
api, partner 등 다른 프로젝트도 같은 방식으로 적용.

#  Spring Boot Application 적용

## Spring Boot Application 적용 - core
* main/resources/jdbc.properties 삭제
* application.properties 추가  
  * project마다 각각 두는 것이 좋겠다.  
  * main/이 아닌 test/에  
    test/resources/application.properties  

### application.properties
```
# LOGGING
logging.level.com.rsupport.psyclone=debug
# DATASOURCE
spring.datasource.platform=mysql
spring.datasource.driver-class-name=@jdbc.driverClassName@
spring.datasource.url=@jdbc.url@
spring.datasource.username=@db.username@
spring.datasource.password=@db.password@
db.id=1
db.type=@db.type@
# DATASOURCE - dbcp2
spring.datasource.type=org.apache.commons.dbcp2.BasicDataSource
spring.datasource.dbcp2.max-total=@jdbc.maxActive@
spring.datasource.dbcp2.max-wait-millis=@jdbc.maxWait@
spring.datasource.dbcp2.initial-size=@jdbc.initialSize@
spring.datasource.dbcp2.validation-query=@jdbc.validationQuery@
spring.datasource.dbcp2.default-query-timeout=@jdbc.queryTimeout@
#spring.datasource.dbcp2.test-on-borrow=true
#spring.datasource.dbcp2.test-on-return=false
#spring.datasource.dbcp2.test-while-idle=false
spring.datasource.dbcp2.time-between-eviction-runs-millis=5000
spring.datasource.dbcp2.min-evictable-idle-time-millis=30000
# JPA
spring.jpa.properties.hibernate.hbm2ddl.auto=@hibernate.hbm2ddl.auto@
spring.jpa.properties.hibernate.format_sql=@hibernate.format_sql@
spring.jpa.properties.hibernate.id.new_generator_mappings=false
spring.jpa.properties.org.hibernate.envers.audit_table_suffix=_h
spring.jpa.properties.hibernate.cache.use_second_level_cache=true
spring.jpa.properties.hibernate.cache.use_query_cache=false
spring.jpa.properties.hibernate.cache.provider_class=org.hibernate.cache.EhCacheProvider
spring.jpa.properties.hibernate.cache.region.factory_class=org.hibernate.cache.ehcache.EhCacheRegionFactory
```
### JpaConfig.java
```
@Configuration
@PropertySource(value = { "classpath:jdbc.properties", "file:${rm_home}/config/jdbc.properties" }, ignoreResourceNotFound = true)
@EnableTransactionManagement
@Slf4j
@EnableJpaRepositories(basePackages = "com.rsupport.seminar.core.repository")
@EnableJpaAuditing
public class JpaConfig implements TransactionManagementConfigurer {
  @Autowired
  private Environment env;
  @Bean
  public AuditorAware<String> auditorProvider() {
    return new SpringSecurityAuditorAware();
  }
  @Override
  public PlatformTransactionManager annotationDrivenTransactionManager() {
    return transactionManager();
  }
  @Bean
  public PersistenceExceptionTranslationPostProcessor persistenceExceptionTranslationPostProcessor() {
    return new PersistenceExceptionTranslationPostProcessor();
  }
  @Bean
  public JpaTransactionManager transactionManager() {
    JpaTransactionManager jpaTransactionManager = new JpaTransactionManager();
    jpaTransactionManager.setEntityManagerFactory(this.entityManagerFactory().getObject());
    jpaTransactionManager.setJpaDialect(new HibernateJpaDialect());
    return jpaTransactionManager;
  }
  @Bean
  public LocalContainerEntityManagerFactoryBean entityManagerFactory() {
    LocalContainerEntityManagerFactoryBean localContainerEntityManagerFactoryBean = new LocalContainerEntityManagerFactoryBean();
    Properties jpaProperties = new Properties();
    jpaProperties.setProperty("hibernate.dialect", env.getProperty("hibernate.dialect"));
    jpaProperties.setProperty("hibernate.query.substitutions", "true 'Y', false 'N'");
    jpaProperties.setProperty("hibernate.show_sql", "false");
    jpaProperties.setProperty("hibernate.format_sql", env.getProperty("hibernate.format_sql"));
    jpaProperties.setProperty("hibernate.use_sql_comments", "false");
    jpaProperties.setProperty("hibernate.hbm2ddl.auto", env.getProperty("hibernate.hbm2ddl.auto"));
    jpaProperties.setProperty("org.hibernate.envers.audit_table_suffix", "_h");
    jpaProperties.setProperty("hibernate.jdbc.use_scrollable_resultset", "false");
    jpaProperties.setProperty("javax.persistence.query.timeout", env.getProperty("jdbc.queryTimeout"));
    jpaProperties.setProperty("hibernate.jdbc_batchsize", env.getProperty("jdbc.batch_size"));
    jpaProperties.setProperty("org.hibernate.envers.store_data_at_delete", "true");
    jpaProperties.setProperty("hibernate.id.new_generator_mappings", "false");
    jpaProperties.setProperty("org.hibernate.envers.global_with_modified_flag", "true");
    jpaProperties.setProperty("org.hibernate.envers.modified_flag_suffix", "_mod");
    // cache common
    jpaProperties.setProperty("hibernate.cache.use_second_level_cache", "true");
    jpaProperties.setProperty("hibernate.cache.use_query_cache", "true");
    // ehcache
    jpaProperties.setProperty("hibernate.cache.provider_class", "org.hibernate.cache.EhCacheProvider");
    jpaProperties.setProperty("hibernate.cache.region.factory_class", "org.hibernate.cache.ehcache.EhCacheRegionFactory");


    localContainerEntityManagerFactoryBean.setPersistenceUnitName(GenericRepositoryJpa.PERSISTENCE_UNIT_NAME);
    localContainerEntityManagerFactoryBean.setJpaProperties(jpaProperties);
    localContainerEntityManagerFactoryBean.setDataSource(this.dataSource());
    localContainerEntityManagerFactoryBean.setPackagesToScan("com.rsupport.seminar.core.domain"); // replace persistence.xml
    localContainerEntityManagerFactoryBean.setPersistenceProvider(new HibernatePersistenceProvider());
    return localContainerEntityManagerFactoryBean;
  }
  @Bean
  public DataSource dataSource() {
    BasicDataSource dataSource = new BasicDataSource();
    dataSource.setDriverClassName(env.getProperty("hibernate.connection.driver_class"));
    dataSource.setUrl(env.getProperty("hibernate.connection.url"));
    dataSource.setUsername(env.getProperty("hibernate.connection.username"));
    dataSource.setPassword(env.getProperty("hibernate.connection.password"));
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
  @Bean
  public DatabaseConfiguration config() {
    return new DatabaseConfiguration(dataSource(), "zt_config", "id", "value");
  }
  @Bean
  public NamedParameterJdbcTemplate namedParameterJdbcTemplate() {
    return new NamedParameterJdbcTemplate(dataSource());
  }
  @Bean
  public JdbcTemplate jdbcTemplate() {
    return new JdbcTemplate(dataSource());
  }
  @Bean
  public PropertiesFactoryBean sqlProperties() {
    PropertiesFactoryBean bean = new PropertiesFactoryBean();
    String dbType = env.getProperty("db.type");
    StringBuilder builder = new StringBuilder();
    builder.append("sql/");
    if (StringUtils.isNotEmpty(dbType)) {
      builder.append(dbType).append("_");
    }
    builder.append("native_query.xml");
    bean.setLocation(new ClassPathResource(builder.toString()));
    return bean;
  }
}
```
->
```
@Configuration
@EnableTransactionManagement
@Slf4j
@EnableJpaRepositories(basePackages = "com.rsupport.seminar.core.repository")
@EnableJpaAuditing
public class JpaConfig {
  @Autowired
  private Environment env;
  @Autowired
  private DataSource dataSource;
  @Bean
  public AuditorAware<String> auditorProvider() {
    return new SpringSecurityAuditorAware();
  }
  @Bean
  public PersistenceExceptionTranslationPostProcessor persistenceExceptionTranslationPostProcessor() {
    return new PersistenceExceptionTranslationPostProcessor();
  }
  @Bean
  public LocalContainerEntityManagerFactoryBean entityManagerFactory(
      EntityManagerFactoryBuilder builder) {
    return builder
        .dataSource(dataSource)
        .packages("com.rsupport.seminar.core.domain")
        .persistenceUnit(GenericRepositoryJpa.PERSISTENCE_UNIT_NAME)
        .build();
  }
  @Bean
  public DatabaseConfiguration config() {
    return new DatabaseConfiguration(dataSource, "zt_config", "id", "value");
  }
  @Bean
  public NamedParameterJdbcTemplate namedParameterJdbcTemplate() {
    return new NamedParameterJdbcTemplate(dataSource);
  }
  @Bean
  public JdbcTemplate jdbcTemplate() {
    return new JdbcTemplate(dataSource);
  }
  @Bean
  public PropertiesFactoryBean sqlProperties() {
    PropertiesFactoryBean bean = new PropertiesFactoryBean();
    String dbType = env.getProperty("db.type");
    StringBuilder builder = new StringBuilder();
    builder.append("sql/");
    if (StringUtils.isNotEmpty(dbType)) {
      builder.append(dbType).append("_");
    }
    builder.append("native_query.xml");
    bean.setLocation(new ClassPathResource(builder.toString()));
    return bean;
  }
}

```
### TestCode
test/CoreApplication.java
```
@SpringBootApplication(scanBasePackages = "com.rsupport.seminar.core")
public class CoreApplication {
}
```
BaseTests.java
```
@ContextConfiguration(classes = { TestConfig.class, TestSecurityConfig.class})
public abstract class BaseTestCase extends AbstractJUnit4SpringContextTests {
  protected final Log log = LogFactory.getLog(getClass());
}
```
->
```
@RunWith(SpringRunner.class)
@SpringBootTest
public abstract class BaseTestCase extends AbstractJUnit4SpringContextTests {
```
BaseTransactionalTestCase.java
```

@ContextConfiguration(classes = { TestConfig.class, TestSecurityConfig.class})
public abstract class BaseTransactionalTestCase extends AbstractTransactionalJUnit4SpringContextTests {
  protected final Log log = LogFactory.getLog(getClass());
  @PersistenceContext(unitName = GenericRepositoryJpa.PERSISTENCE_UNIT_NAME)
  protected EntityManager entityManager;
  protected void flushAndClear() {
    entityManager.flush();
    entityManager.clear();
  }
}
```
->
```
@SpringBootTest
public abstract class BaseTransactionalTestCase extends AbstractTransactionalJUnit4SpringContextTests {
```

## Spring Boot Application 적용 - web

### pom.xml
* spring-boot-maven-plugin 추가  
* jetty-maven-plugin 제거  
* jsp 사용을 위한 설정 추가
```
    <dependency>
      <groupId>org.apache.tomcat.embed</groupId>
      <artifactId>tomcat-embed-jasper</artifactId>
      <scope>provided</scope>
    </dependency>
```    

### WebApplication.java, application.properties
```
@SpringBootApplication(scanBasePackages = "com.rsupport.seminar")
public class WebApplication extends SpringBootServletInitializer {
  public static void main(String[] args) {
    SpringApplication.run(WebApplication.class, args);
  }
  @Override
  protected SpringApplicationBuilder configure(SpringApplicationBuilder application) {
    return application.sources(WebApplication.class);
  }
}
```
### WebAppInitializer 제거, WebConfig 추가
Listener, Filter 설정 변경
WebConfig.java
```
@Configuration
public class WebConfig {

  @Bean
  public IntrospectorCleanupListener introspectorCleanupListener() {
    return new IntrospectorCleanupListener();
  }

  @Bean
  public RequestContextListener requestContextListener() {
    return new RequestContextListener();
  }
  
  @Bean
  public CodeGroupLoader codeGroupLoader() {
    return new CodeGroupLoader();
  }

  @Bean
  public BaseStartupListener BaseStartupListener() {
    return new BaseStartupListener();
  }

  @Bean
  public Filter newHttpServletRequestFilter() {
    return new HttpServletRequestFilter();
  }

  @Bean
  public XSSFilter newXSSFilter() {
    return new XSSFilter();
  }

  @Bean
  public Filter webSslRedirectFilter() {
    return new WebSslRedirectFilter();
  }

  @Bean
  public WebSiteMeshFilter siteMeshFilter() {
    return new WebSiteMeshFilter();
  }
}
```
### /public(public.war)를 배포파일에 추가
```
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-dependency-plugin</artifactId>
        <version>3.1.1</version>
        <executions>
          <execution>
            <id>unpack</id>
            <phase>prepare-package</phase>
            <goals>
              <goal>unpack</goal>
            </goals>
            <configuration>
              <artifactItems>
                <artifactItem>
                  <groupId>${project.groupId}</groupId>
                  <artifactId>public</artifactId>
                  <version>${project.parent.version}</version>
                  <type>war</type>
                  <outputDirectory>${basedir}/target/classes/static/public</outputDirectory>
                  <excludes>WEB-INF/,META-INF/,admin-new/,partner/,solution/</excludes>
                  <overWrite>true</overWrite>
                </artifactItem>
              </artifactItems>
            </configuration>
          </execution>
        </executions>
      </plugin>
```
/resource/static/** 파일이 접근이 안되는 경우 다음 코드 추가
```
 registry.addResourceHandler("/public/**").addResourceLocations("classpath:/static/public/");
```
# Spring Boot Application 적용 - api
## pom.xml
* war -> jar
* spring-boot-maven-plugin 추가
* jetty-maven-plugin 제거
* version 파일 생성 위치 변경
    ```
    <version.filename>src/main/resources/static/version.txt</version.filename>
    ```
## ApiApplication.java, application.properties
```
@SpringBootApplication(scanBasePackages = "com.rsupport.seminar", exclude = {MongoAutoConfiguration.class, MongoDataAutoConfiguration.class, SpringDataWebAutoConfiguration.class})
public class ApiApplication {
  public static void main(String[] args) {
    SpringApplication.run(ApiApplication.class, args);
  }
}
```
### WebAppInitializer 제거, WebConfig 추가
filter에 Url을 지정해야 하는 경우 FilterRegistrationBean을 사용
```
@Configuration
public class WebConfig {
  @Bean
  public WebSecurityFilter webSecurityFilter() {
    return new WebSecurityFilter();
  }
  @Bean
  public FilterRegistrationBean conferenceApiRequestLoggingFilter() {
    ConferenceApiRequestLoggingFilter logginFilter = new ConferenceApiRequestLoggingFilter();
    logginFilter.setIncludePayload(true);
    logginFilter.setIncludeQueryString(true);
    logginFilter.setMaxPayloadLength(10000);
    FilterRegistrationBean filterRegistration = new FilterRegistrationBean();
    filterRegistration.setFilter(logginFilter);
    filterRegistration.setMatchAfter(false);
    filterRegistration.addUrlPatterns("/conference/*");
    return filterRegistration;
  }
  @Bean
  public HttpServletRequestFilter httpServletRequestFilter() {
    return new HttpServletRequestFilter();
  }
  @Bean
  public FilterRegistrationBean serverAuthorizationFilter() {
    ServerAuthorizationFilter serverAuthorizationFilter = new ServerAuthorizationFilter();
    FilterRegistrationBean filterRegistration = new FilterRegistrationBean();
    filterRegistration.setFilter(serverAuthorizationFilter);
    filterRegistration.setMatchAfter(false);
    filterRegistration.addUrlPatterns("/advancedServer", "/advancedServer/*", "/mediaServer", "/mediaServer/*", "/documentServer");
    return filterRegistration;
  }
}
```

