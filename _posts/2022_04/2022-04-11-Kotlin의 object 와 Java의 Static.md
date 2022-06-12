---
layout: post
title: Kotlin의 object와 Java의 Static
date: 2022-04-11
categories: [Kotlin, Java]
tags: [Kotlin, Java]
author: Deokhun Kim
comment: false
update: 2022-04-11
---

### Kotlin에는 Static 키워드가 없다
*[Kotlin에는 Static 키워드가 없다]*

구글에 **Kotlin Static** 을 검색하면 나오는 많은 웹 페이지의 제목이다.

Kotlin에서는 Static 지정자를 지원하지 않으며, 많은 포스팅에서 **object** 혹은 **companion object** 사용을 권한다.

왜 Kotilin에서는 Static을 포기했으며, 실제 동작은 어떤 차이가 있는지 궁금해졌다.


<br/>

### Kotlin to Java (companion object)

IntelliJ에서 지원하는 기능을 통해 Kotlin을 Java bytecode로 변환한 뒤 다시 Java로 decompile 하는 방식으로 Kotlin code -> Java code 를 만들어 비교해보았다.


#### Kotlin Code
```kotlin
class normalClass {
    var nomalVar = "nomalVar"
    fun normalFun() {}

    companion object companionObjectTest {
        var inCompanionObjectDefinition = "Definition"
        fun companionObjectFun(){}
    }
}
```

<br/>

#### Java Code(지저분한 어노테이션이나 get/set 함수등을 제거했음)
```java

public final class normalClass {

    private String nomalVar = "nomalVar";
        
    public void normalFun() {
    }

    
    private static String inCompanionObjectDefinition = "Definition";
    public static final normalClass.companionObjectTest companionObjectTest = new normalClass.companionObjectTest();

    public static final class companionObjectTest {

        public String getIncompanionObjectDefinition() {
            return normalClass.inCompanionObjectDefinition;
        }
        public void setIncompanionObjectDefinition(String var1) {
            normalClass.inCompanionObjectDefinition = var1;
        }

        public void companionObjectFun() {
        }
        
    }
}
```

<br/>

companion object에서 생성한 변수는 companion object 밖 기존 class에서 static으로 선언되어 관리된다.
단 실제 사용은 companion object인 companionObjectTest가 static class로 생성되어 get 메소드를 통해 사용 가능하다.

그러나 companion object에서 생성한 함수는 단지 static class 안에 선언 된 함수 일 뿐이다. 즉 nested class의 함수와 차이가 없다.





<br/>

### Kotlin to Java (object)
그렇다면 그냥 object로 생성하는 것과 무슨 차이가 있을까?


#### Kotlin Code
```kotlin
object objectClass {
    val normalVal = "normalVal"
    const val constVal = "constVal"

    fun objectFun(){}

}
```

<br/>

#### Java Code(지저분한 어노테이션이나 get/set 함수등을 제거했음)
```java
public final class objectClass {

    private static final String normalVal = "normalVal";
    public static final String constVal = "constVal";

    public static final objectClass INSTANCE;

    public void objectFun() {
    }

    private objectClass() {
    }

    static {
        INSTANCE = new objectClass();
        normalVal = "normalVal";
    }
}
```

object는 static 멤버로 자신의 class를 가지며 변수 또한 static 으로 선언되어 있다.

자기 자신의 instance를 static 멤버로 가지며, 생성자를 private로 막아두어 instance가 변질 될 가능성을 막아뒀다.

이건 어디서 많이 본 방법이다. 바로 **Singleton 패턴**이다!
Kotlin은 단순히 object로 생성하는 것으로 Singleton을 쉽게 구현 가능하다.

그리고 한가지 특이한게 val 멤버와 const val 멤버가 조금 다르게 동작하는데,
val는 상수화 되어 변수명으로 가져오지만, const val은 컴파일 단계에서 그 자체가 정말 상수가 된다.
(실제 바이트 코드에도 변수명이 아니라 문자열 그대로 치환되어 있다)
단순히 상수처럼 사용 할 것이라면 컴파일단계에서 미리 상수로 치환시켜주는 const val가 유리해보인다.

