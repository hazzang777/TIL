# Optional
백기선님 `더 자바 로드맵`을 수강하며 정리한 내용 입니다.  
글 밑에 참조에 강의 링크 있습니다.
## Optional 소개

<aside>
💡 자바 프로그래밍에서 NullPointerException 보는 이유

</aside>

→ `null을 리턴`하기 때문 / `null 체크`를 안했기 때문

<aside>
💡 메소드에서 작업 중 특별한 상황에서 값을 제대로 리턴할 수 없는 경우

</aside>

→ 예외를 던진다.

- 비용이 크게 든다. 스택트레이스를 찍기때문

→ null을 리턴한다.

- 비용 문제는 없다. 하지만 그 코드를 사용하는 클라이언트 코드는 주의해야함

→ Java 8 이후 : Optional을 리턴한다.

- 클라이언트 코드에게 명시적으로 빈값 일 수도 있는 것을 알려주고, 빈 값인 경우에는 처리를 강제시킨다.
- `리턴타입으로만 사용하는 것이 권장사항`
- Optional.of(T) 같은 경우에는 무조건 값이 있어야 한다. → 없으면 NullPointerException 발생
- Optional.ofNullable(T) T가 null인 것을 허용한다.

<aside>
💡 Optional

</aside>

- 값이 한 개가 있을 수도 있고 없을 수도 있는 박스

<aside>
💡 주의할 점

</aside>

1. 리턴값으로만 권장
    1. 권장하지 않는 경우
        1. 메소드 매개변수
            1. 더 번거로워 진다.
        2. 맵의 키 타입
            1. `맵의 키는 절대 null이 될 수 없다.`
        3. 인스턴스 필드 타입
2. Optional을 리턴하는 메소드에서 null을 리턴하지말자
    1. `Optional.empty()`를 반환하자

```java
public Optional<Progress> getProgress() {
    return Optional.empty();
}
```

1. 기본형 Optional은 따로 있다. → OptionalInt,OptionalLong….

```java
Optional.of(10); // 박싱 언박싱이 벌어진다. -> 성능이 안좋아진다.
OptionalInt.of(10); // 권장
```

1. Collection, Map, Stream Array, Optional은 Optional로 감싸지 말자.
    1. 컨테이너 성격들을 Optional을 감싸지 말자.
    2. 이 자체로 비어있는지 확인할 수 있는 컨테이너 성격의 인스턴스들이다.
    3. Optional로 감싼다면 두번 감싸는 것이다. 감싸지 않아야 한다.

## Optional API

- Optional 만들기
    - Optional.of()
    - Optional.ofNullable()
    - Optional.empty()
- Optional에 값이 있는지 없는지 확인
    - isPresent()
    - isEmpty() : Java 11 부터 제공

```java
public class App {
    public static void main(String[] args) {
        List<OnlineClass> springClasses = new ArrayList<>();
        springClasses.add(new OnlineClass(1, "spring boot", true));
        springClasses.add(new OnlineClass(2, "spring data jpa", true));
        springClasses.add(new OnlineClass(3, "spring mvc", false));
        springClasses.add(new OnlineClass(4, "spring core", false));
        springClasses.add(new OnlineClass(5, "rest api development", false));

        Optional<OnlineClass> spring = springClasses.stream()
                .filter(oc -> oc.getTitle().startsWith("jpa"))
                .findFirst();

        boolean present = spring.isEmpty(); // 비어있는지 확인
        System.out.println(present); // true
    }
}
```

- Optional에 있는 값 가져오기
    - get()
    - 만약에 비어있다면?

→ RuntimeException 계열의 `NoSuchElementException` 발생!

- Optional에 값이 있는 경우에 그 값을 가지고 ~~를 하라
    - ifPresent(Consumer)

```java
public class App {
    public static void main(String[] args) {
        List<OnlineClass> springClasses = new ArrayList<>();
        springClasses.add(new OnlineClass(1, "spring boot", true));
        springClasses.add(new OnlineClass(2, "spring data jpa", true));
        springClasses.add(new OnlineClass(3, "spring mvc", false));
        springClasses.add(new OnlineClass(4, "spring core", false));
        springClasses.add(new OnlineClass(5, "rest api development", false));

        Optional<OnlineClass> spring = springClasses.stream()
                .filter(oc -> oc.getTitle().startsWith("spring"))
                .findFirst();

        spring.ifPresent(oc -> {
            System.out.println(oc.getTitle()); // spring boot
        });
    }
}
```

- Optional에 값이 있으면 가져오고 없는 경우에 ~~를 리턴하라
    - orElse(T)
    - 인스턴스가 들어간다.
    - 이미 상수로 만들어져있는 인스턴스를 작업할때는 이것이 적합
    - 해당되는게 있더라도 무조건 파라미터에 있는 행위는 실행 된다.
        - 이것이 싫다면 orElseGet(Supplier)

```java
public class App {
    public static void main(String[] args) {
        List<OnlineClass> springClasses = new ArrayList<>();
        springClasses.add(new OnlineClass(1, "spring boot", true));
        springClasses.add(new OnlineClass(2, "spring data jpa", true));
        springClasses.add(new OnlineClass(3, "spring mvc", false));
        springClasses.add(new OnlineClass(4, "spring core", false));
        springClasses.add(new OnlineClass(5, "rest api development", false));

        Optional<OnlineClass> spring = springClasses.stream()
                .filter(oc -> oc.getTitle().startsWith("jpa"))
                .findFirst();

        OnlineClass onlineClass = spring.orElse(createNewClass());
        System.out.println(onlineClass.getTitle());
    }

    private static OnlineClass createNewClass() {
        return new OnlineClass(10, "jpa class", false);
    }
}
```

