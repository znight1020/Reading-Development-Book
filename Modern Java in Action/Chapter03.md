# Chap3. 람다 표현식

## 3.1 람다란 무엇인가?

**람다 표현식**은 메서드로 전달할 수 있는 익명 함수를 단순화 한 것

람다의 특징

- 익명: 보통의 메서드와 달리 이름이 없음
- 함수: 메서드처럼 특정 클래승 종속되지 않음
- 전달: 메서드 인수로 전달하거나 변수로 저장
- 간결성: 자질구레한 코드 구현을 하지 않음

람다 표현식은 세 부분으로 나누어짐

(Apple a1, Apple a2) → a1.getWeight().compareTo(a2.getWeight());

- 파라미터 리스트: Comparator의 compare 메서드 파라미터
- 화살표: 화살표 →는 람다의 파라미터 리스트와 바디 구분
- 람다 바디: 두 사과의 무게를 비교

```java
// 자바 8의 유효한 람다 표현식

(String s) → s.length() // 람다 표현식에는 return이 함축, int 반환

(Apple a) → a.getWeight() > 150 // boolean 반환

(int x, int y) -> {
	System.out.println("Result:");
	System.out.println(x + y);} // void 리턴
 
() -> 42 // 42 반환

(Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight()) // int 반환
```

## 3.2 어디에, 어떻게 람다를 사용할까?

### 함수형 인터페이스

2장에서 만든 Predicate<T> 가 함수형 인터페이스이다.

→ **Predicate<T> 는 오직 하나의 추상 메서드만 지정하기 때문**

<aside>
💡 인터페이스는 default 메서드를 포함할 수 있고, 많은 default 메서드가 있더라도 한 개의 추상 메서드라면 함수형 인터페이스

</aside>

- 전체 표현식을 함수형 인터페이스의 인스턴스로 취급

### 함수 디스크립터

**람다 표현식의 시그니처를 서술하는 메서드**

- 예를 들어, Runnable 인터페이스의 유일한 추상 메서드 run은 인수와 반환값이 없으므로 Runnable 인터페이스는 인수와 반환값이 없는 시그니처로 생각할 수 있다.

<aside>
💡 @FunctionalInterface → 함수형 인터페이스를 가리키는 어노테이션

</aside>

## 3.3 람다 활용: 실행 어라운드 패턴

