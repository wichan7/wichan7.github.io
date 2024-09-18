---
title: "Spring Openfeign 잘 요청했는데 왜 401?"
categories:
  - Web
tags:
  - spring
  - openfeign
toc: true
---

## 개요

Netflix Openfeign 라이브러리를 이용해 elasticsearch와 http 통신하고자 했다.
elasticsearch에는 id/pw 기반 인증이 있어서 `http://{id}:{pw}@{es-endpoint}` 형식의 http basic auth로 요청했는데 401이 돌아오더라.

## 이유

OpenFeign이 `http://{id}:{pw}@{domain}`과 같이 url에 id/pw가 포함된 요청을 잘 처리하지 못한다.
기본적으로 http basic auth는, Authorization 헤더에 {id}:{pw}를 base64 인코딩한 값을 포함하도록 되어있다.

크롬, 사파리와 같은 현대의 웹 에이전트들은 `http://{id}:{pw}@{domain}` 와 같은 요청이 들어오면 base64로 인코딩하고 Authorization 헤더로 변환하는 일련의 과정을 수행해주는데, 아쉽게도 OpenFeign에서는 제공하지 않는 것 같아보인다.

## 해결

우선 `id:pw`를 [여기](https://www.base64decode.org/ko/)서 base64 인코딩했다.  
그 후, ElasticSearch OpenFeign Client가 항상 Authorization을 포함하도록 Config를 작성했다.

```java
// EsFeignConfig.java
package com.a.b.c.httpclient.elasticsearch;

import feign.RequestInterceptor;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;

public class EsFeignConfig {
    @Value("${internal-service.elasticsearch.authorization}")
    private String authorization;

    @Bean
    public RequestInterceptor requestInterceptor() {
        return requestTemplate -> {
            requestTemplate.header("Content-Type", "application/json");
            requestTemplate.header("Authorization", "Basic " + authorization);
        };
    }
}
```

```java
// EsClient.java
package com.a.b.c.httpclient.elasticsearch;

import org.springframework.cloud.openfeign.FeignClient;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;


@FeignClient(name="es-client", url="${internal-service.elasticsearch.url}", configuration = EsFeignConfig.class)
public interface EsClient {
    @PostMapping(value="/events_v1/_doc/{eventId}")
    void upsertEvent(@PathVariable("eventId") Long eventId, @RequestBody EsDto.EventRequest body);
}
```
