# Chap6. 스트림으로 데이터 수집

collect 또한 다양한 요소 누적 방식을 인수로 받아 스트림을 최종 결과로 도출하는 리듀싱 연산을 수행

- collect와 컬렉터로 구현할 수 있는 질의 예제
    - 통화별로 트랜잭션을 그룹화한 다음 통화로 일어난 모든 트랜잭션 합계 계산
        - Map<Currency, Integer> 반환
    - 트랜잭션을 비싼 트랜잭션과 저렴한 트랜잭션 두 그룹으로 분류
        - Map<Boolean, List<Transaction>> 반환
    - 트랜잭션을 도시 등 다수준으로 그룹화
        - Map<String, Map<Boolean, List<Transaction>>> 반환
- 어떤 트랜잭션 리스트가 있는데 이들을 액면 통화로 그룹화

```java
Map<Currency, List<Transaction>> transactionByCurrencies = new HashMap<>(); // 그룹화한 트랜잭션을 저장할 맵 생성

for(Transaction transaction : transactions){
	Currency currency = transaction.getCurrency();
	List<Transaction> transactionForcurreny = transactionsByCurrencies.get(currency);
	if(transactionForCurrency == null){ // 현재 통화를 그룹화하는 맵에 항목이 없으면 항목을 만듦
		transactionForCurrency = new ArrayList<>();
		transactionForCurrencies.put(currency, transactionCurrency);
	}
	transactionForCurrency.add(transaction); // 같은 통화를 가진 트랜잭션 리스트에 현재 탐색 중인 트랜잭션 추가
}

// 해당 코드는 간단한 작업임에도 코드가 너무 긺
// 코드가 무엇을 실행하는지 한눈에 파악 어려움
```

- collect 메서드를 통해 원하는 연산을 간결하게 구현

```java
Map<Currency, List<Transaction>> transactionByCurrencies = transaction.stream().collect(groupingBy(Transaction::getCurrency());))
```

## 6.1 컬렉터란 무엇인가?

- 위 예제는 **함수형 프로그래밍**이 얼마나 편리한지 명확하게 보여줌
    - 함수형 프로그래밍에서는 ‘무엇’을 원하는지 직접 명시할 수 있어 어떤 방법으로 이를 얻을지 신경 쓸 필요가 없음
- 해당 코드는 `groupingBy`를 이용해 ‘각 키 버킷 그리고 각 키 버킷에 대응하는 요소 리스트를 값으로 포함하는 맵 생성’ 동작을 수행

### 고급 리듀싱 기능을 수행하는 컬렉터

- 높은 수준의 조합성, 재사용성
    - collect로 결과를 수집하는 과정을 간단하고 유연한 방식으로 정의 ← 컬렉터의 최대 강점

### 미리 정의된 컬렉터

Collectors 에서 제공하는 메서드의 기능은 크게 세 가지로 구분

1. 스트림 요소를 하나의 값으로 리듀스하고 요약
2. 요소 그룹화
3. 요소 분할
- 분할
    - 한 개의 인수를 받아 boolean 을 반환하는 함수

## 6.2 리듀싱과 요약

컬렉터로 스트림의 항목을 컬렉션으로 재구성 가능

→ 컬렉터로 스트림의 모든 항목을 하나의 결과를 합칠 수 있음

### counting() 예제

- 메뉴에서 요리 수를 계산

```java
long howManyDishes = menu.stream().collect(Collectors.counting());
->
long howManyDishes = menu.stream().count();
```

### 스트림값에서 최댓값과 최솟값 검색

메뉴에서 칼로리가 가장 높은 요리를 찾는다 가정

`Collectors.maxBy`, `Collectors.minBy` 두 개의 메서드 이용해서 계산

```java
Comparator<Dish> dishCaloriesComparator = Comparator.comparingInt(Dish::getCalories);
Optional<Dish> mostCalorieDish = menu.stream().collect(maxBy(dishCaloriesComparator));
```

