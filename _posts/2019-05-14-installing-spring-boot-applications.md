---
layout: post
title:  "Installing Spring Boot Appliations"
date:   2019-05-14 20:00:00 +0900
categories: Backend
tag: SpringBoot
---


* content
{:toc}




# Overview
Spring Boot application(Collector)을 Linux Server에 구성하는 방법에 대해 설명하겠다.  

# 폴더 권한 변경
로그인 계정으로 배포하기 위해 작업디렉토리 권한을 변경한다.
```
su -
chown -R webdev1:webdev1 /DATA/WEB/
```
![폴더권한변경]({{site.url}}/assets/images/2019-05/installing-spring-boot-01.png)  

# Maven 설치 및 구성
maven-dependency-plugin을 사용하여 배포하기 위해 maven을 설치한다.
## Maven 설치
```
sudo wget http://repos.fedorapeople.org/repos/dchen/apache-maven/epel-apache-maven.repo -O /etc/yum.repos.d/epel-apache-maven.repo
sudo sed -i s/\$releasever/6/g /etc/yum.repos.d/epel-apache-maven.repo
sudo yum install -y apache-maven
mvn --version
```
## Maven Configuration
Repository 폴더 생성
```
mkdir /DATA/WEB/.m2
```
경로지정 및 Maven Repo 추가: /etc/maven/settings.xml
```
  <localRepository>/DATA/WEB/.m2/repo</localRepository>
  ...
    <profile>
      <id>rsupport-repo</id>
      <repositories>
        <repository>
          <id>rsupport-public</id>
          <url>http://repo.rsupport.com:8081/repository/maven-public</url>
          <snapshots>
            <enabled>true</enabled>
            <updatePolicy>always</updatePolicy>
          </snapshots>
        </repository>
      </repositories>
    </profile>
  </profiles>
  <activeProfiles>
    <activeProfile>rsupport-repo</activeProfile>
  </activeProfiles>
```

# 소스 배포
배포 스크립트: /DATA/WEB/collector/deploy.sh  
```
#!/bin/bash

if [ "$1" == "" ]
then
  echo "Usage: $0 <VERSION>"
  exit 1
fi

SERVICE_NAME=collector
PACKAGE=war
HOME_DIR=/DATA/WEB/$SERVICE_NAME
VER=$1-SNAPSHOT

function download_file() {
  DEST=$HOME_DIR/$SERVICE_NAME.$PACKAGE
  echo "download file...VER:$VER"
  mvn org.apache.maven.plugins:maven-dependency-plugin:3.0.0:copy -Dartifact=com.rsupport.logcollector:$SERVICE_NAME:$VER:$PACKAGE -DoutputDirectory=$HOME_DIR -Dmdep.useBaseVersion=true
  echo "download complete..."
  cd $HOME_DIR
  /bin/cp $SERVICE_NAME-$VER.$PACKAGE $SERVICE_NAME.$PACKAGE
  rm -f $SERVICE_NAME-$VER.$PACKAGE
  echo "copy complete..."
}

function stop_service() {
  service $SERVICE_NAME stop
}

function start_service() {
  service $SERVICE_NAME start
}
stop_service
download_file
start_service
```

Shell 실행하여 소스 배포하기
```
./deploy.shell  1.0.2
```
![소스배포]({{site.url}}/assets/images/2019-05/installing-spring-boot-01.png)  


conf 파일 작성: collector.conf
```
JAVA_OPTS="-Xmx2G"
#JAVA_OPTS="$JAVA_OPTS -Dspring.config.location=classpath:/application.properties,file:/DATA/WEB/collector/config/application.properties"
RUN_ARGS="--server.port=18080"
LOG_FOLDER=/DATA/WEB/collector/log
```

config: application.properties
```
logging.level.com.rsupport.logcollector=debug
server.hostname=SEA-WL-RC-001
service.name=rc
logcollector.analyzer.url=http://logcollect.rsupport.com:9880/${service.name}
logcollector.crashlog.url=http://logcollect.rsupport.com:9880/crashlog
```

# 서비스 등록
```
chmod a+x collector.war
su -
ln -s /DATA/WEB/collector/collector.war /etc/init.d/collector
chkconfig collector on
```

# 실행
최초 한번은 root 권한으로 실행해야 한다. 안그러면 Operation not permitted (cannot access pid file) 에러 발생함. 이후부터는 로그인 계정(webdev1)으로 실행가능하다.
```
service collector start
```

# logrotate 설정
logback 설정으로 롤링이 가능하지만, 이 프로젝트에서는 logback을 사용하지 않았으므로, logrotate를 사용하여 롤링 설정을 한다.
/etc/logrotate.d/collector
```
/DATA/WEB/collector/log/collector.log {
    copytruncate
    daily
    rotate 30
    compress
    missingok
    notifempty
    dateext
    create 0644 webdev1 webdev1
}
```




