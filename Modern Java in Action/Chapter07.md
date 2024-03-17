# Chap7. 병렬 데이터 처리와 성능

<aside>
💡 이전 장 요약
- 스트림 인터페이스를 이용해서 데이터 컬렉션을 선언형으로 제어하는 방법
- 외부 반복을 내부 반복으로 바꿔 네이티브 자바 라이브러리가 스트림 요소 처리 제어
- 컴퓨터의 멀티코어를 활용해 파이프라인 연산을 실행할 수 있음

</aside>

이 장에서는 스트림으로 데이터 컬렉션 관련 동작을 얼마나 쉽게 병렬로 실행하는지 알아볼 수 있음

## 7.1 병렬 스트림

컬렉션에 parallelStream 호출하면 **병렬 스트림** 생성

- 병렬 스트림: 각각의 스레드에서 처리하도록 스트림 요소를 여러 청크로 분할한 스트림
    
    → 모든 멀티코어 프로세서가 각각의 청크를 처리하도록 할당 가능
    

숫자 n을 인수로 받아 1부터 n까지의 모든 숫자의 합계를 반환하는 메서드 구현

```java
public long sequentialSum(long n){
	return Stream.iterate(1L, i -> i + 1) // 무한 자연수 스트림 생성
			.limit(n) // n개 이하로 제한
			.reduce(0L, Long::sum); // 모든 숫자를 더하는 스트림 리듀싱 연산
}
```

### 순차 스트림을 병렬 스트림으로 변환하기

- 순차 스트림에 parallel 메서드 호출 → 기존의 함수형 리듀싱 연산이 병렬 처리됨

```java
public long parallelSum(long n){
	return Stream.iterate(1L, i -> i + 1) 
			.limit(n) 
			.parallel() // 스트림을 병렬 스트림으로 변환
			.reduce(0L, Long::sum); 
}
```

