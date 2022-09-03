# 빈 순환참조, autowired, 생성자 주입 동작원리

## INTRO
</br>
우선, 많은 자료와 Reference를 보면 생성자 주입을 권장하고 있다.

흔히 불리는 DI 방식 중에서

setter를 이용한 `수정자 주입` /생성자를 이용한 `생성자 주입` 방식들이 있다. 

왜 생성자 주입을 권장하는지 서로 차이점을 알아보며 확인해보자.

## 수정자 주입 방식

스프링을 이용하지 않고 예시를 들어보겠다.

Service.class

```java
public interface Service {
    void doingProcess();
}
```

ServiceImpl.class

```java
public class ServiceImpl implements Service{
    @Override
    public void doingProcess() {
        System.out.println("doing Process!!!");
    }
}
```

Controller.class

```java
public class Controller {

    private Service service;

    public void setService(Service service) {
        this.service = service;
    }

    public void doingByService() {
        service.doingProcess();
    }
}
```

DITest.class

```java
public class DITest {
    public static void main(String[] args) {
        Controller controller = new Controller();

        controller.setService(new ServiceImpl()); // 수정자를 이용한 생성자 주입

        controller.doingByService(); // doing Process!!! 출력
    }
```

Controller는 수정자를 통해 Service의 구현체를 주입받는다.

수정자 주입을 이용한 방식은 낮은 결합도를 가지게 된다.

문제없이 잘 되는 것 같지만 여기서는 몇가지의 문제점이 있다.

