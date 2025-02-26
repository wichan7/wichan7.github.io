---
title: "쿠버네티스 redis-sentinel 배포 & 이관 & Spring 연동"
categories:
  - Infra
tags:
  - Kubernetes
  - Redis
  - Spring
toc: true
---

우리는 Azure의 관리형 쿠버네티스에서 서비스를 제공하고 있다.  
메모리를 옴팡지게 먹는 JVM 기반의 애플리케이션을 위주로 돌리다 보니 메모리가 부족해 파드가 eviction 나는 현상이 생겼다.  

바쁘니까 그냥 scale-out으로 무작정 노드 수를 늘렸었는데, 어느날 열심히 일하고 있는 나와는 달리 세상 편히 놀고있는 CPU가 맘에 안들어 scale-up을 계획했다.  

## 그게 redis-sentinel이랑 무슨 상관인데?
Azure Managed Kubernetes는 Scale-up을 지원하지 않는다.  
AKS에서 Scale-up을 달성하는 방법은 하나 뿐인데, 그것은 새로운 스펙의 노드를 추가하고 파드를 이관한 다음 기존 노드를 종료하는 것이다.  

그러려고 보니 StandAlone으로 떠있는 redis 파드를 발견했고.. 사용자들의 세션을 마구 담고있는 우리 레디스를 그냥 종료시킬 수 없었다.  

## redis를 HA 구성하자
redis HA 구성을 통해 추후 노드에 일어날 수 있는 천재지변과 같은 무엇인가에 대비하자.  

레디스 HA를 달성하는 방법은 크게 sentinel과 cluster로 두가지 방식이 있겠다.  
그 차이에 대해서는 글의 주제와 맞지 않으므로 설명은 생략하고, 결과적으로 나는 sentinel을 선택했다.  

sentinel과 cluster 모두 helm을 통한 손쉬운 설치가 가능한데, sentinel에 대해 소개하도록 하겠다.  

## redis-sentinel 배포 과정
우리 서비스는 아직 규모가 크지 않기에 3개의 노드(최소 사양)로 설정하겠다.  

### 다운로드
``` shell
# bitnami 레포지토리 추가
$ helm repo add bitnami https://charts.bitnami.com/bitnami
    
# 레포지토리 업데이트
$ helm repo update

# 로컬에 redis chart 다운로드
$ helm fetch bitnami/redis
```

### 설정
다음으로는 내 입맛에 맞게 설정을 수정한다.  
``` yaml
...
auth:
  # 비밀번호 사용 해제
  enabled: false
  # 비밀번호 사용 해제 (센티널)
  sentinel: false

...
sentinel:
  enabled: true
```

### 배포
`helm install redis ./redis -n my-namespace`

## 관찰
이렇게 배포하고 나면
* pod/redis-node-0
* pod/redis-node-1
* pod/redis-node-2
* svc/redis (6379, 26379)
* svc/redis-headless (6379, 26379)

요정도의 리소스들이 생겨난다.  

redis-node 파드에는 레디스 컨테이너와 센티널 컨테이너가 함께 뜨고, 각각의 센티널들이 감시하다가 조건에 따라 마스터 노드를 선출한다.  
테스트하고 싶으면 파드를 삭제해보면 된다.  


## 기존 레디스의 데이터를 신규 레디스로 이관
찾아보니 많이 사용하는건 dump.rdb 파일이나 aof 설정을 활용하더라.  
결론부터 이야기하자면 [redis dump go](https://github.com/yannh/redis-dump-go) 오픈소스를 이용했다.  

회사 PC로 윈도우를 사용하고 있어 윈도우 기준으로 설명하자면
### 데이터 덤프
``` powershell
# 구 레디스 포트포워딩
kubectl port-forward --address 0.0.0.0 service/redis-svc 6379:6379 -n my-namespace

# 구 레디스에 접속해 로컬에 덤프파일 생성
docker run --rm -v ${PWD}:/data ghcr.io/yannh/redis-dump-go:latest -host 5.0.55.70 -port 6379 -output resp > dump.resp
```

### 로드
``` powershell
# 신규 레디스 포트포워딩 (* 마스터 노드에 접속해야 write 가능)
kubectl port-forward pod/redis-node-0 6379:6379 --address 0.0.0.0 -n my-namespace

# 로드 
Get-Content ./dump.resp | redis-cli -h 5.0.55.70 -p 6379 --pipe
```

## 스프링 연동
redis-sentinel을 사용하는 경우, 어던 노드가 마스터 노드인지 알아야 애플리케이션에서 write 가능한 노드에 연결할 수 있다.  

26379 포트를 통해 레디스 파드에 접근하는 경우 센티널로 연결되는데, `redis-cli sentinel master {{master-name}}`을 입력하면 현재 마스터의 IP를 알 수 있다.  

spring-data-redis의 경우 마스터 네임과 노드의 주소만 알려주면 위의 과정을 알아서 해준다.  
``` yaml
  data:
    redis:
      sentinel:
        master: mymaster
        nodes: redis.my-namespace.svc.cluster.local:26379
```

## 테스트
redis를 하나씩 죽여가면서 서비스가 정상 동작하는지, redis와 application의 로그를 보면서 이상은 없는지 보면 되겠다.