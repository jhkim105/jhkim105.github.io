---
layout: post
title:  "Migrating from Java to Kotlin"
date:   2019-05-02 22:00:00 +0900
categories: Backend
tag: Kotlin
---

* content
{:toc}


Migrating from Spring Boot Java to Kotlin


------------
# Overview
* Spring Boot Java로 구현된 프로젝트를 Kotlin으로 변경하기
* 가장 간단한 LogCollector 프로젝트를 대상으로 진행
* LogCollector
  * ErrorLog를 FluentD에 전송(http)
  * 첨부파일을 S3에 Upload
  * DB(Repository) 없음
* TestCode부터 진행
* IntelliJ에서 제공하는 기능을 사용

# Convert Java To Kotlin
IntelliJ Convert Java File to Kotlin File 기능 사용
![]({{site.url}}/assets/images/java2kotlin.png)

처음 실행하는 경우 Kotlin 환경 구성 관련 알림이 뜬다
![]({{site.url}}/assets/images/java2kotlin2.png)

![]({{site.url}}/assets/images/java2kotlin3.png)

OK를 누르고 나면 pom.xml에 Kotlin관련 설정이 추가됨
![]({{site.url}}/assets/images/java2kotlin4.png)

![]({{site.url}}/assets/images/java2kotlin5.png)

# TestCode 변환

## CollectorApplicationTests
![]({{site.url}}/assets/images/java2kotlin6-1.png)
  
->  
![]({{site.url}}/assets/images/java2kotlin6-2.png)

## DateUtilsTest
![]({{site.url}}/assets/images/java2kotlin7-1.png)
  
->  
![]({{site.url}}/assets/images/java2kotlin7-2.png)
  
Kotlin code에서 Lombok으로 생성된 메소드(Gettter/Setter)를 접근 못함(Kotlin이 먼저 Compile 되기 때문)
@Slf4j를 직접 코드로 구현. 로깅에 관한 방법은 여러가지가 있다.  
https://discuss.kotlinlang.org/t/best-practices-for-loggers/226/2  
https://www.baeldung.com/kotlin-logging  
가장 간단한 방법으로 구현

```
private val log = LoggerFactory.getLogger(javaClass)
```

## ErrorLogControllerTest
![]({{site.url}}/assets/images/java2kotlin8-1.png)
  
->
![]({{site.url}}/assets/images/java2kotlin8-2.png)

Compile Error 수정
![]({{site.url}}/assets/images/java2kotlin8-3.png)

## S3UtilsTest
![]({{site.url}}/assets/images/java2kotlin9-1.png)  

->  
![]({{site.url}}/assets/images/java2kotlin9-2.png)

접근제어자는 internal, private, protected, public  
지정하지 않으면 public  
lateinit을 사용하여 ?  및 = null 제거

![]({{site.url}}/assets/images/java2kotlin9-3.png)

# VO(DTO)
## CrashLogRequest
![]({{site.url}}/assets/images/java2kotlin10-1.png)  
  
->
Getter/Setter  
![]({{site.url}}/assets/images/java2kotlin10-2.png)
  
Getter  
![]({{site.url}}/assets/images/java2kotlin10-3.png)


Getter 접근시 property를 접근함  
![]({{site.url}}/assets/images/java2kotlin10-4.png)  

Request VO class는 data OR 일반 클래스?  
일반 class로 만들 경우 Java처럼 private, getter, setter 방식으로 하는게 맞을까?   
Kotlin에서 Getter/Setter는 다음과 같이 
```
var stringRepresentation: String
    get() = this.toString()
    set(value) {
        setDataFromString(value) // parses the string and assigns values to other properties
    }
```

