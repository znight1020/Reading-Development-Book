# Chap4. 스트림 소개

## 4.1 스트림이란?

- 선언형으로 컬렉션 데이터를 처리 가능
- 멀티스레드를 구현하지 않아도 데이터를 **투명하게** 병렬 처리 가능

스트림이 어떤 유용한 기능을 제공하는지 코드로 확인!

```java
// 자바 7 코드
public static List<String> getLowCaloricDishesNamesInJava7(List<Dish> dishes) {
    List<Dish> lowCaloricDishes = new ArrayList<>();
    for (Dish d : dishes) {
      if (d.getCalories() < 400) {
        lowCaloricDishes.add(d);
      }
    }
    List<String> lowCaloricDishesName = new ArrayList<>();
    Collections.sort(lowCaloricDishes, new Comparator<Dish>() {
      @Override
      public int compare(Dish d1, Dish d2) {
        return Integer.compare(d1.getCalories(), d2.getCalories());
      }
    });
    for (Dish d : lowCaloricDishes) {
      lowCaloricDishesName.add(d.getName());
    }
    return lowCaloricDishesName;
  }

--------------------------------------------------------------------------------

// 자바 8 코드
public static List<String> getLowCaloricDishesNamesInJava8(List<Dish> dishes) {
    return dishes.stream()
        .filter(d -> d.getCalories() < 400) // 400 칼로리 이하의 요리 선택
        .sorted(comparing(Dish::getCalories)) // 칼로리로 요리 선택
        .map(Dish::getName) // 요리명 추출
        .collect(toList()); // 모든 요리명을 리스트에 저장
  }

위와 같은 코드를 통해 얻는 이익
 1. 선언형으로 코드 구현 가능
 2. filter, sorted, map, collect 와 같은 여러 빌딩 블록 연산을 연결해 복잡한 데이터 처리 파이프라인 제작
```

<aside>
💡 자바 8 스트림 API의 특징
- 선언형: 더 간결하고 가독성이 좋아짐
- 조립할 수 있음: 유연성이 좋아짐
- 병렬화: 성능이 좋아짐

</aside>

## 4.2 스트림 시작하기

### 스트림의 정의

‘**데이터 처리 연산**을 지원하도록 **소스**에서 추출된 **연속된 요소**’

- 연속된 요소
    - 컬렉션과 마찬가지로 특정 요소 형식으로 이루어진 연속된 값 집합의 인터페이스 제공
    - 컬렉션의 주제가 데이터라면 스트림의 주제는 계산
- 소스
    - 컬렉션, 배열, I/O 자원 등의 데이터 제공 소스로 데이터 소비
        
        → 리스트로 스트림을 만들면 스트림의 요소는 리스트의 요소와 같은 순서 유지
        
- 데이터 처리 연산
    - 일반적으로 지원하는 연산과 데이터베이스와 비슷한 연산 지원
        - ex) filter, map, reduce, find, match, sort 등으로 데이터 조작
    - 순차적으로 또는 병렬로 실행

### 스트림의 주요 특징

- **파이프라이닝**: 스트림 연산끼리 연결해서 커다란 파이프라인 구성하도록 스트림 자신 반환
    
    → 그 덕분에 **laziness**, **short-circuiting** 같은 최적화를 얻음
    
- **내부 반복**: 반복자를 이용해서 명시적으로 반복하는 컬렉션과 달리 스트림은 내부 반복 지원

### 예제: 요리 리스트