![3_1](https://github.com/znight1020/Reading-Development-Book/assets/104980470/84ba9408-f1e9-40d6-a0ed-d6bb5fa1525f)

- 실제 자원을 처리하는 코드를 설정과 정리 두 과정이 둘러싸는 형태

```java
public String processFile() throws IOException{
	try (BufferedReader br = new BufferedReader(new FileReader("data.txt"))){
		return br.readLine(); // 실제 필요한 작업을 하는 행
	}
}

// 한 번에 두줄을 읽거나 자주 사용되는 단어를 반환하려면??
```

### 1단계: 동작 파라미터화를 기억하라

- processFile의 동작을 파라미터화

```java
String result = processFile((BufferedReader br) -> br.readLine() + br.readLine());
```

### 2단계: 함수형 인터페이스를 이용해서 동작 전달

- 함수형 인터페이스 `BufferedReaderProcessor` 정의

```java
@FunctionalInterface
public interface BufferedReaderProcessor {
	String processor(BufferedReader b) throws IOException;
}

// 정의한 인터페이스를 processFile 메서드의 인수로 전달
public String processFile(BufferedReaderProcessor p) throws IOException{ . . . }
```

### 3단계: 동작 실행

- processFile 바디 내에서 BufferedReaderProcessor 객체의 process() 호출

```java
public String processFile(BufferedReaderProcessor p) throws IOException{
	try(BufferedReader br = new BufferedReader(new FileReader("data.txt"))){
		return p.process(br) // BufferedReader 객체 처리
	}
}
```

### 4단계: 람다 전달

- 람다를 이용하여 다양한 동작을 processFile 메서드로 전달

```java
// 한 행 처리
String oneLine = processFile((BufferedReader br) -> br.readLine());

// 두 행 처리
String twoLines = processFile((BufferedReader br) -> br.readLine() + br.readLine());
```

## 3.4 함수형 인터페이스 사용

### Predicate

- `java.util.function.Predicate<T>` 인터페이스는 **test** 추상 메서드를 정의 → 제네릭 형식 T의 객체를 인수로 받아 boolean 반환
    - 예제 코드 3.2 참고

### Consumer

- `java.util.function.Consumer<T>` 인터페이스는 제네릭 형식 T 객체를 받아 void 반환하는 **accept** 추상 메서드 정의
    - 예제 코드 3.3 참고

### Function

- `java.util.function.Function <T, R>` 인터페이스는 제네릭 형식 T를 인수로 받아 제네릭 형식 R 객체를 반환하는 **apply** 추상 메서드 정의
    - 예제 코드 3.4 참고

<aside>
💡 **기본형 특화**
자바의 모든 형식은 참조, 기본형에 해당, 하지만 제네릭 파라미터에는 참조형만 사용

자바에서는 기본형을 참조형으로 변환하는 기능 제공 → (박싱 ↔ 언박싱)

</aside>

## 3.5 형식 검사, 형식 추론, 제약

람다 표현식 자체에는 람다가 어떤 함수형 인터페이스를 구현하는지의 정보가 포함 X

→ 람다 표현식을 제대로 이해하려면 **람다의 실제 형식 파악**

### 형식 검사

- **대상 형식**: 어떤 context 에서 기대되는 람다 표현식의 형식
    
    ![3_2](https://github.com/znight1020/Reading-Development-Book/assets/104980470/f4029748-fe1f-42b1-8160-7aa64cfe601d)
   

### 같은 람다, 다른 함수형 인터페이스

- 같은 람다 표현식이라도 호환되는 추상 메서드를 가진 다른 함수형 인터페이스로 사용 가능
- 다음 두 할당문은 모두 유효한 코드
    
    ```java
    Callable<Integer> c = () -> 42;
    PrivilegedAction<Integer> p = () -> 42;
    ```
    

### 형식 추론

**람다 표현식의 형식을 추론**
1. 할당문 콘텍스트
2. 메서드 호출 콘텍스트(파라미터, 반환값)
3. 형변환 콘텍스트

### 지역 변수 사용

- 지금까지 살펴본 모든 람다 표현식은 인수를 자신의 바디 안에서만 사용
    - 익명 함수처럼 **자유 변수**(파라미터로 넘겨진 변수가 아닌 외부 정의 변수) 활용 가능
        
        → 이와 같은 동작을 **람다 캡처링**
        
        - 하지만 지역 변수는 명시적으로 final로 선언되거나 실질적으로 final 변수와 똑같이 사용되어야만 함
        

## 3.7 람다, 메서드 참조 활용하기

처음에 다룬 사과 리스트를 다양한 정렬 기법으로 정렬하는 문제를 더 세련되고 간결하게 해결

### 1단계: 코드 전달

```java
void sort(Comparator<? super E> c)

public class AppleComparator implements Comparator<Apple> {
    @Override
    public int compare(Apple a1, Apple a2) {
      return a1.getWeight().compareTo(a2.getWeight());
	  }
}
inventory.sort(new AppleComparator());
```

### 2단계: 익명 클래스 사용

- 한 번만 사용할 Comparator를 구현하는 것보다 **익명 클래스** 이용

```java
inventory.sort(new Comparator<Apple>(){
	  public int compare(Apple a1, Apple a2) {
	    return a1.getWeight().compareTo(a2.getWeight());
	  }
});
```

### 3단계: 람다 표현식 사용

```java
inventory.sort((Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight()));
```

- 자바 컴파일러는 람다 표현식이 사용된 콘텍스트를 활용해서 람다의 파라미터 **형식 추론**
    
    → 아래와 같이 더 간결하게 나타낼 수 있음
    

```java
inventory.sort((a1, a2) -> a1.getWeight().compareTo(a2.getWeight()));
```

- 가독성을 더 향상시키기

```java
Comparator<Apple> c = Comparator.comparing((Apple a) -> a.getWeight());

//////

import static java.util.Comparator.comparing;
inventory.sort(comparing(apple -> apple.getWeight()));
```

### 4단계: 메서드 참조 사용

```java
inventory.sort(comparing(Apple::getWeight));
```

## 3.8 람다 표현식을 조합할 수 있는 유용한 메서드

- 예제 코드와 함께 설명되는 개념이므로 p.124 ~ p.128 참고

### Comparator 조합

### Predicate 조합

### Function 조합
