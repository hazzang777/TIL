# main() 메소드를 이용한 테스트의 문제점

테스트와 리팩토링은 개발자가 갖추어야 할 중요 역량

## main() 메소드를 이용한 테스트의 문제점

```java
public class Calculator {
    public int add(int i, int j) {
        return i + j;
    }

    public int subtract(int i, int j) {
        return i - j;
    }

    public int multiply(int i, int j) {
        return i * j;
    }

    public int divide(int i, int j) {
        return i / j;
    }

    public static void main(String[] args) {
        Calculator cal = new Calculator();
        System.out.println(cal.add(5,3));
        System.out.println(cal.subtract(5,3));
        System.out.println(cal.multiply(5,3));
        System.out.println(cal.divide(5,3));
    }
}
```

- 계산기 코드는 실제로 서비스를 담당하는 `프로덕션 코드`와 프로덕션 코드의 적절한 수행을 확인하려는 `main() 메소드`가 있다.

첫번째 문제점

- 프로덕션 코드와 테스트 코드가 같은 클래스에 위치
- 테스트 코드의 경우 테스트 단계에서만 필요하기 때문에 굳이 서비스하는 시점에 같이 배포할 필요 X

- 해결
    - 프로덕션 코드와 테스트 코드를 분리한다.
    
    - 프로덕션 코드
    
    ```java
    public class Calculator {
        public int add(int i, int j) {
            return i + j;
        }
    
        public int subtract(int i, int j) {
            return i - j;
        }
    
        public int multiply(int i, int j) {
            return i * j;
        }
    
        public int divide(int i, int j) {
            return i / j;
        }
    }
    ```
    
    - 테스트 코드
    
    ```java
    public class CalculatorTest {
        public static void main(String[] args) {
            Calculator calculator = new Calculator();
            add(calculator);
            subtract(calculator);
            multiply(calculator);
            divide(calculator);
        }
    
        private static void add(Calculator calculator) {
            System.out.println(calculator.add(3,4));
        }
    
        private static void subtract(Calculator calculator) {
            System.out.println(calculator.subtract(5,4));
        }
    
        private static void multiply(Calculator calculator) {
            System.out.println(calculator.multiply(8,4));
        }
    
        private static void divide(Calculator calculator) {
            System.out.println(calculator.divide(4,4));
        }
    }
    ```
    
    - 문제점
        - 테스트를 담당하는 별도의 클래스를 추가했지만 main() 메소드 하나에서 프로덕션 코드의 여러 메소드를 동시에 테스트 하고 있다.
        - 이는 프로덕션의 코드가 증가할수록, main()메소드의 복잡도도 증가하고, main()를 유지하는데 부담이 된다.
        - 문제를 해결하기 위해서는 → 테스트 코드를 각 메소드별로 분리할 수 도 있다.

- 이 또한 최종적인 해결책이 될 수 없음
    - 우리는 프로그래밍을 할 때 한 번에 메소드 하나의 구현에 집중한다.
    - 클래스가 가지고 있는 모든 메소드에 관심이 있는 것이 아니라 `현재 내가 구현하고 있는 메소드에만 집중하고 싶음`
    - 하지만 위 테스트 코드는 Calculaotr 클래스가 가지고 있는 모든 메소드를 테스트 할 수 밖에 없다.

- main() 메소드를 활용한 테스트가 안고 있는 다른 문제점
    - 테스트 결과를 `매번 콘솔에 출력되는 값을 통해 수동`으로 확인해야 한다는 것
    - 위와 같이 로직이 간단한 경우에는 결과 값을 쉽게 예측 가능
    - 로직의 복잡도가 높은 경우, 구현을 완료한 후 한 달이 지난 시점이라고 생각 할 시
        - 프로덕션 코드의 복잡한 로직을 머릿속으로 계산해 결과 값이 정상적으로 출력되는지 일일이 확인해야 함
    - 이를 해결하기 위해 `JUnit` 이다.
        - 내가 관심을 가지는 메소드에 대한 테스트만 가능함
        - 로직을 실행한 후의 결과 값 확인을 프로그래밍을 통해 자동화하는 것이 가능함

