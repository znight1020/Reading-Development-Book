# Chap9. 리팩터링, 테스팅, 디버깅

## 9.1 가독성과 유연성을 개선하는 리팩터링

### 코드 가독성 개선

일반적으로 코드 가독성이 좋다는 것은 ‘어떤 코드를 다른 사람도 쉽게 이해할 수 있음’을 의미

- 코드의 가독성을 높이려면
    - 코드의 문서화
    - 표준 코딩 규칙 준수

세 가지의 리팩터링 예제

1. 익명 클래스를 람다 표현식으로 리팩터링하기
2. 람다 표현식을 메서드 참조로 리팩터링하기
3. 명령형 데이터 처리를 스트림으로 리팩터링하기

### 익명 클래스를 람다 표현식으로 리팩터링

익명 클래스의 많은 코드양과 에러가 발생되기 쉬움

```java
// 익명 클래스를 사용한 이전 코드
Runnable r1 = new Runnable() {
	public void run(){
		System.out.println("Hello");
	}
}

// 람다 표현식을 사용한 최신 코드
Runnable r2 = () -> System.out.println("Hello");
```

하지만 모든 익명 클래스를 람다 표현식으로 변환할 수 있지는 않음

1. 익명 클래스에서 사용하는 this, super는 람다 표현식에서 다른 의미를 갖음
    - 익명 클래스 this: 익명클래스 자신
    - 람다 this: 람다를 감싸는 클래스
2. 익명 클래스는 감싸고 있는 클래스의 변수를 가릴 수 있다
3. 익명 클래스를 람다 표현식으로 바꾸면 콘텍스트 오버로딩에 따른 모호함 초래

```java
interface Task{
	public void execute();
}
public static void doSomething(Runnable r){r.run();}
public static void doSomething(Task a){r.execute();}

// Task 를 구현하는 익명 클래스를 전달할 수 있음
doSomeThing(new Task()){
	public void execute(){
		System.out.println("Danger danger!!");
	}
}

익명 클래스를 람다 표현식으로 바꾸면 메서드를 호출할 때 Runnable과 Task 모두 대상 형식이 될 수 있음
즉, doSomeThing(Runnable), doSomeThing(Task) 중 어느 것을 가리키는지 알 수 없는 모호함
	-> 명시적 형변환으로 해결
```

### 람다 표현식을 메서드 참조로 리팩터링

람다 표현식은 쉽게 전달할 수 있는 짧은 코드이지만 메서드 참조를 이용하면 가독성을 높일 수 있음

### 명령형 데이터 처리를 스트림으로 리팩터링

이론적으로는 반복자를 이용한 기존의 모든 컬렉션 처리 코드를 스트림 API로 바꿔야 함

→ 스트림 API는 데이터 처리 파이프라인의 의도를 더 명확하게 보여주기 때문

```java
// 두 가지 패턴(필터링과 추출) 로 엉킨 코드: 코드의 의도 파악이 어려울 뿐더러 해당 코드를 병렬로 실행하기 매우 어려움
List<String> dishNames = new ArrayList<>();
for(Dish dish: menu){
	if(dish.getCalories() > 300){
		dishNames.add(dish.getName());
	}
}

// 스트림 API 를 이용하면 문제를 더 직접적으로 기술하고 쉽게 병렬화 가능
menu.parallelStream()
	.filter(d -> d.getCalories() > 300)
	.map(Dish::getName)
	.collect(toList());
```

### 코드 유연성 개선: 함수형 인터페이스 적용

