# JVM 이해하기
백기선님 `더 자바 로드맵`을 수강하며 정리한 내용 입니다.  
글 밑에 참조에 강의 링크 있습니다.

## 자바, JVM, JDK 그리고 JRE

![Untitled](https://user-images.githubusercontent.com/68279162/172682925-b9d70612-58c7-456f-a86d-7df0b0ed18de.png)


### JVM

1. 자바 가상 머신으로 자바 바이트 코드(.class)를 OS에 특화된 코드로 변환
    1. 인터프리터와 JIT 컴파일러로 변환 → 실행
2. 바이트 코드를 실행하는 표준이자 구현체
    1. JVM 자체는 표준 / 특정 밴더가 JVM 구현체
3. 특정 플랫폼에 종속적

예) 바이트 코드

```java
Compiled from "HelloJava.java"
public class HelloJava {
  public HelloJava();
    Code:
       0: aload_0
       1: invokespecial #1                  // Method java/lang/Object."<init>":()V
       4: return

  public static void main(java.lang.String[]);
    Code:
       0: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;
       3: ldc           #3                  // String Hello, Java
       5: invokevirtual #4                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
       8: return
}
```

이러한 바이트 코드를 인터프리터와 JIT 컴파일러를 사용하여 네이티브 OS에 맞게 코드(기계어)로 변환후 실행

→ 특정 플랫폼에 종속적!

### JRE : JVM + 라이브러리

1. 자바 Application을 실행할 수 있도록 구성된 배포판
2. JVM과 핵심 라이브러리 및 자바 런타임 환경에서 사용하는 프로퍼티 세팅이나 리소스 파일을 가짐
3. 개발 관련 도구는 포함하지 않음 → 이것은 JDK에서 제공
4. 자바 컴파일러는 제공하지 않음 (javac)

### JDK : JRE + 개발 툴

1. 소스 코드를 작성할 때 사용하는 자바 언어는 플랫폼에 독립적.
2. 오라클은 자바 11부터 JDK만 제공한다.
3. `Write Once Run Anywhere` 

### 자바

1. 프로그래밍 언어
2. JDK에 들어있는 자바 컴파일러(javac)를 사용하여 바이트코드(.class)로 컴파일 할 수 있음

### JVM 언어

1. JVM 기반으로 동작하는 프로그래밍 언어
2. 자바, Kotlin, Scala, …

예) Hello.kt 파일 kotlinc로 컴파일 후 javap로 바이트 코드 확인

```java
> javap -c HelloKt.class
Compiled from "Hello.kt"
public final class HelloKt {
  public static final void main();
    Code:
       0: ldc           #8                  // String Hello, Kotlin
       2: getstatic     #14                 // Field java/lang/System.out:Ljava/io/PrintStream;
       5: swap
       6: invokevirtual #20                 // Method java/io/PrintStream.println:(Ljava/lang/Object;)V
       9: return

  public static void main(java.lang.String[]);
    Code:
       0: invokestatic  #23                 // Method main:()V
       3: return
}
```

---

## JVM 구조

![Untitled 1](https://user-images.githubusercontent.com/68279162/172682942-ad2c8a9f-568c-4987-9a1a-c9ab2213ecd4.png)


`JVM 구조` 는 크게 4 가지로 구성

- 클래스 로더 시스템
    - .class 에서 바이트코드를 읽고 메모리에 저장
    - 로딩: 클래스를 읽어오는 과정
    - 링크: 레퍼런스를 연결하는 과정
    - 초기화: static 값들 초기화 및 변수에 할당
- 메모리
    - 메서드 영역에서는 클래스 수준의 정보를 저장
        - 클래스 이름, 부모 클래스 이름, 메소드, 변수
        - 공유하는 자원들이다.
    - 힙 영역에는 객체를 저장, 공유 자원이다.
    - 스택 영역에는 쓰레드 마다 런타임 스택을 만들고, 그 안에 메소드 호출을 스택 프레임이라 부르는 블럭으로 쌓는다.
        - 쓰레드가 종료하면 런타임 스택도 사라진다.
    - PC 레지스터: 쓰레드 마다 쓰레드 내 현재 실행할 스택 프레임을 가리키는 포인터가 생성됨
    - 네이티브 메소드 스택: 이것도 쓰레드 마다 생성. 네이티브 메소드를 호출할 때 만드는 별도의 스택
        - 네이티브 메소드란?
            - 메소드 앞에 `native` 라는 키워드가 붙고 구현은 `C` 나 `C++` 로 한 것.
            - ex) Thread.currentThread();
    - `스택`, `PC`, `네이티브 메소드 스택`은 특정 쓰레드에 국한된 것.
    - `힙` , `메소드` 영역은 모든 쓰레드에서 사용하는 공유 자원
