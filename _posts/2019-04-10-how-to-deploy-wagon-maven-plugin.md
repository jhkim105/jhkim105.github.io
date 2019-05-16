---
layout: post
title:  "How to deploy - wagon-maven-plugin"
date:   2019-04-10 20:00:00 +0900
categories: DevOps
tag: Maven
---

* content
{:toc}


# Overview
wagon-maven-plugin을 사용하여 배포 환경을 구성할 수 있다. ssh(scp)를 사용하여 배포하므로, 대상서버로 ssh 접근이 가능해야 한다. 실서버 배포에는 보안상 부적합하고, 개발서버나 Staging서버 배포시에 활용하면 좋겠다.

# deploy.xml
```
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <parent>
    <groupId>com.rsupport</groupId>
    <artifactId>logcollector</artifactId>
    <version>1.0.1-SNAPSHOT</version>
  </parent>
  <groupId>com.rsupport.logcollector</groupId>
  <artifactId>deploy</artifactId>
  <version>1.0</version>
  <packaging>pom</packaging>
  <name>deploy</name>
  <properties>
  </properties>
  <build>
    <plugins>
      <plugin>
        <groupId>org.codehaus.mojo</groupId>
        <artifactId>wagon-maven-plugin</artifactId>
        <version>2.0.0</version>
        <configuration>
          <displayCommandOutputs>true</displayCommandOutputs>
          <commands>
            <command>/etc/init.d/${deploy.name} restart</command>
          </commands>
        </configuration>
        <executions>
          <execution>
            <id>upload</id>
            <goals>
              <goal>upload-single</goal>
            </goals>
          </execution>
          <execution>
            <id>restart</id>
            <goals>
              <goal>sshexec</goal>
            </goals>
          </execution>
        </executions>
      </plugin>
    </plugins>
  </build>
</project>
```

# settings.xml
```
<?xml version="1.0" encoding="UTF-8"?>
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 http://maven.apache.org/xsd/settings-1.0.0.xsd">
  <servers>
    <server>
      <id>server-stap</id>
      <configuration>
        <knownHostsProvider implementation="org.apache.maven.wagon.providers.ssh.knownhost.NullKnownHostProvider">
          <hostKeyChecking>no</hostKeyChecking>
        </knownHostsProvider>
      </configuration>
      <username>root</username>
      <password>gosemffj#4430</password>
    </server>
  </servers>
  <profiles>
    <profile>
      <id>rc-stap</id>
      <properties>
        <project.rootdir>/DATA/jenkins/workspace/logcollector-st-build</project.rootdir>
        <deploy.rootdir>/DATA/WEB/collector</deploy.rootdir>
        <wagon.serverId>server-stap</wagon.serverId>
        <wagon.url>scp://<배포대상서버 address></wagon.url>
        <wagon.fromFile>${project.rootdir}/${deploy.name}/target/${deploy.name}-${deploy.version}.war</wagon.fromFile>
        <wagon.toFile>${deploy.rootdir}/${deploy.name}.war</wagon.toFile>
      </properties>
    </profile>
  </profiles>
  <activeProfiles>
    <activeProfile>rc-stap</activeProfile>
  </activeProfiles>
</settings>
```

# Maven command로 IntelliJ에서 실행
```
wagon:upload-single wagon:sshexec -e -Ddeploy.version=1.0.1-SNAPSHOT -Ddeploy.name=collector -f build/deploy.xml -P rc-stap
```
![IntelliJ에서 실행]({{site.url}}/assets/images/2019-05/wagon-intellij.png)   

# Jenkins
![Jenkins-1]({{site.url}}/assets/images/2019-05/wagon-01.png)   
![Jenkins-2]({{site.url}}/assets/images/2019-05/wagon-02.png)  