- **조건부 연기 실행**
    
    다음은 내장 자바 Logger 클래스를 사용한 예제
    
    ```java
    if (logger.isLoggable(Log.FINER)){
    	logger.finer("Problem: " + generateDiagnostic());
    }
    
    // 위 코드는 다음과 같은 사항에 문제가 있음
    - logger의 상태가 isLoggable 이라는 메서드에 의해 클라이언트 코드로 노출
    - 메시지를 로깅할 때마다 logger 객체의 상태를 매번 확인
    
    // 다음처럼 메시지를 로깅하기 전에 logger 객체가 적절한 수준으로 설정되었는는지 내부적으로 확인
    logger.log(Level.FINER, "Problem: " + generateDiagnostic())
    ```
    
    아쉽게도 위 코드로 모든 문제 해결 X
    
    → 인수로 전달된 메시지 수준에서 logger 가 활성화되어 있지 않더라고 항상 로깅 메시지리를 평가하게 됨
    
    - 자바 8 API 설계자는 이와 같은 logger 문제를 해결할 수 있도록 Supplier를 인수로 갖는 오버로드된 log 메서드 제공
    
    ```java
    public void log(Level level, Supplier<String> msgSupplier)
    logger.log(Level.FINER, () -> "Problem: " + generateDiagnostic())
    
    // log 메서드의 내부 구현
    public void log(Level level, Supplier<String> msgSupplier){
    	if(logger.isLoggable(level)){
    		log(level, msgSupplier.get()); // 람다 실행
    	}
    }
    ```
    
- **실행 어라운드**
    
    매번 같은 준비, 종료 과정을 반복적으로 수행하는 코드가 있다면 이를 람다로 변환
    

## 9.2 람다로 객체지향 디자인 패턴 리팩터링

람다를 이용하면 이전 디자인 패턴으로 해결하던 문제를 더 쉽고 간단하게 해결 가능

### 전략

- 한 유형의 알고리즘을 보유한 상태에서 런타임에 적절한 알고리즘 선택
- 예를 들어 오직 소문자 또는 숫자로 이루어져야 하는 텍스트 입력이 다양한 조건에 맞게 포맷되어 있는지 검증
    
    ```java
    interface ValidationStrategy {
        boolean execute(String s);
    }
    
    static private class IsAllLowerCase implements ValidationStrategy {
        @Override
        public boolean execute(String s) {
          return s.matches("[a-z]+");
        }
    }
    
    static private class IsNumeric implements ValidationStrategy {
        @Override
        public boolean execute(String s) {
          return s.matches("\\d+");
        }
    }
    
    static private class Validator {
        private final ValidationStrategy strategy;
        public Validator(ValidationStrategy v) {
          strategy = v;
        }
        public boolean validate(String s) {
          return strategy.execute(s);
        }
    }
    
    // 기존 검증 방식
    Validator v1 = new Validator(new IsNumeric());
    System.out.println(v1.validate("aaaa"));
    Validator v2 = new Validator(new IsAllLowerCase());
    System.out.println(v2.validate("bbbb"));
    
    // 람다 표현식 사용
    Validator v3 = new Validator((String s) -> s.matches("\\d+"));
    System.out.println(v3.validate("aaaa"));
    Validator v4 = new Validator((String s) -> s.matches("[a-z]+"));
    System.out.println(v4.validate("bbbb"));
    ```
    

### 템플릿 메서드

- 알고리즘의 일부를 고칠 수 있는 유연함을 제공해야 할 때 사용
- 간단한 온라인 뱅킹 애플리케이션을 구현한다고 가정

```java
public class OnlineBankingLambda {

  public static void main(String[] args) { // 람다 사용
    new OnlineBankingLambda().processCustomer(1337, (Customer c) -> System.out.println("Hello!"));
  }

  public void processCustomer(int id, Consumer<Customer> makeCustomerHappy) {
    Customer c = Database.getCustomerWithId(id);
    makeCustomerHappy.accept(c);
  }

  // 더미 Customer 클래스
  static private class Customer {}

  // 더미 Database 클래스
  static private class Database {
	  static Customer getCustomerWithId(int id) {
      return new Customer();
    }
  }
}
```

### 옵저버

- 어떤 이벤트가 발생했을 때 한 객체(주제)가 다른 객체 리스트(옵저버)에 자동으로 알림을 보내야 하는 상황에서 사용
- 트위터 같은 커스터마이즈된 알림 시스템을 설계하고 구현

