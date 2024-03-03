# Chap11. null 대신 Optional 클래스

## 11.1 값이 없는 상황을 어떻게 처리할까?

다음처럼 자동차와 자동차 보험을 갖고 있는 사람 객체를 중첩 구조로 구현

```java
public class Person{
	private Car car;
	public Car getCar() { return car; }
}

public class Car{
	private Insurance insurance;
	public Insurance getInsurance() { return insurance; }
}

public class Insurance{
	private String name;
	public String getName() { return name; }
}
```

- 해당 코드에서 발생할 수 있는 문제점
    - 차를 소유하지 않은 사람의 getCar() 호출 시 null 반환
        - getInsurance() 또한 null 반환
    - Person 이 null 인 경우라면?

### 보수적인 자세로 NullPointerException 줄이기

- 대부분의 프로그래머는 다양한 null 확인 코드를 추가하여 null 예외 문제를 회피
- 모든 변수가 null 인지 의심하므로 변수를 접근할 때마다 중첩된 if가 추가되면서 코드 들여쓰기 수준이 증가 → 이와 같은 반복 패턴을 ‘**깊은 의심’** 이라고 부름
    - 이는 코드의 구조가 엉망이 되고 가독성도 떨어지게 됨

### null 때문에 발생하는 문제

- **에러의 근원이다**: 자바에서 가장 흔히 발생하는 에러
- **코드를 어지럽힌다**: 때로는 중첩된 null 확인 코드를 추가해야 하므로 가독성이 떨어짐
- **아무 의미가 없다**: null은 아무 의미도 표현하지 않음
- **자바 철학에 위배된다**: 자바는 개발자로부터 모든 포인터를 숨김, 단 하나의 예외 null 포인터
- **형식 시스템에 구멍을 만든다**: null 은 무형식이며 정보를 포함하고 있지 않으므로 모든 참조 형식에 null을 할당할 수 있음

## 11.2 Optional 클래스 소개

값이 있으면 Optional 클래스는 값을 감싸고 값이 없다면 Optional.empty 메서드로 Optional 반환

- null 대신 Optional<Car> 반환 → 이는 값이 없을 수 있음을 명시적으로 보여줌
    - null 참조가 할당 됐을 때 올바른 값인지 잘못된 값인지 판단할 정보가 없음
- 다음 코드처럼 Optional 사용

```jsx
public class Person{
	private Optional<Car> car;
	public Optional<Car> getCar() { return car; }
}

public class Car{
	private Optional<Insurance> insurance;
	public Optional<Insurance> getInsurance() { return insurance; }
}

public class Insurance{
	private String name;
	public String getName() { return name; }
}
```

## 11.3 Optional 적용 패턴

### Optional 객체 만들기

- 빈 Optional
    - 정적 팩토리 메서드 Optional.empty 로 빈 Optional 객체를 얻을 수 있음
- null이 아닌 값으로 Optional 만들기
    - 정적 팩토리 메서드 Optional.of로 null이 아닌 값을 포함하는 Optional 생성 가능
        - `Optional<Car> optCar = Optional.of(car);`
- null값으로 Optional 만들기
    - 정적 팩토리 메서드 Optional.ofNullable 로 null 값을 저장할 수 있는 Optional 생성
        - `Optioanl<Car> optCar = Optional.ofNullable(car);`
        - car가 null이면 빈 Optional 객체 반환
- Optional에서 값을 가져올 때 get 메서드 사용, 이때 Optional이 비어있으면 get을 호출했을 때 예외가 발생하는데 이는 결국 null을 사용했을 때와 같은 문제를 겪을 수 있음
    
    → 따라서 먼저 Optional 로 명시적인 검사를 제거할 수 있는 방법을 알아볼 것임
    

### 맵으로 Optional의 값을 추출하고 변환하기

- 보험회사의 이름을 추출한다고 가정, 다음과 같이 insurance가 null인지 확인

```java
String name = null;
if(insurance != null){
	name = insurance.getName();
}

// 이런 유형의 패턴에 사용할 수 있도록 Optional은 map 메서드 지원
Optional<Insurance> optInsurance = Optional.ofNullable(insurance);
Optional<String> name = optInsurance.map(Insurance::getName);
```

- 여기서 Optional 객체를 최대 요소의 개수가 한 개 이하인 데이터 컬렉션으로 생각
- 값을 포함하면 map의 인수로 제공된 함수가 값을 바꿈
- Optional이 비어있다면 아무 일도 일어나지 않음
    
    ![11_1.jpg](Chap11%20null%20%E1%84%83%E1%85%A2%E1%84%89%E1%85%B5%E1%86%AB%20Optional%20%E1%84%8F%E1%85%B3%E1%86%AF%E1%84%85%E1%85%A2%E1%84%89%E1%85%B3%202c45623f3e774263a02c49210184c5c7/11_1.jpg)
    
    ```java
    public String getCarInsuranceName(Person person){
    	return person.getCar().getInsurance().getName();
    }
    
    // 이제 여러 메서드를 안전하게 호출가능
    ```
    

### flatMap으로 Optional 객체 연결

