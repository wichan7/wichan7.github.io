---
title: "MSA 설계와 구현 - Prologue"
categories: 
  - MSA
tags:
  - Architecture
  - AWS
  - k8s
  - docker
  - nodejs
  - MSA
---

## Monolithic Architecture
서비스를 위해 하나(mono)의 애플리케이션이 모든 작업을 처리하도록 개발하는 일반적인 아키텍쳐이다.    
구조적으로 복잡하지 않아 작은 서비스를 개발할 때 강력하다.

## Micro Service Architecture
서비스를 위해 여러개의 작은(Micro) 애플리케이션을 개발하는 아키텍쳐이다.   
구조적으로 복잡하지만 애플리케이션을 모듈화하기 때문에 결합도가 낮고, 서비스 장애에 강력하다는 등..   
컴퓨터 공학적으로 모듈이 가진 당연한 이점들을 공유한다.

## 내가 하고자 하는 것
나에게는 아직 생소한 'MSA'를 직접 설계, 구현해보고자 한다.   
내게 DevOps에 대한 관심이 많은 만큼, 프로젝트를 진행하며 많은 새로운 시도를 해보고 싶다.
