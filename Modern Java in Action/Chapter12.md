# Chap12. 새로운 날짜와 시간 API

## 12.1 LocalDate, LocalTime, Instant, Duration, Period 클래스

### LocalDate와 LocalTime 사용

정적 팩토리 메서드 of로 LocalDate 인스턴스를 만들 수 있으며 연도, 달, 요일 등을 반환하는 메서드 제공

```java
// LocalDate 
LocalDate date = LocalDate.of(2017, 9, 21); // 2017-09-21
int year = date.getYear(); // 2017
Month month = date.getMonth(); // SEPTEMBER
int day = date.getDayOfMonth(); // 21
DayOfWeek dow = date.getDayOfWeek(); // THURSDAY
int len = date.lengthOfMonth(); // 31
boolean leap = date.isLeapYear(); // false (윤년이 아님)
LocalDate today = LocalDate.now(); // 현재 날짜 정보

// LocalTime
LocalTime time = LocalTime.of(13,45,20); // 13:45:20
int hour = time.getHour();
int minute = time.getMinute();
int second = time.getSecond();
```

### 날짜와 시간 조합

LocalDateTime은 LocalDate와 LocalTime을 쌍으로 갖는 복합체

```java
// LocalDateTime을 직접 만드는 방법과 날짜와 시간을 조합하는 방법
LocalDateTime dt1 = LocalDateTime.of(2017, Month.MARCH, 21, 13, 45, 20); // 2014-03-18T13:45
LocalDateTime dt2 = LocalDateTime.of(date, time);
LocalDateTime dt3 = date.atTime(13, 45, 20);
LocalDateTime dt4 = date.atTime(time);
LocalDateTime dt5 = time.atDate(date);
```

### Instant 클래스: 기계의 날짜와 시간

- 기계의 관점에서는 연속된 시간에서 특정 지점을 하나의 큰 수로 표현하는 것이 가장 자연스러운 시간 표현 방법
    - 유닉스 에포크 시간(1970년 1월 1일 0시 0분 0초 UTC) 기준으로 특정 지점까지의 시간을 초로 표현
- Instant 클래스는 나노초의 정밀도 제공

```java
Instant.ofEpochSecond(3);
Instant.ofEpochSecond(3, 0);
Instant.ofEpochSecond(2, 1_000_000_000) // 2초 이후의 1억 나노초
Instant.ofEpochSecond(4, -1_000_000_000) // 4초 이전의 1억 나노초
```

### Duration과 Period 정의

- 지금까지 살펴본 클래스는 Temporal 인터페이스를 구현하는 데 사용, Temporal 인터페이스는 특정 시간을 모델링하는 객체의 값을 어떻게 읽고 조작할지 정의
- Duration 클래스의 정적 팩토리 메서드 between으로 두 시간 객체 사이의 지속시간을 만듦

```java
// LocalDateTime과 Instant는 혼합 사용이 안됨
Duration d1 = Duration.between(time1. time2);
Duration d1 = Duration.between(dateTime1, dateTime2)
Duration d2 = Duration.between(instant1, intstant2);

// between을 이용하면 두 LocalDate의 차이를 확인할 수 있음
Period tenDays = Period.between(LocalDate.of(2017, 9, 11), LocalDate.of(2017, 9, 21))

```

## 12.2 날짜 조정, 파싱, 포매팅

withAttribute 메서드로 기존의 LocalDate를 바꾼 버전을 직접 간단하게 만듦

```java
// 절대적인 방식으로 LocalDate 속성 바꾸기
LocalDate date1 = LocalDate.of(2017, 9, 21);
LocalDate date2 = date1.withYear(2011); // 2011-09-21
LocalDate date3 = date2.withDayOfMonth(25); // 2011-09-25
LocalDate date4 = date3.with(ChronoField.MONTH_OF_YEAR, 2); // 2011-09-25

// 상대적인 방식으로 LocalDate 속성 바꾸기
LocalDate date1 = LocalDate.of(2017, 9, 21);
LocalDate date2 = date1.plusWeeks(1); // 2011-09-28
LocalDate date3 = date2.minusYears(25); // 2011-09-28
LocalDate date4 = date3.plus(6, ChronoUnit.MONTHS); // 2012-03-28
```

### TemporalAdjusters 사용하기

때로는 다음 주 일요일, 돌아오는 평일, 어떤 달의 마지막 날 등 좀 더 복잡한 날짜 조정 기능이 필요

```java
import static java.time.temporal.TemporalAdjusters.*;
LocalDate date1 = LocalDate.of(2014, 3, 18);
LocalDate date2 = date1.with(nextOrSame(DayOfWeek.SUNDAY)); // 2014-03-23
LocalDate date3 = date2.with(lastDayOfMonth()); // 2014-03-31
```

### 날짜와 시간 객체 출력과 파싱

날짜와 시간 관련 작업에서 포매팅과 파싱은 서로 떨어질 수 없는 관계, `java.time.format` 에서 가장 중요한 패키지인 DateTimeFormmater

```java
LocalDate date = LocalDate.of(2014, 3, 18);
String s1 = date.format(DateTimeFormmater.BASIC_ISO_DATE); // 20140318
String s1 = date.format(DateTimeFormmater.ISO_LOCAL_DATE); // 2014-03-18

// 반대로 날짜나 시간을 표현하는 문자열을 파싱해서 날짜 객체를 다시 만들 수 있음
LocalDate date1 = LocalDate.parse("20140318", DateTimeFormmater.BASIC_ISO_DATE);
LocalDate date2 = LocalDate.parse("2014-03-18", DateTimeFormmater.ISO_LOCAL_DATE);

// 패턴으로 DateTimeFormmater 만들기
DateTimeFormmater formatter = DateTimeFormatter.ofPattern("dd/MM/yyyy");
LocalDate date1 = LocalDate.of(2014, 3, 18);
String formattedDate = date1.format(formatter);
LocalDate date2 = LocalDate.parse(formattedDate, formatter);
```

## 12.3 다양한 시간대와 캘린더 활용 방법

- 새로운 날짜와 시간 API의 큰 편리함 중 하나는 시간대를 간단하게 처리 가능
- 새로운 클래스를 이용하면 서머타임(DST) 같은 복잡한 사항이 자동으로 처리 됨

### 시간대 사용하기

- 표준 시간이 같은 지역을 묶어서 **시간대** 규칙 집합을 정의
- 다음처럼 지역 ID로 특정 ZoneId를 구분
    - `ZoneId romeZone = ZoneId.of(”Europe/Rome”);`
