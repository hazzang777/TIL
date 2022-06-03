# Date와 Time
백기선님 `더 자바 로드맵`을 수강하며 정리한 내용 입니다.  
글 밑에 참조에 강의 링크 있습니다.

## Date와 Time API 소개

<aside>
💡 자바 8에서 새로운 날짜와 시간 API 생긴 이유

</aside>

- [java.util.Date](http://java.util.Date) 클래스는 변경 가능하기 때문에 쓰레드에 안전하지 않았다.
- Date인데 시간까지 다룬다.
- 버그 발생할 여지가 많다.
    - 타입 안정성이 없음
    - 월이 0부터 시작

```java
public class App {
    public static void main(String[] args) throws InterruptedException {
        Date date = new Date();
        long time = date.getTime(); // Date인데 시간까지 다룸
        System.out.println(date);
        System.out.println(time);

        Thread.sleep(1000 * 3);
        Date after3Seconds = new Date();
        System.out.println(after3Seconds); // 3초후 시간
        date.setTime(time); // mutable 하다
        System.out.println(after3Seconds); // 다시 원래 시간
				// 쓰레드에 안전하지 않다.
    }
}
```

- 자바 8 Date-Time 주요 API
    - 기계용 시간 & 인류용 시간 나눌 수 있음
    - 기계용 시간은 1970년 1월 1일 0시 0분 0초 부터 현재까지의 타임스탬프를 표현
    - 인류용 시간은 연, 월, 일, 시, 분, 초 등을 표현
    - 타임스탬프는 Instant를 사용
    - 특정날짜 - LocalDate
    - 시간 - LocalTime
    - 일시 - LocalDateTime
    - 기간을 표현할때는 Duration (시간 기반) / Period (날짜 기반)
    - DateTimeFormatter를 사용해서 일시를 특정한 문자열로 포매팅 가능

## Date와 Time API

- 기계 시간으로 표현하는 방법
    - [intant.now](http://intant.now) : 현재 UTC(GMT)를 리턴

```java
public class App {
    public static void main(String[] args) throws InterruptedException {
        Instant instant = Instant.now(); // 지금 시간을 기계시간으로 출력
        System.out.println(instant); // 그린위치 기준시로 나온다. 2022-06-03T15:24:55.684792Z 

        ZoneId zone = ZoneId.systemDefault();
        System.out.println(zone); // Asia/Seoul
        ZonedDateTime zonedDateTime = instant.atZone(zone);
        System.out.println(zonedDateTime); // 2022-06-04T00:27:31.139805+09:00[Asia/Seoul]
    }
}
```

- 인류용 일시를 표현하는 방법
    - LocalDateTime.now() : 현재 시스템 Zone에 해당하는 일시를 리턴
    - LocalDateTime.of(int, Month, …) : 로컬의 특정 일시 리턴
    - ZoneDateTime.of(int, Month, …. ,ZoneId) : 특정 Zone의 특정 일시 리턴

```java
public class App {
    public static void main(String[] args) throws InterruptedException {
        LocalDateTime now = LocalDateTime.now();
        System.out.println(now); // 로컬 시스템이 한국이라 한국 기준 시간이 나온다.
        LocalDateTime birthday = LocalDateTime.of(1997, Month.JUNE, 6, 0, 0, 0);
        System.out.println(birthday);
        ZonedDateTime nowInKorea = ZonedDateTime.now(ZoneId.of("Asia/Seoul"));
        System.out.println(nowInKorea);

        Instant nowInstant = Instant.now();
        ZonedDateTime zonedDateTime = nowInstant.atZone(ZoneId.of("Asia/Seoul")); // Instant -> ZonedDateTime
        System.out.println(zonedDateTime);
        zonedDateTime.toInstant(); // Instant로 변환 가능
    }
}
```

- 기간을 표현하는 방법
    - Period / Duration between()
- Period : 인류용

```java
public class App {
    public static void main(String[] args) throws InterruptedException {
        LocalDate today = LocalDate.now();
        LocalDate thisYearBirthday = LocalDate.of(2022, Month.JUNE, 6);

        Period period = Period.between(today, thisYearBirthday);
        System.out.println(period.getDays());

        Period until = today.until(thisYearBirthday);
        System.out.println(until.get(ChronoUnit.DAYS));
    }
}
```

- Duration : 기계용

```java
public class App {
    public static void main(String[] args) throws InterruptedException {
        Instant now = Instant.now();
        Instant plus = now.plus(10, ChronoUnit.SECONDS);
        Duration between = Duration.between(now, plus);
        System.out.println(between.getSeconds()); // 10
    }
}
```

- 파싱 또는 포매팅

→ 포매팅

```java
public class App {
    public static void main(String[] args) throws InterruptedException {
        LocalDateTime now = LocalDateTime.now();
        DateTimeFormatter MMddyyyy = DateTimeFormatter.ofPattern("MM/dd/yyyy");
        System.out.println(now.format(MMddyyyy));
    }
}
```

→ 파싱

```java
public class App {
    public static void main(String[] args) throws InterruptedException {
        DateTimeFormatter MMddyyyy = DateTimeFormatter.ofPattern("MM/dd/yyyy");
        LocalDate parse = LocalDate.parse("06/06/1997", MMddyyyy);
        System.out.println(parse);
    }
}
```

- 레거시 API 지원
    - 예전 API와 호환이 된다. 서로 변환 가능

```java
public class App {
    public static void main(String[] args) throws InterruptedException {
        Date date = new Date();
        Instant instant = date.toInstant(); // Date -> Instant
        Date newDate = Date.from(instant);  // Instant -> Date

        GregorianCalendar gregorianCalendar = new GregorianCalendar();
        LocalDateTime dateTime = gregorianCalendar
                .toInstant().atZone(ZoneId.systemDefault()).toLocalDateTime();
        // GregorianCalendar -> LocalDateTime

        GregorianCalendar from = GregorianCalendar.from(ZonedDateTime.now());
        // ZonedDateTime -> GregorianCalendar
    }
}
```

<aside>
💡 TIP

</aside>

TemporalUnit 파라미터 타입으로 나오는 경우 ChronoUnit으로 찾으면 된다.

Reference:
[더 자바, Java 8 : 백기선](https://www.inflearn.com/course/the-java-java8/dashboard)