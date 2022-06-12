---
layout: post
title: (Design Pattern)Adapter
date: 2022-05-27
categories: [DesignPattern]
tags: [Adapter, DesignPattern, Kotlin]
author: Deokhun Kim
comment: true
update: 2022-05-27
published : true
---

### 개요
어뎁터 (Adapter)

*기존 클래스를 다른 인터페이스로 사용 할 수 있도록 하는 디자인 패턴.*

<br>

기존 클래스를 수정하지 않고(= 그대로 사용 하며) 새로운 방법으로 사용 가능하다.

흔히 **Wrapper** 라는 용어로 많이 사용하기도 하고, 
기존 클래스를 수정하지 않는다는 점에서 **데코레이터(Decorator)** 패턴 과도 유사하다.

<br/>

### 장단점 및 사용 예시
##### 장점
* 기존 객체(클래스)를 수정하지 않는다
  * A(제공자) <--> 어뎁터 <--> B(사용자) 의 구조를 가지기 때문에 기존의 클래스 A,B 는 수정 할 필요가 없다
  * 이것은 SOLID 원칙 중 개방-폐쇄의 원칙을 아주 잘 지키는 예시다


##### 사용예시
기존의 코드에 새로운 코드나 라이브러리를 함께 사용하고 싶은데 인터페이스가 다른 경우



<br/>

### 예시코드 분석
```kotlin
interface Temperature {
  var temperature: Double
}

class CelsiusTemperature(override var temperature: Double) : Temperature

class FahrenheitTemperature(private var celsiusTemperature: CelsiusTemperature) : Temperature {

  override var temperature: Double
    get() = convertCelsiusToFahrenheit(celsiusTemperature.temperature)
    set(temperatureInF) {
      celsiusTemperature.temperature = convertFahrenheitToCelsius(temperatureInF)
    }

  private fun convertFahrenheitToCelsius(f: Double): Double =
    ((BigDecimal.valueOf(f).setScale(2) - BigDecimal(32)) * BigDecimal(5) / BigDecimal(9))
      .toDouble()


  private fun convertCelsiusToFahrenheit(c: Double): Double =
    ((BigDecimal.valueOf(c).setScale(2) * BigDecimal(9) / BigDecimal(5)) + BigDecimal(32))
      .toDouble()
}

class AdapterTest {

  @Test
  fun Adapter() {
    val celsiusTemperature = CelsiusTemperature(0.0)
    val fahrenheitTemperature = FahrenheitTemperature(celsiusTemperature)

    celsiusTemperature.temperature = 36.6
    println("${celsiusTemperature.temperature} C -> ${fahrenheitTemperature.temperature} F")

    assertThat(fahrenheitTemperature.temperature).isEqualTo(97.88)

    fahrenheitTemperature.temperature = 100.0
    println("${fahrenheitTemperature.temperature} F -> ${celsiusTemperature.temperature} C")

    assertThat(celsiusTemperature.temperature).isEqualTo(37.78)
  }
}
```

섭씨(C)에 화씨(F) Adapter 를 만들어서 섭씨/화씨 를 원하는 대로 꺼내서 사용 가능하도록 구현했다.

복잡한 구조는 아니지만 잊고 있었던 문법이 눈길을 끌었다. get,set 을 커스텀해서 사용하는 방법이다.
내부적으로는 섭씨 온도만 가지고 있으며 get,set 에 override 로 변환하는 메소드를 호출해줘서, 실제 사용 할 때는 자동으로 변환되는 것 처럼 보인다.
특히 코틀린 문법에 따라 get set 함수를 호출하는 것이 아니라 대입연산자(=)나 변수를 그대로 사용하더라도 변환이 되는점이 더욱 코드를 간결하게 보이게 한다.



<br/>

### 작성 해본 예시
#### 설정
알맞은 숫자를 반환해주는 class 가 존재한다.

그런데 영어를 모르는 사람을 위해 똑같은 기능을 하지만 한글로 작성된 Adapter 를 작성하려고 한다.
(물론 클래스나 함수명을 한글로 작성하는건 아주 좋지 않은 행위다..)

예를들어 getOne() 대신에 일() 이라고 작성하면 getOne() 의 값을 얻을 수 있다. 

```kotlin
open class Number {
  private val one : Int = 1
  private val two : Int = 2
  private val three : Int = 3

  fun getOne(): Int = one
  fun getTwo(): Int = two
  fun getThree(): Int = three
}

class 숫자 : Number() {
  fun 일():Int = getOne()
  fun 이():Int = getTwo()
  fun 삼():Int = getThree()
}


class AdapterTest {

  @Test
  fun test() {
    var engNumber : Number = Number()
    var 한국숫자 : 숫자 = 숫자()

    println("${engNumber.getOne()} == ${한국숫자.일()}")
    Assertions.assertThat(engNumber.getOne()).isEqualTo(한국숫자.일())
    println("${engNumber.getTwo()} == ${한국숫자.이()}")
    Assertions.assertThat(engNumber.getTwo()).isEqualTo(한국숫자.이())
    println("${engNumber.getThree()} == ${한국숫자.삼()}")
    Assertions.assertThat(engNumber.getThree()).isEqualTo(한국숫자.삼())
  }
}
```




<br/>

***참고 사이트***
- https://en.wikipedia.org/wiki/Adapter_pattern
