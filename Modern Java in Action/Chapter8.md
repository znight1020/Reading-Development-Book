# Chap8. 컬렉션 API 개선

## 8.1 컬렉션 팩토리

자바 9에서는 작은 컬렉션 객체를 쉽게 만들 수 있는 몇 가지 방법 제공

```java
// 휴가를 함께 보내려는 친구 이름을 포함하는 그룹 생성
List<String> friends = new ArrayList<>();
friends.add("Raphael");
friends.add("Olivia");
friends.add("Thibaut");

// 다음처럼 Arrays.asList() 팩토리 메서드로 간단하게
List<String> friends = Arrays.asList("Raphael", "Olivia", "Thibaut")

// 고정 크기의 리스트를 만들었으므로 요소를 갱신할 순 있지만 새 요소 추가하거나 요소 삭제 X
// 요소를 추가하려 하면 UnsupprotedOperationException 발생
```

### UnsupprotedOperationException

- 내부적으로 고정된 크기의 변환할 수 있는 배열로 구현했기 때문

### 리스트 팩토리

List.of 팩토리 메서드를 이용해서 간단하게 리스트 생성 가능

`List<String> friends = List.of("Raphael", "Olivia", "Thibaut")`

- 이 코드 또한 Unsu~ 예외가 발생

### 집합 팩토리

List.of 와 비슷한 방법으로 바꿀 수 없는 집합 생성 가능

`Set<String> friends = Set.of("Raphael", "Olivia", "Thibaut")`

- 중복된 요소로 set() 할 시 IllegalArgumentException 발생

### 맵 팩토리

- 복잡하니 그냥 맵 쓰는게 나을 것 같음

## 8.2 리스트와 집합 처리

자바 8에서는 List, Set 인터페이스에 다음과 같은 메서드 추가

- `removeIf`: 프레디케이트를 만족하는 요소 제거
- `replaceAll`: 리스트에서 이용할 수 있는 기능으로 UnArrayOperator 함수를 이용해 요소를 바꿈
- `sort`: List 인터페이스에서 제공하는 기능으로 리스트 정렬

### removeIf 메서드

```java
// 숫자로 시작되는 참조 코드를 가진 트랜잭션 삭제 코드
for(Iterator<Transaction> iterator = transactions.iterator();
	iterator.hasNext(); ) {
	Transaction trasaction = iterator.next();
	if(Character.isDigit(trasaction.getReferenceCode().charAt(0))) {
		iterator.remove();
	}
}

// 이 코드를 자바 8의 removeIf 메서드로 바꾸면
trasactions.removeIf(trasaction -> 
	Character.isDigit(trasaction.getReferenceCode().charAt(0)));
```

### replaceAll 메서드

```java
// 리스트의 각 요소를 새로운 요소로 바꾸기
referenceCodes.replaceAll(code -> Character.toUpperCase(code.charAt(0)) + code.substring(1));
```

## 8.3 Map 처리

### forEach 메서드

- Map.Entry<K, V> 의 반복자를 이용해 맵의 항목 집합을 반복했던 이전 코드와 달리 자바 8부터는 BiConsumer 를 인수로 받는 forEach 메서드 지원

```java
ageOfFriends.forEach((friend, age) -> System.out.println(friend + " is " + age + " years old"));
```

### 정렬 메서드

다음 두 개의 새로운 유틸리티 이용하여 맵의 항목을 값 또는 키를 기준으로 정렬

- Entry.comparingByValue
- Entry.comparingByKey

```java
Map<String, String> favouriteMovies
			= Map.ofEntries(entry("Raphael", "Star Wars"),
			entry("Cristina", "Matrix"),
			entry("Olivia", "James Bond"));

favouriteMovies
		.entrySet()
		.stream()
		.sorted(Entry.comparingByKey())
		.forEachOrdered(System.out::println); // 사람의 이름을 알파벳 순으로 스트림 요소 처리
```

### getOrDefault 메서드

- 첫 번째 인수로 키, 두 번째 인수로 기본값을 받으며 맵에 키가 존재하지 않을 시 두 번째 인수로 받은 기본값을 반환

```java
Map<String, String> favouriteMovies
			= Map.ofEntries(entry("Raphael", "Star Wars"),
			entry("Olivia", "James Bond"));

System.out.println(favouriteMovies.getOrDefault("Olivia", "Matrix"));
System.out.println(favouriteMovies.getOrDefault("Thibaut", "Matrix"));
```

### 계산 패턴

- 맵에 키가 존재하는지 여부에 따라 어떤 동작을 실행하고 결과를 저장함
    - computeIfAbsent: 제공된 키에 해당하는 값이 없으면, 키를 이용해 새 값 계산하고 맵에 추가
    - computeIfPresent: 제공된 키가 존재하면 새 값을 계산하고 맵에 추가
    - compute: 제공된 키로 새 값을 계산하고 맵에 저장
