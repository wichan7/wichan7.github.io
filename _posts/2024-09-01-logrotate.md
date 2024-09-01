---
title: "logrotate로 nginx 로그 날짜 별로 돌리기 (feat. k8s, docker)"
categories:
  - Web
tags:
  - nginx
  - logrotate
  - k8s
  - docker
toc: true
---

## 개요

업무 중 nginx의 access log를 날짜 별로 관리해야 할 일이 생겼다.  
쉬운 일이겠거니 했지만 많은 삽질이 있었는데, 삽질 과정과 해결 방법을 소개한다.

바쁜 사람은 결론만 보시라.

## 환경

- Azure Managed Kubernetes
- nfs를 마운트한 pv
- /var/log/nginx를 마운트
- nginx: 1.27.1

## 1차 시도 (실패)

인터넷에 찾아보니 리눅스의 logrotate 패키지를 이용하는 예시가 정말 많았다.

골자는 다음과 같다.

1. logrotate는 단순히 파일명을 변경해주는 패키지이다.  
   스케쥴 기능이 없으므로 cron을 설치해 매일매일 logrotate가 실행되도록 설정한다.

2. cron이 logrotate를 실행하면 access.log 파일이 access.log-yyyymmdd 파일로 rename된다.

3. logrotate는 단순히 파일을 rename하므로, nginx가 rename된 파일에 계속 로그를 쓰고있는다. 우리는 새로운 access.log가 생기게 하고 싶으므로 `kill -USR1` 명령으로 nginx에게 파일을 reopen 하라는 신호를 보낸다.

```dockerfile
# dockerfile
FROM nginx:1.27.1

RUN apt-get install -y logrotate cron
COPY ./logrotate.conf /etc/logrotate.conf

RUN sh -c cron && nginx -g 'daemon off'
```

```sh
# logrotate.conf
/var/log/nginx/*.log {
	daily
	rotate 365
	dateext
	dateyesterday
	missingok
	compress
	delaycompress
	ifempty
	sharedscripts
	postrotate
      if [ -f /var/run/nginx.pid ]; then
        kill -USR1 `cat /var/run/nginx.pid`
      fi
	endscript
}
```

이렇게 구현하니 문제가 있었는데,

1. access.log-yyyymmdd 파일에 계속 로그를 기록하고 있었다.  
   -> nginx가 파일을 reopen하지 않는다... 이 때는 이유를 몰랐다.
2. 컨테이너 재시작 후 첫 날에는 logrotate가 돌지 않았다.  
   -> logrotate는 `/var/lib/logrotate/status` 경로에 last-rotate 시간을 기록하는데, 첫 날 (컨테이너 재시작 후)에는 파일이 없으므로 로테이트가 돌지 않는다.
3. 0시 0분에 돌게 하고싶었는데. 6시 25분에 실행됐다.  
   -> cron.daily의 기본 실행 시간이 6시 25분이더라...

## 2차 시도 (해결.. 인줄 알았지만 실패)

### 문제1 조치

원인을 파악할 수 없어 nginx가 파일을 reopen하지 않아도 회전될 수 있도록, logrotate의 copytruncate 옵션을 사용해보았다.  
기존은 쓰고 있던 파일의 이름을 변경하는 방식이다. copytruncate는 새로운 파일을 생성하고 새 파일에 로그 내용을 복사한다. 그 후 원본 파일에서 그 만큼의 이력을 지운다.

따라서 새 파일을 열지 않더라도 nginx는 같은 파일에 계속 쓰면 된다.

### 문제2 조치

logrotate -f 옵션을 사용해 강제로 회전되도록 한다.  
`/usr/sbin/logrotate -v -f /home/logrotate.conf`

### 문제3 조치

cron이 0 0 \* \* \* 에 실행되도록 설정한다.

### 아이디어

copytruncate를 사용하면 프로세스에게 시그널 보내는게 필요 없기 때문에, 굳이 같은 컨테이너에서 로테이트시킬 필요 없다! 쿠버네티스를 사용 중이니 cronjob으로 logrotate를 실행해 다른 nginx app도 rotate 시켜주자! 라는게 아이디어였다.

### dockerfile

logrotate가 cronjob에 의해 실행되게 하고싶었다.

```dockerfile
FROM alpine:3.14

USER root
RUN apk --update add --no-cache logrotate && \
    rm -f /etc/logrotate.d/*
ADD logrotate.conf /etc/logrotate.conf
RUN chmod 0400 /etc/logrotate.conf

CMD ["/usr/sbin/logrotate", "-v", "-f", "--state","/tmp/logrotate.status", "/etc/logrotate.conf"]
```

### configmap

```
apiVersion: v1
kind: ConfigMap
metadata:
  name: logrotate-config
  namespace: wichan
data:
  nginx-logrotate.conf: |
    /var/log/app/nginx/*/log/*.log {
        daily
        rotate 365
        dateext
        dateyesterday
        missingok
        compress
        delaycompress
        ifempty
        copytruncate
        endscript
    }
```

### cronjob

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: job-logrotate
  namespace: wichan