```java
// map을 이용한 코드 재구현
Optional<Person> optPerson = Optional.of(person);
Optional<String> name = 
	optPerson.map(Person::getCar)
		.map(Car::getInsurance)
		.map(Insurance::getName);

// 아쉽게도 위와 같은 코드는 컴파일 되지 않음 <- 중첩 Optional 객체 구조 반환, Optional<Optional<Car>>
// 이 문제를 해결하려면 스트림의 flatMap 메서드 사용 -> 이차원 Optional을 일차원 Optional로 평준화

public String getCarInsuranceName(Optional<Person> person){
	return person.flatMap(Person::getCar)
		.flatMap(Car::getInsurance)
		.map(Insurance::getName)
		.orElse("Unknown"); // 결과 Optional이 비어있으면 기본값 사용
}
```

- Optional을 이용한 Person/Car/Insurance 참조 체인
    
    ![11_2.jpg](Chap11%20null%20%E1%84%83%E1%85%A2%E1%84%89%E1%85%B5%E1%86%AB%20Optional%20%E1%84%8F%E1%85%B3%E1%86%AF%E1%84%85%E1%85%A2%E1%84%89%E1%85%B3%202c45623f3e774263a02c49210184c5c7/11_2.jpg)
    

### Optional 스트림 조작

```java
public Set<String> getCarInsuranceNames(List<Person> persons){
	return persons.stream()
		.map(Person::getCar)
		.map(optCar -> optCar.flatMap(Car::getInsurance))
		.map(optIns -> optIns.map(Insurance::getName))
		.flatMap(Optional::stream)
		.collect(toSet());
}
```

- 첫 번째 map 변환을 수행하고 Stream<Optional<Car>> 얻음
- 이어지는 두 개의 map 연산을 이용해 Optional<Car> 를 Optional<Insurance>로 변환
- 각각을 Optional<String> 변환

### 디폴트 액션과 Optional 언랩

Optional 인스턴스에 포함된 값을 읽는 다양한 방법

- **get()**은 값을 읽는 가장 간단한 메서드면서 동시에 가장 안전하지 않은 메서드
래핑된 값이 있으면 해당 값을 반환, 값이 없으면 NoSuchElement 예외 발생
- **orElse** 메서드를 사용하여 Optional이 값을 포함하지 않을 때 기본값 제공
- **orElseGet(Supplier<? extends T> other)** 는 orElse 메서드에 대응하는 게으른 버전의 메서드
Optional에 값이 없을 때만 Supplier 실행
- orElseThrow(Supplier<? extends X> exceptionSupplier) 는 Optional이 비어있을 때 예외를 발생시키며 발생시킬 예외의 종류를 선택 가능
- ifPresent(Consumer<? super T> consumer) 를 이용하면 값이 존재할 때 인수로 넘겨준 동작 실행 가능, 값이 없으면 아무 일도 일어나지 않음
- ifPresentOrElse(Consumer<? super T> action, Runnable emptyAction) 이 메서드는 Optional이 비었을 때 실행할 수 있는 Runnable을 인수로 받는다는 점만 ifPersent와 다름

### 두 Optional 합치기

```java
// Person과 Car 정보를 이용해서 가장 저렴한 보험료를 제공하는 보험회사를 찾는 비즈니스 로직을 구현한 외부 서비스가 있다고 가정
public Insurance findCheapestInsurance(Person person, Car car){
	// 다양한 보험회사가 제공하는 서비스 조회
	// 모든 결과 데이터 비교
	return cheapestCompany;
}

// 이제 두 Optional을 인수로 받아 Optional<Insurance> 반환하는 메서드 구현
public Optional<Insurance> nullSafeFindCheapestInsurance(
		Optional<Person> person, Optional<Car> car){
	if(person.isPresent() && car.isPresent()){
		return Optional.of(findCheapestInsurance(person.get(), car.get()))
	}
}
```

### 필터로 특정값 거르기

```java
Optional<Insurance> optInsurance = ...;
optInsurance.filter(insurance -> "CambridgeInsurance).equals(insurance.getName())
		.ifPresent(x -> System.out.println("ok"));
```

## 11.4 Optional을 사용한 실용 예제

### 잠재적으로 null이 될 수 있는 대상을 Optional로 감싸기

- 기존 자바 API에서는 null을 반환하면서 요청한 값이 없거나 계산에 실패하는 상황 발생
- 다음과 같은 코드를 이용하여 null일 수 있는 값을 Optional로 안전하게 변환
    - `Optional<Object> value = Optional.ofNullable(map.get(”key”));`

### 예외와 Optional 클래스

- 자바 API는 어떤 이유에서 값을 제공할 수 없을 때 null 반환 대신 예외를 발생시킴
- Integer.parseInt(String) → 문자열을 정수로 변환하지 못했을 때 NumberFormat 예외 발생
- 정수로 변환할 수 없는 문자열 문제를 빈 Optional로 해결

```java
public static Optional<Integer> stringToInt(String s){
	try{
		return Optional.of(Integer.parseInt(s));
	} catch{
		return Optional.empty();
	}
}
```