- 정보를 캐시할 때 computeIfAbsent 를 활용
    - 파일 집합의 각 행을 파싱해 SHA-256 을 계산
    
    ```java
    Map<String, byte[]> dataToHash = new HashMap<>();
    MessageDigest messageDigest = MessageDigest.getInstance("SHA-256");
    
    // 이제 데이터를 반복하며 결과를 캐시
    lines.forEach(line -> 
    	dataToHash.computeIfAbsent(line, this::calculateDigest)); 
    	// line은 맵에서 찾을 키
    	// 키가 존재하지 않으면 동작을 실행
    
    private byte[] calculateDigest(String key) { // 헬퍼가 제공된 키의 해시를 계산할 것
    	return messageDigest.digest(key.getBytes(StandardCharsets.UTF_8));
    }
    ```
    
- 여러 값을 저장하는 맵을 처리할 때도 이 패턴은 유용

```java
friendsToMovies.computeIfAbsent("Raphael", name -> new ArrayList<>()).add("Star Wars")
```

### 삭제 패턴

```java
favouriteMovies.remove(key, value);
```

### 교체 패턴

맵의 항목을 바꾸는 데 사용할 수 있는 두 개의 메서드

- `replaceAll`: BiFunction 을 적용한 결과로 각 항목의 값 교체
- `Replace`: 키가 존재하면 맵의 값을 바꿈

```java
Map<String, String> favouriteMovies = new HashMap<>();
favouriteMovies.put("Raphael", "Star Wars");
favouriteMovies.put("Olivia", "james bond");
favouriteMovies.replaceAll((friend, movie) -> movie.toUpperCase());
System.out.println(favouriteMovies);
```

### 합침

- 두 개의 맵에서 값을 합치거나 바꿔야 한다면?
- 두 영화의 문자열을 합치는 방법

```java
Map<String, String> everyone = new HashMap<>(family);
    friends.forEach((k, v) -> everyone.merge(k, v, (movie1, movie2) -> movie1 + " & " + movie2));
    System.out.println(everyone);
```

- merge 를 이용해 초기화 검사 구현 가능

```java
Map<String, Long> moviesToCount = new HashMap<>();
String movieName = "JamesBond";
long count = moviesToCount.get(movieName);
if(count == null){
	moviesToCount.put(movieName, 1);
}
else{
	moviesToCount.put(movieName, count + 1);
}

// 위 코드를 다음처럼 구현 가능
moviesToCount.merge(movieName, 1L, (count, increment) -> count + 1L);
```

## 8.4 개선된 ConcurrentHashMap

ConcurrentHashMap 클래스는 동시성 친화적이며 최신 기술 반영 HashMap

- 내부 자료구조의 특정 부분만 잠궈 동시 추가, 갱신 작업 허용
- 동기화된 Hashtable 버전에 비해 읽기 쓰기 연산 성능이 월등
    - 표준 HashMap 은 비동기로 동작

### 리듀스와 검색

**ConcurrentHashMap 은 세 가지 새로운 연산 제공**

- forEach: 각 (키, 값) 쌍에 주어진 액션을 실행
- reduce: 모든 (키, 값) 쌍을 제공된 리듀스 함수를 이용해 결과로 합침
- search: 널이 아닌 값을 반환할 때까지 각 (키, 값) 쌍에 함수를 적용

**네 가지 연산 형태 지원**

- 키, 값으로 연산: forEach, reduce, search
- 키로 연산: forEachKey, reduceKeys, searchKeys
- 값으로 연산: forEachValue, reduceValues, searchValues
- Map.Entry 객체로 연산: forEachEntry, reduceEntries, searchEntries

**이들 연산은 ConcurrentHashMap 의 상태를 잠그지 않고 연산을 수행한다는 점을 주목**

→ 이들 연산에 제공한 함수는 계산이 진행되는 동안 바뀔 수 있는 객체, 값, 순서 등에 의존 X

**이들 연산에 병렬성 기준값을 지정해야 함**

→ 맵의 크기가 주어진 기준값보다 작으면 순차적으로 연산 실행

⇒ 기준값을 1로 지정하면 공통 스레드 풀을 이용해서 병렬성을 극대화

⇒ Long.MAX_VALUE 를 기준값으로 설정할 시 한 개의 스레드로 연산 실행

```java
ConcurrentHashMap<String, Long> map = new ConcurrentHashMap<>();
long parallelismThreshold = 1;
Optional<Integer> maxValue = 
	Optional.ofNullable(map.reduceValues(parallelismThreshold, Long::max));
```

### 계수

ConcurrentHashMap 클래스는 맵의 매핑 개수를 반환하는 mappingCount 메서드 제공

- 기존의 size 메서드 대신 새 코드에서는 long 반환하는 mappingCount 메서드 사용이 적절
    - 매핑의 개수가 int의 범위를 넘어서는 이후의 상황 대처 가능

### 집합뷰

ConcurrentHashMap 클래스는 ConcurrentHashMap 을 집합 뷰로 반환하는 KeySet 메서드 제공

- 맵을 바꾸면 집합도 바뀌고 반대로 집합을 바꾸면 맵도 영향을 받음
