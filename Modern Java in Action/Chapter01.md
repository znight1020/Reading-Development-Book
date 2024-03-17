# Chap1. 자바 8, 9, 10, 11

## 1.1 역사의 흐름

### 자바 8버전 이후로 코드의 획기적인 변화

- 기존 자바(v8 이전) 프로그램에서는 멀티코어를 사용하기 위해서 스레드를 사용
    - 스레드는 관리하기 어렵고 많은 문제가 발생할 가능성이 있음
- 멀티코어 CPU 대중화와 같은 하드웨어적 변화가 자바 8에 영향을 미침
    - 기존 자바에서 부족했던 특성인 멀티코어 병렬성을 쉽게 이용할 수 있도록 진화

### 자바 8에서 제공하는 새로운 기술

- 스트림 API

<aside>
💡 병렬 연산을 지원하는 스트림 API
스트림을 이용하면 비용이 훨씬 비싼 **synchronized** 키워드를 사용하지 않아도 됨

</aside>

- 메서드에 코드를 전달하는 기법
- 인터페이스의 디폴트 메서드

<aside>
💡 스트림 API의 등장 → [메서드에 코드를 전달하는 기법], [인터페이스의 디폴트 메서드]

</aside>

### 사과 목록을 무게순으로 정렬해보자

```java
/* 자바 8 이전 버전 코드*/
Colletions.sort(inventory, new Comparator<Apple>(){
	public int compare(Apple a1, Apple a2){
		return a1.getWeight().compareTo(a2.getWeight());
	}
});

/*자바 8 이후 버전 코드*/
inventory.sort(comparing(Apple::getWeight));
```

## 1.2 왜 아직도 자바는 변화하는가?

### 스트림 처리

- 스트림이란, 한 번에 한 개씩 만들어지는 연속적인 데이터 항목들의 모임

<aside>
💡 **유닉스 시스템에서는 해당 명령들을 병렬로 실행**

- cat: 두 파일을 연결해서 스트림 생성
tr: 스트림의 문자 번역
sort: 스트림의 행 정렬
tail -3: 스트림의 마지막 3개 행 제공
- `cat file1 file2 | tr “[A-Z]” “[a-z]” | sort | tail -3`
    - 파이프 ‘|’ 를 이용한 명령 연결
</aside>

- 유닉스의 복잡한 명령어로 파이프라인을 구성했던 것처럼 **자바의 스트림 API는 파이프라인을 만드는 데 필요한 많은 메서드 제공**
- 자바 8에서는 기존 1개씩 수행하던 작업을 고수준으로 추상화하여 스트림으로 만들어 처리 
→ 스레드와 같은 복잡한 작업을 사용하지 않으면서도 **공짜로 병렬성을 얻을 수 있음**

### 동작 파라미터화로 메서드에 코드 전달

- sort 메서드에 파라미터를 추가하여 사용자가 원하는 순서대로 자료를 정리하고 싶은 경우
    1. 이전 자바 코드와 같이 Comparator 객체 만든 후 sort에 넘겨줌
    2. 자바 8 이후 메서드를 다른 메서드의 인수로 넘겨줌
        - 이와 같은 방법을 **동작 파라미터화** 라고 부름

### 병렬성과 공유 가변 데이터

- 스트림 메서드로 전달하는 코드는 다른 코드와 동시에 실행 하더라도 안전한 실행이 필요
    - 다른 코드와 동시에 실행 시 **공유 가변 데이터에 접근을 막아야 함** ← 순수 함수
    → 예) 두 프로세스가 공유된 변수를 동시에 접근
    - 기존처럼 synchronized 사용하게 된다면 공유 데이터를 보호할 수는 있지만 병렬이라는 목적을 무력화 시키면서 성능에 악영향을 끼치게 됨
- 자바 8 스트림을 이용하여 기존의 자바 스레드 API보다 쉽게 병렬성 활용가능

<aside>
💡 위에 설명한 동작 파라미터화, 공유되지 않은 가변 데이터의 활용은 **함수형 프로그래밍**의 핵심 사항

</aside>

## 1.3 자바 함수

