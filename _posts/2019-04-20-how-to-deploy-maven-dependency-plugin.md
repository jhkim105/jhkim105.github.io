---
layout: post
title:  "How to Deploy - maven-depedency-plugin"
date:   2019-04-20 10:55:00 +0900
categories: DevOps
tag: Maven
---

* content
{:toc}


maven-depdency-plugin을 사용하여 배포하기


------------

# Overview

현재 회사에서 배포와 관련에서 사용하는 Maven Plugin은 cargo2-maven-plugin, wagon-maven-plugin, maven-depdency-plugin이 있다.
 
이번에 RemoteView 배포에 적용한 maven-depdency-plugin을 사용해서 WAR를 배포하는 방법에 대해서 알아보자.
이 방법을 사용하게 된 배경은 cargo2-maven-plugin은 Tomcat Manager를 사용해서 http로 배포하는데, 속도가 느라고 서버에서 tomcat manager를 열어줘야 하는 단점이 있다. wagon-maven-plugin은 scp를 사용하기 때문에 속도는 빠르지만 빌드서버에서 scp로 운영서버에 접근이 가능해야하므로 보안상 꺼려질 수 있어서, maven-depdency-plugin을 사용해서 Nexus에서 직접 war파일을 다운로드 하는 방법을 사용하게 되었다.


# Configure Maven

먼저 서버에  Maven이 설치되어 있어야 한다.
```
sudo wget http://repos.fedorapeople.org/repos/dchen/apache-maven/epel-apache-maven.repo -O /etc/yum.repos.d/epel-apache-maven.repo
sudo sed -i s/\$releasever/6/g /etc/yum.repos.d/epel-apache-maven.repo
sudo yum install -y apache-maven
```

 Local repositoyr path 및 Nexus Repository 등록 (/etc/maven/settings.xml)
```
<localRepository>/DATA/WEB/.m2/repository</localRepository>
```
```
    <profile>
      <id>default</id>
      <repositories>
        <repository>
          <id>rsupport-public</id>
          <url>http://xxx.xxx.xxx/repository/maven-public</url>
          <snapshots>
            <enabled>true</enabled>
            <updatePolicy>always</updatePolicy>
          </snapshots>          
        </repository>
      </repositories>
    </profile>
```
```
<activeProfiles>
    <activeProfile>default</activeProfile>
</activeProfiles>
```
![Deploy result]({{site.url}}/assets/images/maven-installed-version.png)


모듈 다운로드 테스트
```
mvn org.apache.maven.plugins:maven-dependency-plugin:3.0.0:copy -Dartifact=com.rsupport.rc5x:rc5x-public:6.0.24.1-SNAPSHOT:war -DoutputDirectory=D:\@ -Dmdep.useBaseVersion=true
```

# Create deploy script

deploy.sh
```
#!/bin/bash

if [ "$1" == "" ] || [ "$2" == "" ] || [ "$3" == "" ] || [ "$4" == "" ]
then
  echo "Usage: $0 <SERVICE_NAME> <SOURCE_NAME> <TARGET_NAME> <VERSION>"
  echo "$0 web web ROOT 1.0.1"
  echo "$0 web jms jms 1.0.1"
  exit 1
fi

SERVICE_NAME=$1
SOURCE_NAME=$2
TARGET_NAME=$3
VER=$4-SNAPSHOT
PACKAGE=war
HOME_DIR=/DATA/WEB/rv5x/webapps/$SERVICE_NAME
WORK_DIR=/DATA/WEB/rv5x/webapps/tmp

function remove_dir() {
  echo "remove dir...$HOME_DIR/$TARGET_NAME"
  rm -rf $HOME_DIR/$TARGET_NAME 2>/dev/null
  echo "remove complete..."
}

function download_file() {
  echo "download file...VER:$VER"
  mvn org.apache.maven.plugins:maven-dependency-plugin:3.0.0:copy -Dartifact=com.rsupport.rv5x:$SOURCE_NAME:$VER:$PACKAGE -DoutputDirectory=$WORK_DIR -Dmdep.useBaseVersion=true
  echo "download complete..."
  cd $WORK_DIR
  /bin/cp $SOURCE_NAME-$VER.$PACKAGE $HOME_DIR/$TARGET_NAME.$PACKAGE
  rm -f $SOURCE_NAME-$VER.$PACKAGE
  echo "copy complete..."
}


remove_dir
download_file
```
download-web.sh
```
#!/bin/bash
if [ "$1" == "" ]
then
  echo "Usage: $0 <VERSION>"
  exit 1
fi
VER=$1
function deploy() {
  /DATA/WEB/rv5x/bin/deploy.sh web $1 $2 $VER
}
deploy "web" "ROOT"
deploy "batch" "batch"
deploy "jms" "jms"
deploy "public" "public"
deploy "web" "v2"
```
deploy-all.sh
```
#!/bin/bash
if [ "$1" == "" ]
then
  echo "Usage: $0 <VERSION>"
  exit 1
fi
VER=$1
/etc/intit.d/tomcat8 stop
/DATA/WEB/rv5x/bin/deploy-web.sh $VER
/DATA/WEB/rv5x/bin/deploy-admin.sh $VER
/DATA/WEB/rv5x/bin/deploy-upload.sh $VER
/etc/init.d/tomcat8 start
```
동일한 방식으로 RemoteCall Beta 적용함
![RC beta]({{site.url}}/assets/images/deploy-rc-shell.png)


## Deploy to embedded Tomcat(Spring Boot)

Embedded Tomcat에서 운영되는 Spring Boot jar/war인 경우에는 웹 소스 폴더를 삭제해줄 필요가 없어 배포 스크립트에 해당 부분이 필요없다.


로그수집서버(Collector)에 적용한 script
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

![Deploy result]({{site.url}}/assets/images/collector-deploy-result.png)

service XXX stop으로 서비스가 종료되지 않는 경우가 있다(RabbitMQ를 사용하는 경우). 다음과 같이 직접 프로세스를 Kill하는 script가 필요한 경우가 있다.

stop.sh
```
#!/bin/bash
if [ "$1" == "" ]
then
  echo "Usage: $0 <name>"
  exit 1
fi

SERVICE_NAME=$1
PID_DIR=/var/run/$SERVICE_NAME
PID_FILE=$SERVICE_NAME.pid
IDENTIFIER=$SERVICE_NAME.jar

function kill_process() {
  echo "$SERVICE_NAME destroy started..."

  PID=`cat $PID_DIR/$PID_FILE 2>/dev/null`
  if [ ! -z $PID ]; then
    echo "shutting down $SERVICE_NAME. pid=$PID."
    # kill -9 $PID
  else
    echo "Not running (pidfile not found)"
  fi

  process_count=`ps aux | grep $IDENTIFIER | grep -v grep | wc -l`
  if [ ! $process_count -eq 0 ]; then
    echo "shuttind down $SERVICE_NAME. identifier=$IDENTIFIER."
    ps aux | grep $IDENTIFIER | grep -v grep | awk '{print $2}' | xargs kill -9
  fi

  process_count=`ps aux | grep $IDENTIFIER | grep -v grep | wc -l`
  if [ $process_count -eq 0 ]; then
    echo "$SERVICE_NAME destroy completed."
  else
    echo "$SERVICE_NAME destroy failed."
  fi
}

function clear_pid() {
  if [ -f  $PID_DIR/$PID_FILE ]; then
    echo "rm $PID_DIR/$PID_FILE."
    rm $PID_DIR/$PID_FILE
  fi
}

kill_process
clear_pid
```
