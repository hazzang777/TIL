# 열거형(enums)

## 열거형이란?

- 서로 관련된 상수를 편리하게 선언하기 위한 것
- 열거형이 갖는 값뿐만 아니라 타입도 관리 할 수 있다.

```java
class Chicken {
	....
	....
	final int kind;
	final int num;
}

// enum 사용 한다면
class Chicken {
	...
	...
	enum Kind {후라이드, 양념, 간장, 뿌링클} // 열거형 Kind 정의
  enum Value {ONE, TWO, THREE, FOUR} // 열거형 Value 정의

	final Kind kind;
	final Value value;
}
```

- `타입에 안전한 열거형` 실제값이 같아도 타입이 다르면 컴파일 에러 발생
    - 값뿐만 아니라 타입까지 체크함

```java
if(Chicken.Kind.후라이드 == Chicken.Value.ONE) // 컴파일 에러. 값은 같지만 타입이 다름
```

→ 만약 상수의 값이 바뀌면, 해당 상수를 참조하는 모든 소스를 다시 컴파일 해야함

- `열거형 상수를 사용하면`, 기존의 소스를 다시 컴파일 하지 않아도 됨.

## 열거형의 정의와 사용

- 열거형을 정의하는 방법

```java
enum 열거형이름 {상수명1, 상수명2,...}
```

ex) 치킨 종류 → 후라이드, 양념, 간장, 뿌링클 열거형 정의 (ChickenType)

```java
enum ChickenType {후라이드, 양념, 간장, 뿌링클}
```

- 열거형 사용하는 방법
    - `열거형이름.상수명`  (클래스의 스태틱 변수 참조하는 것과 동일)

```java
ChickenType.후라이드
```

<aside>
💡 열거형 상수간의 비교에는 == 가 가능 → 빠른 성능 제공

</aside>

- 그대신 대소비교는 못하지만 `compareTo()` 는 사용 가능
    - 같으면 `0` , 왼쪽이 크면 `양수` , 오른쪽이 크면 `음수`
- switch 문에서도 사용 가능
    - 주의할점 - case 문에서 열거형의 이름은 적지 않고, `상수의 이름만 적어야 함`

```java
switch(chickenType) {
	case ChickenType.후라이드 <- X
	case 후라이드 <- (O)
	...
}
```

## 모든 열거형의 조상 - java.lang.Enum

- 열거형의 모든 상수 출력하는 법

```java
enum ChickenType {후라이드, 양념, 간장, 뿌링클}

ChickenType[] ctArr = ChickenType.values();
```

→ `values()` 는 열거형의 모든 상수를 배열에 담아 반환

- Enum 클래스에 정의된 메서드

| String name() | 열거형 상수의 이름을 문자열로 반환 |
| --- | --- |
| int ordinal() | 열거형 상수가 정의된 순서를 반환 (0부터 시작) |
| T valueOf(Class<T> enumType, String name) | 지정된 열거형에서 name과 일치하는 열거형 상수를 반환한다. |

```java
public class App{
    public static void main(String[] args) {
        ChickenType 양념 = ChickenType.양념;
        ChickenType 간장 = ChickenType.valueOf("간장");
        ChickenType 뿌링클 = Enum.valueOf(ChickenType.class, "뿌링클");

        System.out.println("양념 = " + 양념);
        System.out.println("간장 = " + 간장);
        System.out.println("뿌링클 = " + 뿌링클);

        System.out.println("===========================");
        ChickenType[] types = ChickenType.values();
        for (ChickenType type : types) {
            System.out.println(type.name() + " : " + type.ordinal());
        }
    }
}
```

## 열거형에 멤버 추가하기

- Enum 클래스에 정의된 ordinal()이 열거형 상수가 정의된 순서를 반환한다.
    - 하지만 이 값을 열거형 상수의 값으로 사용하는 것은 좋지않다.
    - 열거형 상수옆에 원하는 값을 괄호()와 함께 작성할 수 있다.

