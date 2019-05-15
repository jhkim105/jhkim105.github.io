---
layout: post
title:  "Spring Boot Local DynamoDB"
date:   2019-05-13 20:00:00 +0900
categories: Backend
tag: SpringBoot
---

* content
{:toc}


# Overview
Spring Boot을 사용하여 Local DynamoDB와 연동하는 샘플 코드를 작성해보았다. 언어는 요즘 학습중인 Kotlin을 사용하였다.

# Run DynamoDB with docker
```
docker run -p 8000:8000 amazon/dynamodb-local
```

# Spring Boot DynamoDB
## maven dependency
```
  <dependencyManagement>
    <dependencies>
      <dependency>
        <groupId>org.springframework.data</groupId>
        <artifactId>spring-data-releasetrain</artifactId>
        <version>Hopper-SR10</version>
        <type>pom</type>
        <scope>import</scope>
      </dependency>
    </dependencies>
  </dependencyManagement>

  <dependencies>
    <!-- dynamodb -->
    <dependency>
      <groupId>com.amazonaws</groupId>
      <artifactId>aws-java-sdk-dynamodb</artifactId>
      <version>1.11.545</version>
    </dependency>
    <dependency>
      <groupId>com.github.derjust</groupId>
      <artifactId>spring-data-dynamodb</artifactId>
      <version>4.3.1</version>
    </dependency>
    <!-- //dynamodb -->
  </dependencies>
  ```

## application.properties
Local DynamoDB 접속시에는 endpoint만 있으면 되고, accessKey, secretKey, region은 없어도 된다. DynamoDB setting시에 accessKey를 지정할 수 있게 되어 있는데, 그 키를 사용하지 않아도 접속이 된다. AWS DynamoDB를 사용할때에는 accessKey, secretKey, region 정보가 필요하다.
```
amazon.aws.accessKey=
amazon.aws.secretKey=
amazon.dynamoDB.region=
amazon.dynamodb.endpoint=http://10.1.100.164:8000
```

## DynamoDB Config
Local DynamoDB를 사용하려면 endpoint를 지정해야하므로, endpoint가 있을 경우와 없을 경우를 나누어 처리했다.
```
@Configuration
@EnableDynamoDBRepositories(basePackages = ["com.rsupport.emailchecker"])
class DynamoDBConfig {
    @Value("\${amazon.aws.accessKey}")
    lateinit var amazonAWSAccessKey: String
    @Value("\${amazon.aws.secretKey}")
    lateinit var amazonAWSSecretKey: String
    @Value("\${amazon.dynamodb.region}")
    lateinit var amazonDynamoDBRegion: String
    @Value("\${amazon.dynamodb.endpoint}")
    lateinit var amazonDynamoDBEndpoint: String
    @Bean
    fun amazonDynamoDB(): AmazonDynamoDB {
        return if(amazonDynamoDBEndpoint.isBlank()) {
            AmazonDynamoDBClientBuilder.standard()
                    .withRegion(Regions.fromName(amazonDynamoDBRegion))
                    .withCredentials(amazonAWSCredentialProvider())
                    .build()
        } else {
            AmazonDynamoDBClientBuilder.standard()
                    .withEndpointConfiguration(
                            AwsClientBuilder.EndpointConfiguration(amazonDynamoDBEndpoint, amazonDynamoDBRegion))
                    .withCredentials(amazonAWSCredentialProvider())
                    .build()
        }
    }
    @Bean
    fun amazonAWSCredentials() = BasicAWSCredentials(amazonAWSAccessKey, amazonAWSSecretKey)
    @Bean
    fun amazonAWSCredentialProvider() = AWSStaticCredentialsProvider(amazonAWSCredentials())
}
```
## Entity
식별자 지정을 위한 @DynamoDBHashKey, 자동생성을 위한 @DynamoDBAutoGeneratedKey 애노테이션을 지정한다. JPA에서@Id, @GeneratedValue와 같은 역할을 한다.  
```
@DynamoDBTable(tableName = "MailSendLog")
data class MailSendLog(
        @DynamoDBAttribute
        var email: String?,
        @DynamoDBAttribute
        var sendDate: Date?
) {
    @DynamoDBHashKey
    @DynamoDBAutoGeneratedKey
    var id: String? = null
    constructor(): this(null, null)
}
```

## Repository

```
@EnableScan
interface MailSendLogRepository: CrudRepository<MailSendLog, String>
```

## TestCode
```
@RunWith(SpringJUnit4ClassRunner::class)
@SpringBootTest
class MailSendLogRepositoryTest  {

    private val log = LoggerFactory.getLogger(javaClass)

    @Autowired
    private lateinit var amazonDynamoDB: AmazonDynamoDB

    @Autowired
    private lateinit var repository: MailSendLogRepository

    private lateinit var dynamoDBMapper: DynamoDBMapper

    @Before
    fun setUp() {
        dynamoDBMapper = DynamoDBMapper(amazonDynamoDB)
        val tableRequest = dynamoDBMapper.generateCreateTableRequest(MailSendLog::class.java)
        tableRequest.provisionedThroughput = ProvisionedThroughput(1L, 1L)
        amazonDynamoDB.createTable(tableRequest)
    }

    @After
    fun tearDown() {
        amazonDynamoDB.deleteTable("MailSendLog")
    }

    @Test
    fun save() {
        val mailSendLog = MailSendLog("jihwan.kim@rsupport.com", Date())
        repository.save(mailSendLog)

        val result = repository.findAll()
        assertThat(result.toList().size, `is`(greaterThan(0)))
    }


    @Test
    fun findAll() {
        save()
        var result = repository.findAll()
        log.debug("result -> ${result.toList()}")
    }
}
```