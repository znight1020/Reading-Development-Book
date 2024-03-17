# Chap5. 스트림 활용

## 5.1 필터링

스트림의 요소를 선택하는 방법

### 프레디케이트로 필터링

- filter 메서드는 **프레디케이트**를 인수로 받아 일치하는 모든 요소를 포함하는 스트림 반환
- 다음과 같은 코드로 모든 채식요리를 필터링해서 채식 메뉴를 만듦

```java
List<Dish> vegetarianMenu = menu.stream()
																.filter(Dish::isVegetarian)
																.collect(toList());
```

### 고유 요소 필터링

- distinct 메서드 지원, ex) 리스트의 모든 짝수를 선택 및 중복 필터링

```java
List<Integer> nubmers = Arrays.asList(1,2,1,3,3,2,4);
nubmers.stream()
			.filter(i -> i % 2 == 0)
			.distinct()
			.forEach(System.out::println)
```

## 5.2 스트림 슬라이싱

스트림의 요소를 선택하거나 스킵하는 다양한 방법

### 프레디케이트를 이용한 슬라이싱

- **takeWhile 활용**
    
    ```java
    // 기존의 코드에서 320칼로리 이하의 요리를 선택한다 했을 때의 코드
    List<Dish> filteredMenu
    		= specialMenu.stream()
    								.filter(dish -> dish.getCalories() < 320)
    								.collect(toList());
    // 위의 specialMenu 가 이미 칼로리 순으로 정렬되어 있다고 했을 때takeWhile을 활용하여 더 효율적인 코드 동작이 가능
    List<Dish> slicedMenu1
    		= specialMenu.stream()
    								.takeWhile(dish -> dish.getCalories() < 320)
    								.collect(toList());
    ```
    
- **dropWhile 활용**
    
    ```java
    // 그렇다면 320 칼로리보다 낮은 메뉴는 선택됐고 나머지 데이터들은 어떻게 탐색?
    List<Dish> slicedMenu2
    		= specialMenu.stream()
    								.dropWhile(dish -> dish.getCalories() < 320)
    								.collect(toList());
    ```
    
    - dropWhile은 takeWhile 과 정반대의 작업 수행
    - dropWhile은 무한한 남은 요소를 가진 **무한 스트림에서도 동작**

### 스트림 축소

- 주어진 값 이하의 크기를 갖는 새로운 스트림 반환하는 limit(n) 메서드 지원

```java
// 그렇다면 320 칼로리보다 낮은 메뉴는 선택됐고 나머지 데이터들은 어떻게 탐색?
List<Dish> dishes
		= specialMenu.stream()
								.filter(dish -> dish.getCalories() > 300)
								.limit(3) // 3개의 요소
								.collect(toList());
```

### 요소 건너뛰기

- 처음 n개 요소를 제외한 스트림을 반환하는 skip(n) 메서드 지원

```java
// 그렇다면 320 칼로리보다 낮은 메뉴는 선택됐고 나머지 데이터들은 어떻게 탐색?
List<Dish> dishes
		= specialMenu.stream()
								.filter(d -> d.getCalories() > 300)
								.skip(2) // 3개의 요소
								.collect(toList());
```

## 5.3 매핑

특정 객체에서 특정 데이터를 선택하는 작업

### 스트림의 각 요소에 함수 적용하기

- 함수를 인수로 받는 map 메서드 지원
- 예를 들어 Dish::getName을 map 메서드로 전달 후 스트림의 요리명 추출

```java
// map 메서드의 출력 스트림은 Stream<String> 형식을 갖음
List<String> dishNames = menu.stream()
														.map(Dish::getName)
														.collect(toList());

// 요리명의 길이를 알고 싶다면?
List<String> dishNameLengths = menu.stream()
																	.map(Dish::getName)
																	.map(String::length)
																	.collect(toList());
```

### 스트림 평면화

- 리스트에서 **고유 문자**로 이루어진 리스트 반환