spec:
  schedule: "0 0 * * *"
  timeZone: Asia/Seoul
  concurrencyPolicy: Forbid
  jobTemplate:
    spec:
      template:
        spec:
          containers:
            - name: container
              image: my-logrotate:latest
              imagePullPolicy: Always
              volumeMounts:
                - name: logrotate-conf
                  mountPath: /etc/logrotate.d
                - name: nfs
                  mountPath: /var/log/app
                  subPath: App
                - name: timezone
                  mountPath: /etc/localtime
          imagePullSecrets:
            - name: ghcrsecret
          volumes:
            - name: logrotate-conf
              configMap:
                name: logrotate-config
            - name: nfs
              persistentVolumeClaim:
                claimName: pvc-wichan
            - name: timezone
              hostPath:
                path: /usr/share/zoneinfo/Asia/Seoul
          restartPolicy: Never
```

이걸로 /var/log/app/nginx/\*/log가 0시 0분마다 잘 회전했고, 성공한 줄로만 알았다.

### 새로 발생한 문제

어느 날 보니 로그에 ^@^@^@와 같은 null 문자가 잔뜩 들어있었다.

nginx가 로그를 쓰는 도중에 logrotate가 n번째 줄까지 자르게 되면, nginx가 n+1 줄에 다음 로그를 쌓는데 이 때 0 ~ n번째 줄이 null char로 대체되는 것이었다.

## 3차 시도 (해결)

근본적인 해결을 위해 nginx 프로세스에게 reopen signal을 보내는게 필수임을 알았다.

다만 logrotate 컨테이너로 돌리는 경우, nginx process에 접근할 수 없기 때문에 cronjob을 이용하는 방식(2차)를 폐기했다.

1차 시도로 돌아와 nginx 프로세스가 파일을 reopen하지 못하는 이유를 확인했더니..  
-> 알고 보니 nfs 마운트에 의해 nginx 프로세스에게 파일을 열 수 있는 권한이 없었다.

nfs 마운트된 디렉토리의 소유를 nginx로 변경하려 했는데, nfs는 소유변경을 할 수 없단다.  
그래서 권한을 777로 변경했는데 오류가 발생하더라.

> because parent directory has insecure permissions (It's world writable or writable by group which is not "root") Set "su" directive in config file to tell logrotate which user/group should be used for rotation.

로테이트 디렉토리에 world-writable한 권한이 있으면 보안 상 로테이트를 실패시킨다.

수많은 삽질 끝에.. nginx 프로세스가 root 권한으로 실행되도록 설정했다.

```nginx
user  root; # 여기
worker_processes  1;

http {
  ...
}
```

## 최종 해결 방법

```sh
# logrotate.conf
/var/log/nginx/*.log {
	daily
	rotate 365
	dateext
	dateyesterday
	missingok
	compress
	delaycompress
	ifempty
	sharedscripts
	postrotate
    if [ -f /var/run/nginx.pid ]; then
      kill -USR1 `cat /var/run/nginx.pid`
    fi
	endscript
}
```

```nginx
# nginx.conf
user  root; # 여기
worker_processes  1;

error_log  /var/log/nginx/error.log notice;
pid        /var/run/nginx.pid;

events {
    worker_connections  1024;
}

http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile        on;

    keepalive_timeout  65;

    include /etc/nginx/conf.d/*.conf;
}
```

```dockerfile
# Dockerfile
FROM nginx:latest

# 타임존 설정
RUN ln -snf /usr/share/zoneinfo/Asia/Seoul /etc/localtime

# 패키지 설치
RUN apt-get update
RUN apt-get -y install logrotate cron

# 빌드 파일 복사
COPY ./build /usr/share/nginx/html

# nginx 설정 복사
RUN rm /etc/nginx/nginx.conf
COPY ./nginx.conf /home/tmp/nginx.conf
RUN tr -d '\r' < /home/tmp/nginx.conf > /etc/nginx/nginx.conf

RUN rm /etc/nginx/conf.d/default.conf
COPY ./default.conf /home/tmp/default.conf
RUN tr -d '\r' < /home/tmp/default.conf > /etc/nginx/conf.d/default.conf

# nginx-logrotate 설정 복사
COPY ./logrotate.conf /home/tmp/logrotate.conf
RUN tr -d '\r' < /home/tmp/logrotate.conf > /home/logrotate.conf

# logrotate cron 파일 생성
RUN echo "0 0 * * * /usr/sbin/logrotate -v -f /home/logrotate.conf" > /etc/cron.d/logrotate-nginx
RUN chmod 0644 /etc/cron.d/logrotate-nginx

# crontab에 cron 등록
RUN crontab /etc/cron.d/logrotate-nginx

EXPOSE 1020

CMD ["sh", "-c", "cron && nginx -g 'daemon off;'"]
```

핵심은 다음과 같다.

- nginx가 /var/log/nginx에 파일을 쓸 수 있도록 root 권한을 주는 것
- postrotate 스크립트를 통해 nginx에게 reopen signal을 보내는 것
- logrotate status 파일이 없어도 로테이트 되게 하는 것 (또는 status를 마운트해서 pirsist하게 관리)
- cron을 0시 0분에 실행시키는 것

## 사족

LogStash가 일자별 nginx 로그를 읽을 수 있도록 하는 요구사항이었다.  
별거 아닌것 처럼 보이는 일이 이렇게까지 속썩일 줄 몰랐다.

이름 모르는 누군가에게 이 글이 도움되었으면 좋겠다. 그대여 삽질하지 마시오....
