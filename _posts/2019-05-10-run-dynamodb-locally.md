---
layout: post
title:  "Run DynamoDB locally - Windows"
date:   2019-05-10 21:00:00 +0900
categories: Backend
tag: SpringBoot
---

* content
{:toc}

## docker로 실행하기
```
docker run -p 8000:8000 amazon/dynamodb-local
```
  
![Local DynamoDB]({{site.url}}/assets/images/2019-05/dynamodb-local-01.png)

## Console 접속
localhost:8000/shell/ 로 접속하여 credential setting  
![Local DynamoDB]({{site.url}}/assets/images/2019-05/dynamodb-local-02.png)  

![Local DynamoDB]({{site.url}}/assets/images/2019-05/dynamodb-local-03.png)  

## AWS CLI 사용하기
위에서 설정한 accessKeyId로 변경한 후 해보자
![Local DynamoDB]({{site.url}}/assets/images/2019-05/dynamodb-local-04.png)  


아무거나로 셋팅해보면
![Local DynamoDB]({{site.url}}/assets/images/2019-05/dynamodb-local-05.png)  

aws cli로 접속시 local db는 credential 필요없다

## References
[AWS-developerguide](https://docs.aws.amazon.com/ko_kr/amazondynamodb/latest/developerguide/DynamoDBLocal.DownloadingAndRunning.html)  
[https://hub.docker.com/r/amazon/dynamodb-local](https://hub.docker.com/r/amazon/dynamodb-local)