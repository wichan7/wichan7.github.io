---
title: "[TroubleShoot] Private key must be accompanied by certificate chain"
categories:
  - Tomcat
  - Springboot
tags:
  - Tomcat
  - Springboot
---

Tomcat에서 TLS를 설정하면서 발생한 메시지인데, 구글이나 스택 오버플로우를 찾아봐도 잘 안나와서 내가 쓴다.  

내 경우에는 `key-store-password`를 `key-password`로 잘못 썼다.

``` yaml
server:
  port: 443
  ssl:
    enabled: true
    key-store: /writely/cert/keystore.p12
    key-store-type: PKCS12
    key-store-password: blah.. # correct
    key-password: blah..       # incorrect
```
