# 코틀린에서 object 키워드를 다루는 방법

인프런에서 `최태현 지식 공유자`님의   
`자바 개발자를 위한 코틀린 입문`을 수강하며  정리한 내용 입니다.  
하단에 강의 링크 있습니다.

## static 함수와 변수

코틀린은 static이 라는게 없다. → `companion object` 

```kotlin
class Person private constructor(
    var name: String,
    var age: Int,
){
    companion object{
        const val MIN_AGE = 1 // (1)
        fun newBaby(name: String): Person{
            return Person(name, MIN_AGE)
        }
    }
}
```

1. `static`: 클래스가 인스턴스화 될 때 새로운 값이 복제되는게 아니라 정적으로 인스턴스끼리의 값을 공유
2. `companion object`:  클래스와 동행하는 유일한 오브젝트
    1. 하나의 객체로 간주된다.
    2. 이름을 붙일 수도 있다.
    3. 인터페이스를 구현할 수도 있다.

(1) 저기에 const를 안붙이게 되면 런타임시에 변수가 할당 된다. 그러나 const를 붙이면 컴파일 시에 변수가 할당 된다.

- `const` : 진짜 상수에 붙이기 위한 용도. 기본 타입과 String에 붙일 수 있음

```kotlin
interface Log{
	fun log()
}

class Person private constructor(
    var name: String,
    var age: Int,
){
    companion object Factory : Log{ // (1)
        private const val MIN_AGE = 1
        fun newBaby(name: String): Person{
            return Person(name, MIN_AGE)
        }

        override fun log() { // (2)
            println("구현할 수 있어용")
        }
    }
}
```

(1) companion object는 이름을 붙일 수도 있다.

(2) 인터페이스를 구현할 수도 있다.

→ 하나의 객체로 간주되기 때문이다.

<aside>
💡 companion object에 유틸성 함수를 넣어도 되지만, 최상단 파일을 활용하는 것이 좋다.

</aside>

자바에서 코틀린의 companion object를 사용하려면 `@JvmStatic`을 붙여야 한다.

## 싱글톤

싱글톤: 단 하나의 인스턴스만을 갖는 클래스

코틀린에서 싱글톤을 만드는 방법은 되게 간단하다.

→ `object` 만 붙여주면 된다.

```kotlin
object Singleton
```

- 싱글톤은 유일한 하나의 인스턴스이므로 바로 이름으로 접근한다.

```kotlin
fun main() {
    println(Singleton.a) // 0
}

object Singleton{
    var a: Int = 0
}
```

## 익명 클래스

익명클래스: 특정 인터페이스나 클래스를 상속받은 구현체를 일회성으로 사용할 때 쓰는 클래스

```kotlin
fun main() {
    moveSomething(object : Movable { // (1)
        override fun move() {
            print("움직임")
        }

        override fun fly() {
            println("날음")
        }
    })
}

private fun moveSomething(movable: Movable) {
    movable.move()
    movable.fly()
}

interface Movable {
    fun move()
    fun fly()
}
```

(1) 자바에서는 new 타입 이름() {} / 코틀린에서는 `object : 타입이름` 그리고 중괄호

Reference:  
[자바 개발자를 위한 코틀린 입문](https://www.inflearn.com/course/java-to-kotlin/dashboard)