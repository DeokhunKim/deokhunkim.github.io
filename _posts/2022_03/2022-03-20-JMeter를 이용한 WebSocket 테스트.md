---
layout: post
title: JMeter를 이용한 WebSocket 테스트
date: 2022-03-20
categories: [JMeter]
tags: [KPP, WebMessenger]
author: Deokhun Kim
comment: true
update: 2022-03-20
published : true
---

### 개요
단순한 웹페이지의 GET/POST Request 테스트는 어렵지 않다.

하지만 내가 테스트 하고자 하는 기능은
1. 로그인을 통해 Session을 얻어서
2. 채팅 페이지로 들어가
3. Socket 데이터를 주고 받는 것을
4. 여러 사용자를 통해 동시에 수행 해야하는

다소 이것저것 작업이 필요한 테스트였다.

하나하나의 작업은 지금 생각해보면 어렵지 않으나, 처음 해보는 입장에서 많이 헤맸으므로 기록을 하려고 한다.

프로그램 설치, URL 설정 이런 사소한것들은 대충대충 넘어가고, **위 순서에 적힌 동작을 어떻게 하는지** 중점으로 알아보자.

<br/>

### JMeter 설치 및 Plugin 설치

JMeter는 그냥 설치하면 되고, WebSocket 접속을 위해 플러그인 추가로 받아야한다.

`WebSocket Sampler` 플러그인을 받으면 되는데 2가지 버젼이 있다.
* WebSocket Sampler by Maciej Zaleski
* WebSocket Sampler by Peter Doornbosch

여기서 꼭 **by Peter Doornbosch** 버젼으로 받자.

Maciej Zaleski 버젼은 업데이트가 되지 않아 동작하지 않는 경우가 많다. 
내가 시간을 제일 많이 소모한 부분 중 하나 ㅠㅠ


<br/>

### Sampler 구성

* `Thread Group`
  * `HTTP Cookie Manager`: 단순히 추가만 하면 된다. Thread Group 내의 Cookie를 유지시켜주는 역할을 한다.
  * `Once Only Controller`
    * `HTTP Request`: Login 요청을 위한 POST Request로 설정을 해주자. Parameters에 테스트에 사용할 아이디/비밀번호를 작성하면 되는데, 
    나같은 경우는 3개의 스레드로 돌릴 것이기에 *User Defined Variables* 를 통해 3개 계정의 정보를 등록하여 변수화 시켰다.
    * `WebSocket Open Connection`: Socket에 연결하는 부분인데, **Path 뒤에 꼭 '/websocket'을 붙이자!!** 이것 때문에 정말정말 많이 헤맸다.
    Spring WebSocketHandler 특징인 것 같은데 socket에 대한 처리 경로를 내부적으로 /websocket 추가해서 처리하고 있는 것 같다.(추측)
    * `Loop Controller`: 접속 후에 채팅을 할 만큼 Loop Count에 세팅하자.
      * `WebSocket request-response Sampler`: Connection은 **use existing connection**을 해야 위에서 접속한 세션을 통해 계속 실행한다. 
      Data는 **Text**로 하고, Request Data는 구현한 대로 작성해주자. 나는 JSON을 사용했으므로 **JSON**으로 작성했다.
    * `WebSocket Close`: 소켓 접속 종료 요청
  * `Agregate Report`: 수행한 결과를 보기 위한 Listener
* `User Defined Variables`
* `CSV Data Set Config`


<br/>


<img src="/assets/postimg/2022_03/testplan.png" width="400px"/>