- 프로그래밍 언어의 핵심은 **값의 수정 ←** 일급 시민
- 전달할 수 없는 일부 구조체 ← 이급 시민

<aside>
💡 기존의 자바에서는 int, double 과 같은 기본값은 일급 시민, method와 class 등은 이급 시민이였지만 자바 8 이후부터는 **일급 시민으로 바꿀 수 있는 기능**을 제공

</aside>

### 예제: 메서드 참조(인수에 정보(코드) 전달)

```java
// 디렉토리에서 모든 숨겨진 파일을 필터링한다고 가정
// File 클래스에는 isHidden() 메서드 존재, 이 메서드는 File클래스를 인수로 받아 boolean을 반환
// FileFilter 객체 내부에 위치한 isHidden의 결과를 File.listFiles 메서드로 전달

/* 자바 8 버전 이전 코드*/
File[] hiddenFiles = new File(".").listFiles(new FileFilter(){
	public boolean accept(File file){
		return file.isHidden(); // 숨겨진 파일 필터링
	}
});

/* 자바 8 버전 이후 코드*/

File[] hiddenFiles = new File(".").listFiles(File::isHidden);

/*
이미 isHidden이라는 함수는 준비되어 있으므로 메서드 참조 ::('이 메서드를 값으로 사용하라')
를 이용하여 listFiles에 직접 전달이 가능
*/

* 기존처럼 객체를 주고 받는 것이 아닌 메서드 참조를 만들어 정보(코드) 전달
```

### 람다 : 익명 함수

- 메서드 뿐 아니라 **람다**를 포함하여 함수도 값으로 취급 가능
    - (int x) → x + 1, x라는 인수로 호출하면 x+1 반환
- **람다 문법 형식으로 구현된 프로그램을 함수형 프로그래밍**
    - ‘함수를 일급값으로 넘겨주는 프로그램을 구현한다.’ 라고 할 수 있음

### 예제: 코드 넘겨주기

- 특정 항목을 선택해서 반환하는 동작을 **필터** 라고 한다.

```java
// 모든 녹색 사과를 선택해서 리스트를 반환하는 프로그램
public static List<Apple> filterGreenApples(List<Apple> inventory) {
    List<Apple> result = new ArrayList<>();
    for (Apple apple : inventory) {
      if ("green".equals(apple.getColor())) {
        result.add(apple);
      }
    }
    return result;
  }
// 150그램 이상의 사과 필터링
public static List<Apple> filterHeavyApples(List<Apple> inventory) {
    List<Apple> result = new ArrayList<>();
    for (Apple apple : inventory) {
      if (apple.getWeight() > 150) {
        result.add(apple);
      }
    }
    return result;
  }
```

```java
// 자바 8에서는 앞의 코드를 다음과 같이 구현할 수 있다.
public static boolean isGreenApple(Apple apple) {
    return "green".equals(apple.getColor());
  }

public static boolean isHeavyApple(Apple apple) {
    return apple.getWeight() > 150;
  }

public static List<Apple> filterApples(List<Apple> inventory, Predicate<Apple> p) {
// 메서드가 p라는 이름의 프레디케이트 파라미터로 전달됨
    List<Apple> result = new ArrayList<>();
    for (Apple apple : inventory) {
      if (p.test(apple)) {
        result.add(apple);
      }
    }
    return result;
}

// 정의된 메서드들을 아래와 같이 호출할 수 있다.
filterApples(inventory, Apple::isGreenApple);
filterApples(inventory, Apple::isHeavyApple);
```

<aside>
💡 **Predicate란 무엇인가?**
수학에서는 인수로 값을 받아 true, false로 반환하는 함수를 프레디케이트라고 한다. 자바 8에서도 Function<Apple, Boolean> 같이 코드를 구현할 수 있지만 일반적으로 Predicate<Apple>을 사용하는 것이 더 표준적인 방식

</aside>

### 메서드 전달에서 람다로