```java
public enum ChickenType {
    후라이드(1), 양념(2), 간장(3), 뿌링클(4); // 끝에 ; 추가해야함
    
    private final int value; // (1) 정수를 저장할 필드(인스턴스 변수)추가
    ChickenType(int value) { // (2) 생성자 추가
        this.value = value;
    }
    public int getValue(){
        return value;
    }
}
```

- 특이사항

(1) 반드시 final이어야 할 필요는 없다. value는 열거형 상수를 저장하기 위한 것이므로 final을 붙였을 뿐

(2) 열거형에 생성자가 있지만, 외부에서 열거형 객체를 생성할 수는 없다.

- 열거형의 생성자는 private이기 때문이다.

→ 그리고 하나의 열거형 상수에 여러 값 지정 가능

- 다만 그에 맞게 인스턴스 변수와 생성자에 추가해주어야 함

```java
public enum ChickenType {
    후라이드(1, "Fri"), 양념(2, "Sour"), 간장(3, "Soy"), 뿌링클(4, "Pow");

    private final int value;
    private final String symbol;
    ChickenType(int value, String symbol) {
        this.value = value;
        this.symbol = symbol;
    }
    public int getValue(){
        return value;
    }

    public String getSymbol() {
        return symbol;
    }
}
```

## 열거형에 추상 메서드 추가하기

- 후라이드, 양념, 간장, 뿌링클이라는 4가지 치킨을 제공하는 치킨집이 있는데 배달을 할때 배달 되는 거리에 따라 부과되는 금액이 치킨 종류에 따라 달라진다고 가정해보자.

```java
public enum ChickenType {
    후라이드(150), 양념(160), 간장(165), 뿌링클(180);

    private final int PRICE;
    ChickenType(int price) {
        PRICE = price;
    }
    int price(){ // 치킨 가격을 반환
        return PRICE;
    }
}
```

→ 배달 거리에 따라 치킨 금액을 계산하는 방식이 각 치킨 종류에 따라 달라지기 때문에 추상 메서드 `price(int distance)` 를 선언하면 각 열거형 상수가 이 추상 메서드를 반드시 구현해야함.

```java
public enum ChickenType {
    후라이드(150){
        @Override
        public int price(int distance) {
            return distance * PRICE;
        }
    },
    양념(160){
        @Override
        public int price(int distance) {
            return distance * PRICE * 2;
        }
    },
    간장(165){
        @Override
        public int price(int distance) {
            return distance * PRICE * 3;
        }
    },
    뿌링클(180){
        @Override
        public int price(int distance) {
            return distance * PRICE * 4;
        }
    };

    protected final int PRICE; // (1) protected로 해야 각 상수에서 접근 가능

    ChickenType(int price) {
        PRICE = price;
    }

    public abstract int price(int distance);// 배달 거리에 따른 치킨 금액 계산 추상 메서드

    public int getOriginalPrice(){
        return PRICE;
    }
}
```

(1) protected로 해야 각 상수에서 접근 가능

ex) 사용 예시

```java
public class App{
    public static void main(String[] args) {
        System.out.println("후라이드 가격 = " + ChickenType.후라이드.price(200));
        System.out.println("간장 가격 = " + ChickenType.간장.price(200));
    }
}

/*
후라이드 가격 = 30000
간장 가격 = 99000
*/
```

## 열거형의 이해

```java
enum ChickenType {후라이드, 양념, 간장, 뿌링클}
```

- 열거형 상수 하나하나가 ChickenType 객체이다.
- 이것을 클래스로 표현한다면

```java
class ChickenType {
	static final ChickenType 후라이드 = new ChickenType("후라이드");
	static final ChickenType 양념 = new ChickenType("양념");
	static final ChickenType 간장 = new ChickenType("간장");
	static final ChickenType 뿌링클 = new ChickenType("뿌링클");
	
	private String name;

	private ChickenType(String name) {
		this.name = name;
	}

}
```

- `ChickenType 클래스`의 static 상수 후라이드, 양념, 간장, 뿌링클의 값은 객체의 참조값이다.
    - 바뀌지 않기 때문에 `==` 로 비교가 가능!


Reference : 
 [자바의 정석](http://www.yes24.com/Product/Goods/24259565)