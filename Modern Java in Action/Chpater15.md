# Chap15. CompletableFuture와 리액티브 프로그래밍 컨셉의 기초

최신 소프트웨어 개발 방법을 획기적으로 뒤집는 두 가지 추세

1. 애플리케이션을 실행하는 하드웨어와 관련된 것
    - 멀티코어 프로세서가 발전하면서 애플리케이션의 속도는 멀티코어 프로세서를 얼마나 잘 활용할 수 있도록 소프트웨어를 개발하는가에 따라 달라질 수 있음을 확인
2. 애플리케이션을 어떻게 구성하는가와 관련된 것
    - 인터넷 서비스에서 사용하는 애플리케이션이 증가하고 있는 현상을 반영
    - 하나의 거대한 애플리케이션 대신 작은 서비스로 애플리케이션을 나누는 것
        - 서비스가 작아진 대신 네트워크 통신이 증가
            
            → 즉 앞으로 만들 웹 App은 다양한 소스의 콘텐츠를 가져와서 사용자가 삶을 풍요롭게 만들도록 합치는 **매시업** 형태가 될 가능성이 큼
            
            ![15_1](https://github.com/znight1020/Reading-Development-Book/assets/104980470/9f1ac1ae-dfdb-4bfd-bfd0-5730e38577bc)
            
            - 이런 App을 구현하려면 인터넷으로 여러 웹 서비스에 접근
            

<aside>
💡 해당 15장에서 등장하는 코드는 실생활의 자바 코드를 거의 포함하지 않음

</aside>

## 15.1 동시성을 구현하는 자바 지원의 진화

1. 초기의 자바는 Runnable, Thread를 동기화된 클래스와 메서드를 이용해 잠구었음
2. 자바 5에서는 ExecutorService 인터페이스, 높은 수준의 결과 즉, Runnable, Thread의 변형을 반환하는 Callalbe<T> and Future<T>, 제네릭 등을 지원
3. 자바 7에서는 java.util.concurrent.RecursiveTask 추가
4. 자바 8에서는 스트림과 새로 추가된 람다 지원에 기반한 병렬 프로세싱 추가
    - 자바는 Future를 조합하는 기능을 추가하면서 동시성을 강화
5. 자바 9에서는 분산 비동기 프로그래밍을 명시적으로 지원

위와 같은 API는 매쉬업 App 즉, 다양한 웹 서비스를 이용하고 이들 정보를 실시간으로 조합해 사용자에게 제공하거나 추가 웹 서비스를 통해 제공하는 종류의 App을 개발하는데 필수적인 기초 모델과 툴킷 제공

→ 이 과정을 ‘**리액티브 프로그래밍’** 이라고 부르며 **‘발행-구독 프로토콜’** 로 이를 지원한다.

### 스레드와 높은 수준의 추상화

멀티코어 설정에서 스레드의 도움없이 프로그램이 노트북의 컴퓨팅 파워를 모두 활용할 수 없음

→ 각 코어는 한 개 이상의 프로세스나 스레드에 할당될 수 있지만 프로그램이 스레드를 사용하지 않는다면 효율성을 고려해 여러 프로세서 코어 중 한 개만을  사용할 것임

실제로 4개의 코어를 가진 CPU에서 이론적으로는 프로그램을 4개의 코어에서 병렬로 실행함으로써 실행 속도를 4배까지 향상시킬 수 있음

```java
// 숫자 1,000,000 개를 저장한 배열을 처리하는 예제

// not using thread
long sum = 0;
for (int i = 0; i < 1_000_000; i++){
	sum += stats[i];
}

// using thread
long sum0 = 0;
for (int i = 0; i < 250_000; i++){
	sum0 += stats[i];
}
. . .
for (int i = 750_000; i < 1_000_000; i++){
	sum3 += stats[i];
}
sum = Arrays.stream(stats).parallel().sum();
```

- 스트림을 이용해 스레드 사용 패턴을 **추상화**
    - 쓸모 없는 코드가 라이브러리 내부로 구현되면서 복잡성도 줄어듦
- java.util.concurrent.RecursiveTask 로 인해 포크/조인 스레드 추상화로 분할 그리고 정복 알고리즘을 병렬화하면서 멀티코어 머신에서 배열의 합을 효율적으로 계산하는 높은 수준의 방식 제공

### 스레드에 무엇을 바라는가?

일반적으로 모든 하드웨어 스레드를 활용해 병렬성의 장점을 극대화하도록 만드는 것 즉, 프로그램을 작은 task 단위로 구조화하는 것이 목표

## 15.2 동기 API와 비동기 API

두 가지 단계로 병렬성을 이용

- 첫 번째로 외부 반복(명시적 for 루프) 을 내부 반복(스트림 메서드 사용)으로 바꾸어야 함
- 스트림에 parallel() 메서드를 이용하므로 자바 런타임 라이브러리가 복잡한 스레드 작업을 하지 않고 병렬로 요소가 처리되도록 할 수 있음

```java
// 같은 시그니처를 갖는 f, g 두 메서드의 호출을 합하는 예제
int f(int x);
int g(int g);

// 두 메서드를 호출하고 합계를 출력
int y = f(x);
int z = g(x);
System.out.println(y + z);
```

- f와 g를 실행하는데 오랜 시간이 걸린다고 가정
    - f와 g가 서로 상호작용하지 않는다는 사실을 알고 있거나 상호작용을 전혀 신경쓰지 않는다면 f와 g를 별도의 CPU 코어로 실행
    - 별도의 스레드로 f와 g를 실행해 이를 구현
        - 의도는 좋지만 단순했던 코드가 복잡하게 변함

```java
public class ExecutorServiceExample{
	public static void main(String[] args){
		throws ExecutionException, InterruptedException{
			int x = 1337;
			
			ExecutorService executorService = Executors.newFixedThreadPool(2);
			Future<Integer> y = executorService.submit(() -> f(x));
			Future<Integer> z = executorService.submit(() -> g(x));
			System.out.println(y.get() + z.get());
			
			executorService.shutdown();
		}
	}
}
```

- Runnable 대신 Future API 인터페이스를 이용해 코드를 단순화할 수 있지만 여전히 해당 코드도 명시적인 submit 메서드 호출 같은 불필요한 코드로 오염
- 문제의 해결은 **비동기 API**라는 기능으로 API를 바꿔서 해결
    - 첫 번째 방법인 자바의 Future를 이용하면 이 문제를 조금 개선시킬 수 있음

### Future 형식 API

```java
Future<Integer> f(int x);
Future<Integer> g(int x);

Future<Integer> y = f(x);
Future<Integer> z = g(x);
System.out.println(y.get() + z.get());
```

- 메서드 f는 호출 즉시 자신의 원래 바디를 평가하는 태스크를 포함한 Future를 반환
- 마찬가지로, 메소드 g도 Future를 반환
- 세 번째 코드는 get() 메서드를 이요해 두 Future가 완료되어 결과가 합쳐지기를 기다림

### 리액티브 형식 API

- 두 번째 대안에서 핵심은 f, g의 시그니처를 바꿔서 콜백 형식의 프로그래밍을 이용하는 것
    - `void f(int x, IntConsumer dedalWithResult);`

```java
public class CallbackStyleExample{
	public static void main(String[] args){
			int x = 1337;
			
			Result result = new Result();
			
			f(x, (int y) -> {
				result.left = y;
				System.out.println((result.left + result.right))
			});
			
			g(x, (int z) -> {
				result.right = z;
				System.out.println((result.left + result.right))
			});
	}
}
```

- f와 g의 호출 합계를 정확하게 출력하지 않고 상황에 따라 먼저 계산된 결과를 출력
- 다음처럼 두 가지 방법으로 이 문제를 보완
    - if-then-else를 이용해 적절한 락을 이용해 두 콜백이 모두 호출되었는지 확인한 다음 출력
    - 리액티브 형식의 API는 보통 한 결과가 아니라 일련의 이벤트에 반응하도록 설계되었으므로 Future를 이용하는 것이 더 적절

### 잠자기(그리고 기타 블로킹 동작)는 해로운 것으로 간주

sleep() 메서드를 사용하면 스레드는 잠들어도 여전히 시스템 자원을 점유

- 스레드 풀에서 잠을 자는 태스크는 다른 태스크가 시작되지 못하게 막으므로 자원을 소비
- 모든 블록 또한 마찬가지

### 현실성 확인

- 현실적으로 ‘모든 것은 비동기’ 라는 설계 원칙을 어겨야 함

## 15.3 박스와 채널 모델

동시성 모델을 가장 잘 설계하고 개념화하려면 그림이 필요

→ 우리는 이 기법을 **박스와 채널 모델** 이라고 부름

```java
int t = p(x);
Future<Integer> a1 = executorService.submit(() -> q1(t));
Future<Integer> a2 = executorService.submit(() -> q2(t));
System.out.println( r(a1.get(), a2.get()));
```

- 이 예제에서는 박스와 채널 다이어그램의 모양상 p와 r을 Future로 감싸지 않음
- 병렬성을 극대화하려면 모든 다섯 함수를  Future로 감싸야 함
- 시스템에서 많은 작업이 동시에 실행되고 있지 않다면 이 방법도 잘 동작할 수 있음
    - 하지만, 시스템이 커지고 내부적으로 자신만의 박스와 채널을 사용하면 문제가 달라짐
    - 위와 같은 상황은 많은 task가 get() 메서드를 호출해 Future가 끝나기를 기다리는 상태에 놓일 수 있음
        - 자바 8에서는 CompletableFuture와 콤비네이터를 이용해 문제를 해결

## 15.4 CompletableFuture와 콤비네이터를 이용한 동시성

- 자바 8에서는 Future 인터페이스의 구현인 CompletableFuture를 이용해 Future를 조합할 수 있는 기능을 추가
- f(x)와 g(x)를 동시에 실행해 합계를 구하는 코드를 다음처럼 구현

```java
public class CFComplete {

  public static void main(String[] args) throws ExecutionException, InterruptedException {
      ExecutorService executorService = Executors.newFixedThreadPool(10);
      int x = 1337;

      CompletableFuture<Integer> a = new CompletableFuture<>();
      executorService.submit(() -> a.complete(f(x)));
      int b = g(x);
      System.out.println(a.get() + b);

      executorService.shutdown();
  }

}
```

- 위 코드는 f(x)의 실행이 끝나지 않거나 아니면 g(x)의 실행이 끝나지 않는 상황에서 get()을 기다려야 하므로 프로세싱 자원을 낭비할 수 있음
    
    → 자바 8의 CompletableFuture를 이용하면 이 상황을 해결할 수 있음
    

```java
public class CFCombine {

  public static void main(String[] args) throws ExecutionException, InterruptedException {
      ExecutorService executorService = Executors.newFixedThreadPool(10);
      int x = 1337;

      CompletableFuture<Integer> a = new CompletableFuture<>();
      CompletableFuture<Integer> b = new CompletableFuture<>();
      CompletableFuture<Integer> c = a.thenCombine(b, (y, z)-> y + z);
      executorService.submit(() -> a.complete(f(x)));
      executorService.submit(() -> b.complete(g(x)));

      System.out.println(c.get());
      executorService.shutdown();
  }

}
```

- thenCombine 행이 핵심

## 15.5 발행-구독 그리고 리액티브 프로그래밍

Future와 CompletableFuture은 독립적 실행과 병렬성이라는 정식적 모델에 기반

→ 연산이 끝나면 get()으로 Future의 결과를 얻음, 따라서 **한 번**만 실행해 결과 제공

반면, 리액티브 프로그래밍은 시간이 흐르면서 여러 Future 같은 객체를 통해 여러 결과를 제공

- 프로그램이 스트림 모델에 잘 맞는 상황이라면 가장 좋은 구현이 될 수 있음
    - 하지만, 보통 리액티브 프로그래밍 패러다임은 비싼 편
    - 주어진 자바 스트림은 한 번의 단말 동작으로 소비될 수 있음
- 자바 9에서는 java.util.concurrent.Flow 의 인터페이스에 발행-구독 모델을 적용해 리액티브 프로그래밍 제공
- 세 가지 플로 API 정리
    1. **구독자**가 구독할 수 있는 **발행자**
    2. 이 연결을 **구독**이라 한다.
    3. 이 연결을 이용해 **메시지**(또는 **이벤트**로 알려짐)을 전송

### 두 플로를 합치는 예제

```java
public class SimpleCell implements Publisher<Integer>, Subscriber<Integer> {

  private int value = 0;
  private String name;
  private List<Subscriber<? super Integer>> subscribers = new ArrayList<>();

  public static void main(String[] args) {
    SimpleCell c3 = new SimpleCell("C3");
    SimpleCell c2 = new SimpleCell("C2");
    SimpleCell c1 = new SimpleCell("C1");

    c1.subscribe(c3);

    c1.onNext(10); // C1의 값을 10으로 갱신
    c2.onNext(20); // C2의 값을 20으로 갱신
  }

  public SimpleCell(String name) {
    this.name = name;
  }

  @Override
  public void subscribe(Subscriber<? super Integer> subscriber) {
    subscribers.add(subscriber);
  }

  public void subscribe(Consumer<? super Integer> onNext) {
    subscribers.add(new Subscriber<>() {

      @Override
      public void onComplete() {}

      @Override
      public void onError(Throwable t) {
        t.printStackTrace();
      }

      @Override
      public void onNext(Integer val) {
        onNext.accept(val);
      }

      @Override
      public void onSubscribe(Subscription s) {}

    });
  }

  private void notifyAllSubscribers() {
    subscribers.forEach(subscriber -> subscriber.onNext(value));
  }

  @Override
  public void onNext(Integer newValue) {
    value = newValue;
    System.out.println(name + ":" + value);
    notifyAllSubscribers();
  }

  @Override
  public void onComplete() {}

  @Override
  public void onError(Throwable t) {
    t.printStackTrace();
  }

  @Override
  public void onSubscribe(Subscription s) {}
}

/*
발행자 -> 구독자로 데이터가 흐르것에 착안해 개발자는 이를 **업스트림 or 다운스트림** 이라 부름
위 예제에서 데이터 newValue는 업스트림 onNext() 메서드로 전달되고 
notifyAllSubscribers() 호출을 통해 다운스트림 onNext() 호출로 전달된다.
*/
```

- 플로 인터페이스의 개념을 복잡하게 만든 두 가지 기능은 **압력, 역압력**
    - 스레드 활용에서 이들 기능은 필수
- 기존 온도계 예제에서 온도계가 매 초마다 온도를 보고했는데 기능이 업그레이드되면서 매밀리초마다 온도계를 보고한다고 가정
    - 우리 프로그램이 이렇게 빠른 속도로 발생하는 이벤트를 아무 문제없이 처리 가능?
        
        ⇒ 이런 상황을 **압력**이라고 부름
        

## 15.6 리액티브 시스템 vs 리액티브 프로그래밍

- 리액티브 시스템
    - 런타임 환경이 변화에 대응하도록 전체 아키텍처가 설계된 프로그램을 가리킴
    - 리액티브 시스템이 가져야 할 공식적인 속성
        - 반응성: 작업을 처리하느라 질의의 응답을 지연하지 않고 실시간으로 입력에 반응
        - 회복성: 한 컴포넌트의 실패로 전체 시스템이 실패하지 않음
        - 탄력성: 시스템이 자신의 작업 부하게 맞게 적응하며 작업을 효율적으로 처리함
- java.util.concurrent.Flow 관련된 자바 인터페이스에서 제공하는 **리액티브 프로그래밍** 형식을 이용하는 것도 주요 방법 중 하나
- 이들 인터페이스 설계는 Reactive Manifesto의 네 번째이자 마지막 속성 즉 **메시지 주도**속성을 반영