- 예제와 같이 메서드를 값으로 전달하는 것은 분명 유용하지만 한두 번만 사용할 메서드를 매번 정의하는 것은 어려움
    - 아래와 같은 코드로 더 짧고 간결하게 표현 가능
        
        ```java
        filterApples(inventory, (Apple a) -> GREEN.equals(a.getColor()) );
        
        filterApples(inventory, (Apple a) -> a.getWeight() > 150 );
        
        filterApples(inventory, (Apple a) -> a.getWeight() < 80 || RED.equals(a.getColor()) );
        /*
         람다를 사용하면 이렇게 간결해질 수 있지만 
         더 복잡한 동작을 수행하는 경우 메서드 참조를 활용하는 것이 바람직함
         -> 코드가 수행하는 일을 잘 나타내는 **명확성**이 우선시되어야 함
        */
        ```
        
    

## 1.4 스트림

<aside>
💡 예제 코드 p.55

</aside>

- 스트림 API를 이용하면 컬렉션 API와는 상당히 다른 방식으로 데이터 처리 가능
    - 컬렉션: 반복 과정을 직접 처리(for-each) 사용하여 각 요소 작업 수행: **외부 반복**
    - 스트림 API: 라이브러리 내부에서 모든 데이터 처리: **내부 반복**
- 거대한 리스트를 처리하는 데 컬렉션은 매우 비효율적, 서로 다른 CPU 코어에 작업을 할당하여 효과적으로 수행 가능

### 멀티스레딩은 어렵다

- 이전 자바에서는 멀티스레딩을 활용해 병렬성을 이용하기 쉽지 않았음
    - 각각의 스레드가 동시에 공유된 데이터 접근
    - 스레드의 잘못된 제어는 문제를 일으킬 수 있음
- 자바 8에서는 ‘컬렉션을 처리하면서 발생하는 모호함과 반복적인 코드 문제’ 그리고 ‘멀티코어 활용 어려움’ 이라는 두 문제를 모두 해결

### 스트림 API 기능

자주 반복되는 패턴으로 주어진 조건에 따라 데이터를 **필터링**

데이터를 **추출**

데이터를 **그룹화**

### 병렬화 과정

1. 5개의 사과 리스트를 두개의 CPU에 각각 2, 3개씩 분할 → **포킹 단계**
2. 각각의 CPU는 자신이 맡은 절반의 리스트 처리
3. 하나의 CPU가 두 결과를 정리
    - 구글 검색도 이와 같은 방식으로 작동

<aside>
💡 **컬렉션 vs 스트림**

- 컬렉션은 어떻게 데이터를 저장하고 접근할지에 중점
- 스트림은 데이터에 어떤 계싼을 할 것인지 묘사하는 것에 중점

컬렉션을 필터링할 수 있는 가장 빠른 방법: 

1. 컬렉션을 스트림으로 바꾸고
2. 병렬로 처리
3. 리스트로 다시 복원
</aside>

## 1.5 디폴트 메서드와 자바 모듈

- 패키지의 인터페이스를 바꿔야 하는 상황에서는 Impl 클래스의 구현을 모두 바꿔야 했음
- 자바 8에서는 인터페이스를 쉽게 바꿀 수 있도록 **디폴트 메서드를 지원**

<aside>
💡 디폴트 메서드를 이용하면 기존의 코드를 건드리지 않고도 원래의 인터페이스 설계를 자유롭게 확장 가능

여러 인터페이스의 다중 디폴트 메서드 존재 → 결국 다중 상속 허용?
    → 어느 정도는 맞는말, 9장에서 더 자세히 설명

</aside>

## 1.6 함수형 프로그래밍에서 가져온 다른 유용한 아이디어

지금까지의 핵심 아이디어 두가지

1. 메서드와 람다를 일급값으로 사용
2. 가변 공유 상태가 없는 병렬 실행을 이용 → 효율적이고 안전한 함수, 메서드 호출
    - 스트림 API는 이 두 가지 아이디어 모두 활용
- 자바 8에서는 NullPointer 예외를 피할 수 있는 Optional<T> 클래스 제공
    - 값을 갖거나 갖지 않을 수 있는 컨테이너 객체
    - 형식 시스템을 이용하여 어떤 변수에 값이 없을 때 어떻게 처리할지 명시 가능
- (구조적) 패턴 매칭 기법의 자세한 내용(스칼라 프로그래밍 언어로 패턴 매칭 사용)은 20장에서 설명