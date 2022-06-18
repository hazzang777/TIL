# 코틀린의 scope function

인프런에서 `최태현 지식 공유자`님의   
`자바 개발자를 위한 코틀린 입문`을 수강하며  정리한 내용 입니다.  
하단에 강의 링크 있습니다.

## scope function이란?

`scope function` : 일시적인 영역을 형성하는 함수

```kotlin
fun printPerson(person: Person?) {
    if (person != null) {
        println(person.name)
        println(person.age)
    }
}
```

→ 이 코드를 `일시적인 영역` 과 함께 리팩토링!

```kotlin
fun printPerson(person: Person?) {
    person?.let { 
        println(it.name)
        println(it.age)
    }
}
```

- Safe Call 사용: null이 아닐때 let을 호출
- `let` 은 scope function의 한 종류
    - 확장함수. 람다를 받아, 람다 결과를 반환한다.
- scope function: `람다` 를 사용해 일시적인 영역을 만들고 코드를 더 간결하게 만들거나, `method chaining`에 활용하는 함수

## scope function의 분류

1. `let`
    1. 람다의 결과 반환, it 사용
2. `run`
    1. 람다의 결과 반환, this 사용
3. `also`
    1. 객체 그 자체 반환, it 사용
4. `apply`
    1. 객체 그 자체 반환, this 사용
5. `with`
    1. this를 사용해 접근하고, this는 생략 가능, 확장함수가 아니다.

`this` : 생략이 가능한 대신, 다른 이름을 붙일 수 없다.

`it`: 생략이 불가능한 대신, 다른 이름을 붙일 수 있다. 

<aside>
💡 확장함수에서는 본인 자신을 this로 호출하고, 생략할 수 있다.

</aside>

run과 apply에서는 확장 함수를 받고 있기 때문에 this를 사용하는 것이다.

## 언제 어떤 scope function을 사용해야 할까?

- let
    - 하나 이상의 함수를 call chain 결과로 호출 할 때
    - (1) non-null 값에 대해서만 code block을 실행시킬 때 → 자주 사용 된다.
    - (2) 일회성으로 제한된 영역에 지역 변수를 만들 때 → 잘은 사용하지 않는다.

```kotlin
// (1)
val length = str?.let{
		it.length
}

// (2)
val numbers = listOf("one", "two")
val modifiedFirstItem = numbers.first()
	.let { firstItem ->
	if(firstItem.length >= 5) firstItem else "$firstItem!!!"
}.uppercase()
```

- run
    - 객체 초기화와 반환 값의 계산을 동시에 해야 할 때
    - 객체를 만들어 DB에 바로 저장하고, 그 인스턴스를 활용할 때

```kotlin
val person = Person("hardy", 100).run(personRepository::save)
```

→ 개인적으로 잘 사용하지는 않는다.

반복되는 생성 후처리는 생성자, 프로퍼티, init block으로 넣는 것이 좋다.

- apply
    - 객체 그 자체가 반환
    - 객체를 설정을 할 때에 객체를 수정하는 로직이 call chain 중간에 필요할 때
        - Test Fixture를 만들 때

```kotlin
fun createPerson(
	name: String,
	age: Int,
	hobby: String,
): Person {
	return Person(
		name = name,
		age = age,
	).apply {
		this.hobby = hobby
	}
}
```

- also
    - 객체 그 자체가 반환
    - 객체를 수정하는 로직이 call chain 중간에 필요할 때

```kotlin
mutableListOf(1,2,3,4)
	.also {println("4추가 전 지금 상태: $it")}
	.add(4)
```

- with
    - 특정 객체를 다른 객체로 변환해야 하는데, 모듈 간의 의존성에 의해 정적 팩토리 혹은 toClass 함수를 만들기 어려울 때

```kotlin
return with(person) {
	PersonDTO(
		name = name,
		age = age,
	)
}
```

this를 생략할 수 있어 필드가 많아도 코드가 간결해진다.

## scope function과 가독성

🧱 scope function을 사용한 코드가 그렇지 않은 코드보다 가독성 좋은 코드일까?

```kotlin
// 1번
if(person != null && person.isAdult){
		view.showPerson(person)
} else {
	view.showError()
}

// 2번
person?.takeIf{it.isAdult}
	?.let(view::showPerson)
	?: view.showError()
```

→ 1번을 좋은 코드라고 생각한다.

- 2번은 숙련된 코틀린 개발자만 더 알아보기 쉽거나 숙련된 코틀린 개발자도 잘 이해하지 못할 수 있다.
- 1번이 디버깅이 쉽다.
- 1번이 수정이 쉽다.

🤩 **사용 빈도가 적은 관용구는 코드를 더 복잡하게 만들고 이런 관용구들을 한 문장 내에서 조합해 사용하면 복잡성이 훨씬 증가한다.**

`하지만` **scope function을 사용하면 안되는 것도 아니다! 적절한 convention을 적용하면 유용하게 활용할 수 있음**

Reference:  
[자바 개발자를 위한 코틀린 입문](https://www.inflearn.com/course/java-to-kotlin/dashboard)