```java
public static void main(String[] args) { // 람다 사용
    Feed f = new Feed();
    f.registerObserver(new NYTimes());
    f.registerObserver(new Guardian());
    f.registerObserver(new LeMonde());
    f.notifyObservers("The queen said her favourite book is Java 8 & 9 in Action!");

    Feed feedLambda = new Feed();

    feedLambda.registerObserver((String tweet) -> {
      if (tweet != null && tweet.contains("money")) {
        System.out.println("Breaking news in NY! " + tweet);
      }
    });
    feedLambda.registerObserver((String tweet) -> {
      if (tweet != null && tweet.contains("queen")) {
        System.out.println("Yet another news in London... " + tweet);
      }
    });
}

interface Observer {
  void inform(String tweet);
}

interface Subject {
  void registerObserver(Observer o);
  void notifyObservers(String tweet);
}

static private class NYTimes implements Observer {
  @Override
  public void inform(String tweet) {
    if (tweet != null && tweet.contains("money")) {
      System.out.println("Breaking news in NY!" + tweet);
    }
  }
}

static private class Guardian implements Observer {
  @Override
  public void inform(String tweet) {
    if (tweet != null && tweet.contains("queen")) {
      System.out.println("Yet another news in London... " + tweet);
    }
  }
}

static private class LeMonde implements Observer {
  @Override
  public void inform(String tweet) {
    if (tweet != null && tweet.contains("wine")) {
      System.out.println("Today cheese, wine and news! " + tweet);
    }
  }
}

static private class Feed implements Subject {
  private final List<Observer> observers = new ArrayList<>();
  @Override
  public void registerObserver(Observer o) {
    observers.add(o);
  }
  @Override
  public void notifyObservers(String tweet) {
    observers.forEach(o -> o.inform(tweet));
  }
}
```

### 의무체인

- 작업 처리 객체의 체인(동작 체인 등)을 만들 때 사용, 한 객체가 어떤 작업을 처리한 다음 다른 객체로 결과 전달, 다른 객체도 해야 할 작업 처리 한 다음 또 다른 객체로 전달
- 작업 처리 객체 예시

```java
private static abstract class ProcessingObject<T> {
  protected ProcessingObject<T> successor;
  public void setSuccessor(ProcessingObject<T> successor) {
    this.successor = successor;
  }
  public T handle(T input) {
    T r = handleWork(input);
    if (successor != null) {
      return successor.handle(r);
    }
    return r;
  }
  abstract protected T handleWork(T input);
}

private static class HeaderTextProcessing extends ProcessingObject<String> {
    @Override
    public String handleWork(String text) {
      return "From Raoul, Mario and Alan: " + text;
    }
}
private static class SpellCheckerProcessing extends ProcessingObject<String> {
    @Override
    public String handleWork(String text) {
      return text.replaceAll("labda", "lambda");
    }
}

// 다음과 같이 andThen 메서드로 이들 함수를 조합해 체인을 생성
ProcessingObject<String> p1 = new HeaderTextProcessing();
ProcessingObject<String> p2 = new SpellCheckerProcessing();
p1.setSuccessor(p2);
String result1 = p1.handle("Aren't labdas really sexy?!!");
System.out.println(result1);

UnaryOperator<String> headerProcessing = (String text) -> "From Raoul, Mario and Alan: " + text;
UnaryOperator<String> spellCheckerProcessing = (String text) -> text.replaceAll("labda", "lambda");
Function<String, String> pipeline = headerProcessing.andThen(spellCheckerProcessing);
String result2 = pipeline.apply("Aren't labdas really sexy?!!");
System.out.println(result2);
```

### 팩토리

- 인스턴스화 로직을 클라이언트에 노출하지 않고 객체를 만들 때 사용
- 다양한 상품을 만드는 Factory 클래스

<aside>
💡 p.311 참고

</aside>

## 9.3 람다 테스팅

일반적으로 좋은 소프트웨어 공학자라면 프로그램이 의도대로 동작하는지 확인할 수 있는 **단위 테스팅** 진행

