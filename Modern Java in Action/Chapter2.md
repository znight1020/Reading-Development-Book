# Chap2. 동작 파라미터화 코드 전달하기

시시각각 변하는 사용자 요구사항에 효과적인 대응 → **동작 파라미터화**

<aside>
💡 **동작 파라미터**
어떻게 실행할 지 결정하지 않은 코드 블록

해당 코드 블록은 나중에 프로그램에서 호출 → 코드 블록의 실행이 나중으로 미뤄짐

- 예) 컬렉션 처리 시 다음과 같은 메서드를 구현
    - 리스트의 모든 요소에 대해서 ‘어떤 동작’ 을 수행
    - 리스트 관련 작업을 끝낸 다음에 ‘어떤 다른 동작’ 을 수행
    - 에러가 발생하면 ‘정해진 어떤 다른 동작’ 을 수행
</aside>

## 2.1 변화하는 요구사항에 대응하기

기존의 농장 재고목록 애플리케이션에 리스트에서 녹색 사과만 필터링하는 기능 추가

### 첫 번째 시도: 녹색 사과 필터링

```java
enum Color {RED, GREEN}

public static List<Apple> filterGreenApples(List<Apple> inventory) {
    List<Apple> result = new ArrayList<>();
    for (Apple apple : inventory) {
      if (GREEN.equals(apple.getColor())) { // 녹색 사과 선택 시 필요한 조건
        result.add(apple);
      }
    }
    return result;
}
```

- 농부가 녹색 사과가 아닌 빨간 사과도 필터링하고 싶어한다면?
    - if 문의 조건을 빨간색으로 바꿈
        
        → 더 다양한 색 필터링 시 적절하게 대응 X
        

### 두 번째 시도: 색을 파라미터화

```java
public static List<Apple> filterApplesByColor(List<Apple> inventory, Color color) {
    List<Apple> result = new ArrayList<>();
    for (Apple apple : inventory) {
      if (apple.getColor() == color) {
        result.add(apple);
      }
    }
    return result;
}

List<Apple> greenApples = filterApplesByColor(inventory, GREEN);
List<Apple> redApples = filterApplesByColor(inventory, RED);
```

- 농부가 색 이외에도 사과의 무게를 구분하고 싶어한다면?
    - 파라미터에 weight 를 전달하여 무게를 구분
        - 구현 코드를 보면 색 필터링 코드와 대부분 중복
            
            → DRY(Don’t Repeat Yourself) 원칙을 어기는 것
            

### ~~세 번째 시도: 가능한 모든 속성으로 필터링~~(추천 X)

모든 속성 추가 시 

```java
List<Apple> greenApples = filterApplesByColor(inventory, GREEN, 0, true);
List<Apple> redApples = filterApplesByColor(inventory, null, 150, false);
```

- 형편없는 코드

## 2.2 동작 파라미터화

파라미터를 추가하는 방법이 아닌 사과의 어떤 속성에 기초해서 boolean 값을 반환

    → **프레디케이트**를 사용하여 **선택 조건을 결정하는 인터페이스** 정의

![img1](https://github.com/znight1020/Reading-Development-Book/assets/104980470/e96797ff-443e-4fc6-bd40-0945c41f0912)

- 전략 디자인 패턴 적용
    - 알고리즘 패밀리 → `ApplePredicate`
    - 전략 → `AppleGreenColorPredicate`, `AppleGreenColorPredicate`

### 네 번째 시도: 추상적 조건으로 필터링

```java
public static List<Apple> filterApples(List<Apple> inventory, ApplePredicate p) {
    List<Apple> result = new ArrayList<>();
    for (Apple apple : inventory) {
      if (p.test(apple)) {
        result.add(apple);
      }
    }
    return result;
  }

static class AppleRedAndHeavyPredicate implements ApplePredicate {

    @Override
    public boolean test(Apple apple) {
      return apple.getColor() == Color.RED && apple.getWeight() > 150;
    }

}

List<Apple> redAndHeavyApples = filterApples(inventory, new AppleRedAndHeavyPredicate());
```

- 코드/동작 전달하기
    - 이전 코드는 color, weight **변수**를 파라미터로
    - 위의 코드는 AppleRedAndHeavyPredicate **메서드**를 넘겨 **메서드의 동작을 파라미터화**

### 한 개의 파라미터, 다양한 동작

![img2](https://github.com/znight1020/Reading-Development-Book/assets/104980470/fff67ab3-2489-4716-a30f-26f1346d2ebe)

**동작 파라미터화**의 강점

- 탐색 로직
- 각 항목에 적용할 동작을 분리
    
    → 유연한 API 정의 시 중요한 역할
    

<aside>
💡 지금까지의 코드 구현은 클래스를 구현하여 인스턴스화하는 과정이 거추장스럽게 느껴질 수도 있음

</aside>

## 2.3 복잡한 과정 간소화

현재 filterApples 메서드로 새로운 동작을 전달하려면 ApplePredicate 인터페이스를 구현하는 여러 클래스를 정의 후 인스턴스화 → 번거로운 작업, 시간 낭비

### 다섯 번째 시도: 익명 클래스 사용

**익명 클래스**

- 클래스의 선언과 인스턴스화를 동시에 수행
- 하지만, 람다 표현식을 사용해 더 가독성 있는 코드 구현 가능

```java
List<Apple> redApples = filterApples(inventory, new ApplePredicate() {
      @Override
      public boolean test(Apple apple) {
        return RED.equals(apple.getColor());
  }
});

// 결국 객체를 만들고 명시적으로 새로운 동작을 정의하는 메서드를 구현해야 한다는 점은 변하지 않음
```

### 여섯 번째 시도: 람다 표현식 사용

```java
List<Apple> result = filterApples(inventory, (Apple apple) -> RED.equals(apple.getColor()));
```

### 일곱 번째 시도: 리스트 형식으로 추상화

```java
public interface Predicate<T>{
	boolean test(T t);
}

public static <T> List<T> filter(List<T> list, Predicate<T> p){
    List<T> result = new ArrayList<>();
    for(T e: list){
        if(p.test(e)) {
            result.add(e);
        }
    }
		return result;
}

// 형식 파라미터 사용
// 바나나, 오렌지, 정수, 문자열 리스트에 필터 메서드 적용 가능
```

- 지금까지 살펴 본 예제들을 통해 자바8로 인한 **유연성**과 **간결함**의 향상을 알 수 있음

![img3](https://github.com/znight1020/Reading-Development-Book/assets/104980470/57412578-4c72-429b-a2f1-bfa00655ff3c)

## 2.4 실전 예제(람다 표현식 사용)

### Comparator로 정렬하기

```java
inventory.sort( (Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight()));
```

### Runnable로 코드 블록 실행하기

```java
Thread t = new Thread( () -> System.out.println("Hello World"));
```

### Callable을 결과로 반환하기

해당 개념이 낯설겠지만, 지금 당장은 Callable 인터페이스를 이용해 결과를 반환하는 task 를 만든다는 사실만 기억 → Runnable 의 업그레이드 버전

```java

Future<String> threadName = executorService.submit( () -> Thread.currentThread().getName() );
```

### GUI 이벤트 처리하기

```java
button.setOnAction( (ActionEvent event) -> label.setText("Sent!!") );
```