- 실행 엔진
    - 인터프리터: 바이트 코드를 한줄 씩 실행
    - JIT 컴파일러: 인터프리터 효율을 높이기 위해, 인터프리터가 반복되는 코드를 발견하면 JIT 컴파일러로 반복되는 코드를 모두 네이티브 코드로 바꿔둔다. 그 다음부터 인터프리터는 네이티브 코드로 컴파일된 코드를 바로 사용
    - `GC`(Garbage Collector): 더이상 참조되지 않는 객체를 모아서 정리
- 네이티브 메소드 인터페이스 + 네이티브 메소드 라이브러리
    - JNI (Java Native Interface)
        - 자바 Application에서 C, C++, 어셈블리로 작성된 함수를 사용할 수 있는 방법 제공
        - Native 키워드를 사용한 메소드 호출
    - 네이티브 메소드 라이브러리
        - C, C++로 작성된 라이브러리
    
    ---
    
    ## 클래스 로더
    
    ![Untitled 2](https://user-images.githubusercontent.com/68279162/172682956-1ca356a2-4dec-462e-979a-c7f8dbe10ff5.png)

    

### 클래스 로더

- 로딩 → 링크 → 초기화 순으로 진행
- 로딩
    - 클래스 로더가 `.class` 파일을 읽은 다음 적절한 바이너리 데이터를 만들고 `메소드` 영역에 저장
    - 메소드 영역에 저장하는 데이터
        - FQCN : 패키지 경로까지 포함한 클래스 이름
        - 클래스 | 인터페이스 | 이넘
        - 메소드와 변수
    - 로딩이 끝나면 해당 클래스 타입의 Class 객체를 생성하여 `힙` 영역에 저장

```java
public class App 
{
    public static void main( String[] args ) {
        ClassLoader classLoader = App.class.getClassLoader();
        System.out.println(classLoader); // (1)현재 App 클래스를 읽어온 클래스 로더 출력
        System.out.println(classLoader.getParent()); // (2)클래스로더의 부모 출력
        System.out.println(classLoader.getParent().getParent()); // (3)부모의 부모 출력
    }
}

/**
(1) jdk.internal.loader.ClassLoaders$AppClassLoader@3b192d32
(2) jdk.internal.loader.ClassLoaders$PlatformClassLoader@6bdf28bb
(3) null
*/
```

(3) `null이 나오는 이유`는 최상위 클래스로더인 `부트스트랩 클래스 로더`는 `네이티브 코드`로 구현되어 있기 때문에 자바 코드에서 참조해서 출력할 수가 없기에 null이 출력되게 된다.

- 링크
    - Verify → Prepare → Resolve 세 단계로 나눠짐
    - 검증: .class 파일 형식이 유효한지 체크
    - Preparation: 클래스 변수(static 변수)와 기본값에 필요한 메모리
        - 메모리를 준비하는 과정
    - Resolve: 심볼릭 메모리 레퍼런스를 메소드 영역에 있는 실제 레퍼런스로 교체
        - `선택적이다.` 이 단계에서 실제 Heap 영역에 있는 레퍼런스를 참조 할 수도 있지만 실제로 사용할때 참조하도록 교체할 수 도 있기에 선택적이다.
    - 초기화
        - static 변수의 값을 할당한다. (static 블럭이 있다면 이때 실행된다.)

- 클래스 로더는 계층 구조로 이뤄져 있고 기본적으로 세 가지 클래스 로더가 제공
    1. 부트스트랩 클래스 로더
        1. JAVA_HOME\lib에 있는 코어 자바 API를 제공
        2. 최상위 우선순위를 가짐
    2. 플랫폼 클래스 로더
        1. JAVA_HOME\lib\ext 폴더 또는 java.ext.dirs 시스템 변수에 해당하는 위치에 있는 클래스를 읽음
    3. 애플리케이션 클래스 로더
        1. 애플리케이션 classpath(애플리케이션 실행할 때 주는 -classpath 옵션 또는 java.class.path 환경 변수의 값에 해당하는 위치)에서 클래스를 읽는다.

Reference:
[더 자바, 코드를 조작하는 다양한 방법 : 백기선](https://www.inflearn.com/course/the-java-code-manipulation/dashboard)        