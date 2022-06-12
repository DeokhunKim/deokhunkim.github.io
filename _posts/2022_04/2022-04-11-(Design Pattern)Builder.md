---
layout: post
title: (Design Pattern)Builder
date: 2022-04-11
categories: [DesignPattern]
tags: [Builder, DesignPattern, Kotlin]
author: Deokhun Kim
comment: true
update: 2022-04-11
published : true
---

### 개요
빌더 (Builder)

객체 인스턴스를 생성하기 위한 생성 패턴

*빌더 패턴이란 복합 객체의 생성 과정과 표현 방법을 분리하여 동일한 생성 절차에서 서로 다른 표현 결과를 만들 수 있게 하는 패턴이다. (2단어 요약 : 생성자 오버로딩)*

##### 내 생각
생성자를 유저가 마음대로 사용하는게 아니라 Builder를 통해 만들어 내는 것은 다른 생성자 패턴과 동일하나, 문제가 없는 부분에선 유연하게, 필요 할 경우에는 제한적으로 사용할 수 있도록 해주는 패턴


<br/>

### 장단점 및 사용 예시
##### 장점
* 기본값을 쉽게 설정 가능
  * 객체 생성시 파라미터가 필요 없는 경우 더미값을 넣거나 새로운 형태의 생성자를 만들어야 했지만 Builder를 통해 편리하게 처리 가능하다
* 확장성에 유리함
  * 객체 내 새로운 멤버가 생겼더라도 이미 구현된 생성자들을 수정 안하더라도 오류 없이 동작되도록 할 수 있다
  * 이미 구현된 생성자들의 수정이 필요 할 경우에는 오류가 강제되도록 처리도 가능하다
* 가독성이 좋음
  * 생성자 파라미터로 나열했을 경우 각 요소가 어떤값인지 확인이 힘드나, Builder의 경우 메소드명을 통해 직관적으로 확인 가능하다

##### 사용예시
생성자의 매개변수 혹은 멤버변수가 다양하고 동적인 경우에 유용하게 사용 가능하다.

특히 제약을 두고 싶은 부분, 유연하게 처리 할 부분 등을 구별 가능.



<br/>

### 예시코드 분석
```kotlin

class Dialog {

  fun setTitle(text: String) = println("setting title text $text")
  fun setTitleColor(color: String) = println("setting title color $color")
  fun setMessage(text: String) = println("setting message $text")
  fun setMessageColor(color: String) = println("setting message color $color")
  fun setImage(bitmapBytes: ByteArray) = println("setting image with size ${bitmapBytes.size}")

  fun show() = println("showing dialog $this")
}

// Builder
class DialogBuilder() {

  constructor(init: DialogBuilder.() -> Unit) : this() {
    init()
  }

  private var titleHolder: TextView? = null
  private var messageHolder: TextView? = null
  private var imageHolder: File? = null

  fun title(attributes: TextView.() -> Unit) {
    titleHolder = TextView().apply { attributes() }
  }

  fun message(attributes: TextView.() -> Unit) {
    messageHolder = TextView().apply { attributes() }
  }

  fun image(attributes: () -> File) {
    imageHolder = attributes()
  }

  fun build(): Dialog {
    println("build")
    val dialog = Dialog()

    titleHolder?.apply {
      dialog.setTitle(text)
      dialog.setTitleColor(color)
    }

    messageHolder?.apply {
      dialog.setMessage(text)
      dialog.setMessageColor(color)
    }

    imageHolder?.apply {
      dialog.setImage(readBytes())
    }

    return dialog
  }

  class TextView {
    var text: String = ""
    var color: String = "#00000"
  }
}

//Function that creates dialog builder and builds Dialog
fun dialog(init: DialogBuilder.() -> Unit): Dialog =
  DialogBuilder(init).build()

class BuilderTest {

  @Test
  fun Builder() {
    println("Build dialog")

    val dialog: Dialog =
      dialog {
        title {
          text = "Dialog Title"
        }
        message {
          text = "Dialog Message"
          color = "#333333"
        }
        image {
          File.createTempFile("image", "jpg")
        }
      }

    println("Show dialog")

    dialog.show()
  }
}
```