- 스트림에 있는 객체의 숫자 필드의 합계나 평균 등을 반환하는 연산에도 리듀싱 기능이 자주 사용 → **요약 연산**

### 요약 연산

- Collectors.summingInt(Double, Long) 메서드 제공
- summingInt 는 객체를 int로 매핑하는 함수를 인수로 받음
    
    → collect 메서드로 전달되어 요약 작업 수행
    
    `int totalCalories = menu.stream().collect(summingInt(Dish::getCalories));`
    
- 이 밖에도 평균값을 계산하는 avragingInt, averagingLong… 등이 있음

<aside>
💡 summarizingInt 메서드
- 하나의 요약 연산으로 메뉴에 있는 요소 수, 칼로리 합계, 평균, 최댓값, 최솟값 계산

`IntSummaryStatistics menuStatistics = menu.stream().collect(summarizingInt(Dish::getCalories));`

다음과 같은 정보를 얻을 수 있음
IntSummaryStatistics{count=9, sum=4300, min=120, average=477.7778, max=800}

</aside>

### 문자열 연결

`joining` 메서드를 이용하여 스트림의 각 객체에 toString 메서드 호출해 추출한 모든 문자열을 하나의 문자열로 연결해서 반환

```java
// joining 메서드는 내부적으로 StringBuilder 를 이용해서 문자열을 하나로 만듦

String shortMenu = menu.stream().map(Dish::getName).collect(joining());
-> Dish class가 요리명을 반환하는 toString 메서드 포함 시 map 추출 과정 생략 가능
String shortMenu = menu.stream().collect(joining());
-> 연결된 두 요소 사이에 구분 문자열 넣기
String shortMenu = menu.stream().map(Dish::getName).collect(joining(", "));
```

### 범용 리듀싱 요약 연산

- 지금까지 살펴본 모든 컬렉터는 reducing 팩토리 메서드로도 정의 가능

```java
// 메뉴의 모든 칼로리 합계 계산
int totalCalories = menu.stream().collect(reducing(0, Dish::getCalories, (i, j) -> i + j));

// 가장 칼로리가 높은 요리 찾기
Optional<Dish> mostCalorieDish = menu.stream().collect(reducing(d1, d2) -> d1.getCalories() > d2.getCalories() ? d1 : d2));
```

## 6.3 그룹화

메뉴를 그룹화한다고 가정

1. 고기를 포함하는 그룹
2. 생선을 포함하는 그룹
3. 나머지 그룹

```java
Map<Dish.Type, List<Dish>> dishesByType = menu.stream().collect(groupingBy(Dish::getType));
```

