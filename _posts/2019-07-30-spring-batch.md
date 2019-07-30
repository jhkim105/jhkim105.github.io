---
layout: post
title:  "Spring Batch를 활용하여 Data Migrator 구현"
date:   2019-07-30 19:00:00 +0900
categories: Backend
tag: SpringBoot
---

* content
{:toc}



# Overview
솔루션 납품 요구사항 중 고객사 사용자 DB를 연동하는 요구사항을 처리하기 위해, Spring Batch를 활용해서 구현해보았다.  
고객사 DB는 Oracle이고, 자사 솔루션은 MariaDB를 사용하므로 Mulitiple DataSource를 설정해야 한다.

# Multiple DataSoruce
## DatabaseConfig.java
```
@Configuration
public class DatabaseConfig {


  @Bean
  @Primary
  @ConfigurationProperties("spring.datasource")
  public DataSourceProperties dataSourceProperties() {
    return new DataSourceProperties();
  }


  @Bean
  @Primary
  public DataSource dataSource() {
    return dataSourceProperties().initializeDataSourceBuilder().build();
  }


  @Bean
  @ConfigurationProperties("legacy.datasource")
  public DataSourceProperties legacyDataSourceProperties() {
    return new DataSourceProperties();
  }


  @Bean
  public DataSource legacyDataSource() {
    return legacyDataSourceProperties().initializeDataSourceBuilder().build();
  }
}
```

# Spring Batch 
## Maven dependency  
```
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-batch</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework.batch</groupId>
      <artifactId>spring-batch-test</artifactId>
      <scope>test</scope>
    </dependency>
```

## BatchConfig.java
```
@Configuration
@EnableBatchProcessing
public class BatchConfig {

  public static final int CHUNK_SIZE = 1000;

  public static final int THROTTLE_LIMIT = 10;
}
```

## UserJobConfig.java
```
@Configuration
public class UserJobConfig {

  @Autowired
  private JobBuilderFactory jobs;

  @Autowired
  private StepBuilderFactory steps;

  @Autowired
  MigrationService migrationService;

  @Resource(name = "legacyDataSource")
  DataSource legacyDataSource;

  @Bean
  public Job userMigrationJob() {
    return this.jobs.get("userMigrationJob").start(userMigrationStep()).build();
  }

  private Step userMigrationStep() {
    return this.steps.get("userMigrationStep")
        .chunk(CHUNK_SIZE)
        .reader(userItemReader())
        .writer(userItemWriter())
        .throttleLimit(THROTTLE_LIMIT)
        .build();
  }

  @Bean
  JdbcCursorItemReader<LinkedUser> userItemReader() {
    JdbcCursorItemReader itemReader = new JdbcCursorItemReader();
    itemReader.setDataSource(legacyDataSource);
    itemReader.setSql("SELECT * FROM TB_USER");
    itemReader.setRowMapper(new UserRowMapper());
    return itemReader;
  }

  class UserRowMapper implements RowMapper {
    @Override
    public LinkedUser mapRow(ResultSet rs, int i) throws SQLException {
      LinkedUser user = LinkedUser.builder()
          .id(rs.getString("USERNAME"))
          .build();

      return user;
    }
  }

  @Bean
  public ItemWriter<Object> userItemWriter() {
    ItemWriterAdapter itemWriterAdapter = new ItemWriterAdapter();
    itemWriterAdapter.setTargetObject(migrationService);
    itemWriterAdapter.setTargetMethod("migrate");
    return itemWriterAdapter;
  }
}
```

## Run
* Application을 실행하면 batch 관련 테이블이 생성되고, 작업 결과가 기록됨  
![spring-batch-01]({{site.url}}/assets/images/2019-07/spring-batch-01.png)

* 다시 어플리케이션을 실행하면 batch가 실행되지 않는다. 계속 실행되게 하고 싶으면 param을 다르게 해서 실행해야 한다.  
![spring-batch-02]({{site.url}}/assets/images/2019-07/spring-batch-02.png)  
![spring-batch-03]({{site.url}}/assets/images/2019-07/spring-batch-03.png)  

## JobLauncher로 실행하기
* BatchService.java
```
@Service
@RequiredArgsConstructor
public class BatchService {
  private final JobLauncher jobLauncher;


  private final Job userMigrationJob;


  public void runUserMigrationJob() {
    JobParameters jobParameters = new JobParametersBuilder().addLong("time", System.currentTimeMillis()).toJobParameters();
    try {
      jobLauncher.run(userMigrationJob, jobParameters);
    } catch (Exception e) {
      throw new RuntimeException(e);
    }
  }
}
```
![spring-batch-04]({{site.url}}/assets/images/2019-07/spring-batch-04.png)  
![spring-batch-05]({{site.url}}/assets/images/2019-07/spring-batch-05.png)  

* App 실행시 batch가 실행되는 것을 막으려면 다음 설정 추가
```
spring.batch.job.enabled=false
```

# Scheduling
@EnableScheduling 

@Scheduled(cron = "${batch.user-migration.cron}")

batch.user-migration.cron=/10 * * * * *

pool개수나 이름 등을 지정하고 싶다면
```
@Configuration
@EnableScheduling
public class SchedulingConfig {


  private static final int SCHEDULER_POOL_SIZE = 5;


  @Bean
  public ThreadPoolTaskScheduler taskScheduler() {
    ThreadPoolTaskScheduler taskScheduler = new ThreadPoolTaskScheduler();
    taskScheduler.setPoolSize(SCHEDULER_POOL_SIZE);
    taskScheduler.setThreadNamePrefix("scheduling-");
    return taskScheduler;
  }


}
```

Spring Boot 2.1에서는 property로 가능  
![spring-batch-06]({{site.url}}/assets/images/2019-07/spring-batch-06.png)  