사실 Kotlin에서 Builder 패턴은 유용성이 낮다.
왜냐면 Kotlin은 생성자에서 초기값을 설정 가능 하기 때문에 Builder를 굳이 만들지 않아도 동일한 효과를 만들어 낼 수 있기 때문이다.

그럼에도 Build 패턴을 구현할 수 있으나, 위 코드는 마음에 안드는 것이 있는데 **Builder를 통하지 않고 직접 Dialog를 생성 가능**한 점이다.

생성을 위한 인터페이스(패턴)을 만들어 놨는데 다른 방법으로 사용이 가능하다는 점은 Builder를 통해 전부가 통제되지 못하다는 뜻이기 때문에 
예기치 않은 곳에서 문제가 발생할 리스크가 있다는 것이 개인적인 의견이다.

그리고 내가 Kotlin에 덜 익숙해서 그런지 모르겠지만, 코드의 가독성이 떨어진다고 생각된다.


<br/>

### 작성 해본 예시
위 코드에서 마음에 안드는 점을 고쳐서 작성 해 보았다.

* House는 HouseBuilder를 통해 만들어진다.
* 방이나 화장실 등은 없을 수도 있고 여러 개가 있을 수도 있다. 
없다면 Builder에서 호출을 안하면 될 것이고, 여러 개라면 여러 번 호출하면 된다.
* House의 이름과 주소는 없어서 안되므로 Build() 메소드에서 필수적으로 파라미터로 받는다.

특히 아래코드는 House가 private constructor를 가지고 있기 때문에 Builder를 통하지 않고는 생성하지 못한다.
추후 House의 구조가 변경 되더라도 House가 사용되는 모든 곳에서 Builder를 통해 유연하게 처리 가능 할 것이다.

그리고 나는 setXXXX()로 통일을 했는데, 만약 List를 가지는 멤버는 addXXXX()로 구별해서 사용했다면 
가독면에서 더 좋았을 거란 생각이 들었다.

```kotlin
typealias Room = String
typealias Toilet = String
typealias Veranda = String

class House {
  private var homeName: String
  private var address: String
  private var rooms = mutableListOf<Room>()
  private var toilets = mutableListOf<Toilet>()
  private var verandas = mutableListOf<Veranda>()

  private constructor(
    homeName: String,
    address: String,
    rooms: MutableList<Room>,
    toilets: MutableList<Toilet>,
    veradas: MutableList<Veranda>){
    this.homeName = homeName
    this.address = address
    this.rooms = rooms
    this.toilets = toilets
    this.verandas = veradas
  }


  fun printInfo() {
    println("---- Room Info ----")
    println("집 주소는 ${homeName} ${address}입니다.")
    println("방의 갯수는 ${rooms.size}개고, ${rooms}이 있습니다.")
    println("화장실의 갯수는 ${toilets.size}개고, ${toilets}이 있습니다.")
    println("베란다의 갯수는 ${verandas.size}개고, ${verandas}이 있습니다.")
    println("--------------------")
  }

  class HouseBuilder {
    var rooms = mutableListOf<Room>()
    var toilets = mutableListOf<Toilet>()
    var verandas = mutableListOf<Veranda>()

    fun build(homeName: String, address:String): House {
      return House(homeName, address, this.rooms, this.toilets, this.verandas)
    }

    fun setRoom(room:Room): HouseBuilder {
      this.rooms.add(room)
      return this
    }
    fun setToilet(toilet:Toilet): HouseBuilder {
      this.toilets.add(toilet)
      return this
    }
    fun setVeranda(veranda: Veranda): HouseBuilder {
      this.verandas.add(veranda)
      return this
    }
  }

}

class BuilderTest {

  @Test
  fun Builder() {

    var house = House.HouseBuilder()
      .setRoom("small room")
      .setRoom("big room")
      .setToilet("outer toilet")
      .setToilet("inner toilet")
      .setVeranda("veranda")
      .build("Kotlin아파트","224호")

    println(house.printInfo())
  }
}
```




#### Test 결과
<img src="/assets/postimg/2022_04/Builder_001.png" />

<br/>

#### 도식
<img src="/assets/postimg/2022_04/builder_002.svg" />


<br/>

***참고 사이트***
- https://github.com/dbacinski/Design-Patterns-In-Kotlin
- https://ko.wikipedia.org/wiki/빌더_패턴
