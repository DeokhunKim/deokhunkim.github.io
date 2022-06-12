---
layout: post
title: (Design Pattern)Abstract Factory
date: 2022-04-02
categories: [DesignPattern]
tags: [Abstract Factory, DesignPattern, Kotlin]
author: Deokhun Kim
comment: true
update: 2022-04-02
published : true
---

### 개요
추상 팩토리 (Abstract Factory)

객체 인스턴스를 생성하기 위한 생성 패턴

Factory Method와 Abstract Factory의 차이
- Factory Method
  - 하나의 Factory가 알맞은 요소의 인스턴스틑 **직접 결정**한다
- Abstract Factory
  - 각 요소별로 Factory가 있어 **어떤 Factory를 선정할지만 걸정** 하고, 각 인스턴스를 생성은 각 Factory가 담당한다

   



<br/>

### 장단점 및 사용 예시
##### 다양한 구성 요소 별로 '객체의 집합'을 생성해야 할 때 유용하다
단순히 개체를 구별해야 할 경우에는 복잡하게 추상 팩토리를 사용하지 않고 팩토리 메서드로 충분하다.
하지만 집합 혹은 그룹별로 생성해야 할 경우는 추상 팩토리를 사용하면 유용하다.


예를들어 단순히 아이폰과 갤럭시폰을 구별하거나 애플워치와 갤럭시워치를 구별하기 위해서 사용할 필요는 없지만,
애플유저(아이폰, 애플워치) 그룹과, 갤럭시그룹(갤럭시폰, 갤럭시워치) 그룹을 생성하기 위해 각 그룹별로 Factory를 만들어 담당하면 된다.

<br/>

### 예시코드 분석
```kotlin

interface Plant

class OrangePlant : Plant

class ApplePlant : Plant

abstract class PlantFactory {

    abstract fun makePlant(): Plant

    companion object {
        inline fun <reified T : Plant> createFactory(): PlantFactory =
            when (T::class) {
                OrangePlant::class -> OrangeFactory()
                ApplePlant::class -> AppleFactory()
                else -> throw IllegalArgumentException()
            }
    }
}

class AppleFactory : PlantFactory() {
    override fun makePlant(): Plant = ApplePlant()
}

class OrangeFactory : PlantFactory() {
    override fun makePlant(): Plant = OrangePlant()
}


class AbstractFactoryTest {

    @Test
    fun `Abstract Factory`() {
        val plantFactory = PlantFactory.createFactory<OrangePlant>()
        val plant = plantFactory.makePlant()
        println("Created plant: $plant")

        assertThat(plant).isInstanceOf(OrangePlant::class.java)
    }
}
```
PlantFactory를 생성 할 때 AppleFactory와 OrangeFactory를 선택하여 호출한다.

PlantFactory는 어떤 Factory를 호출 할 지 결정만 해주며, 구체적인 생성 방법은 AppleFactory 또는 OrangeFactory가 한다.




<br/>

### 작성 해본 예시
```kotlin
interface Phone
class IPhone : Phone
class GalaxyPhone : Phone

interface Watch
class AppleWatch : Watch
class GalaxyWatch : Watch

open class User(var phone: Phone, var watch: Watch )
class AppleUser(phone: Phone, watch: Watch ) : User(phone, watch)
class GalaxyUser(phone: Phone, watch: Watch ) : User(phone, watch)

abstract class UserFactory {
    abstract fun giveMobile(): User

    companion object {
        inline fun <reified T : User> makeUser(): User =
            when (T::class) {
                AppleUser::class -> AppleUserFactory().giveMobile()
                GalaxyUser::class -> GalaxyUserFactory().giveMobile()
                else -> throw IllegalArgumentException()
            }
    }
}

class AppleUserFactory : UserFactory() {
    override fun giveMobile(): User {
        return AppleUser(IPhone(), AppleWatch())
    }
}

class GalaxyUserFactory : UserFactory() {
    override fun giveMobile(): User {
        return GalaxyUser(GalaxyPhone(), GalaxyWatch())
    }
}

class AbstractFactoryTest {
    @Test
    fun runAbstractFactoryTest() {
        println("Start Test!")
        val appleUser = UserFactory.makeUser<AppleUser>()
        val galaxyUser = UserFactory.makeUser<GalaxyUser>()

      appleUser.run { println("팩토리가 만든 애플유저의 클래스: ${appleUser::class.simpleName}") }
      appleUser.run { println("애플유저의 Phone은 ${phone::class.simpleName}이고 Watch는 ${watch::class.simpleName}입니다.") }
      assertThat(appleUser).isInstanceOf(AppleUser::class.java)
      assertThat(appleUser).isNotInstanceOf(GalaxyUser::class.java)
      assertThat(appleUser.phone).isInstanceOf(IPhone::class.java)
      assertThat(appleUser.watch).isInstanceOf(AppleWatch::class.java)

      galaxyUser.run { println("팩토리가 만든 갤럭시유저 클래스: ${galaxyUser::class.simpleName}") }
      galaxyUser.run { println("갤럭시유저의 Phone은 ${phone::class.simpleName}이고 Watch는 ${watch::class.simpleName}입니다.") }
      assertThat(galaxyUser).isNotInstanceOf(AppleUser::class.java)
      assertThat(galaxyUser).isInstanceOf(GalaxyUser::class.java)
      assertThat(galaxyUser.phone).isInstanceOf(GalaxyPhone::class.java)
      assertThat(galaxyUser.watch).isInstanceOf(GalaxyWatch::class.java)

        println("End Test!")
    }
}
```

유저를 만드는 Factory가 있으며 AppleUser일 경우 AppleUserFactory를 호출하고, 
GalaxyUser일 경우 GalaxyUserFactory를 호출한다.

유저에게 Phone과 Watch를 어떤것을 줄지는 각 Factory가 담당한다.

#### Test 결과
<img src="/assets/postimg/2022_04/AbstractFactory002.png" />

<br/>

#### 도식
<img src="/assets/postimg/2022_04/AbstractFactory001.svg" />


<br/>

***참고 사이트***
- https://github.com/dbacinski/Design-Patterns-In-Kotlin
- https://ko.wikipedia.org/wiki/%EC%B6%94%EC%83%81_%ED%8C%A9%ED%86%A0%EB%A6%AC_%ED%8C%A8%ED%84%B4
- https://beomseok95.tistory.com/246
- https://www.bsidesoft.com/8187