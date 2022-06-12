---
layout: post
title: JPA에서 pageable과 limit의 쿼리 차이
date: 2022-03-19
categories: [Spring]
tags: [KPP, WebMessenger]
author: Deokhun Kim
comment: true
update: 2022-03-19
published : true
---

### JPQL에는 LIMIT 문법이 없다!
메신저에서 유저가 특정 대화방의 목록을 불러올때 한번에 모든 채팅 내역을 불러올 필요는 없다.
가장 최신의 대화내역 n개를 불러온뒤, 필요하면 추가로 채팅 내역을 필요한 만큼 불러 오는 것이 효율적이고 보통의 메신저가 동작하는 방식이다.

일반적으로 query를 통해 n개의 대화 내역을 조회 하려면 LIMIT를 사용하는데, JPQL에서는 LIMIT를 지원하지 않더라(!!).

그럼 어떻게 사용 해야 하는가 하니 스프링에서는 first와 top 키워드를 제공 하는 것으로 확인된다.
*(https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#repositories.limit-query-result)*

근데 이건 defined query method를 사용해야만 한다. **내가 직접 JPQL을 작성해서 질의 하고 싶을 때는 사용 할 수가 없다.**

또 다른 방법으론 pageable을 이용하는 방법이 있었다.*(https://duooo-story.tistory.com/49)*

이 방법은 상당히 유용해 보이고, 특히 page 단위로 이전 채팅 내역을 불러 올 때 활용 또한 가능해 보여서 pageable로 구현 하려고 했다.

그런데 이런 의문이 들었다.
> 혹시나 전체 채팅을 메모리에 올린 뒤 일부분만 잘라서 가져오는게 아닐까?


<br/>

### page로 조회하는 쿼리를 확인해보자

Test 코드를 통해 확인된 prepare query는 아래의 형태를 가졌다.

```sql
select message 
from chat
where chat.room_id=? 
order by chat.time DESC 
limit ? 
offset ?
```

※ 실제 확인된 쿼리: `select chat0_.id as id1_0_, chat0_.message as message2_0_, chat0_.room_id as room_id5_0_, chat0_.time as time3_0_, chat0_.writer as writer4_0_ from chat chat0_ where chat0_.room_id=? order by chat0_.time DESC limit ? offset ?
`

우려와 다르게 쿼리에서 limit를 사용하여 필요한 만큼만 질의하였고, page 처리를 위하여 offset을 사용하는 간단한 쿼리였다.

page request에서 page를 0으로 주고 질의하면 offset=0 으로 쿼리가 수행 될 것이고, 단순 limit 쿼리와 동일하게 처리 가능할 것으로 보인다.

<br/>

### **결론**
> limit가 필요할 땐 pageable을 사용하자



