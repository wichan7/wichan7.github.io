---
title: "[TroubleShoot] Azure AppGateway 환경에서 SSE Timeout 현상"
categories:
  - Azure
  - Infra
tags:
  - Azure
  - Infra
---

## 개요
클라이언트에게 실시간 서버 변경사항을 내려주어야 하는 요구사항이 생겼다.  
이를 Http Streaming(SSE) 프로토콜로 구현했고, 로컬에서 잘 되길래 클라우드 환경에 올려보았다.  

그런데 클라우드 환경에 올리고 보니 SSE 요청만 응답이 제 때 오지 않았다. (30초 후에 응답과 함께 연결이 끊어졌다.)  

## 해결
문제는 Ingress를 구현한 Azure Application Gateway에 있었다.  
우리의 서비스는 애저 관리형 쿠버네티스를 사용중으로 AppGateway를 통해 트래픽을 리버스 프록시 하고있다.  

AppGateway가 내부적으로 nginx를 사용하는데 nginx의 Response Buffering 기능이 활성화되어 있다고 한다.  
궁금한 부분은 왜 하필 SSE 응답만 버퍼링되어 응답이 도착하지 않은 것인가였는데, 좀 찾아봐도 버퍼링되는 사이즈와 조건을 찾을 수는 없었다. ㅠㅠ  
(혹자는 SSE 응답이 Transfer-Encoding: chunked로, 정확한 사이즈를 몰라 nginx의 기준이되는 버퍼링 사이즈가 찰 때까지 버퍼링된다고 하는데 정확한 내용인지는 잘 모르겠다. )  


그래도 증상에 대한 해결은 했다.  
스택 오버플로우 답변에 따라, SSE Emitter Response 내려줄 때 http 헤더에 `x-accel-buffering: no`를 추가해 주었다.  

[Stack Overflow Ref](https://stackoverflow.com/questions/76161274/delay-in-getting-sse-events-when-configured-via-azure-application-gateway)  