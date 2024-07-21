---
title: "Spring Batch, 메타테이블 DB 분리와 datasource 설정"
categories:
  - Spring Framework
tags:
  - spring
  - batch
  - mybatis
toc: true
---

## 목표

스프링 배치를 사용하면 배치가 언제, 어디까지 이루어졌는지 등의 메타 정보를 관리하는 테이블이 생겨나게 된다.  
이 메타 테이블을 서비스와 다른 DB에 생성하고, 배치 서버는 서비스에도 접근 가능해야 하도록 설정하고 싶었다.

## 요약

1. dataSource 설정을 mybatis dataSource, batch dataSource로 분리
2. mybatis가 mybatis용 dataSource를 사용하도록 설정

## 내용

### 1. dataSource 설정

application.yml에서는 여러개의 datasource 설정을 지원하지 않는다.  
따라서 @Configuration을 통해 직접 datasource 빈을 주입한다.

```yaml
# application.yml
spring:
  application:
    name: my-batch
  batch:
    jdbc:
      initialize-schema: never
    job:
      name: ${job.name:NONE}
  datasource:
    batch:
      jdbc-url: jdbc:mysql://{batch-database-uri}/batch
      driver-class-name: com.mysql.cj.jdbc.Driver
      username: batch
      password: batch123
    service:
      jdbc-url: jdbc:mysql://{service-database-uri}/service
      driver-class-name: com.mysql.cj.jdbc.Driver
      username: service
      password: service123
```

```java
// com.dev.wichan.config.DataSourceConfig
@Configuration
public class DataSourceCOnfig {

  @Bean
  @Primary
  @ConfigurationProperties(prefix="spring.datasource.batch")
  public HikariConfig batchHikariConfig() {
      return new HikariConfig();
  }

  // 스프링 기본 데이터소스로 사용할 batchDatasource 생성
  @Bean
  @Primary
  @Qualifier("batchDataSource")
  public DataSource batchDataSource() {
      return new HikariDataSource(batchHikariConfig());
  }

  @Bean
  @ConfigurationProperties(prefix="spring.datasource.service")
  public HikariConfig serviceHikariConfig() {
      return new HikariConfig();
  }

  // Mybatis에서 사용할 serviceDataSource 빈 생성
  @Bean
  @Qualifier("serviceDataSource")
  public DataSource serviceDataSource() {
      return new HikariDataSource(serviceHikariConfig());
  }
}
```

### 2. Mybatis Datasource 변경

Qualifier 어노테이션으로 마이바티스에서 사용할 데이터소스 지정

```java
// com.dev.wichan.config.MyBatisConfig
@Configuration
public class MybatisConfig {

  @Primary
  @Bean
  public SqlSessionFactory sqlSessionFactory (
    @Qualifier("serviceDataSource") DataSource dataSource,
    ApplicationContext applicationContext
  ) throws Exception {
    SqlSessionFactoryBean sessionFactoryBean = new SqlSessionFactoryBean();
    sessionFactoryBean.setDataSource(dataSource);
    sessionFactoryBean.setMapperLocations(applicationContext.getResources("classpath:mapper/**/*.xml"));

    org.apache.ibatis.session.Configuration mybatisConfig = new org.apache.ibatis.session.Configuration();
    mybatisConfig.setMapUnderscoreToCamelCase(true);
    mybatisConfig.setDefaultExecutorType(ExecutorType.BATCH);

    sessionFactoryBean.setConfiguration(mybatisConfig);
    return sessionFactoryBean.getObject();
  }

}

```