![5_1](https://github.com/znight1020/Reading-Development-Book/assets/104980470/2108d62c-e1f4-454a-b1c9-8983db578b9a)

## 5.4 검색과 매칭

특정 속성이 데이터 집합에 있는지 여부를 검색하는 데이터 처리도 자주 사용

### 프레디케이트가 적어도 한 요소와 일치하는지 확인(anyMatch )

```java
if(menu.stream().anyMatch(Dish::isVegetarian)){
	System.out.println("The menu is (somewhat) vegetarian friendly!!");
}
```

### 프레디케이트가 모든 요소와 일치하는지 검사(allMatch)

```java
boolean isHealthy = menu.stream()
												.allMatch(dish -> dish.getCalories < 1000);
```

<aside>
💡 **noneMatch**
주어진 프레디케이트와 일치하는 요소가 없는지 확인

</aside>

### 요소 검색(findAny)

- 현재 스트림에서 임의의 요소 반환

```java
Optional<Dish> dish = 
		menu.stream()
				.filter(Dish::isVegetarian)
				.findAny();
```

- 스트림 파이프라인은 내부적으로 단일 과정으로 실행할 수 있도록 최적화

<aside>
💡 **Optional 이란?**
- 값의 존재나 부재 여부를 표현하는 컨테이너 클래스
- findAny는 아무 요소도 반환하지 않을 수 있음 → null 발생

isPresent() 는 Optional 값을 포함하면 return true, 포함하지 않으면 return false
ifPresoent() 는 값이 있으면 주어진 블록을 실행

`menu.stream().filter(Dish::isVegetarian).findAny().ifPresent(dish → sout(dish.getName()));`

</aside>

### 첫 번째 요소 찾기

```java
List<Integer> someNumbers = Arrays.asList(1,2,3,4,5);
Optional<Integer> firstSquareDivisibleByThree = 
		someNubers.stream()
							.map(n -> n * n)
							.filter(n -> n % 3 ==0)
							.findFirst(); // 9
```

## 5.5 리듀싱

지금까지 살펴본 최종 연산은 boolean(allMatch), void(forEach), 또는  Optional(findAny) 객체 반환

- **리듀싱 연산**: 모든 스트림 요소를 처리해서 값으로 도출, 같은 의미로 ‘**폴드**’ 라고 부름

### 요소의 합

- reduce를 사용하여 다음처럼 스트림의 모든 요소를 더함

```java
// reduce 는 2개의 인수를 갖음
int sum = numbers.stream().reduce(0, (a, b) -> a + b);

// 메서드 참조를 사용해 좀 더 간결하게 만들 수 있음
int sum = nubers.stream().reduce(0, Integer::sum);

// 다른 람다, 즉 (a, b) -> a * b를 넘겨주면 모든 요소의 곱셈
// 초기값을 설정하지 않는다면 Optional<Integer> 객체를 사용해야 함
```

![5_2](https://github.com/znight1020/Reading-Development-Book/assets/104980470/2ff7113d-679d-41a5-b651-37c8045629f8)

### 최댓값과 최솟값

- reduce 는 두 인수 사용
    - 초깃값
    - 스트림의 두 요소를 합쳐서 하나의 값으로 만드는 데 사용할 람다
    
    ```java
    Optional<Integer> max = nmubers.stream().reduce(Integer::max);
    Optional<Integer> min = nmubers.stream().reduce(Integer::min);
    ```
    

<aside>
💡 ※부록※
[reduce 메서드의 장점과 병렬화] p.174
[스트림 연산: 상태 없음과 상태 있음] p.175

</aside>

### 지금까지의 다양한 스트림 연산 점검

![5_3](https://github.com/znight1020/Reading-Development-Book/assets/104980470/aef24f2d-77d7-4cb5-b82b-a20f32355c42)

<aside>
💡 지금까지 배운 내용을 연습 → [5.6 실전 연습] p.177

</aside>

## 5.7 숫자형 스트림

```java
// 아래와 같은 코드의 경우 숨어있는 박싱 비용 존재
int calories = menu.stream()
									.map(Dish::getCalories)
									.reduce(0, Integer::sum);

// sum 메서드를 호출한다면?
int calories = menu.stream()
									.map(Dish::getCalories)
									.sum();

-> 아쉽게도 sum 메서드를 직접 호출할 수 없음, map 메서드가 Stream<T> 를 생성하기 때문
-> 스트림 API 숫자 스트림을 효율적으로 처리하는 **기본형 특화 스트림** 제공
```

### 기본형 특화 스트림

- 자바 8에서는 세 가지 기본형 특화 스트림 제공
    - IntStream
    - DoubleStream
    - LongStream
- 각각의 인터페이스는 sum(), max() 와 같이 자주 사용하는 리듀싱 연산 수행 메서드 제공
또한, 필요할 때 다시 객체 스트림으로 복원하는 기능도 제공
    
    <aside>
    💡 특화 스트림은 오직 박싱 과정에서 일어나는 효율성과 관련 있으며 스트림에 추가 기능을 제공하지는 않음
    
    </aside>
    

**※ 숫자 스트림으로 매핑 ※**

```java
int calories = menu.stream()
									.mapToInt(Dish::getCalories)
									.sum();
```

**※ 객체 스트림으로 복원하기 ※**

```java
IntStream intStream = menu.stream().mapToInt(Dish::getCalories); // 스트림을 숫자 스트림으로 변환
Stream<Integer> stream = intStream.boxed(); // 숫자 스트림을 스트림으로 변환
```

**※ 기본값 : OptionallInt ※**

- 합계 예제에서는 0 이라는 기본값 존재 하지만, IntStream 에서 최댓값을 찾을 때는 0 이라는 기본값 때문에 잘못된 결과 도출 가능성 있음
- Optional 또한 다음과 같은 세 가지 기본형 특화 스트림 버전 제공
    - OptionalInt
    - OptionalDouble
    - OptionalLong
    
    ```java
    OptionalInt maxCalories = menu.stream()
    															.mapToInt(Dish::getCalories)
    															.max();
    
    // 아래와 같은 코드로 최댓값이 없는 상황에 사용할 기본값 명시 가능
    int max = maxCalories.orElse(1);
    ```
    

### 숫자 범위

프로그램에서 특정 범위의 숫자를 이용해야 하는 상황 자주 발생

→ IntStream, LongStram 에서는 다음과 같은 정적 메서드 제공

- range
    - 종료값이 결과에 포함되지 않는다
- rangeClosed
    - 시작값과 종료값 모두 결과에 포함된다
    
    → 두 메서드 모두 첫 번째 인수로 시작값, 두 번째 인수로 종료값
    

<aside>
💡 **※ 숫자 스트림 활용 ※**
p.184 [피타고라스 수]
    p.185 [세 수 표현하기]
    p.185 [좋은 필터링 조합]
    p.186 [집합 생성]
    (중략)

</aside>

## 5.8 스트림 만들기

일련의 값, 배열, 파일, 심지어 함수를 이용한 무한 스트림 만들기 등 다양한 방식의 스트림을 만드는 방법 설명

### 값으로 스트림 만들기

- 스트림의 모든 문자열을 대문자로 변환한 후 문자열을 하나씩 출력

```java
Stream<String> stream = Stream.of("Modern", "Java", "In", "Action");
stream.map(String::toUpperCase).forEach(System.out::println);

// 다음처럼 empty 메서드를 이용해 스트림을 비울 수 있음
Stream<String> emptyStream = Stream.empty();
```

### null이 될 수 있는 객체로 스트림 만들기

때로는 null이 될 수 있는 객체를 스트림으로 만들어야 함

- `System.getProperty` 는 제공된 키에 대응하는 속성이 없으면 null 반환

```java
String homeValue = System.getProperty("home");
Stream<String> homeValueStream = homeValue == null ? Stream.empty() : Stream.of(value);

// Stream.ofNullable 을 이용해 다음처럼 코드 구현 가능
Stream<String> homeValueStream = Stream.ofNullable(System.getProperty("home"));

// 아래 코드는 null이 될 수 있는 객체를 포함하는 스트림값을 flatMap과 함께 사용 시 더 유용한 패턴
Stream<String> values = Stream.of("config", "home", "user")
															.flatMap(key -> Stream.ofNullable(System.getProperty(key)));
```

### 배열로 스트림 만들기

```java
int[] numbers = {2,3,5,7,11,13};
int sum = Arrays.stream(numbers).sum();
```

### 함수로 무한 스트림 만들기

- `Stream.iterate`, `Stream.generate` 두 연산을 이용하여 **무한 스트림** 즉, 크기가 고정되지 않은 스트림 제작 가능
    - 하지만 보통 무한한 값을 출력하지 않도록 limit(n) 함수를 함께 연결해서 사용
        - (?) 그럼 왜 알려주는겨

**※ iterate 메서드 ※**

```java
// iterate 는 요청할 때마다 값을 생산할 수 있으며 끝이 없으므로 **무한 스트림** 을 만듦
// 이러한 스트림을 '언바운드 스트림' 이라고 표현

Stream.iterate(0, n -> n + 2)
			.limit(10)
			.forEach(System.out::println);

-> 일반적으로 연속된 일련의 값 생성 시 iterate 사용
-> 예를 들어, 1월 13일, 2월 1일등의 날짜 생성
```

- iterate 메서드는 프레디케이트 지원
    
    → 0에서 시작해서 100보다 크면 숫자 생성 중단하는 코드
    
    ```java
    // forEach
    IntStream.iterate(0, n -> n < 100, n -> n + 4)
    				.forEach(System.out::println);
    
    // filter 또한 가능할까?
    IntStream.iterate(0, n -> n + 4)
    				.filter(n -> n < 100)
    				.forEach(System.out::println);
    
    // 안타깝게도 이와 같은 방법으로 같은 결과 얻을 수 없음
    // filter 메서드는 언제 이 작업을 중단해야 하는지 알 수 없기 때문
    
    // takeWhile 이용하는 것이 해법
    IntStream.iterate(0, n -> n + 4)
    				.takeWhile(n -> n < 100)
    				.forEach(System.out::println);
    ```
    

**※ generate 메서드 ※**

- iterate와 달리 generate는 생산된 각 값을 연속적으로 계산 X

```java
Stream.generate(Math::random)
			.limit(5)
			.forEach(System.out::println);

// 이 코드는 0~1 사이의 임의의 더블 숫자 다섯 개 생성
// 이번에도 명시적으로 limit() 메서드 사용하여 스트림 크기 한정
//     -> 없다면 해당 스트림은 언바운드 상태
```