## CrashLog - Builder 구현
Lombok에서 만든 코드를 Kotlin에서 참조할 수 없으므로 Builder를 직접 구현해야 한다.
![]({{site.url}}/assets/images/java2kotlin11-1.png)
->
```
class CrashLog : Serializable {
    var logKey: String? = null
        private set
    var productType: String? = null
        private set
    var errorMessage: String? = null
        private set
    var extAddress: String? = null
        private set
    var extCode: String? = null
        private set
    var extNumber: String? = null
        private set
    var clientIp: String? = null
    var userName: String? = null
        private set
    var userEmail: String? = null
        private set
    var userTel: String? = null
        private set
    var content: String? = null
        private set
    var filePath: String? = null
    var fileUrl: String? = null


    @JsonFormat(shape = JsonFormat.Shape.STRING, pattern = "yyyy-MM-dd'T'HH:mm:ssZ")
    var errorDate: Date? = null
        private set
    constructor(logKey: String?,
                errorDate: Date?,
                productType: String?,
                errorMessage: String?,
                extAddress: String?,
                extCode: String?,
                extNumber: String?,
                clientIp: String?,
                userName: String?,
                userEmail: String?,
                userTel: String?,
                content: String?,
                filePath: String?,
                fileUrl: String?) {
        this.logKey = logKey
        this.errorDate = errorDate
        this.productType = productType
        this.errorMessage = errorMessage
        this.extAddress = extAddress
        this.extCode = extCode
        this.extNumber = extNumber
        this.clientIp = clientIp
        this.userName = userName
        this.userEmail = userEmail
        this.userTel = userTel
        this.content = content
        this.filePath = filePath
        this.fileUrl = fileUrl
    }
    companion object {
        private const val serialVersionUID = 5749490740387087329L
        fun builder() = Builder()
    }


    override fun toString(): String {
        return "CrashLog(logKey=$logKey, productType=$productType, errorMessage=$errorMessage, extAddress=$extAddress, extCode=$extCode, extNumber=$extNumber, clientIp=$clientIp, userName=$userName, userEmail=$userEmail, userTel=$userTel, content=$content, filePath=$filePath, fileUrl=$fileUrl, errorDate=$errorDate)"
    }


    data class Builder(
            var logKey: String? = null,
            var errorDate: Date? = null,
            var productType: String? = null,
            var errorMessage: String? = null,
            var extAddress: String? = null,
            var extCode: String? = null,
            var extNumber: String? = null,
            var clientIp: String? = null,
            var userName: String? = null,
            var userEmail: String? = null,
            var userTel: String? = null,
            var content: String? = null,
            var filePath: String? = null,
            var fileUrl: String? = null
    ) {
        fun logKey(logKey: String) = apply { this.logKey = logKey }
        fun errorDate(errorDate: Date?) = apply {this.errorDate = errorDate}
        fun productType(productType: String?) = apply {this.productType = productType}
        fun errorMessage(errorMessage: String?) = apply {this.errorMessage = errorMessage}
        fun extAddress(extAddress: String?) = apply {this.extAddress = extAddress}
        fun extCode(extCode: String?) = apply {this.extCode = extCode}
        fun extNumber(extNumber: String?) = apply {this.extNumber = extNumber}
        fun clientIp(clientIp: String?) = apply {this.clientIp = clientIp}
        fun userName(userName: String?) = apply {this.userName = userName}
        fun userEmail(userEmail: String?) = apply {this.userEmail = userEmail}
        fun userTel(userTel: String?) = apply {this.userTel = userTel}
        fun content(content: String?) = apply {this.content = content}
        fun filePath(filePath: String?) = apply {this.filePath = filePath}
        fun fileUrl(fileUrl: String?) = apply {this.fileUrl = fileUrl}
        fun build() = CrashLog(logKey, errorDate, productType, errorMessage, extAddress, extCode, extNumber, clientIp, userName, userEmail, userTel, content, filePath, fileUrl)
    }
}
```


# Maven Build 순서 지정하기(Kotlin First)
maven build하면 target/에 class파일이 생성 안된다. CrashLogController.java에서 앞에서 Kotlin으로 변경하는 코드를 참조하는데, maven build에서는 기본적으로 maven-compiler가 가장 먼저 실행된다. 다음과 같이 설정을 추가해줘야 한다.
![]({{site.url}}/assets/images/java2kotlin12-1.png)

