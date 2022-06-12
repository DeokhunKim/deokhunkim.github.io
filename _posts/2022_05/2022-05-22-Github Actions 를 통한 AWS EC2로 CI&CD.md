---
layout: post
title: Github Actions 를 통한 AWS EC2 로 CI/CD
date: 2022-05-22
categories: [AWS]
tags: [Git, Github, AWS, Docker]
author: Deokhun Kim
comment: false
update: 
---

### CI / CD
처음엔 Jenkins 를 이용하려고 했으나 Jenkins 를 구동 할 서버가 여의치 않아 Github actions 를 이용하기로 했다.
참 아쉬운데 다음에 꼭 Jenkins 로 구성해 보고 싶다.

목표는 이렇다
1. Github 에 push 가 되면
2. 자동으로 Gradle Build 를 하고
3. Docker Build 로 Image 를 만들어서
4. Docker Hub 에 업로드 하고
5. AWS EC2 서버에서 Docker Hub 를 통해 Image Pull 받아
6. 기존에 서버에서 실행 중인 서비스를 종료 후 새로운 서비스를 실행한다
7. 이 모든 작업이 Github 에 소스코드가 push 되면 자동으로 수행된다.


<img src="/assets/postimg/2022_05/CICD.png">

<br/>

EC2 셋팅이나 소스코드 내용 등은 생략하고 Github Actions 의 script 의 workflow 를 따라가면서 작성하려고 한다.


### 전체 workflow
``` yml
# This workflow uses actions that are not certified by GitHub.
# They are provided by a third-party and are governed by
# separate terms of service, privacy policy, and support
# documentation.
# This workflow will build a Java project with Gradle and cache/restore any dependencies to improve the workflow execution time
# For more information see: https://help.github.com/actions/language-and-framework-guides/building-and-testing-java-with-gradle

name: Java CI/CD with Gradle

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

permissions:
  contents: read

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v3
    
    - name: Set up JDK 11
      uses: actions/setup-java@v3
      with:
        java-version: '11'
        distribution: 'temurin'
        
    - name: Grant execute permission for gradlew
      run: chmod +x gradlew
      
    - name: Build with Gradle
      run: ./gradlew build -x test
           
    - name: Docker build and push to docker hub
      run: |
        docker login -u ${ { secrets.DOCKER_ID } } -p ${ { secrets.DOCKER_PASSWORD } } 
        docker build -t glossaryapi . 
        docker tag glossaryapi ${ { secrets.DOCKER_ID } }/glossaryapi:${GITHUB_SHA::7} 
        docker push ${ { secrets.DOCKER_ID } }/glossaryapi:${GITHUB_SHA::7}
    
    - name: Deploy to AWS EC2
      uses: appleboy/ssh-action@master
      with:
        host: 서버주소
        username: ${ { secrets.AWS_USERNAME } }
        key: ${ { secrets.AWS_PEM } }
        envs: GITHUB-SHA
        script: |
          docker pull ${ { secrets.DOCKER_ID } }/glossaryapi:${GITHUB_SHA::7}
          docker tag ${ { secrets.DOCKER_ID } }/glossaryapi:${GITHUB_SHA::7} glossaryapi
          docker stop glossaryapi
          docker run -d --rm --name glossaryapi --network kpp -e SPRING_DATASOURCE_URL=jdbc:mysql://mysql:3306/glossary?useSSL=false glossaryapi     
```
<br/>

#### Check out
``` yaml
- uses: actions/checkout@v3
```
Build 를 위한 repository 를 check out 을 제일 먼저 받는다

<br/>

#### JDK 
``` yaml
- name: Set up JDK 11
      uses: actions/setup-java@v3
      with:
        java-version: '11'
        distribution: 'temurin'
```
JDK 를 받아오는 작업인데 java version 은 필요한 version 으로 받아오면 되고, 
temurin 에서 받아오는 특별한 이유는 없고 github 에서 gradle 로 CI 작업에 대한 기본 템플릿에 포함되어 있어 temurin 을 권장하는 것으로 보여 그대로 사용 했다. 

<br/>