- JUnit을 활용한 main() 메소드 문제점 극복
    - 단위 테스트 프레임워크 중 하나
- Junit을 이용한 단위 테스트 코드

```java
package study;

import calculator.Calculator;
import org.junit.jupiter.api.*;

import static org.junit.jupiter.api.Assertions.*;

public class CalculatorTest {
    private Calculator cal;

    @BeforeEach
    public void setUp() {
        cal = new Calculator();
        System.out.println("before");
    }

    @Test
    public void add() {
        assertEquals(9, cal.add(6,3));
        System.out.println("add");
    }

    @Test
    public void subtract() {
        assertEquals(5, cal.subtract(10,5));
        System.out.println("subtract");
    }

    @AfterEach
    public void teardown() {
        System.out.println("teardown");
    }
}
```

### 한 번에 메소드 하나에만 집중

`@Test` 애노테이션을 추가한 후 add() 메소드에 대해 실행할 수 있다.

위와 같이 JUnit 기반으로 테스트 코드를 구현하면 CalculatorTest 클래스가 가지는

전체 메소드를 한번에 실행할 수도 있으며, add(), subtract() 메소드 각각을 실행할 수도 있다.

프로덕션 코드의 메소드만 실행해 볼 수 있다.

<aside>
💡 즉, 다른 메소드에 영향을 받지 않기 때문에 내가 현재 구현하고 있는 프로덕션 코드에 집중할 수 있는 효과를 얻을 수 있음

</aside>

### 결과 값을 눈이 아닌 프로그램을 통해 자동화

main() 메소드의 두 번째 문제점은 실행 결과를 눈으로 직접 확인해야 한다는 것

JUnit은 이 같은 문제점을 극복하기 위해 `assertEquals()` 메소드를 제공한다.

- assertEquals() 메소드는 첫 번째 인자는 기대하는 결과 값이고, → expected
- 두 번째 인자는 프로덕션 코드의 메소드를 실행한 결과 값이다. → actual
- 이 외에도
    - true/false 인지를 확인할 수 있는 assertTrue(), assertFalse()
    - 결과 값이 null 유무를 판단할 수 있는 assertNull(), assertNotNull()
    - 배열 값이 같은지를 검증하는 assertArrayEquals()

  

### 2.2.3 테스트 코드 중복 제거

개발자가 가져야 할 좋은 습관 중의 하나는 중복 코드를 제거하는 것이다.

→ 현재 코드에서는 Calculator 인스턴스를 생성하는 부분이다.

```java
private Calculator cal = new Calculator();
```

이렇게 할 수도 있지만 JUnit은 테스트를 실행하기 위한 초기화 작업을 위와 같이 구현하는 것을 추천 X

JUnit은 위 구현을 @BeforeEach 애노테이션을 활용해 다음과 같이 구현 가능

Calculator 인스턴스를 매 테스트마다 생성하는 이유는 add() 테스트 메소드를 실행할 때 영향을 미칠 수 있기 때문

이와 같이 테스트 메소드 간에 영향을 미칠 경우 테스트 실행 순서나 Calculator 상태 값에 따라 테스트가 성공하거나 실패할 수 있다.

`@AfterEach` 애노테이션은 메소드 실행이 끝난 후 실행됨으로써 후처리 작업을 담당

![Untitled](https://user-images.githubusercontent.com/68279162/171431854-ddf8bc3f-4ff3-4067-9f33-5f3ae70c6034.png)

- 각 테스트 메소드가 실행될 때 매번 @BeforeEach, @AfterEach 애노테이션으로 설정한 메소드가 실행되면서 초기화와 후처리 작업을 하는 것을 확인할 수 있음
- 이와 같이 매번 초기화, 후처리 작업을 통해 각 테스트 간에 영향을 미치지 않으면서 독립적인 실행이 가능하도록 지원

REFERENCE:  
[자바 웹 프로그래밍 Next Step](http://www.yes24.com/Product/Goods/31869154)