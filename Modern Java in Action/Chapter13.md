# Chap13. 디폴트 메서드

- 인터페이스를 바꾸면 이전에 해당 인터페이스를 구현했던 모든 클래스의 구현도 고쳐야 하는 경우 발생
- 자바 8에서는 이 문제를 해결하는 새로운 기능 제공
    1. 인터페이스 내부에 정적 메서드를 사용
    2. 인터페이스의 기본 구현을 제공할 수 있도록 **디폴트 메서드** 기능 사용
- 디폴트 메서드가 없던 시절에는 인터페이스에 메서드를 추가하면서 여러 문제 발생
    
    ← 인터페이스에 새로 추가된 메서드를 구현하도록 인터페이스를 구현하는 기존 클래스 수정
    
- 디폴트 메서드는 다중상속 동작이라는 유연성을 제공, 프로그램 구성에도 도움을 줌

## 13.1 변화하는 API

- 자바 그리기 라이브러리 설계자가 되었다고 가정
    - Resizable 인터페이스를 구현하는 Ellipse 라는 클래스 구현
    - 몇 개월이 지나면서 Resizable에 몇 가지 기능이 부족하다는 사실을 알게 됨

### API 버전 1

- 초기 버전은 다음과 같은 메서드 포함

```java
public interface Resizable extends Drawble{
	int getWidth();
	int getHeight();
	void setWidth(int width);
	void setHeight(int height);
	void setAbsoluteSize(int width, int height);
}

// 사용자 구현
public class Ellipse implements Resizable{
	. . .
}

public class Game(){
	public static void main(String[] args){
		List<Resizable> resizableShapes = Arrays.asList(new Square(), new Rectangle(), new Ellipse());
		Utils.paint(resizableShapes);
	}
}

public class Utils{
	public static void paint(List<Resizable> l){
		l.forEach(r -> {
			r.setAbsoluteSize(42, 42);
			r.draw();
		})
	}
}
```

### API 버전 2

- 몇 개월이 지나자 Resizable을 구현하는 Square와 Rectangle 구현을 개선해달라는 많은 요청을 받음
    - `void setRelativeSize(int wFactor, int hFactor)` 메서드 추가
        - 사용자가 겪는 문제
            1. Resizable을 구현하는 모든 클래스는 `setRelativeSize` 메서드 구현
            2. 사용자가 Ellipse를 포함하는 전체 애플리케이션을 재빌드할 때 컴파일 에러 발생

## 13.2 디폴트 메서드란 무엇인가?

- 자바 8에서는 호환성을 유지하면서 API를 바꿀 수 있도록 새로운 기능인 **디폴트 메서드** 제공
- 자신을 구현하는 클래스에서 메서드를 구현하지 않을 수 있는 새로운 메서드 시그니처 제공
- 인터페이스를 구현하는 클래스에서 구현하지 않은 메서드는 인터페이스 자체에서 기본 제공
    - 예제로 다시 돌아가 setRelativeSize() 를 수정해보자

```java
**default** void setRelativeSize(int wFactor, int hFactor){
	setAbsoluteSize(getWidth() / wFactor, getHeight() / hFactor);
} ****
```

## 13.3 디폴트 메서드 활용 패턴

### 선택형 메서드

- Iterator 인터페이스의 remove()는 잘 사용하지 않으며, 이를 빈 구현 제공
- 디폴트 메서드를 이용하면 빈 구현을 제공하지 않아도 됨

### 동작 다중 상속

- 자바의 클래스는 한 개의 클래스만 상속할 수 있지만 인터페이스는 여러 개 구현할 수 있음

```java
public class ArrayList<E> extends AbstractList<E> 
		implements List<E>, RandomAccess, Colneable, Serializable {}
```

- 다중 상속 형식
    - 결과적으로 ArrayList는 AbstractList, List, RandomAccess, Colneable, Colneable, Serializable, Iterable, Collection의 **서브형식**이 됨
- 기능이 중복되지 않는 최소의 인터페이스
    - Rotateable 인터페이스
    - Moveable 인터페이스
    - Resizable 인터페이스
        - 각각 디폴트 구현 제공
- 인터페이스 조합
    - Moster 클래스는 Rotateable, Moveable, Resizable 인터페이스의 디폴트 메서드를 자동으로 상속받음

## 13.4 해석 규칙

같은 시그니처를 갖는 디폴트 메서드를 상속받을 때 어떤 인터페이스의 디폴트 메서드를 사용하게 될까?

### 알아야 할 세 가지 해결 규칙

1. 클래스는 항상 이긴다.
    - 클래스나 슈퍼클래스에서 정의한 메서드가 디폴트 메서드보다 우선권을 갖음
2. 1번 규칙 이외의 상황에서는 서브인터페이스가 이긴다.
    - 상속관계를 갖는 인터페이스에서 같은 시그니처를 갖는 메서드를 정의할 때는 서브인터페이스가 이김 → B가 A를 상속바든낟면 B가 A를 이김
3. 여전히 디폴트 메서드의 우선순위가 결정되지 않았다면 여러 인터페이스를 상속받는 클래스가 명시적으로 디폴트 메서드를 오버라이드하고 호출해야 한다.
