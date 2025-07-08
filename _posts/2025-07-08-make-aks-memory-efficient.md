---
title: "Azure Kubernetes 노드 메모리 효율적으로 관리하기"
categories:
  - Infra
tags:
  - infra
  - azure
  - k8s
---

## 문제 상황

- 개발기에서 메모리로 인한 간헐적인 eviction 발생
- 운영기에 선제적인 메모리 증설 작업 필요

```bash
*** 운영기
C:\Users\wichan>kubectl top node           ( workingset / allocatable mem ) 
NAME                                CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%
aks-agentpool-xxxxxxxx-vmss000000   182m         9%     5582Mi          104%
aks-agentpool-xxxxxxxx-vmss000001   154m         8%     4850Mi          90%
aks-agentpool-xxxxxxxx-vmss000002   122m         6%     5422Mi          101%
aks-agentpool-xxxxxxxx-vmss000003   123m         6%     5502Mi          102%
aks-agentpool-xxxxxxxx-vmss00000a   114m         6%     4467Mi          83%
aks-agentpool-xxxxxxxx-vmss00000b   129m         6%     4708Mi          88%

*** 운영기
C:\Users\wichan>kubectl top node --show-capacity=true     (workingset / mem capacity )
NAME                                CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%
aks-agentpool-xxxxxxxx-vmss000000   183m         9%     5594Mi          70%
aks-agentpool-xxxxxxxx-vmss000001   157m         7%     4844Mi          61%
aks-agentpool-xxxxxxxx-vmss000002   118m         5%     5413Mi          68%
aks-agentpool-xxxxxxxx-vmss000003   123m         6%     5492Mi          69%
aks-agentpool-xxxxxxxx-vmss00000a   108m         5%     4466Mi          56%
aks-agentpool-xxxxxxxx-vmss00000b   128m         6%     4698Mi          59%
```

## VM 스펙
```
# Azure VM spec
name: Standard_D2as_v4
CPU: 2vCPU
MEM: **8192MiB**
...

# on Kubernetes
MEM/Allocatable: **5346MiB**
```

## 의문점
왜 노드당 8192MiB가 아닌 5346MiB만 할당 가능한가? (8192 - 5346 = 2846은 어디로 갔는가..)

### Allocatable Memory 의 기준

 `[Allocatable] = [Node Capacity] - [kube-reserved] - [system-reserved] - [Hard-Eviction-Thresholds]`

### Allocatable Memory 계산

참조: https://learn.microsoft.com/ko-kr/azure/aks/node-resource-reservations

- 현재 사용 중인 클러스터 기준 (AKS v1.28, capacity 8GiB)

`kube reserved`: 4 * 0.25 + 4 * 0.2 = 1.8GiB = **1843.2MiB**

`hard-eviction-thresholds`: **750MiB**


## 해결 방안

### 1. 예약 메모리 감축

[내용](https://learn.microsoft.com/ko-kr/azure/aks/node-resource-reservations)에 따라

- AKS v1.29 이상부터 **max-pods-per-node**에 따라 예약된 메모리 효율화 가능
    - max-pods-per-node 50으로 설정 시, 기존 110에 비해 노드 당 950MiB 추가 확보 가능
- AKS v1.29 이상부터 **eviction-threshold**가 100MiB로 감소 (노드 당 650MiB 추가 확보 가능)

### 2. 물리 메모리 증대 (scale-up)

유휴 상태의 CPU 리소스가 많음 (9% 사용 중)
메모리 집약적 VM 선택으로 노드 수 감축 및 비용 효율화

- 참고사항
    - max-pods-per-node 작업 시 nodepool 신규 생성 필요
    - VM 변경 작업 시 nodepool 신규 생성 필요
        
        [Vertical scaling of azure kubernetes cluster](https://stackoverflow.com/questions/67491883/vertical-scaling-of-azure-kubernetes-cluster)
        

## 설정

### max-pods-per-node

---

운영 중인 클러스터 내의 파드 수 및 향후 생길 애플리케이션들을 고려하여 설정해야 함.

- 클러스터 운영을 위한 기본 생성되는 파드(kube-system ns)가 존재함. ( 10 pods / node )
파드 당 100MiB 이내
- Spring Application
파드 당 최소 300MiB의 메모리를 요구
- Nginx ( *2 )
파드 당 100MiB 이내의 메모리를 요구
- Elasticsearch ( *3 )
파드 당 2GiB 메모리를 요구

```
(단위: MiB)

추가 가능한 spring pod 수: (14841 - 100 * 10 - 100 * 2 - 2000 * 3) / 300
= 25.47

kube-system, nginx, spring 등을 포함한 총 pod 수: 10 + 2 + 3 + 25.47
= 40.47
```

추후 spring pod 만 추가하는 경우, max-pods : 40 설정으로 커버가 가능.
⇒ spring 이외에 메모리를 덜 사용하는 애플리케이션이 추가되는 경 고려하여 50으로 설정