```java
// Point 클래스가 있고 생성자, getter 가 정의 되어있으며, moveRightBy(int x) 메서드를 포함한다고 하자
public Point moveRightBy(int x){
	return new Point(this.x + x, this.y + y);
}

// 다음은 moveRightBy 메서드가 의도한 대로 동작하는지 확인하는 단위 테스트
@Test
public void testMoveRightBy() throws Exception{
	Point p1 = new Point(5,5);
	Point p2 = p1.moveRightBy(10);
	assertEquals(15, p2.getX());
	assertEquals(5, p2.getY());
}
```

### 보이는 람다 표현식의 동작 테스팅

- 테스트 케이스 내부에서 Point 클래스 코드를 테스트할 수 있음
- 하지만 람다는 익명이므로 테스트 코드 이름을 호출할 수 없음
    
    → 따라서, 필요하다면 람다를 필드에 저장해서 재사용할 수 있으며 람다의 로직을 테스트 할 수 있음
    

### 람다를 사용하는 메서드의 동작에 집중

- 람다의 목표는 정해진 동작을 다른 메서드에서 사용할 수 있도록 하나의 조각으로 캡슐화하는 것 → 세부 구현을 포함하는 람다 표현식을 공개하지 말아야 함

### 복잡한 람다를 개별 메서드로 분할하기

- 복잡한 람다 표현식을 어떻게 테스트 할 것인지?
    - 8절에서 설명한 것처럼 람다 표현식을 메서드 참조로 바꿈

### 고차원 함수 테스팅

- 함수를 인수로 받거나 다른 함수를 반환하는 메서드(고차원 함수) 는 사용하기 어려움
- 이때는 함수형 인터페이스의 인스턴스로 간주하고 함수의 동작을 테스트

## 9.4 디버깅

문제가 발생한 코드를 디버깅할 때 다음 두 가지 먼저 확인

- 스택 트레이스
- 로깅

하지만 람다 표현식과 스트림은 기존의 디버깅 기법을 무력화

### 스택 트레이스 확인

프로그램이 멈췄다면 프로그램이 어떻게 멈추게 되었는지 프레임별로 보여주는 스택 트레이스를 얻을 수 있음 → 즉, 문제가 발생한 지점에 이르게 된 메서드 호출 리스트를 얻을 수 있음

- 람다와 스택 트레이스
    - 람다 표현식은 이름이 없기 때문에 조금 복잡한 스택 트레이스 발생
        
        → at Debugging$$Lambda$5/2874720968.apply(Unknown Source)
        
    - 미래의 자바 컴파일러가 개선해야 할 일

### 정보 로깅

스트림의 파이프라인 연산을 디버깅한다고 가정, 다음처럼 forEach 로 스트림 결과를 출력, 로깅

```java
List<Integer> numbers = Arrays.asList(2, 3, 4, 5);

numbers.stream()
	.map(x -> x + 17)
	.filter(x -> x % 2 == 0)
	.limit(3)
	.forEach(System.out::println);

// 출력 결과
20
22
```

- forEach를 호출하는 순간 전체 스트림 소비, 스트림 파이프라인에 적용된 각각의 연산(map, filter, limit) 이 어떤 결과를 도출하는지 확인하고 싶다면 → **peek**

```java
public static void main(String[] args) {
    List<Integer> result = Stream.of(2, 3, 4, 5)
        .peek(x -> System.out.println("taking from stream: " + x)) // 소스에서 처음 소비한 요소 출력
        .map(x -> x + 17)
        .peek(x -> System.out.println("after map: " + x)) // map 동작 실행 결과 출력
        .filter(x -> x % 2 == 0)
        .peek(x -> System.out.println("after filter: " + x)) // limit 동작 후 선택된 숫자 출력
        .limit(3)
        .peek(x -> System.out.println("after limit: " + x))
        .collect(toList());
}
```

<aside>
💡 peek 은 스트림의 각 요소를 소비한 것처럼 동작을 실행
peek 은 자신이 확인한 요소를 파이프라인의 다음 연산으로 그대로 전달

</aside>
