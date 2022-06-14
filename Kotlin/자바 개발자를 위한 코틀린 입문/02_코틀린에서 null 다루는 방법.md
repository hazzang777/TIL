# 코틀린에서 null을 다루는 방법

인프런에서 `최태현 지식 공유자`님의   
`자바 개발자를 위한 코틀린 입문`을 수강하며  정리한 내용 입니다.  
하단에 강의 링크 있습니다.

## Kotlin에서의 null 체크

```kotlin
fun startWithA(str: String?): Boolean? {
    if (str == null) {
        return null
    }
    return str.startsWith("A")
}

// str에는 null이 될 수 있다.
// 반환값도 null이 될 수 있다.
```

→ 코틀린에서는 `null이 가능한 타입`을 완전히 다르게 취급

→ 코틀린은 문맥상 위에서 null을 한 번 체크해주면 아래에서 null이 아니겠거니 컴파일러가 자동 추측해준다.

→ 코틀린에서는 null이 될수있는 값이 바로 함수를 호출 못하게끔 막는다.

```kotlin
fun func(str: String?){
	str.startsWith("A") // 컴파일 에러
}
```

<aside>
💡 코틀린에는 null이 가능한 타입만을 위한 기능은 있을까?

</aside>

---

## Safe Call과 Elvis 연산자

1. Safe Call
    1. null이 아니면 실행하고, null이면 실행하지 않음 (그대로 null)
    2. 즉, null이 아닌 경우에만 실행

```kotlin
val str: String? = "hardy"
str.length // X
str?.length // OK
```

1. Elvis 연산자
    1. 앞의 연산 결과가 null이면 뒤의 값을 사용
    2. 즉, null인 경우에만 호출

```kotlin
val name: String? = "hardy"
name?.length ?: 0 // name?.length가 null이면 0을 반환
```

위의 `startWithA()` 함수를 코틀린스럽게 코드를 바꿔보면

```kotlin
fun startWithA(str: String?): Boolean? {
    return str?.startsWith("A")
}
```

**Elvis연산은 early return에도 사용할 수 있음**

## 널 아님 단언!!

→ nullable type이지만, 아무리 생각해도 null이 될 수 없는 경우!

- `!!` 를 사용한다.

```kotlin
fun startWith(str: String?): Boolean{
    return str!!.startsWith("A")
}
```

→ null값이 만약들어온다면 `NPE` 발생!

- 정말 null이 아닌게 확실한 경우에만 `널 아님 단언!!` 을 사용해야 한다.

## 플랫폼 타입

<aside>
💡 코틀린에서 자바 코드를 가져다 사용할 때 어떻게 처리될까?

</aside>

예) Person : Java

```java
public class Person {
    private final String name;

    public Person(String name) {
        this.name = name;
    }

    @Nullable // null 허용
    public String getName() {
        return name;
    }
}
```

예) 사용 : kotlin

```kotlin
fun main(args: Array<String>) {
    val person = Person("hardy")
    print(startWith(person.name)) // 컴파일 에러 발생
}

fun startWith(name: String): Boolean {
    return name.startsWith("A")
}
```

`컴파일 에러가 나는 이유`는 코틀린의 startWith() 함수의 파라미터 `name`을 보면 `null이 들어올 수 없다`고 되있다. 그러나 자바 코드로 짠 `Person 클래스`를 보면 `@Nullable`이라고 null을 허용함을 알 수 있다. 즉 null이 될 수 있다고 어노테이션이 알려주고 있으므로 코틀린에서는 사용할 수 없다.

`즉, null과 관련된 어노테이션을 사용하면 코틀린에서 이것을 인식하고 이해해서 활용할 수 있다.`

**플랫폼 타입**

- 코틀린이 null 관련 정보를 알 수 없는 타입
- Runtime 시 Exception이 날 수 있음

<aside>
💡 코틀린에서 자바 코드를 사용할 때 플랫폼 타입에 사용에 유의해야 함

</aside>

Reference:  
[자바 개발자를 위한 코틀린 입문](https://www.inflearn.com/course/java-to-kotlin/dashboard)