# Java 8 날짜 관련하여 작성해보자

## INTRO

</br>

개발을 하다보면 항상 날짜와 시간 관련해서 어떻게 할지 헷갈리는 경우가 많다.

일단 처음부터 결론을 말하자면 자바 8부터 나온 날짜와 시간관련 API를 사용하자!

자바8에서 날짜와 시간과 관련된 API가 나왔다.

</br>

- LocalDateTime
- ZonedDateTime
- Instant
- ,,,,

</br>

다들 비슷해보이지만 완전히 다르다.

먼저 날짜와 시간과 관련된 간단하게 개념을 한번 정리 한 후 

왜 자바 8 이전의 날짜와 시간과 관련된 API를 지양하는지 

그리고 자바 8부터 나온 각 API에 대해서 알아본 후

사용예시를 한번 알아보겠습니다.

</br>

## GMT? UTC? 타임존? 시차?

</br>

![https://sixalberts.files.wordpress.com/2014/08/timezones_text.jpg](https://sixalberts.files.wordpress.com/2014/08/timezones_text.jpg)

</br>

다들 이 시간 관련 검색을 해보면 이러한 지도를 많이 봤을 것이다.

여기서 사진에 보면 GMT와 UTC가 있다. 

</br>

***GMT***

GMT와 UTC는 서로 아주 비슷하지만 다른 의미를 가지고 있다.

먼저 GMT는 영국에 있는 `그린위치 천문대(경도 0도)`를 기준으로 하는 태양 시간을 의미한다.

쉽게 좌우로 15도씩 1시간 차이를 나타내며 동쪽으로 +1시간씩 서쪽으로는 -1시간 이라고 생각하면 된다.

현재 한국은 동쪽으로 약 135도에 위치해있으며 GMT 기준으로 9시간이 차이가 난다.

그래서 보통 한국의 타임존은 GMT+09:00으로 표현된다.

</br>

***UTC***

UTC는 GMT와 어떤 점이 다르나면 `지구의 자전주기의 흐름이 점점 늦어지고 있는 문제를 해결하기 위해 나온 시간대이다.` 시간적으로 따지자면 GMT와 거의 다르지는 않지만 미세한 차이가 있다.

그래서 `소프트웨어를 사용할 때에는 UTC가 더 정확하다.`

</br>

***타임존과 시차(OFFSET)***

</br>

**타임존**

타임존은 동일한 `로컬 시간을 따르는 지역`을 의미 한다.

이것은 나라에서 법적으로 지정하는 것이다.

현재 한국은 - 서울이지만 미국 처럼 땅이 넓은 나라는 

하나의 국가에 지정된 타임존이 여러개인 경우도 있다.

</br>

**시차(OFFSET)**

흔히 자바에서 개발을 하다보면 OFFSET이라는 단어를 한번쯤은 봤을 것이다.

말 그대로 쉽게 `시차`라고 생각하면 된다.

현재 한국은 UTC+09:00이며 UTC 기준 9시간 빠르다고 생각하면 된다.

</br>

***그럼 시차와 타임존은 1:1 관계일까?***

아니다! 현재 +09:00의 시차를 사용하고 있는 나라는 한국도 있지만 일본, 인도네시아 등 많은 지역에서 사용하고 있기에 시차와 타임존은 `1:N` 관계이다.

이제 시간과 관련된 개념들을 확인했으니

왜 자바 8 이전에 나온 Date와 Calendar를 지양하는지 대표적인 3가지 이유를 알아보자.

</br>

## Date와 Calendar를 지양하는 이유

</br>

먼저 가장 첫번째로 Date와 Calendar API는 `Mutable` 하다

변경이 가능하기 때문에 쓰레드에 안전하지가 않다!

그리고 두번째로는 Date 클래스를 봤을 시 날짜관련된 클래스라고 다들 생각하기 쉽다.

하지만 Date 클래스는 시간까지 다루기 때문에 혼동하기 쉽다.

마지막으로 Calendar를 이용하요 9월을 가져온다고 가정해보자

```java
public class DateTimeTest {
    public static void main(String[] args) {
        Calendar calendar = Calendar.getInstance();
        int 과연9가나올까 = calendar.get(Calendar.MONTH); // 현재 날짜 : 2022-09-06
        System.out.println(과연9가나올까); // 8
    }
}
```

당연히 9월이라 9가 나올꺼라 기대했지만 `8` 이 나오게 된다.

여기서 알 수 있듯이 월이 0부터 시작하기 때문에 개발을 하다보면 헷갈리는 경우가 많다.

이로써 지양하는 이유를 알아보았으니

드디어 자바 8부터 나온 Date와 Time에 관련된 API를 살펴보자.

</br>

## LocalDateTime

</br>

![https://i.stack.imgur.com/PJSsX.jpg](https://i.stack.imgur.com/PJSsX.jpg)

![https://i.stack.imgur.com/WXBy8.jpg](https://i.stack.imgur.com/WXBy8.jpg)

![https://i.stack.imgur.com/cmEdq.png](https://i.stack.imgur.com/cmEdq.png)

LocalDateTime은 먼저 타임존을 가지고 있지 않기 때문에 특정 지역에 시간대를 저장하거나 나타내지 않는다.

즉, 생년월일과 같은 날짜와 현지 시간을 합친거라고 생각하면 됩니다.

정확한 순간을 나타내는 것이라고 할 수 없습니다.

</br>

LocalDateTime = LocalDate + LocalTime

```java
public class DateTimeTest {
    public static void main(String[] args) {
        LocalDate localDate = LocalDate.of(2022, 9, 6);
        LocalTime localTime = LocalTime.of(12, 0);
        LocalDateTime localDateTime = LocalDateTime.of(localDate, localTime);
        System.out.println(localDateTime);
				// 2022-09-06T12:00
    }
}
```

코드로 확인해볼 수 있듯이 `LocalDateTime = LocalDate + LocalTime`을 확인할 수 있고, static 메소드로 인스턴스를 생성합니다.

여러가지 인스턴스 생성 정적메소드를 오버로딩해서 제공하기 때문에 편하게 사용할 수 있습니다.

</br>

## Instant

![https://i.stack.imgur.com/9c9c8.png](https://i.stack.imgur.com/9c9c8.png)

Instant는 1970년 부터 현재 시간까지를 계산한 nano초 동안의 시간이며 하나의 순간을 나타냅니다. `대부분의 로직과 데이터들은 UTC와 같은 정확한 시간으로 계산되어야 하므로 자주 사용하는 클래스입니다.`

</br>

Instant 생성

```java
public class DateTimeTest {
    public static void main(String[] args) {
        Instant instant = Instant.now();
        System.out.println(instant); // 2022-09-05T15:17:14.741133Z
    }
}
```

현재 시간은 9월 6일 오전 12시 17분이다. UTC 기준으로 시간이 출력되는 것을 확인할 수 있다.

그렇다면 한국 시간으로 나오게 할려면 어떻게 해야할까?

```java
public class DateTimeTest {
    public static void main(String[] args) {
        Instant instant = Instant.now();
        
        // 타임존 (한국/서울)의 ZoneId를 넘겨준다.
        ZonedDateTime zonedDateTime = instant.atZone(TimeZone.getDefault().toZoneId());
        System.out.println(zonedDateTime);
				// 2022-09-06T00:18:48.519207+09:00[Asia/Seoul]
    }
}
```

</br>

## ZonedDateTime

![https://i.stack.imgur.com/2j5Ui.png](https://i.stack.imgur.com/2j5Ui.png)

ZonedDateTime = ZoneId + Instant라고 생각하면 된다.

쉽게 생각한다면 특정 지역의 현재 순간을 나타낸다고 생각하면 됩니다.

</br>

뉴욕의 현재 순간을 알아보자

```java
public class DateTimeTest {
    public static void main(String[] args) {
        // 뉴욕의 현재 순간을 알아보자.
        ZoneId zoneId = ZoneId.of("America/New_York");
        ZonedDateTime newyork = ZonedDateTime.now(zoneId);
        System.out.println(newyork);
    }
}
```

</br>

그럼 개발에서는 어떻게 사용하는가?

백엔드에서는 `정확한 날짜의 순간을 계산하기 위해서는 UTC 형식이어야 한다.`

하지만 `사용자가에게 나타낼때에는 현재 사용자가 위치한 지역에 맞게 보여줘야 하므로 이렇게 ZonedDateTime이 존재합니다.`

각각의 특징을 알아보았으니 실제 날짜와 시간 관련 개발 예시를 들어 보겠습니다.

</br>

## 예시 (이벤트 기간 체크 클래스)

</br>

만약 개발을 하다보며 이벤트 부분을 개발하게 되는 경우가 있다.

하지만 이벤트는 상시로 운영하는 이벤트도 있겠지만 대부분 일정 기간동안 진행하는 이벤트가 대다수 이다.

여기서 각 메서드마다 시간 체크 하는 로직을 넣는 다면 중복될 것이며 나중에 유지보수 하기가 어려워진다. 

이러한 점을 고려하여 시작날짜와 종료날짜를 인자로 받아 현재 지금 순간이 이벤트 기간인지를 체크해주는 클래스를 한번 개발해보자.

</br>

EventManager.class

```java
public class EventManager {
		// (1)
    private static final DateTimeFormatter DATE_TIME_FORMATTER = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");

		// (2)
    private long startEventTimeMillis;
    private long endEventTimeMillis;

		// (3)
    public static EventManager of(String startEventDateTime, String endEventDateTime, TimeZone timeZone) {
        ZonedDateTime startEventZonedDateTime = createEventDateTime(startEventDateTime, timeZone);
        ZonedDateTime endEventZonedDateTime = createEventDateTime(endEventDateTime, timeZone);
        
        EventManager eventManager = new EventManager();
        eventManager.setStartEventTimeMillis(convertToTimeMillis(startEventZonedDateTime));
        eventManager.setEndEventTimeMillis(convertToTimeMillis(endEventZonedDateTime));
        
        return eventManager;
    }

		// (4)
    private static ZonedDateTime createEventDateTime(String strDateTime, TimeZone timeZone) {
        DateTimeFormatter zonedDateTimeFormatter = DATE_TIME_FORMATTER.withZone(timeZone.toZoneId());
        return ZonedDateTime.parse(strDateTime, zonedDateTimeFormatter);
    }

		// (5)
    private static long convertToTimeMillis(ZonedDateTime zonedDateTime) {
        return zonedDateTime.toInstant().toEpochMilli();
    }

		// (6)
    public boolean isEventPeriod() {
        long currentTimeMillis = System.currentTimeMillis();
        return this.startEventTimeMillis <= currentTimeMillis && currentTimeMillis <= endEventTimeMillis;
    }

    public void setStartEventTimeMillis(long startEventTimeMillis) {
        this.startEventTimeMillis = startEventTimeMillis;
    }

    public void setEndEventTimeMillis(long endEventTimeMillis) {
        this.endEventTimeMillis = endEventTimeMillis;
    }
}
```

</br>

1. DateTimeFormatter 지정 우리는 클라이언트에게서 `2022-09-06 14:00:00` 와 같은 시간 형식의 문자열을 받을 것이며 이것을 포매팅하기 위해서 선언해주며 EventManager 공통으로 사용할 형식이기 때문에 `static final`로 선언

2. 빠른 시간계산을 위해 밀리세컨드 단위로 변환시켜 멤버 변수로 가지고 있는다.

3. 정적 생성 메소드를 이용해서 EventManager 객체를 생성해준다. (추후에 정적 메소드를 사용하는 이유로 찾아뵙겠다.)

4. 파라미터로 받은 문자열을 EventManager에 선언된 DateTimeFormatter에 맞게 ZonedDateTime을 생성해주며, ZonedDateTime을 만들기 위해서는 DateTimeFormatter에 ZoneId가 필요하기 때문에 TimeZone을 파라미터로 받는다.
    
    
5. 변환시킨 ZonedDateTime을 빠른 계산을 위해 millisecond로 변환한다.

6. 이벤트 기간인지 체크해주는 멤버 메소드

</br>

**실제 클라이언트에서 사용해보자.**

</br>

```java
public class DateTimeClient {
    public static void main(String[] args) {
        TimeZone timeZone = TimeZone.getDefault();
        EventManager eventManager = EventManager.of("2022-08-01 14:00:00", "2022-08-20 14:00:00", timeZone);
        System.out.println(eventManager.isEventPeriod()); // false
    }
}
```

현재 시간은 2022-09-06 12:59 AM(작성일 기준)이므로 `false` 가 출력되며 이벤트 기간이 아니라는 것을 확인할 수 있다.

이렇게 자바8부터 나온 날짜와 시간과 관련된 API는

- 불변
- 직관적
- 여러가지 편리한 기능을 제공

,,,

등등 많은 장점을 가지고 있기때문에 잘 사용한다면 아주 좋은 API들이므로 

각자 환경에 맞게 잘 사용하기 바라면서 이상 날짜와 시간과 관련된 글 마치도록 하겠습니다.

</br>

*****참조*****
1. [Date와 Time](https://dev-hazzang.tistory.com/19)
2. [자바스크립트에서 타임존 다루기](https://meetup.toast.com/posts/125)
3. [What's the difference between Instant and LocalDateTime?](https://stackoverflow.com/questions/32437550/whats-the-difference-between-instant-and-localdatetime)