![7_1](https://github.com/znight1020/Reading-Development-Book/assets/104980470/7412d30c-2d00-4586-930e-c1beb31627af)

- `stream.parallel().filter(…).sequential().map(…).parallel().reduce();`
- 해당 코드 실행 시 `parallel`, `sequential` 두 메서드 중 최종적으로 parallel 이 호출 되었으므로 파이프라인은 전체적으로 병렬로 실행 됨

### 스트림 성능 측정

- 성능을 최적화할 때는 세 가지 황금 규칙을 기억
    1. 측정
    2. 측정
    3. 측정!
- 자바 마이크로벤치마크 하니스(JMH) 라이브러리 이용
    
    <aside>
    💡 JMH 를 이용하면 간단하고, 어노테이션 기반 방식 지원, 안정적인 자바 프로그램 or JVM 을 대상으로 하는 다른 언어용 벤치마크 구현 가능
    
    </aside>
    
    ```xml
    <dependency>
          <groupId>org.openjdk.jmh</groupId>
          <artifactId>jmh-core</artifactId>
          <version>1.21</version>
        </dependency>
        <dependency>
          <groupId>org.openjdk.jmh</groupId>
          <artifactId>jmh-generator-annprocess</artifactId>
          <version>1.21</version>
        </dependency>
    
    <build>
        <plugins>
          <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-shade-plugin</artifactId>
            <version>3.1.1</version>
            <executions>
              <execution>
                <phase>package</phase>
                <goals>
                  <goal>shade</goal>
                </goals>
                <configuration>
                  <finalName>benchmarks</finalName>
                  <transformers>
                    <transformer implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
                      <mainClass>org.openjdk.jmh.Main</mainClass>
                    </transformer>
                  </transformers>
                </configuration>
              </execution>
            </executions>
          </plugin>
        </plugins>
      </build>
    ```
    
    - JMH 를 사용하기 위해 pom.xml 을 작성한 것
    - 두 가지 문제 발견
        1. 반복 결과로 박싱된 객체가 만들어지므로 숫자를 더하려면 언박싱 해야 함
        2. 반복 작업은 병렬로 수행할 수 있는 독립 단위로 나누기 어려움
    - 이처럼 병렬 프로그래밍은 까다롭고 때로는 이해하기 어려운 함정이 숨어 있음
    심지어 병렬 프로그래밍 오용 시 오히려 전체 프로그램의 성능 악화
    - rangedSum(), parallelRangedSum() 메서드를 통해 순차 실행보다 빠른 성능을 갖는 병렬 리듀싱을 만듦
    
    ```java
    // Benchmark                             Mode Cnt Score      Error   Units
    // ParallelStreamBenchmark.rangedSum     avgt  40 5.315  +-  0.285   ms/op
    
    // Benchmark                                  Mode Cnt Score      Error   Units
    // ParallelStreamBenchmark.parallelRangedSum  avgt  40 2.677  +-  0.214   ms/op
    ```
    
    - 올바른 자료구조를 선택해야 병렬 실행도 최적의 성능을 발휘

### 병렬 스트림의 올바른 사용법

병렬 스트림을 잘못 사용하면서 발생하는 많은 문제는 공유된 상태를 바꾸는 알고리즘을 사용하기 때문에 발생

```java
// 다음은 n까지의 자연수를 더하면서 공유된 누적자를 바꾸는 프로그램
public long sideEffectSum(long n){
	Accumulator accumulator = new Accmulator();
	LongStream.rangeClosed(1, n).forEach(accmulator::add);
	return accmulator.total;
}

public class Accmulator{
	public long total = 0;
	public void add(long value) { total += value; }
}

// 병렬 실행 코드
public static long sideEffectParallelSum(long n) {
    Accumulator accumulator = new Accumulator();
    LongStream.rangeClosed(1, n).parallel().forEach(accumulator::add);
    return accumulator.total;
}
// 각 실행 결과 도출
System.out.println("SideEffect parallel sum done in: " + measurePerf(ParallelStreams::sideEffectParallelSum, 10_000_000L) + " msecs");

Result: 5959989000692
Result: 7425264100768
Result: 7192970417739
...
	SideEffect parallel sum done in: 49 msecs
```

- 메서드의 성능은 둘째 치고, 올바른 결과값 도출 X
- 18장, 19장에서 함수형 프로그래밍을 살펴보며 상태 변화를 피하는 방법 설명

### 병렬 스트림 효과적으로 사용하기

- 어떤 상황에서 병렬 스트림을 사용할 것인지 약간의 수량적 힌트를 정하는 것이 도움
    - 확신이 서지 않으면 직접 측정
    - 박싱 주의
    - 순차 스트림보다 병렬 스트림에서 성능이 떨어지는 연산 존재
    - 스트림에서 수행하는 전체 파이프라인 연산 비용 고려
    - 소량의 데이터에서는 병렬 스트림 도움 X
    - 스트림을 구성하는 자료구조가 적절한지 확인
    - 스트림의 특성과 파이프라인의 중간 연산이 스트림의 특성을 어떻게 바꾸는지에 따라 분해 과정의 성능 달라짐
    - 최종 연산의 병합 과정 비용을 살펴봄

| 소스 | 분해성 |
| --- | --- |
| ArrayList | 훌륭함 |
| LinkedList | 나쁨 |
| IntStream.range | 훌륭함 |
| Stream.iterate | 나쁨 |
| HashSet | 좋음 |
| TreeSet | 좋음 |

## 7.2 포크/조인 프레임워크

- 병렬화 할 수 있는 작업을 재귀적으로 작은 작업으로 분할 → 서브태스크 각각의 결과를 합쳐서 전체 결과를 만들도록 설계
- 포크/조인 프레임워크에서는 서브태스크를 스레드 풀의 작업자 스레드에 분산 할당하는 ExecutorService 인터페이스 구현

### RecursiveTask 활용

- 스레드 풀 이용하려면 RecursiveTask<R>의 서브클래스 생성 → compute 메서드 구현

<aside>
💡 compute 메서드는 다음과 같은 의사코드 형식 유지
if(태스크가 충분히 작거나 더 이상 분할할 수 없으면){
    순차적으로 태스크 계산
} else {
    태스크를 두 서브태스크로 분할
    태스크가 다시 서브태스크로 분할되도록 이 메서드를 재귀적으로 호출함
    모든 서브태스크의 연산이 완료될 때까지 기다림
    각 서브태스크의 결과를 합침
}

</aside>

- 포크/조인 프레임워크를 이용해서 병렬 합계 수행
- 코드의 경우 제공 소스 [chap07] → [ForkJoinSumCalculator.java] 확인
    
    ![7_2](https://github.com/znight1020/Reading-Development-Book/assets/104980470/898fa876-8472-4ded-9e3f-b2809396937f)
    
### 포크/조인 프레임워크를 제대로 사용하는 방법

- Join 메서드를 태스크에 호출하면 태스크가 생산하는 결과가 준비될 때까지 호출자를 블록
- RecursiveTask 내에서는 ForkJoinPool의 invoke 메서드 사용 금지
- 서브태스크에 fork 메서드를 호출해 ForkJoinPool의 일정 조절
- 포크/조인 프레임워크 이용하는 병렬 계산은 디버깅이 어려움
- 병렬 스트림에서 살펴본 것처럼 멀티코어에 포크/조인 프레임워크를 사용하는 것이 순차 처리보다 무조건 빠를 거라는 생각은 버림

### 작업 훔치기

- 코드 예제에서는 덧셈을 수행할 숫자가 10,000 개 이하 시 서브태스크 분할 중단
- 천만 개 항목을 포함하는 배열 사용 시 천 개 이상의 서브 태스크를 포크
    - 대부분의 기기의 코어는 4개이므로 천 개 이상의 서브태스크는 자원 낭비로 보임
    - 실제로 코어 개수와 관계없이 적절 크기로 분할된 많은 태스크를 포킹하는 것이 바람직

→ 포크/조인 프레임워크에서는 **작업 훔치기** 라는 기법으로 이 문제 해결

- 각각의 스레드는 자신에게 할당된 태스크를 포함하는 이중 연결 리스트를 참조하면서 작업이 끝날 때마다 큐의 헤드에서 다른 태스크를 가져와 작업을 처리
- 다른 스레드는 바쁘게 일하고 있는데 한 스레드는 할일이 다 떨어진 상황이 된다면 **다른 스레드 큐의 꼬리에서 작업을 훔쳐옴**

## 7.3 Spliterator 인터페이스

Spliterator 는 ‘분할할 수 있는 반복자’ 라는 의미

- Iterator 처럼 소스의 요소 탐색 기능을 제공하는건 같지만 **병렬 작업에 특화**
- 자바 8은 컬렉션 프레임워크에 포함된 모든 자료구조에 사용할 수 있는 디폴트 Spliteraotr 제공

```java
// Spliterator 인터페이스
public interface Spliterator<T> {
	boolean tryAdvance(Consumer<? super T> action); // 요소를 하나씩 순차적으로 소비, 탐색 요소가 남아있으면 참을 반환
	Spliterator<T> trySplit(); // Spliterator 의 일부 요소를 분할해 두 번째 Spliterator 생성
	long estimateSize(); // 탐색해야 할 요소 수 정보를 제공
	int characteristics(); // Spliterator 자체의 특성 집합을 포함하는 int 반환
}
```

### 분할 과정

- trySplit 의 결과가 null 이 될 때까지 재귀적으로 일어남
    
    ![7_3](https://github.com/znight1020/Reading-Development-Book/assets/104980470/def8bb18-1132-4ee3-9aa6-0d0fb85613f5)
    
    ### Spliterator 특성
    
    | 특성 | 의미 |
    | --- | --- |
    | ORDERED | 리스트처럼 요소에 정해진 순서가 있으므로, 이 순서에 유의 |
    | DISTINCT | x, y 두 요소를 방문했을 때 x.equals(y)는 항상 false 반환 |
    | SORTED | 탐색된 요소는 미리 정의된 정렬 순서 따름 |
    | SIZED | 크기가 알려진 소스로 생성했으므로 estimatedSize()는 정확한 값 반환 |
    | NON-NULL | 탐색하는 모든 요소는 null이 아님 |
    | IMMUTABLE | 이 Spliterator 의 소스는 불변, 탐색하는 동안 추가, 삭제, 수정 불가 |
    | CONCURRENT | 동기화 없이 Spliterator 의 소스를 여러 스레드에서 동시에 고칠 수 있음 |
    | SUBSIZED | 이 Spliterator 그리고 분할되는 모든 Spliterator는 SIZED 특성을 갖음 |

### 커스텀 Spliterator 구현하기

- p.266 함수형으로 단어 수를 세는 메서드 재구현하기 예제
