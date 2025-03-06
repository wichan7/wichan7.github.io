---
title: "마인크래프트 서버 클라우드 서빙하기"
categories:
  - Cloud
tags:
  - Cloud
  - Minecraft
toc: true
---

## 개요

나와 우리는 99년생으로 26살이지만 정신 연령은 아직 고등학교 그 시절 언저리에 머물러 있다.  
우리는 마인크래프트를 함꼐 플레이하기로 했는데, 고등학생 때와는 달리 각자의 스케쥴이 생겨버린 우리는 늘 함께하기에 시간대가 맞지 않았고, 원할 때 언제나 접속할 수 있는 마인크래프트 서버가 필요했다.

오늘은 클라우드에서 마인크래프트 서버를 서빙하는 것에 대한 이야기이다.

## 1차 시도

### amazon ec2의 free-tier 한도로 가능할까?

마인크래프트 서버를 24시간 제공하고 싶어서 많이 사용해본 AWS에 올리고자 했고, 돈을 들이고싶지는 않았기에 free-tier 한도에 한해 끝내고 싶었다.

마인크래프트 자바 에디션의 서버는 최 최 최소 1GB 이상의 가용 RAM을 필요로 한다.  
aws ec2에서는 t2.micro 컴퓨팅을 무료로 사용할 수 있는데, t2 micro는 RAM을 1GB만 제공하기에 OS를 돌리고 나면 가용 램이 얼마 남지 않는다. ([https://aws.amazon.com/ko/ec2/instance-types/t2/](https://aws.amazon.com/ko/ec2/instance-types/t2/))

### 그래서...

내겐 더 나은 사양이 필요했고 amazon lightsail이 이를 해결해줄 수 있을 것 같았다.  
lightsail은 ec2보다 더 쉽게 사용가능한 인터페이스를 제공하는 컴퓨팅 서비스라고 보면 되는데, 아마존이 밀고 싶은 서비스인지 무료로 꽤 괜찮은 성능의 컴퓨팅을 제공한다.

| cpu     | memory      | storage           | transfer      |
| ------- | ----------- | ----------------- | ------------- |
| 2 vCPUs | 2 GB Memory | 60 GB SSD Storage | 3 TB Transfer |

lightsail로 돌려보자.

### lightsail 설정

친구들을 대상으로한 서비스라고는 하지만 쾌적하게는 서비스하고 싶었다.  
그래서 과거 해보지 않았던 리눅스 환경에서 돌려보고자 했고, ubuntu os로 채택했다.

마인크래프트는 디폴트 25565포트이므로 25565 inbound를 allow한다.

### ubuntu 설정

요새 도커에 익숙해져서 도커부터 설치했다.

```sh
$ sudo apt-get update

$ sudo apt-get install apt-transport-https ca-certificates curl gnupg-agent software-properties-common

$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

$ sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"

$ sudo apt-get update

$ sudo apt-get install docker-ce docker-ce-cli containerd.io

$ sudo systemctl status docker
```

### 마인크래프트 서버 이미지 풀 받기

누가 잘 만들어놓은 마인크래프트 서버 이미지가 있더라. 스테이블 버전으로 받았다.

```sh
docker pull itzg/minecraft-server:stable
```

### docker-compose 작성

latest나 stable 버전으로 돌리면 신규 서버 버전이 나왔을 때 알아서 업그레이드해준다.

```yaml
services:
  mc:
    image: itzg/minecraft-server:stable
    tty: true
    stdin_open: true
    ports:
      - "25565:25565"
    environment:
      EULA: "TRUE"
    volumes:
      # attach the relative directory 'data' to the container's /data path
      - ./data:/data
```

### 확인

```sh
$ docker-compose up -d
$ docker-compose ps
$ docker ps
```

마운트된 ./data/logs에 들어가서 java -Xms1G -Xmx1G 로 1GB 램을 사용한 서버가 잘 구동됨을 확인했다.

### 그래서 잘 됐는가?

둘이 접속해서 테스트하니 처음에는 쾌적하고 잘 돌아갔는데, 겉날개 아이템을 사용했을 때 문제가 발생했다.  
이동속도가 빠른 겉날개를 사용하니 청크 로딩이 순식간에 많이 요청되어 램이 부족해지더라.

더 나은 방법을 생각해보자.  

## 2차 시도

램이 부족하니 docker를 사용해서 생기는 overhead를 줄이고, linux swap memory를 활용해 부족한 램을 채우고자 했다.

### openjdk 설치

```sh
$ apt install openjdk-22-jre-headless
$ java --version
```

### minecraft-paper 다운로드

[https://papermc.io/downloads](https://papermc.io/downloads)

10년전에 쓰던 minecraft-bukkit은 지원이 중단됐다고 하더라.  
성능도 새로나온 paper이 괜찮다고 해서 paper 서버 구동기를 다운로드했다.

### 스왑메모리 설정

스토리지는 널널하니 20GB flex를 해보자.

```sh
$ sudo fallocate -l 20G /mnt/swapfile

$ sudo chmod 600 /mnt/swapfile

$ sudo mkswap /mnt/swapfile

$ sudo swapon /mnt/swapfile

$ sudo swapon --show
```

### 메모리 플렉스하기

```sh
$ java -Xms4G -Xmx16G -jar paper-1.21-106.jar
```

### 그래서 잘 됐는가?

우선 1차 시도보다도 체감 성능이 안좋아졌는데, 이유는 swap-memory 사용으로 인한 오버헤드로 생각한다.

ssd가 빠르다고는 하지만 메모리와 보조기억 간 액세스 타임 차이는 분명했고, 스왑 메모리 스위칭 작업을 위한 cpu 작업이 발생할 수 밖에 없는데, 이에 따라 1차 시도 때보다 더 많은 cpu를 사용하는 것을 볼 수 있었다.

![cpu-usage-statistics](/assets/images/cloud/minecraft/cpu-usage.png)

플레이어가 접속하지 않았을 때 cpu-usage가 12% 수준에서 머물렀고, 한 플레이어가 접속해서 활동할 때 cpu usage가 50% 수준까지 치솟았는데, 물론 여러명의 접속을 테스트해보지는 않았지만 하루에 하나 이상의 플레이어가 6시간 이상 플레이한다고 가정하면 cpu-burst credit 사용률이 전혀 지속가능하지 않았다.

성능도 더 낮고 지속가능하지 않은 방법이기에 2차 시도도 실패로 끝났다.

## 3차 시도

azure에서 한달 간 $200 credit을 제공한다고 한다.  
잘 됐다. 돈이 최고다.  