## kotlin-maven-plugin
```
      <plugin>
        <groupId>org.jetbrains.kotlin</groupId>
        <artifactId>kotlin-maven-plugin</artifactId>
        <version>${kotlin.version}</version>
        <executions>
          <execution>
            <id>compile</id>
            <phase>compile</phase>
            <goals>
              <goal>compile</goal>
            </goals>
            <configuration>
              <sourceDirs>
                <sourceDir>${project.basedir}/src/main/kotlin</sourceDir>
                <sourceDir>${project.basedir}/src/main/java</sourceDir>
              </sourceDirs>
              <jvmTarget>${java.version}</jvmTarget>
              <args>
                <arg>-Xjvm-default=enable</arg>
              </args>
            </configuration>
          </execution>
          <execution>
            <id>test-compile</id>
            <goals>
              <goal>test-compile</goal>
            </goals>
            <configuration>
              <jvmTarget>${java.version}</jvmTarget>
            </configuration>
          </execution>
        </executions>
        <configuration>
          <args>
            <arg>-Xjsr305=strict</arg>
          </args>
          <compilerPlugins>
            <plugin>spring</plugin>
          </compilerPlugins>
        </configuration>
        <dependencies>
          <dependency>
            <groupId>org.jetbrains.kotlin</groupId>
            <artifactId>kotlin-maven-allopen</artifactId>
            <version>${kotlin.version}</version>
          </dependency>
        </dependencies>
      </plugin>
```
## maven-compiler-plugin
```
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-compiler-plugin</artifactId>
        <configuration>
          <source>${java.version}</source>
          <target>${java.version}</target>
        </configuration>
        <executions>
          <!-- Replacing default-compile as it is treated specially by maven -->
          <execution>
            <id>default-compile</id>
            <phase>none</phase>
          </execution>
          <!-- Replacing default-testCompile as it is treated specially by maven -->
          <execution>
            <id>default-testCompile</id>
            <phase>none</phase>
          </execution>
          <execution>
            <id>java-compile</id>
            <phase>compile</phase>
            <goals>
              <goal>compile</goal>
            </goals>
          </execution>
          <execution>
            <id>java-test-compile</id>
            <phase>test-compile</phase>
            <goals>
              <goal>testCompile</goal>
            </goals>
          </execution>
        </executions>
      </plugin>

```


# Controller
![]({{site.url}}/assets/images/java2kotlin13-1.png)
  
->  
![]({{site.url}}/assets/images/java2kotlin13-2.png)
  
![]({{site.url}}/assets/images/java2kotlin13-3.png)

```
@RestController
@RequestMapping(path = ["/cl", "/svc/content/collect.aspx"])
class CrashLogController(private val runtimeProperties: RuntimeProperties,
                         private val logFileUploader: LogFileUploader) {

    private val log: Logger = LoggerFactory.getLogger(javaClass)
    @Value("\${logcollector.crashlog.url}")
    private val url: String? = null
    @Value("\${storage.url}")
    private val storageUrl: String? = null

    @PostMapping
    fun collect(crashLogRequest: CrashLogRequest, multipartRequest: MultipartRequest, request: HttpServletRequest): ResponseEntity<Any> {
        log.debug("req:{}", crashLogRequest)
        if (!runtimeProperties.isCollectorEnabled) {
            log.info("collector not enabled.")
        }
        val crashLog = crashLogRequest.toCrashLog()
        crashLog.clientIp = request.remoteAddr
        val multipartFile = multipartRequest.getFile("ErrFile")
        if (multipartFile != null && !multipartFile.isEmpty) {
            val path = logFileUploader.upload(crashLog.logKey, multipartFile)
            crashLog.filePath = path
            crashLog.fileUrl = getFileUrl(path)
        }
        log.debug("sendJson:{}", JsonUtils.toString(crashLog))
        sendLog(crashLog)
        return ResponseEntity(HttpStatus.OK)
    }

    private fun sendLog(crashLog: CrashLog) {
        val headers = HttpHeaders()
        headers.contentType = MediaType.APPLICATION_JSON_UTF8
        headers.accept = Arrays.asList(MediaType.APPLICATION_JSON_UTF8)
        val entity = HttpEntity(crashLog, headers)
        val timeout = 5 * 1000
        val restTemplate = RestTemplateUtils.getRestTemplate(timeout)
        log.debug("send log:url:{}", url)
        restTemplate.postForEntity(url, entity, String::class.java)
    }
    private fun getFileUrl(path: String): String {
        return if (storageUrl!!.endsWith("/")) storageUrl + path else "$storageUrl/$path"
    }
}
```
@Value 지정시 \를 앞에 추가해야 함 "\${}"

# data class
ErrorLogRequest는 data class로 해보자

## ErrorLogRequest
```
data class ErrorLogRequest(
        var errorCode: String?,
        var errorMessage: String?,
        var timestamp: Long?,
        var correlationId: String?,
        var fingerPrint: String?,
        var errorDetail: String?
) {
    private var log: Logger = LoggerFactory.getLogger(javaClass)
    fun toErrorLog(): ErrorLog {
        val errorLog = ErrorLog()
        BeanUtils.copyProperties(this, errorLog, "errorDetail")
        if (StringUtils.isNotBlank(this.errorDetail)) {
            try {
                val errorDetailMap = JsonUtils.toObject<Map<String, Any>>(this.errorDetail, HashMap::class.java)
                errorLog.errorDetail = errorDetailMap
            } catch (ex: RuntimeException) {
                log.warn("Invalid errorDetail. errorMessage:{}", ex.toString())
                errorLog.errorDetailString = this.errorDetail
            }
        }
        if (this.timestamp != null)
            errorLog.errorDate = Date(this.timestamp!!)
        return errorLog
    }
}
```

