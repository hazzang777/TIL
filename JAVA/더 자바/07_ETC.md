# ETC
백기선님 `더 자바 로드맵`을 수강하며 정리한 내용 입니다.  
글 밑에 참조에 강의 링크 있습니다.
## 애노테이션의 변화

애노테이션 관련 두가지 큰 변화

- 자바 8부터 애노테이션을 타입 선언부에도 사용 가능
    - 제네릭 타입
    - 변수 타입
    - 매개변수 타입
    - 예외 타입
    - …
- 자바 8부터 애노테이션을 중복해서 사용 가능

<aside>
💡 타입에 사용할 수 있으려면

</aside>

1. TYPE_PARAMETER: 타입 변수에만 사용 가능
2. TYPE_USE: 타입 변수를 포함해서 모든 타입 선언부에 사용 가능

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE_PARAMETER)
public @interface Hardy {

}

public class App {
    public static void main(String[] args) {

    }
    static class FeelsLikeHardy<@Hardy T> {
				public static <@Hardy C> void print(C c) {
            System.out.println(c);
        }
    }
}
```

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE_USE)
public @interface Hardy {

}

public class App {
    public static void main(String[] args) throws @Hardy RuntimeException {
        List<@Hardy String> names = Arrays.asList("HARDY");
    }
    static class FeelsLikeHardy<@Hardy T> {
        public static <@Hardy C> void print(C c) {
            System.out.println(c);
        }
    }
}
```

<aside>
💡 중복 사용할 수 있는 애노테이션 만들기

</aside>

→ 여러개의 애노테이션을 감싸고 있을 컨테이너 애노테이션 타입을 선언해야 한다.

→ `컨테이너 애노테이션은 중복 애노테이션과 @Retension 및 @Target이 같거나 더 넓어야 한다.` 

→ 자기가 감싸고 있을 어노테이션을 배열로 감싸고 있으면 된다.

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE_USE)
public @interface HardyContainer {
    Hardy[] value();
}

@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE_USE)
@Repeatable(HardyContainer.class)
public @interface Hardy {

}
```

- 컨테이너 애노테이션으로 중복 애노테이션 참조하기

```java
@Hardy("자바")
@Hardy("개발자")
@Hardy("백엔드")
public class App {
    public static void main(String[] args) {
        HardyContainer hardyContainer = App.class.getAnnotation(HardyContainer.class);
        Arrays.stream(hardyContainer.value()).forEach(System.out::println);
    }
}
```

## 배열 Parallel 정렬

Arrays.parallelSort()

- Fork/Join 프레임워크를 사용해서 배열을 병렬로 정렬한다.

병렬 정렬 알고리즘

- 배열을 둘로 계속 나눈다.
- 합치면서 정렬한다.

```java
public class App {
    public static void main(String[] args) {
        int size = 1500;
        int[] numbers = new int[size];
        Random random = new Random();
        IntStream.range(0, size).forEach(i -> numbers[i] = random.nextInt());

        long start = System.nanoTime();
        Arrays.sort(numbers);
        System.out.println("직렬 정렬 시간 " + (System.nanoTime() - start));

        IntStream.range(0, size).forEach(i -> numbers[i] = random.nextInt());
        start = System.nanoTime();
        Arrays.parallelSort(numbers);
        System.out.println("병렬 정렬 시간 " + (System.nanoTime() - start));
    }
}
```

- 직렬 정렬 시간 411875
- 병렬 정렬 시간 300834
- 효율성은 같다

## Metaspace

JVM의 여러 메모리 영역중에 PermGen 메모리 영역은 없어짐

→ Metaspace 영역이 생김

- Permgen
    - 클래스 메타데이터를 담는 곳
    - `Heap 영역에 속함`
    - 제한된 크기를 가짐
    - -XX:PermSize=N 초기 사이즈 설정
    - -XX:MaxPermSize=N 최대 사이즈 설정
- Metaspace
    - 클래스 메타데이터를 담는 곳
    - Heap 영역이 아니라, `Native 메모리 영역!`
    - `기본값으로 제한된 크기를 가지고 있지 않음`
        - 필요한 만큼 늘어난다.
    - 자바 8부터는 Permgen 관련 옵션 무시
    - -XX:MetaspaceSize=N 초기 사이즈 설정
    - -XX:MaxMetaspaceSize=N 최대 사이즈 설정 (중요!)

Reference:
[더 자바, Java 8 : 백기선](https://www.inflearn.com/course/the-java-java8/dashboard)