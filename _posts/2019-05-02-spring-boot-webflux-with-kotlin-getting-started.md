---
layout: post
title:  "Spring Boot Webflux with Kotlin Getting Started"
date:   2019-05-02 20:00:00 +0900
categories: Backend
tag: SpringBoot
---

* content
{:toc}

## Lambda-based Approach - Routes, Handler 작성

Routes.kt
```
@Configuration
class Routes(private val handler: Handler) {
    private val log = LoggerFactory.getLogger(javaClass)


    @Bean
    fun router() = router {
        GET("/check", handler::checkMail)
    }
}
```

Handler.kt
```
@Component
class Handler {
    fun checkMail(req: ServerRequest) = ServerResponse.ok().body("true".toMono().map{"result: $it"}, String::class.java)
}
```

  
![실행화면]({{site.url}}/assets/images/2019-05/webflux-kotlin.png)


## Test by WebTestClient 
```
@RunWith(SpringRunner::class)
@SpringBootTest
class RoutersTest {
    lateinit var client: WebTestClient
    @Autowired
    lateinit var handler: Handler
    @Before
    fun init() {
        this.client = WebTestClient.bindToRouterFunction(Routes(handler).router()).build()
    }


    @Test
    fun whenRequestToCheck_thenStatusShouldBeOk() {
        client.get()
                .uri("/check")
                .exchange()
                .expectStatus().isOk
    }
}
```

## Test by TestRestTemplate
```
@RunWith(SpringRunner::class)
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
class RoutersTest {
    @Autowired
    lateinit var restTemplate: TestRestTemplate
    @Test
    fun `check 정상적인 경우`() {
        val entity = restTemplate.getForEntity<String>("/check")
        assertThat(entity.statusCode).isEqualTo(HttpStatus.OK)
    }
}
```

## References
[Spring Webflux with Kotlin](https://www.baeldung.com/spring-webflux-kotlin)