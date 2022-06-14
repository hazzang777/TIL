# 코틀린에서 Type을 다루는 방법

인프런에서 `최태현 지식 공유자`님의   
`자바 개발자를 위한 코틀린 입문`을 수강하며  정리한 내용 입니다.  
하단에 강의 링크 있습니다.

## 기본 타입

**코틀린에서는 `선언된 기본값`을 보고 타입을 추론한다.**

```kotlin
val number1 = 10 // Int
val number2 = 10L // Long
```

- 코틀린 vs 자바
    - 자바에서는 기본 타입간의 변환은 암시적으로 이루어질 수 있다.
    - 코틀린에서는 기본 타입간의 변환은 명시적으로 이루어져야 한다.

```kotlin
val number1 = 4
val number2: Long = number1 // 에러: Type mismatch
```

**코틀린에서 기본 타입간의 명시적 변환**

→ `to변환타입()` 을 사용

```kotlin
val num1 = 10
val num2: Long = num1.toLong() // Int -> Long
```

**변수가 nullable이라면 적절한 처리가 필요!**

```kotlin
val num1: Int? = 10
val num2: Long = num1?.toLong() ?: 0L
```

→ `SafeCall`과 `Elvis 연산자`를 이용하여 Long타입을 반환하게끔 적절하게 처리해야함

## 타입 캐스팅

<aside>
💡 기본 타입이 아닌 일반 타입은 어떨까요?

</aside>

```kotlin
fun printAgeIfPerson(obj: Any) {
    if (obj is Person) { // (1)
        val person = obj as Person // (2)
        println(person.name)
    }
}
```

(1) 자바의 instanceof와 기능 동일

- obj가 Person 이면: true 반환
- obj가 Person 이면: false 반환

(2) 자바의 (Person) obj 와 동일

- as Person 생략 가능
- 바로 obj.name으로 접근 가능한데 → 코틀린의 `스마트 캐스트` !
- 코틀린 컴파일러가 컨텍스트를 분석해서 if안에서 타입체크를 해줬네 그러면 이 타입으로 간주될수 있겠구나를 인지하고 있는 것.

**instanceof의 반대가 있을까?**

```kotlin
if(obj !is Person) // obj가 Person이 아니라면
```

**value `is` Type**

- value가 Type이면 → true
- value가 Type이 아니면 → false

**value `!is` Type**

- value가 Type이 아니면 → true
- value가 Type이면 → false

**value `as` Type**

- value가 Type이면 → Type으로 캐스팅
- value가 Type이 아니면 → 예외 발생

**value `as?` Type : 안전한 타입 형변환**

- value가 Type이면 → Type으로 타입 캐스팅
- value가 null이면 → null
- value가 Type이 아니면 → null

## Kotlin의 특이한 타입 3가지

**Any**

- Java의 Object 역할 : 모든 객체의 최상위 타입
- 모든 Primitive Type의 최상위 타입도 Any
- Any 자체로는 null을 포함할 수 없다.
    - null을 포함하고 싶다면 Any?
- Any에 equals/ hashCode/ toString 존재

**Unit**

- Java의 void와 동일한 역할
- void와 다르게 Unit은 그 자체로 타입 인자로 사용 가능
- 함수형 프로그래밍에서 Unit은 단 하나의 인스턴스만 갖는 타입을 의미
    - 즉, 코틀린에서 Unit은 실제 존재하는 타입

**Nothing**

- 함수가 정상적으로 끝나지 않았다는 사실을 표현하는 역할
- 무조건 예외를 반환하는 함수 / 무한 루프 함수 등

```kotlin
fun fail(message: String): Nothing {
	throw IllegalArgumentException(message)
}
```

## String interpolation / String indexing

```kotlin
val hardy = Person("hardy", 26)
val log = "나는 ${hardy.name} 나이는 ${hardy.age}"
println(log) // 나는 hardy 나이는 26
```

`${변수}` 를 사용하면 값이 들어간다

- 중괄호 생략도 가능하다.

```kotlin
val name = "hardy"
println("이름: $name") // 이름: hardy
```

<aside>
💡 TIP

</aside>

변수 이름만 사용하더라도 `${변수}` 를 사용하는 것의 장점

1. 가독성
2. 일괄 변환
3. 정규식 활용

**코틀린에서 여러 줄에 걸친 문자열을 작성해야 할 때**

→ 큰따옴표 세 개(`”””`)를 사용하면 편하다.

```kotlin
fun main(args: Array<String>) {
    val str = """
        안녕하세요
        저는 dev.hardy입니다.
        반갑습니다.
    """.trimIndent() // Indentation 제거
    print(str)
}
```

**특정 문자 가져오기**

```kotlin
fun main(args: Array<String>) {
    val name = "hardy"
    println(name[0]) // h
    println(name[1]) // a
    println(name[2]) // r
    println(name[3]) // d
    println(name[4]) // y
}
```

→ `문자열[index]` 로 접근한다.

Reference:  
[자바 개발자를 위한 코틀린 입문](https://www.inflearn.com/course/java-to-kotlin/dashboard)