## ErrorLog
```
@Data
data class ErrorLog(var serverName: String?,
                    var logKey: String?,
                    @JsonFormat(shape = JsonFormat.Shape.STRING, pattern = "yyyy-MM-dd'T'HH:mm:ssZ")
                    var createdDate: Date?,
                    var errorCode: String?,
                    var errorMessage: String?,
                    @JsonFormat(shape = JsonFormat.Shape.STRING, pattern = "yyyy-MM-dd'T'HH:mm:ssZ")
                    var errorDate: Date?,
                    var correlationId: String?,
                    var fingerPrint: String?,
                    var errorDetail: Any?,
                    var errorDetailString: String?,
                    var filePath: String?,
                    var fileUrl: String?,
                    var clientIp: String?,
                    var clientInfo: ClientInfo?
) {


    init {
        this.logKey = UUID.randomUUID().toString()
        this.createdDate = Date()
    }
}
```
다음 코드에서 compile Error
```
val errorLog = ErrorLog()
```
->
다음과 같이 초기화를 해주면 default constructor를 호출할 수 있음
```
@Data
data class ErrorLog(var serverName: String? = null,
                    var logKey: String? = null,
                    @JsonFormat(shape = JsonFormat.Shape.STRING, pattern = "yyyy-MM-dd'T'HH:mm:ssZ")
                    var createdDate: Date? = null,
                    var errorCode: String? = null,
                    var errorMessage: String? = null,
                    @JsonFormat(shape = JsonFormat.Shape.STRING, pattern = "yyyy-MM-dd'T'HH:mm:ssZ")
                    var errorDate: Date? = null,
                    var correlationId: String? = null,
                    var fingerPrint: String? = null,
                    var errorDetail: Any? = null,
                    var errorDetailString: String? = null,
                    var filePath: String? = null,
                    var fileUrl: String? = null,
                    var clientIp: String? = null,
                    var clientInfo: ClientInfo? = null
) {
    init {
        this.logKey = UUID.randomUUID().toString()
        this.createdDate = Date()
    }
}
```

# Application

## CollectorApplication
```
@SpringBootApplication
class CollectorApplication : SpringBootServletInitializer() {
    override fun configure(application: SpringApplicationBuilder): SpringApplicationBuilder {
        return application.sources(CollectorApplication::class.java)
    }
    companion object {
        @JvmStatic
        fun main(args: Array<String>) {
            SpringApplication.run(CollectorApplication::class.java, *args)
        }
    }
}
```

처음부터 spring-boot kotlin으로 project를 생성할 경우 Applcation class는 다음과 같이 생성된다.
```
@SpringBootApplication
class DemoApplication

fun main(args: Array<String>) {
    runApplication<DemoApplication>(*args)
}
```
위와 같은 형식으로 바꾸면
```
@SpringBootApplication
class CollectorApplication : SpringBootServletInitializer() {
    override fun configure(application: SpringApplicationBuilder): SpringApplicationBuilder {
        return application.sources(CollectorApplication::class.java)
    }
}
fun main(args: Array<String>) {
    runApplication<CollectorApplication>(*args)
}
```

다음 Warning Log
```
D:\dev-logcollector\workspace\logcollector\collector\src\main\kotlin\com\rsupport\logcollector\collector\crashlog\CrashLogController.kt: (66, 36) Type mismatch: inferred type is String? but String was expected
```
![]({{site.url}}/assets/images/java2kotlin14-1.png)
  
![]({{site.url}}/assets/images/java2kotlin14-2.png)
  
->  
```
 @Value("\${logcollector.crashlog.url}")
    private lateinit var url: String
```

# TODO
- Kotlin에서 Logging하기  
https://discuss.kotlinlang.org/t/best-practices-for-loggers/226/2  
https://www.baeldung.com/kotlin-logging

- Request DTO는 data class?
- Request DTO를 일반 클래스로 만들 경우 private, getter, setter 사용?
- Builder pattern 사용하기
- Serializable을 구현해야 하나?
``` 
    companion object {
        private const val serialVersionUID = 5749490740387087329L
    }
```
- Controler/Service/Repository TestCase


