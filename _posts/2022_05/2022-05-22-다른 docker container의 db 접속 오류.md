---
layout: post
title: 다른 docker container 의 db에 접속할 때 오류 날 경우 
date: 2022-05-22
categories: [Docker]
tags: []
author: Deokhun Kim
comment: false
update: 
---

### Docker 로 실행하면 DB 연결 오류가 발생

Docker 를 통해서 Web Service 와 DB 를 실행하면 Spring Data 에서 DB 연결 못하는 문제가 자주 발생했다.

오류 메세지가 단순히 접속실패로 발생하기 때문에 원인이 다양하고 찾기가 까다로워 해결했던 경험을 기록하려고 한다.

※ MySql 을 사용 했으므로, MySql 기준으로 작성

<br/>

#### 1. 접속 시 localhost 로 접속 한 경우
로컬에서 Docker 를 통해 MySql Container 를 띄워놓고 Web Service 개발을 할땐 
localhost:외부포트 로 문제 없이 접속이 됐다.

하지만 Web Service 로 image 화 시켜서 서버에서 container 로 띄우면 DB 연결에 실패를 했다.

원인은 Container 로 수행을 하면 Container 내에서 별도의 Private IP 를 할당 받는다.
그렇기 때문에 Container 를 통해 실행했을 때 localhost 가 서버 자신의 IP 를 가르키지 않기 때문에 발생하는 문제다.

Docker 의 기본적인 원리에 대해 알고 있다면 당연히 알 수 있는 사실이지만, 
개인 PC IDE 에서 개발할 때는 접속이 잘 되기 때문에 미처 생각지도 못하고 많이 헤맨 문제다.

해결 방법은 localhost 대신에 DB 의 Container Name 을 적어주면 Container 내부 IP 로 치환되어 접속이 된다.

`jdbc:mysql://localhost:3306/testDB` 

-> `jdbc:mysql://mysql-name:3306/testDB`


하지만, 이 방법은 Web Container 와 DB Container 가 동일한 **Docker Network** 상에 있어야 된다.
만약 Docker Compose 를 통해 2개의 서비스를 실행했다면 별도의 작업을 하지 않아도 동일한 Network 로 묶이기 때문에 정상적으로 동작한다.
하지만 2개의 서비스를 각각 따로 Run 을 했다면 같은 Docker Network 에 속해있지 않기 때문에 여전히 접속오류가 발생한다.
이 경우에는 아래 2번 작업이 필요하다.

<br/>

#### 2. 동일한 Docker Network 에 속해있지 않은 경우(Docker v1.9.0 이상)
Docker Compose 가 아니라 각각 별도로 Container 를 동작했다면 Network 로 묶어주지 않으면 서로 연결이 되지 않는다.

이전에는 --link 옵션을 통해 처리했지만, Docker v1.9.0 에서는 --network 옵션을 지원한다.

먼저 network group 을 생성한다.
> `$docker network create [network-name]`

그리고 각 container 를 시작 할 때 `--network [network-name]` 옵션을 추가 하면 된다.

docker compose 를 통해 실행하는 경우에는 아래 처럼 옵션을 주면 된다.
``` yaml
services:
  mysql-db:
    image: mysql:5.7
    container_name: mysql
    ports:
      - "3307:3306"
    restart: always
    environment:
      - MYSQL_DATABASE=db
      - MYSQL_ROOT_PASSWORD=admin
    volumes:
      - ~/data_dir:/var/lib/mysql
    networks:
      - kpp

networks:
  kpp:
    name: [network-name]
```




<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>


##### 참고 사이트
* https://github.com/confirm/docker-mysql-backup/issues/13
* http://pyrasis.com/book/DockerForTheReallyImpatient/Chapter06/02