#### Build Gradle
```yaml
- name: Grant execute permission for gradlew
  run: chmod +x gradlew

- name: Build with Gradle
  run: ./gradlew build -x test
```
로컬에서 빌드 하듯이 gradlew 커맨드를 사용했다.

해당 프로젝트에는 DB 에 접속이 필요한 unit test code 가 작성되어 있어 build 시에 에러가 발생한다.
`-x test` 옵션을 통해서 test 는 생략했다.

> 물론 build 전에 DB 또한 세팅하여 테스트 코드가 정상적으로 동작하도록 할 수도 있지만, 
> 하루종일 시도를 했음에도 결국은 알 수 없는 이유로 DB Connect 에 실패하였다...
> 
> 거기다가 업친 데 덮친격으로 내 실수로 EC2 인스턴스를 통채로 날려버렸다...
> 
> 도저히 멘탈관리가 되지 않아 분하지만 DB 접속을 포함한 CI 는 다음에 다시 도전하기로 했다.

<br/>

#### Docker Build And Push To Docker Hub
```yaml
- name: Docker build and push to docker hub
    run: |
      docker login -u ${ { secrets.DOCKER_ID } } -p ${ { secrets.DOCKER_PASSWORD } } 
      docker build -t glossaryapi . 
      docker tag glossaryapi ${ { secrets.DOCKER_ID } }/glossaryapi:${GITHUB_SHA::7} 
      docker push ${ { secrets.DOCKER_ID } }/glossaryapi:${GITHUB_SHA::7}
```
터미널에 docker 명령어 치는 것과 동일하게 작성하면 된다.
* Docker Hub 에 Push 권한을 위해 `login`
* jar File 로 Build 되어 있는 서비스를 Image 로 `build`
    * 사전에 repository 에 Dockerfile 을 만들어 둬야 한다
* version 관리를 위해 SHA 7자리를 이름으로 사용해 `tag` 생성
* 만들어진 tag 를 Docker Hub 에 `push`


**다만** 민감한 계정정보는 직접 script 에 담지 말고 Github 에서 제공하는 secrets 변수를 통해 작성하도록 하자.

<br/>

#### Deploy to AWS EC2
``` yaml
 - name: Deploy to AWS EC2
      uses: appleboy/ssh-action@master
      with:
        host: 서버주소
        username: ${ { secrets.AWS_USERNAME } }
        key: ${ { secrets.AWS_PEM } }
        envs: GITHUB-SHA
        script: |
          docker pull ${ { secrets.DOCKER_ID } }/glossaryapi:${GITHUB_SHA::7}
          docker tag ${ { secrets.DOCKER_ID } }/glossaryapi:${GITHUB_SHA::7} glossaryapi
          docker stop glossaryapi
          docker run -d --rm --name glossaryapi --network kpp -e SPRING_DATASOURCE_URL=jdbc:mysql://mysql:3306/glossary?useSSL=false glossaryapi        
```

ssh 로 EC2 에 접속하여 docker 명령어를 내리는 작업이다.

key 에는 인스턴스 생성시 발급 받은 pem 을 넣었다. 여기서 주의 할 점은, 시작과 끝쪽에 ---- BEGIN 어쩌구 END 어쩌구 하면서 주석처럼 적혀있는 부분이 있는데, 그 부분도 같이 적어야지 동작한다.

script 는
* 아까 push 한 image 를 `pull`
* push 할 때 붙였던 `tag` 를 이번엔 제거
* 기존에 실행 중인 container 를 `stop` 
* 새로 받은 image 를 적절한 옵션값과 함께 `run`

서버로 전송된 서비스가 DB 에 접속못하는 문제가 발생했는데, 이 부분은 [별도로 포스팅을 작성](https://deokhunkim.github.io/posts/docker/%EB%8B%A4%EB%A5%B8-docker-container%EC%9D%98-db-%EC%A0%91%EC%86%8D-%EC%98%A4%EB%A5%98.html)했다. 

<br/>


<br/>
<br/>
<br/>


##### 참고 사이트
* https://itcoin.tistory.com/685

