---
title: "HTML 태그로 캐시를 제어하지 마시오"
categories:
  - Web
tags:
  - web
  - http
toc: true
---

업무 중 특정 html 파일의 캐시를 무효화해야 했다.  

nginx 설정을 건드리지 않고 적용하고 싶어서, 눈에 익던 메타 태그를 통해 캐시를 제어해보고자 했다.  

``` html
<meta http-equiv="Pragma" content="no-cache">
<meta http-equiv="Cache-Control" content="no-cache">
...
```

결론부터 이야기해보자면, 해당 메타 태그는 **사용하면 안된다**.


## 이유
가장 큰 이유로 [HTML5 스펙](https://html.spec.whatwg.org/multipage/semantics.html#pragma-directives)에 없다는 점이다.  
스펙 pragma-directives 목차를 보면, http-equiv에 올 수 있는 enumerated attributes가 나열되어 있고 그 중에 `Pragma`, `Cache-Control` 은 **없다**.

HTML5 스펙만을 기준으로 구현하는 브라우저의 경우 해당 내용에 따라 캐시 컨트롤 태그가 동작하지 않는다.  

설령 브라우저 개발자의 인심에 따라 태그에 따른 동작이 구현되었더라도, 브라우저 별로 동작이 달라진다는 점에서 해당 태그를 사용하는 것은 안티 패턴이다.  


## 그래서 어떻게 하라고?
웹서버가 cache-control 관련 http 헤더를 설정하는 정석적인 방법으로 구현해라.