- Optional에 값이 있으면 가져오고 없는 경우에 ~~를 하라
    - orElseGet(Supplier)
    - orElse(T)와 달리 람다가 들어가기 때문에 Lazy하게 다룰 수 있다.
    - 동적으로 작업해서 만들어야 할때는 이것이 적합
    - 반환 값이 있다면 람다는 실행되지 않는다.

```java
public class App {
    public static void main(String[] args) {
        List<OnlineClass> springClasses = new ArrayList<>();
        springClasses.add(new OnlineClass(1, "spring boot", true));
        springClasses.add(new OnlineClass(2, "spring data jpa", true));
        springClasses.add(new OnlineClass(3, "spring mvc", false));
        springClasses.add(new OnlineClass(4, "spring core", false));
        springClasses.add(new OnlineClass(5, "rest api development", false));

        Optional<OnlineClass> spring = springClasses.stream()
                .filter(oc -> oc.getTitle().startsWith("spring"))
                .findFirst();

        OnlineClass onlineClass = spring.orElseGet(App::createNewClass);
        System.out.println(onlineClass.getTitle());
    }

    private static OnlineClass createNewClass() {
        System.out.println("create new class");
        return new OnlineClass(10, "jpa class", false);
    }
}
```

- Optional에 값이 있으면 가져오고 없는 경우에 에러를 던져라
    - orElseThrow()
    - 기본적으로는 NoSuchElementException을 던짐
    - 원하는 에러가 있다면 orElseThrow(Supplier)로 제공해주면 된다.

```java
public class App {
    public static void main(String[] args) {
        List<OnlineClass> springClasses = new ArrayList<>();
        springClasses.add(new OnlineClass(1, "spring boot", true));
        springClasses.add(new OnlineClass(2, "spring data jpa", true));
        springClasses.add(new OnlineClass(3, "spring mvc", false));
        springClasses.add(new OnlineClass(4, "spring core", false));
        springClasses.add(new OnlineClass(5, "rest api development", false));

        Optional<OnlineClass> spring = springClasses.stream()
                .filter(oc -> oc.getTitle().startsWith("jpa"))
                .findFirst();

        OnlineClass onlineClass = spring.orElseThrow(IllegalArgumentException::new);
        System.out.println(onlineClass.getTitle());

    }

    private static OnlineClass createNewClass() {
        System.out.println("create new class");
        return new OnlineClass(10, "jpa class", false);
    }
}
```

- Optional에 들어있는 값 걸러내기
    - Optional filter(Predicate)
    - filter조건에 충족하지 않으면 Optional.empty() 반환

```java
public class App {
    public static void main(String[] args) {
        List<OnlineClass> springClasses = new ArrayList<>();
        springClasses.add(new OnlineClass(1, "spring boot", true));
        springClasses.add(new OnlineClass(2, "spring data jpa", true));
        springClasses.add(new OnlineClass(3, "spring mvc", false));
        springClasses.add(new OnlineClass(4, "spring core", false));
        springClasses.add(new OnlineClass(5, "rest api development", false));

        Optional<OnlineClass> spring = springClasses.stream()
                .filter(oc -> oc.getTitle().startsWith("spring"))
                .findFirst();

        Optional<OnlineClass> onlineClass = spring
                .filter(oc -> !oc.isClosed()); // 닫혀있는지 확인
        
        System.out.println(onlineClass.isEmpty()); // true

    }

    private static OnlineClass createNewClass() {
        System.out.println("create new class");
        return new OnlineClass(10, "jpa class", false);
    }
}
```

- Optional에 들어있는 값 변환하기
    - Optional map(Function)
    - Optional flatMap(Function): Optional 안에 들어있는 인스턴스가 Optional인 경우에 사용하면 편리
    - Function에서 처리할때 리턴하는 타입에 따라 Optional에 담기는 값 타입이 달라진다.

```java
public class App {
    public static void main(String[] args) {
        List<OnlineClass> springClasses = new ArrayList<>();
        springClasses.add(new OnlineClass(1, "spring boot", true));
        springClasses.add(new OnlineClass(2, "spring data jpa", true));
        springClasses.add(new OnlineClass(3, "spring mvc", false));
        springClasses.add(new OnlineClass(4, "spring core", false));
        springClasses.add(new OnlineClass(5, "rest api development", false));

        Optional<OnlineClass> spring = springClasses.stream()
                .filter(oc -> oc.getTitle().startsWith("spring"))
                .findFirst();

        Optional<Integer> integer = spring.map(OnlineClass::getId);
        System.out.println(integer.isPresent());
    }

    private static OnlineClass createNewClass() {
        System.out.println("create new class");
        return new OnlineClass(10, "jpa class", false);
    }
}
```

- flatMap() 사용 예시

```java
Optional<Optional<Progress>> progress = spring.map(OnlineClass::getProgress);
Optional<Progress> progress1 = spring.flatMap(OnlineClass::getProgress);
```

- Stream이 제공하는 flatMap과 다르다.
    - Stream에서 사용하는 것은 input은 하나지만 output은 여러개일수 있다.
- Optional에서의 flatMap은 input이 하나면 output도 하나!

Reference:
[더 자바, Java 8 : 백기선](https://www.inflearn.com/course/the-java-java8/dashboard)