아래 kotlin code 와 변환된 java btye code 를 비교해보면 상당히 많은 차이가 있다.
* const val 을 통해 생성한 문자열은 컴파일 단계에서 그대로 문자열은 완성해 그대로 대입하고 끝이다.
* 하지만 val 을 통해 생성한 문자열은 val 문자열을 가져오는 작업과 StringBuilder 를 통해 문자열을 합치는 작업까지 runtime 에 발생한다.

Kotlin code
```kotlin
val objectClass1 = objectClass.constVal + "a" // const val (L14)
val objectClass2 = objectClass.normalVal + "b" // val (L15)
```

Byte code
```
L14
    LINENUMBER 64 L14
    LDC "constVala" 
    ASTORE 9
L15
    LINENUMBER 65 L15
    NEW java/lang/StringBuilder
    DUP
    INVOKESPECIAL java/lang/StringBuilder.<init> ()V
    GETSTATIC mypatterns/objectClass.INSTANCE : Lmypatterns/objectClass;
    INVOKEVIRTUAL mypatterns/objectClass.getNormalVal ()Ljava/lang/String;
    INVOKEVIRTUAL java/lang/StringBuilder.append (Ljava/lang/String;)Ljava/lang/StringBuilder;
    LDC "b"
    INVOKEVIRTUAL java/lang/StringBuilder.append (Ljava/lang/String;)Ljava/lang/StringBuilder;
    INVOKEVIRTUAL java/lang/StringBuilder.toString ()Ljava/lang/String;
    ASTORE 10
```


<br/>

### 왜 Kotlin 은 Static 지정자를 포기하였는가?
여러 사이트를 검색해서 찾아보고 나름대로 의미를 정리해 보았다. **(나의 주관적 해석과 고민이 들어 갔기 때문에 이 내용이 정답은 아니다)**


#### 객체 지향 관점에서 static 보다 명확하다
객체지향의 설명에 많이 인용하는 붕어빵과 붕어빵 틀 관점에서 인스턴스 객체와 정적 개체는 반대의 개념에 가깝다고 생각된다.
분명 붕여빵틀(class)의 멤버는 붕어빵(instance)의 상태를 나타내야 하는데, 붕어빵(instance)와는 무관한 정적(static) 멤버가 뒤섞여있는 혼란이 있는 것이다.

Kotlin은 이 문제를 언어차원에서 제약을 두어 해결 했다고 생각한다. (내부적인 동작은 일단 뒤로하고, 사용자 관점에서!)

여기저기 static 을 남발하듯이 사용하는 것이 아니라 companion 이라는 **객체(object)**를 통해서만 사용 가능하도록 통제하였다.
비록 내부적인 동작은 static class 와 다를게 없더라도 companion object 는 현재의 class 와는 분명 다른것이라고 구분된다.
이름부터 companion class 가 아니라 companion object 니까.

static 을 사용할때 나는 분명 속으로 이렇게 생각을 했을 것이다.
> A Class **안에** static 멤버 s가 있고...

하지만 companion object 를 사용하면 이제 자연스럽게 이렇게 생각이 된다.
> A Class **와 함께 사용**하는 객체 s가 있고...

똑같이 정적 객체를 두고 사용하는 것이지만 class 와 명확하게 구분지어 생각하게 된다. 
즉 static 키워드 때문에 혼란해졌던 class 와 object 의 구분을, companion object 라는 제약 하나로 명확하게 구분 지어졌기 때문에 언어적 디자인 관점에서 일관성이 유지됐다.





<br/>

### 결론
#### object 를 사용 하는 경우
* 클래스 전체를 정적으로 사용하고 싶다면
* singleton 클래스를 사용하고 싶다면

#### companion object 를 사용 하는 경우
* 클래스 내에서 일부 변수/함수를 정적으로 호출하여 사용 하고 싶은 경우

<br/>

### 참고 사이트
* https://kotlinlang.org/docs/object-declarations.html#semantic-difference-between-object-expressions-and-declarations
* https://www.quora.com/Why-doesnt-Kotlin-programming-language-have-static
* https://discuss.kotlinlang.org/t/what-is-the-advantage-of-companion-object-vs-static-keyword/4034