- 스트림의 각 요리에서 Dish.Type 과 일치하는 모든 요리를 추출하는 함수를 groupingBy 메서드로 전달
- 이 함수를 기준으로 스트림이 그룹화되며 이를 **분류 함수** 라고 부름
    
    ![6_1](https://github.com/znight1020/Reading-Development-Book/assets/104980470/76f21833-2ac7-48c9-a209-0982469f3580)
    

```java
// 칼로리 별 그룹화
public enum CaloricLevel {DIET, NORMAL, FAT}

Map<CaloricLevel, List<Dish>> dishesByCaloricLevel = menu.stream().collect(
		groupingBy(dish -> {
			if(dish.getCalories() <= 400) return CaloricLevel.DIET;
			else if(dish.getCalories() <= 700) return CaloricLevel.NORMAL;
			else return CaloricLevel.FAT;
		}));
```

### 그룹화된 요소 조작

- 각 결과 그룹의 요소를 조작하는 연산 필요

```java
// 500 칼로리가 넘는 요리만 필터한다고 가정
Map<Dish.Type, List<Dish>> caloricDishesByType = menu.stream().filter(dish -> dish.getCalories() > 500). collect(groupingBy(Dish::getType));
```

- 단점 존재
    - 메뉴 요리는 다음처럼 맵 형태로 되어 있어, 위 기능을 사용하려면 맵에 코드 적용 필요
        - {OTHER= [french fries, pizza], MEAT=[pork, beef]}
    - 필터 프레디케이틀 만족하는 FISH 종류 요리는 없으므로 결과 맵에서 해당 키 삭제
    - 아래와 같이 두 번째 Collector 안으로 필터 프레디케이트 이동
    
    ```java
    Map<Dish.Type, List<Dish>> caloricDishesByType = menu.stream().
    			collect(groupingBy(Dish::getType, filtering(dish -> dish.getCalories() > 500, toList())));
    ```
    
    - {OTHER=[french fries, pizza], MEAT=[pork, beef], FISH=[]}

### 다수준 그룹화

지금까지 칼로리 같은 한 가지 기준으로 메뉴의 요리를 그룹화 → 두 가지 이상의 기준 동시 적용하기

```java
Map<Dish.Type, Map<CaloricLevel, List<Dish>>> dishesByTypeCaloricLevel = menu.stream().collect(
	groupingBy(Dish::getType, // 첫 번째 수준의 분류 함수
		groupingBy(dish -> { // 두 번째 수준의 분류 함수
			if(dish.getCalories() <= 400) return CaloricLevel.DIET;
			else if(dish.getCalories() <= 700) return CaloricLevel.NORMAL;
			else return CaloricLevel.FAT;
		})
	)
);
```

- 그룹화의 결과로 다음과 같은 두 수준의 맵이 만들어짐

MEAT={DIET=[chicken], NORMAL=[beef], FAT=[pork], FISH={DIET=[prawns], NORMAL=[salmon]}, OTHER={DIET=[rice, seasonal fruit], NORMAL=[french fries, pizza]}}

![6_2](https://github.com/znight1020/Reading-Development-Book/assets/104980470/a3a02543-6e0b-43a5-bad3-47b87392f19b)

- 보통 groupingBy의 연산을 ‘버킷’ 개념으로 생각
    - 첫 번째 groupingBy는 각 키의 버킷 생성
        - 준비된 각각의 버킷을 서브스트림 컬렉터로 채워가기 반복

### 서브그룹으로 데이터 수집

- 첫 번째 groupingBy로 넘겨주는 컬렉터의 형식은 제한이 없음
- 아래와 같은 코드처럼 g 컬렉터에 두 번째 인수로 counting 컬렉터를 전달해서 메뉴에서 요리의 수를 종류별로 계산

```java
Map<Dish.Type, Long> typesCount = menu.stream().collect(groupingBy(Dish::getType, counting()));

// 결과: {MEAT=3, FISH=2, OTHER=4}
```

- 분류 함수 한 개의 인수를 갖는 groupingBy(f) 는 groupingBy(f, toList())의 축약형
- 요리의 **종류**를 분류하는 컬렉터로 메뉴에서 가장 높은 칼로리를 가진 요리를 찾는 프로그램 구현

```java
Map<Dish.Type, Optional<Dish>> mostCaloricByType = menu.stream()
			.collect(groupingBy(Dish::getType, maxBy(comparingInt(Dish::getCalories))));

// 그룹화의 결과로 요리의 종류를 키로, Optional<Dish> 를 값으로 갖는 맵 반환
// {FISH=Optional[salmon], OTHER=Optional[pizza], MEAT=Optional[pork]}
```

### 컬렉터 결과를 다른 형식에 적용하기

- 마지막 그룹화 연산에서 맵의 모든 값을 Optional로 감쌀 필요가 없으므로 Optional 삭제 가능
- 아래처럼 팩토리 메서드 Collectors.collectingAndThen 으로 컬렉터가 반환한 결과를 다른 형식으로 활용

```java
Map<Dish.Type, Dish> mostCaloricByType = menu.stream()
			.collect(groupingBy(Dish::getType, // 분류 함수
				collectingAndThen(maxBy(comparingInt(Dish::getCalories)), // 감싸인 컬렉터
						Optional::get))); // 변환 함수

// {FISH=salmon, OTHER=pizza, MEAT=pork}
```

![6_3](https://github.com/znight1020/Reading-Development-Book/assets/104980470/e2bf19f8-37cd-4167-8712-9aef34704c4a)

### groupingBy와 함께 사용하는 다른 컬렉터 예제

- p.218

## 6.4 분할

**분할 함수**라 불리는 프레디케이트를 분류 함수로 사용하는 특수한 그룹화 기능

- boolean 반환 → 맵의 키 형식 Boolean
- 결과적으로 그룹화 맵은 최대 (true or false) 두 개의 그룹으로 분류

채식 요리 vs 채식이 아닌 요리

```java
Map<Boolean, List<Dish>> partitionedMenu = menu.stream().collect(partitioningBy(Dish::isVegetarian));

// 다음과 같은 맵 반환
// {false = [pork, beef, chicken, prawns, salmon], true=[french fries, rice, season fruit, pizza]}

// 참값의 키로 맵에서 모든 채식 요리를 얻을 수 있음
List<Dish> vegetarianDishes = partitionedMenu.get(true);
```

### 분할의 장점

- 이전 코드를 활용하면 채식 요리와 채식 요리가 아닌 요리 각각의 그룹에서 가장 칼로리가 높은 요리 찾기 가능

```java
Map<Boolean, Dish> mostCaloricPartitionedByVegetarian = menu.stream().collect(
						partitioningBy(Dish::isVegeterian, collectingAndThen(maxBy(comparingInt(Dish::getCalories)),
							Optional::get)));

// 실행 결과
// {false=pork, true=pizza}
```

### 숫자를 소수와 비소수로 분할하기

```java
// 소수인지 아닌지 판단하는 프레디케이트
public boolean isPrime(int candidate){
	return IntStream.range(2, candidate) // 2부터 candidate 미만 사이의 자연수 생성
		.noneMatch(i -> candidate % i == 0); // 스트림의 모든 정수로 candidat를 나눌 수 없으면 참 반환
}

// isPrime 메서드를 이용해 partitioningBy 컬렉터로 리듀스 해서 소수와 비소수 분류
public Map<Boolean, List<Integer>> partitionPrimes(int n){
	return IntStream.rangeClosed(2, n).boxed()
			.collect(partitioningBy(candidate -> isPrime(candidate)));
}

```

## 6.5 Collector 인터페이스

리듀싱 연산을 어떻게 구현할지 제공하는 메서드 집합으로 구성

### Collector 인터페이스의 메서드 살펴보기

- **supplier** 메서드: 새로운 결과 컨테이너 만들기
- **accumulator** 메서드: 결과 컨테이너에 요소 추가하기
- **finisher** 메서드: 최종 변환값을 결과 컨테이너로 적용하기
- **combiner** 메서드: 두 결과 컨테이너 병합
- **Characteristics** 메서드
    - 다음 세 항목을 포함하는 열거형
        - UNORDERED: 리듀싱 결과는 스트림 요소의 방문 순서나 누적 순서에 영향을 받지 않음
        - CONCURRENT: 다중 스레드에서 accmulator 함수를 동시에 호출할 수 있으며 이 컬렉터는 스트림의 병렬 리듀싱을 수행
        - IDENTITY_FINISH: finisher 메서드가 반환하는 함수는 단순히 identity 를 적용할 뿐이므로 이를 생략 가능

### 자신만의 커스텀 ToListCollector 구현해보자

- 일단 전 안할게요, 참고 p.230

### 커스텀 컬렉터를 구현해서 성능 개선하기

1. Collector 클래스 시그니처 정의
2. 리듀싱 연산 구현
3. 병렬 실행할 수 있는 컬렉터 만들기(가능하다면)
4. finisher 메서드와 컬렉터의 characteristics 메서드

## 6.6 컬렉터 성능비교

팩토리 메서드 partitioningBy로 만든 코드와 커스텀 컬렉터로 만든 코드의 기능은 같지만 partitioningBy에 비해 커스텀 컬렉터의 성능이 좋은가?

- 간단한 하니스를 만들어서 테스트 할 수 있음

<aside>
💡 성능비교 코드작성은 p.238 참고하기

</aside>