![Untitled](https://user-images.githubusercontent.com/68279162/188271235-e8fab371-5724-4c91-8c4b-1e7526d80c92.png)

만약 Controller에 Service를 주입하지 않아도 문제없이 Controller 객체는 생성이 가능하다.

그리고 Controller에 있는 `doingByService` 메소드 호출도 가능하게 된다.

하지만 doingByService 메소드는 Service의 doingProcess 메소드를 호출하므로 Service가 주입되지 않았기 때문에

***`NullPointerException이 발생하게 된다!`***

그리고 `Controller를 정상적으로 동작시키기 위해 필요한 객체들이 주입되지 않아도 얼마든지 객체를 생성할 수 있는 것이 문제!`

생성자 주입은 이러한 문제들을 해결할 수 있다.

## 생성자 주입 방식

우선 생성자 주입 방식으로 Controller 코드를 변경해보자.

Controller.class

```java
public class Controller {

    private final Service service; // final 가능

    public Controller(Service service) {
        this.service = service;
    }

    public void doingByService() {
        service.doingProcess();
    }
}
```

DITest.class

```java
public class DITest {
    public static void main(String[] args) {
        Controller controller = new Controller(new ServiceImpl()); // 생성자를 이용한 주입 방식
        controller.doingByService(); // doing Process!!! 출력
    }
}
```

생성자 주입 방식을 이용한다면

![Untitled 1](https://user-images.githubusercontent.com/68279162/188271239-f85ed424-6662-4167-af0a-a7216a96af6c.png)

1. 확인할 수 있듯이 Controller에 필요한 객체를 주입하지 않는다면 객체를 생성할 수 없는 것을 확인할 수 있다. (컴파일 타입에서 오류 체크)
2. 그리고 NullPointerException이 발생하지 않는다.
3. final 사용가능하다. 
    
    final을 사용하면 좋은점은 필요한 객체를 변경할 수 없이 `Immutable` 하기 때문에 안전하다.
    

## Spring에서의 DI 주입 방식

스프링에서는 수정자 주입, 생성자 주입 그리고 추가적으로 필드 주입이 있다.

필드 주입은 수정자 방식과 유사하 방식으로 동작하기 때문에 수정자 주입방식의 단점을 그대로 가지게 된다.

각각 방식마다 어떻게 주입하는지 알아보자.

1. 스프링에서의 생성자 주입 방식

```java
@Service
public class OrderServiceImpl implements OrderService{
    
    private final PayService payService;

    @Autowired
    public OrderServiceImpl(PayService payService) {
        this.payService = payService;
    }

    @Override
    public void order() {
        payService.pay();
    }
}
```

1. 스프링에서의 수정자 주입 방식

```java
@Service
public class OrderServiceImpl implements OrderService{

    private PayService payService;

    @Autowired
    public void setPayService(PayService payService) {
        this.payService = payService;
    }

    @Override
    public void order() {
        payService.pay();
    }
}
```

1. 스프링에서의 필드 주입 방식

```java
@Service
public class OrderServiceImpl implements OrderService{

    @Autowired
    private PayService payService;

    @Override
    public void order() {
        payService.pay();
    }
}
```

## Spring 에서 순환참조 문제

먼저 Bean이 A, B, C 3개가 있고 

Bean A → Bean B → Bean C

A가 B를 의존하고 있고, B가 C를 의존하고 있는 순환 종속성이 있지 않는 의존관계라고 

가정해보자

이런 경우에 Spring은 먼저 C를 생성한다음 B를 생성하고 B에 C를 주입한다.

그리고 A를 생성한다음 B를 주입하게 된다.

그러나 순환 종속성이 있는 경우

Bean A → Bean B / Bean B → Bean A

A도 B를 의존하고 있고, B도 A를 의존하고 있기 때문에

Spring은 먼저 어떤 빈을 생성해야 할지 결정할 수 없기 때문에 

Spring Context를 구성하는 동안에 `BeanCurrentlyInCreationException` 발생시키게 됨.

여기서 Spring에서는 생성자 주입을 사용한다면 

이러한 스프링 컨텍스트 로딩 시점에서  순환 참조 방식을 막을 수 있다.

먼저 순환 참조가 일어나는 구조를 만들어 보자.

PayService.class

```java
public interface PayService {
    void pay();
}
```

PayServiceImpl.class

```java
@Service
public class PayServiceImpl implements PayService{

    private final OrderService orderService;

    @Autowired
    public PayServiceImpl(OrderService orderService) {
        this.orderService = orderService;
    }

    @Override
    public void pay() {
        orderService.order();
    }
}
```

OrderService.class

```java
public interface OrderService {
    void order();
}
```

OrderServiceImpl.class

```java
@Service
public class OrderServiceImpl implements OrderService{

    private final PayService payService;

    public OrderServiceImpl(PayService payService) {
        this.payService = payService;
    }

    @Override
    public void order() {
        payService.pay();
    }
}
```

PayService와 OrderService가 서로 참조하는 구조이다. 

여기서 실행을 해보자.

![Untitled 2](https://user-images.githubusercontent.com/68279162/188271254-75c9e755-ca07-4933-8669-27a5076d47bd.png)

**스프링 컨테이너가 빈을 생성하는 시점에 사이클 관계가 생기기 때문에 순환 참조를 캐치하게 된다.**

😲 그렇다면 필드 주입에서는 어떻게 될까?

OrderServiceImpl.class

```java
@Service
public class OrderServiceImpl implements OrderService{

    @Autowired
    private PayService payService;

    @Override
    public void order() {
        payService.pay();
    }
}
```

PayServiceImpl.class

```java
@Service
public class PayServiceImpl implements PayService{

    @Autowired
    private OrderService orderService;

    @Override
    public void pay() {
        orderService.order();
    }
}
```

![Untitled 3](https://user-images.githubusercontent.com/68279162/188271255-54dd6d63-9006-40f1-bb8d-c73e5f74f25d.png)

문제 없이 실행되는 것을 알 수 있다!

😗 **그렇다면 왜 문제없이 빈 생성이 잘되는 것일까?**

먼저 생성자 주입은 `객체 생성 시점에서 순환 참조`가 일어나는 것이기 때문에 빈 생성이 제대로 이루어지지 않는다. 하지만 수정자 주입이나 필드 주입은 해당 `빈이 필요한 시점에 주입`되기 때문에 생성 시점이 아닌 `비즈니스 로직 상에서 순환참조`가 일어나는 것이기 때문에 빈 생성이 잘 된다. 그래서 `수정자 주입이나 필드 주입에서는 순환참조를 알 수 있는 방법이 없다.`

그래서 수정자 주입이나 필드 주입 경우에는 해당 순환참조를 가지는 부분에서 호출이 되는 경우 StackOverflowError를 에러를 발생시킨다. `즉, 에러를 발생시킬 수 밖에 없는 로직을 가지고 애플리케이션이 실행되는 것이기 때문에 매우 위험하다.`

![Untitled 4](https://user-images.githubusercontent.com/68279162/188271257-1e67a9ac-e13d-4b95-9025-4b9f62ba8c34.png)


**Spring Boot 2.7.3 기준**

최신 스프링 부트 버전으로 필드 주입시에 순환 참조 가지는 경우에서 스프링을 실행시켜 본 결과

![Untitled 5](https://user-images.githubusercontent.com/68279162/188271258-0bac8ae4-6fb5-4831-8af8-9085947b6749.png)

🤔 생성자 주입 방식과 마찬가지로 빈 생성 시점에서 순환 참조를 캐치하는 것을 볼 수 있다. (추후에 자세히 알아보겠다.)

하지만 실무에서는 스프링 부트 버전을 전부다 최신을 사용하는 것도 아니고 막상 버전 올리는 일이 쉬운일이 아니니 `생성자 주입 방식` 을 권장한다.

그리고 최신 버전을 사용한다하더라도 여러가지 장점이 있으니 `생성자 주입 방식`을 사용하는 것을 추천한다.

</br>

*****참조*****
1. [매우 도움된 블로그 글](https://yaboong.github.io/spring/2019/08/29/why-field-injection-is-bad/)
2. [Spring의 순환 참조 - baeldug](https://www.baeldung.com/circular-dependencies-in-spring)