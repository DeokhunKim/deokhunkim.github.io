---
layout: post
title: (JPA) Join 과 Fetch Join, 그리고 Lazy Loading 과 Eager Loading
date: 2022-04-20
categories: [Spring]
tags: [JPA, DB, Java]
author: Deokhun Kim
comment: false
update: 
---

### FETCH JOIN ?
JPQL 에서 그냥 Join 을 하더라도 검색한 Entity 만 조회 될 뿐 Join 에 연관된 Entity 는 조회되지 않는다.

하지만 Fetch Join 을 사용하면 연관된 모든 Entity 를 조회하여 가지고 있는다.


##### JPA 가 만들어내는 JOIN 쿼리와 FETCH JOIN 쿼리
Join
```java
@Query("SELECT d FROM Document d JOIN d.content WHERE d.isUse = true")
```
```sql
select
    document.*
from
    document document
inner join
    content content
on document.content_id = content.content_id
where
    document.is_use = true
```

<br/>
Fetch Join 

```java
@Query("SELECT d FROM Document d JOIN FETCH d.content WHERE d.isUse = true")
```
```sql
select
    document.*,
    content.*
from
    document document 
inner join
    content content 
on document.content_id = content.content_id 
where
    document.is_use = true
```
<br/>

하지만 단지 그 차이 뿐일까?

한 사이트에서는 Fetch Join 의 가이드에 이렇게 적어놨다.
>Although this query looks very similar to other queries, there is one difference; the Employees(※Join 된 테이블) are **eagerly loaded**.
>
>...
>
> **However, we must be aware of the memory trade-off.** We may be more efficient because we only performed one query, but we also loaded all Departments and their employees into memory at once.
> 
> https://www.baeldung.com/jpa-join-types


Fetch Join 을 사용함으로 **eagerly load** 를 하게되고, 그로인해 **메모리 사용에 주의**를 말하고 있다.

그런데 eagerly load 가 뭘까?

<br/>

### Lazy Loading vs Eager Loading
> [ENG]
> 
> **Eager Loading** is a design pattern in which data initialization occurs on the spot.
> 
>**Lazy Loading** is a design pattern that we use to defer initialization of an object as long as it's possible.
> 
> [KOR]
> 
> **Eager Loading** 은 데이터 초기화가 그 자리에서 일어나는 디자인 패턴입니다.
> 
> **Lazy Loading** 은 가능한 한 객체 초기화를 연기하는 데 사용하는 디자인 패턴입니다.
> 
> https://www.baeldung.com/hibernate-lazy-eager-loading

Eager loading 은 쿼리 수행 즉시 메모리에 로드를 시키고, 
Lazy loading 은 명시 적으로 호출 할 때 메모리에 로드를 시킨다.

아래 코드로 예시를 들면, Eager loading 은 **1라인**에서 즉시 초기화되고, Lazy loading은 **2라인**에서 초기화 된다.
```java
Line 1: List<UserLazy> users = sessionLazy.createQuery("From UserLazy").list();
Line 2: UserLazy userLazyLoaded = users.get(3);
Line 3: return (userLazyLoaded.getOrderDetail());
```

즉 실제 데이터가 필요한지 여부에 상관없이 모든 데이터를 먼저 불러오는 Eager 방식은 메모리를 많이 차지할 우려가 있다는 사실을
Fetch Join 의 가이드에서는 경고했던 것이었다.

#### 장단점
* **Lazy Loading**
  * 장점
    * 짧은 초기 로드시간
    * 적은 메모리 소비
  * 단점
    * 원치 않는 순간 성능에 영향을 줄 가능성
    * 예외처리의 리스크(In some cases we need to handle lazily initialized objects with special care, or we might end up with an exception.)
* **Eager Loading**
  * 장점
    * 최초 로딩 이후 지연에 관한 성능 문제 없음
  * 단점
    * 긴 초기화 시간
    * 불필요한 데이터를 너무 많이 로드 하여 성능 하락의 가능성


### 결론
연관된 Entity 가 필요하다면 Fetch Join 을 활용 하자.

남용 하면 불필요한 데이터까지 불러오는 오버헤드가 커질 수 있으므로 꼭 필요한 경우에만 사용하자.

검색 조건에만 필요하고 실제 Entity 가 불필요 하다면 일반 Join 으로 충분하다.


<br/>

##### 참고 사이트
* https://cobbybb.tistory.com/18
* https://incheol-jung.gitbook.io/docs/q-and-a/spring/n+1
* https://www.baeldung.com/jpa-join-types
* https://www.baeldung.com/hibernate-lazy-eager-loading