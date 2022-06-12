---
layout: post
title: MongoDB의 Collection 내부의 List 삽입 속도
date: 2022-03-17
categories: [MongoDB]
tags: [KPP, WebMessenger]
author: Deokhun Kim
comment: true
update: 2022-03-17
published : true
---

### MongoDB는 INSERT에 유리하고 UPDATE에 불리하다?
NoSQL의 장단점을 검색하면 항상 나오는 말이다.

Document가 정형화 되지 않고 자유롭기 때문에 다양한 형태의 데이터를 삽입하기에 좋지만, 
반대로 기존의 데이터를 수정하려면 Document 하나하나 수정해줘야 하기 때문에 불리하다 라고 나는 이해를 했다.
지금 생각해도 틀린말은 아니다.


**그런데 나는 단단히 오해한 점이 있었다.**

<br/>

### Document에 List를 만들어놓고 거기다가 우겨 넣으면 되겠네!

> "아니 INSERT가 빠르다며?"
> 
> "Document 하나 만들어두고 List에 데이터 계속 추가하면 되겠네."
> 
> "그게 INSERT 아님?"

나의 얄팍한 지식은 이런 참사를 만들어 냈다.

NoSQL을 XML 혹은 JSON과 비유를 많이 하곤 하는데, 
나는 그런 계단식 구조의 비유만 듣고 DB가 동작하는 방법에 대해서는 생각하지 못했던 것이었다.

<br/>

### 처음의 구상은 이러했다

구상
* 각 대화방 별로 Document를 생성한다.
* Document 내부의 필드로 `CHAT`이라는 Document List를 가지면서 대화로그를 쌓는다.
* 어차피 시간 순서대로 쌓여있으니 데이터를 쌓으면서 필요할경우 끝부분만 읽으면 된다.

그렇게 생각하고 Document를 구성했다.
```java
@Document(collection = "roomInfo")
public class RoomInfo {

    @Id
    private String id;
    private String roomName;
    private Set<String> members = new HashSet<>();
    private List<Chat> chats = new ArrayList<>();
}
```

그리고 내 머릿속에 있던 상상의 구조는 이랬고,
xml에 한줄 **insert** 하는거니까 빠르겠지? 하는 생각을 혼자 하고 있었던 것이다.

```xml
<RoomInfo>
    <id> id </id>
    <roomName> roomName </roomName>
    <members> member1 </members>
    <members> member2 </members>
    <Chat> ~~~ </Chat>
    <Chat> ~~~ </Chat>
    <Chat> ~~~ </Chat>
    ...
</RoomInfo>
```

간단한 테스트 코드 동작 확인 후 문제가 없음에 룰루랄라 나머지 기능의 구현을 전부 끝내고,
실제 웹으로 채팅도 해보고. DB에 적재된 데이터 확인도 해보았을 때 전부 잘 동작 했다.

이제 과부화 테스트 해 볼 차례다. JMeter로 과부화 테스트를 해보았다.

*RDB 보다 얼마나 빠를려나?*

<br/>

### 느려도 너~~~무 느렸다

RDB 보다 얼마나 빠를까? 2배? 3배? 10배? 열심히 행복회로를 돌린 나의 기대를 무참하게 부셔버리는 결과.

**RDB 보다 8배나 느렸다.**

애써 당황하지 않은 척 하며 다른 곳에서 원인을 찾아보았다. 채팅내역을 불러오는 부분, DB에 connection 하는 부분 이곳저곳 찔러보고 다녔지만, 
확실하게 나오는 원인분석 결과는 *Chat 로그를 쌓는 부분* 에서 너무나도 느려졌다.

그리고 더욱 절망적인 것은, 처음엔 빠르다가 채팅로그가 늘어날 수록 점점 느려진다는 점이었다.
그렇게 그날 밤새 문제 원인을 검색하며 찾아봤지만 뚜렷한 답을 얻지 못했다.

나 처럼 무식한 생각을 한 사람은 거의 없었으니까...

<br/>

### 내가 한 것은 INSERT 가 아니라 UPDATE 였다

*Document 내부에서 List를 add 하는 것은 INSERT가 아니라 UPDATE다.*

내가 내린 결론이다.

나는 그저 Document 속의 Document를 하나 더 추가하는 것이라고 생각을 했지만,
실제로는 하나의 Document를 통채로 업데이트 하는 작업이었고,
심지어 그 Document는 로그가 쌓일수록 점점 덩치가 거대해진 무거운 데이터 덩어리 였던 것이었다.

<img src="/assets/postimg/2022_03/Document.png" width="400px"/>



### 별도의 Collection을 만들어 해결했지만..
Document 내부의 데이터를 추가하는 것이 아니라, Document 자체가 추가 되어야 한다.

결국 채팅 로그만을 위한 별도의 Collection을 만들어 테스트를 진행했고 **아주 빠른 속도를 보여줬다.**

하지만 내 마음속엔 여전한 찜찜함이 남아 있었다.

> 결국 대화방에 대한 정보와 채팅 로그가 분리가 된다면, 
join 작업을 하지 않았을 뿐이지 각 Collection이 Relation을 가지고 있는 것이고, 
이건 RDB와 다를게 없지 않은가?
>
> NoSQL이 가진 장점을 어떻게 살려야 하는거지?

애석하게도 이 글을 쓰는 지금까지 위 질문의 대한 정답은 해결하지 못했다.

온라인 커뮤니티에 질문을 하자니 나의 글 솜씨로는 내가 가진 의문의 개념을 잘 설명 하지 못할 것 같고, 
주변에 물어봐도 MongoDB에 대한 지식이 충분한 사람은 없었다.

이럴땐 정말 퇴사한게 아쉽게 느껴진다. 궁금증을 잘 설명해 줄 수 있는 선배, 같이 고민해줄 수 있는 동료들이 아쉬운 순간이다..

이 질문에 대한 답은 @TODO 를 만들어 놔서 언젠간 해결하리...