![4_1](https://github.com/znight1020/Reading-Development-Book/assets/104980470/2151673a-8aa3-4779-9fef-35cc6fe77fe1)

```java
import static java.util.stream.Collectors.toList;
List<String> threeHighCaloricDishNames =
	menu.stream()
				.filter(dish -> dish.getCalories() > 300)
				.map(Dish::getName)
				.limit(3)
				.collect(toList());
```

- filter: 람다를 인수로 받아 스트림에서 특정 요소 제외
- map: 람다를 이용해서 한 요소를 다른 요소로 변환하거나 정보 추출
- limit: 정해진 개수 이상의 요소가 스트림에 저장되지 못하게 스트림 크기 축소
- collect: 스트림을 다른 형식으로 변환

## 4.3 스트림과 컬렉션

일상생활에서 찾아보는 스트림, 컬렉션

- 컬렉션: DVD
- 스트림: 인터넷 스트리밍

<aside>
💡 데이터를 언제 계산하느냐가 컬렉션과 스트림의 가장 큰 차이

</aside>

### 컬렉션

- 현재 자료구조가 포함하는 모든 값을 메모리에 저장
    - 모든 요소는 컬렉셔네 추가하기 전에 계산

### 스트림

- 이론적으로 **요청할 때만 요소를 계산** 하는 고정된 자료구조
    - 스트림의 요소를 추가, 삭제 불가능
    - 사용자가 요청하는 값만 스트림에서 추출

<aside>
💡 컬렉션: 생산자 중심 ← 팔기도 전에 창고를 가득 채움
스트림: 요청 중심 ← 사용자가 요청할 때만 제작

</aside>

### 딱 한 번만 탐색할 수 있다

- 반복자와 마찬가지로 스트림도 한 번만 탐색 가능 → 탐색된 스트림 요소는 소비
    - 한 번 탐색한 요소를 다시 탐색하려면 초기 데이터 소스에서 새로운 스트림 제작

### 외부 반복과 내부 반복

- 외부 반복
    - **컬렉션** 인터페이스를 사용하기 위해 사용자가 직접 요소 반복 (for - each)
        - 명시적으로 컬렉션 항목을 하나씩 가져와서 처리
    - 병렬성을 **스스로 관리**
        1. 병렬성을 포기
        2. synchronized 로 시작하는 힘들고 긴 전쟁을 시작
- 내부 반복
    - **스트림** 라이브러리는 함수에 어떤 작업을 수행할지만 지정하면 모든 것이 알아서 처리
    - 장점: 작업을 투명하게 병렬로 처리하거나 더 최적화된 다양한 순서로 처리

```java
// 컬렉션: for-each 루프 이용하는 외부 반복
List<String> names = new ArrayList<>();
for(Dish dish: menu){ // 메뉴 리스트를 명시적으로 순차 반복
	names.add(dish.getName()); // 이름을 추출해서 리스트에 추가
}

// 컬렉션: 내부적으로 숨겨졌던 반복자(iterator) 를 사용한 외부 반복

...

// 스트림: 내부 반복
List<Stirng> names = menu.stream()
											.map(Dish::getName) // map 메서드를 getName 메서드로 파라미터화해서 요리명 추출
											.collect(toList()); // 파이프라인을 실행, 반복자 필요 없음
 
```

## 4.4 스트림 연산

이전 예제 코드를 살펴보자

```java
List<String> threeHighCaloricDishNames =
	menu.stream()
				.filter(dish -> dish.getCalories() > 300)
				.map(Dish::getName)
				.limit(3)
				.collect(toList());
```

- 연산을 두 그룹으로 구분 가능
    - filter, map, limit 는 서로 연결되어 파이프라인 형성
    - collect 로 파이프라인을 실행한 다음 닫음
- 위처럼 연결할 수 있는 스트림 연산을 **중간 연산**
- 스트림을 닫는 연산을 **최종 연산**

### 중간 연산

- filter 나 sorted 같은 중간 연산은 다른 스트림을 반환
- 중요한 특징으로는 단말 연산을 스트림 파이프라인에 실행하기 전까지는 아무 연산도 수행하지 않음
    
    → **lazy** 즉, ‘게으르다’ → 중간 연산을 합친 다음에 합쳐진 중간 연산을 최종 연산으로 한 번에 처리
    

### **최종 연산**

- 스트림 파이프라인에서 결과를 도출
- List, Integer, void 등 스트림 이외의 결과가 반환

### **스트림 이용하기**

- 질의를 수행할 (컬렉션 같은) 데이터 소스
- 스트림 파이프라인을 구성할 중간 연산 연결
- 스트림 파이프라인을 실행하고 결과를 만들 최종 연산

- 중간 연산 && 최종 연산

![4_2](https://github.com/znight1020/Reading-Development-Book/assets/104980470/7cfc069a-53ff-4a4e-80bd-e2d0ce5fb77d)
