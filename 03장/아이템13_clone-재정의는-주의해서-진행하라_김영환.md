# Item13. clone 재정의는 주의해서 진행하라!

- `Cloneable`은 복제해도 되는 클래스임을 명시하는 용도의 `mixin interface`지만, 아쉽게도 의도한 목적을 제대로 이루지 못했다.
- 이유? 가장 큰 문제는 `clone` 메서드의 선언 위치다.
  `clone` 메서드가 선언된 곳이 `Cloneable`이 아닌 `Object`이고, 그마저도 `protected`다.
  이러한 이유로 `Cloneable`을 구현하는 것만으로 외부 객체에서 `clone` 메서드를 호출하는 것은 불가능하다.

- 이번 아이템에서는 `clone` 메서드를 잘 동작하게끔 해주는 구현방법, 언제 그렇게 구현 해야 하는지, 가능한 다른 방법에 대해 이야기한다.



## Cloneable 인터페이스가 하는 일

- `Object`의 `protected` 메서드인 `clone`의 동작 방식을 결정
- `Cloneable`을 구현한 클래스의 인스턴스에서 `clone` 호출 시, 그 객체의 필드들을 하나하나 복사한 객체를 반환
- `Cloneable`을 구현하지 않은 클래스의 인스턴스에서 호출 시, `CloneNotSupportedException` 발생

### Cloneable 인터페이스의 이례적인 모습

- 일반적으로 인터페이스를 구현한다는 것은 인터페이스에서 정의한 기능을 해당 클래스가 제공한다는 것을 선언하는 것이다.
- 하지만 `Cloneable`의 경우, 상위 클래스에 정의된 `protected` 메서드의 동작 방식을 변경한다.



## Clone 메서드의 허술한 일반 규약

> 허술함의 이유
>
> 실무에서 Cloneable을 구현한 클래스는 clone 메서드를 public으로 제공하며, 사용자는 당연히 복제가 제대로 이뤄질 것이라 기대한다. 이 기대를 만족시키기 위해 해당 클래스와 모든 상위 클래스는 복잡하고, 강제할 수 없으며 허술하게 기술된 프로토콜을 지켜야만 한다. 이러한 이유로 깨지기 쉽고, 위험하고, 모순적인 메커니즘이 탄생했고, 생성자를 호출하지 않고도 객체를 생성할 수 있게 되는 현상이 발생한다.

- 이 객체의 복사본을 생성해 반환한다. 

- '복사'의 정확한 뜻은 그 객체를 구현한 클래스에 따라 다를 수 있다.

- 일반적 의도는 다음과 같다. 

  ~~~java
  x.clone() != x; // true
  
  x.clone().getClass() == x.getClass(); // true
  ~~~

- 위에서 말한 일반적 의도 이상의 요구를 반드시 만족할 필요는 없다.
  ~~~java
  x.clone().equals(x); // 일반적으로 참이지만, 그것이 필수는 아니다.
  ~~~

- 관례상, `clone` 메서드가 반환하는 객체는 `super.clone` 호출을 통해 얻는다.
  ~~~java
  x.clone().getClass() == x.getClass(); // true
  ~~~

- 관례상, 반환된 객체와 원본 객체는 독립적이어야 한다. 
  이를 만족하려면 `super.clone`으로 얻은 객체 필드 중 하나 이상을 반환 전 수정해야 할 수 있다.



### 생성자 연쇄와 살짝 비슷한 매커니즘

- 강제성이 없다는 점만 빼면 생성자 연쇄와 비슷
- 즉, `clone` 메서드가 `super.clone`이 아닌, 생성자를 호출해 얻은 인스턴스를 반환해도 컴파일러는 불평하지 않을 것이다.
- 그럼 이 클래스의 하위 클래스에서 `super.clone`을 호출한다면 어떻게 될까?
  잘못된 클래스 객체가 만들어져 결국 하위 클래스의 `clone` 메서드가 제대로 동작하지 않을 것이다.
- 위의 문제를 해결하기 위해 상위 클래스의 `clone` 반환값이 아닌, 하위 클래스의 타입으로 변환한 반환값을 사용하면 된다.
- `clone`을 재정의한 클래스가 `final`이라면 위 문제에 대한 걱정을 하지 않아도 된다.(`final` 클래스는 하위 클래스 없음)



### 제대로 동작하는 clone 메서드를 가진 상위 클래스를 상속해 Cloneable을 구현할 경우

- 먼저 `super.clone`을 호출한다.
- 위 호출을 통해 얻은 객체는 완벽한 복제본이다.
- 모든 필드가 `primitive` 타입이건 불변 객체를 참조한다면 이 객체는 완벽한 상태이다. 
  - 쓸데 없는 복사를 지양한다는 관점에서 불변 클래스는 굳이 `clone` 메서드를 제공하지 않는 게 좋다.

- 클래스 선언에 `Cloneable` 구현 여부를 추가해준다.
- 재정의한 메서드의 반환 타입은 상위 클래스의 메서드가 반환하는 타입의 하위 타입일 수 있다.(자바의 공변 반환 타이핑 지원으로 가능)
- `Object`의 `clone` 메서드는 `checked exception`인 `CloneNotSupportedException`을 던지도록 설계되어 있다. `Cloneable`을 구현한 클래스의 `super.clone`이 성공할 것을 알고 있는 상태이니 `unchecked exception` 처리를 하는 것이 맞다. 그러니 `super.clone` 호출을 `try-catch` 블록으로 감싸 처리한다.



## 가변 객체를 참조하는 경우

~~~java
public class Stack {
    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public Stack() {
        this.elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }

    public void push(Object e) {
        ensureCapacity();
        elements[size++] = e;
    }

    public Object pop() {
        if (size == 0)
            throw new EmptyStackException();
        Object result = elements[--size];
        elements[size] = null; // 다 쓴 참조 해제
        return result;
    }

    private void ensureCapacity() {
        if (elements.length == size) {
            elements = Arrays.copyOf(elements, 2 * size + 1);
        }
    }
}
~~~

- `clone` 메서드가 단순히 `super.clone`의 결과를 그대로 반환한다면?
  - 반환된 `Stack` 인스턴스의 `size` 필드는 올바른 값을 가질 것.
  - `elements` 필드는 원본 `Stack` 인스턴스와 똑같은 배열을 참조할 것
  - 즉, `Deep copy`가 일어나지 않는다. 이는 불변식을 해친다.
  - 따라서 프로그램의 버그나 `NullPointerException`을 던질 것이다.
- `Stack` 클래스의 하나뿐인 생성자를 호출한다면 이러한 상황은 절대 일어나지 않는다.
  - `clone` 메서드는 사실상 생성자와 같은 효과
  - 즉, `clone`은 원본 객체에 아무런 해를 끼치지 않는 동시에 복제된 객체의 불변식을 보장해야 함.
  - 스택 내부 정보를 복사하는 쉬운 방법? `elements` 배열의 `clone`을 재귀적으로 호출해주는 것.

~~~java
public class Stack implements Cloneable {
    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public Stack() {
        this.elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }

    public void push(Object e) {
        ensureCapacity();
        elements[size++] = e;
    }

    public Object pop() {
        if (size == 0)
            throw new EmptyStackException();
        Object result = elements[--size];
        elements[size] = null; // 다 쓴 참조 해제
        return result;
    }

    private void ensureCapacity() {
        if (elements.length == size) {
            elements = Arrays.copyOf(elements, 2 * size + 1);
        }
    }

  	// 가변 상태를 참조하는 클래스용 clone 메서드
    @Override
    public Stack clone() {
        try {
            Stack clone = (Stack) super.clone();
            clone.elements = elements.clone(); // 배열을 복제할 때는 배열의 clone 메서드를 사용하길 권장
            return clone;
        } catch(CloneNotSupportedException e) {
            throw new AssertionError();
        }
    }
}
~~~

### 위 방식의 근본적인 문제점

- `elements` 필드가 `final` 이었다면? 동작하지 않는다. `final` 필드에는 새로운 값 할당이 안되기 때문이다.
- `Cloneable` 아키텍처는 '가변 객체를 참조하는 필드는 final로 선언하라'는 일반 용법을 적용할 수 없음을 이야기하기도 한다.
  - 복제 가능한 클래스를 만들기 위해 일부 필드에서 `final` 한정자 제거해야 할 수 있다.
- `clone`을 재귀적으로 호출하는 것만으로 충분하지 않은 경우가 존재한다.
  - 복제 시 원본과 같은 연결 리스트를 참조하여 원본과 복제본 모두 예기치 않게 동작할 가능성을 고려해야 한다.
  - 이 문제를 해결하기 위해 해시테이블을 구성하는 각 버킷을 구성하는 연결 리스트를 복사해야 한다.

- 연결 리스트의 복제
  - `deep copy`로 인해 자신을 재귀적으로 호출하게 된다.
  - 재귀 호출 때문에 리스트의 원소 수만큼 스택 프레임을 소비한다. 이는 리스트가 길 경우 오버플로를 일으킬 위험이 있다.
  - 이러한 문제의 해결을 위해 `deep copy` 대신 반복자를 사용해 순회하는 방식을 고려한다.



## 복잡한 가변 객체를 복제하는 방법

- super.clone을 호출 -> 얻은 객체의 모든 필드를 초기 상태로 설정

- 고수준 메서드 호출 -> 원본 객체의 상태를 다시 생성

  > 고수준 API 활용해 복제하는 경우
  >
  > 간단하고 제법 우아한 코드를 얻게 된다. 하지만 저수준에서 바로 처리하는 것보다 느리다. 또 Cloneable 아키텍처의 기초가 되는 필드 단위 객체 복사를 우회한다. 이는 전체 Cloneable 아키텍처와는 어울리지 않는 방식이다.

### 고려할 사항

- 생성자에서는 재정의될 수 있는 메서드를 호출하지 않아야 한다. `clone` 메서드도 마찬가지다.
- 만약 `clone`이 하위 클래스에서 재정의한 메서드 호출 시, 하위 클래스는 복제 과정에서 자신의 상태 교정 기회를 잃게 된다. 이는 원본과 복제본의 상태가 달라지게 할 가능성이 있다.

### 주의 사항 - 예외 처리

- 앞서 확인한 내용처럼 `Object`의 `clone` 메서드는 `CloneNotSupportedException`을 던진다. 하지만 재정의한 메서드는 그렇지 않아야 한다.
- `public`인 `clone` 메서드에서는 `throws` 절을 없애준다.
- `unchecked exception`으로의 수정을 통해 사용성을 얻는다.

### 상속해서 쓰기 위한 클래스

- 상속용 클래스는 `Cloneable`을 구현해서는 안된다.

- `Object`의 방식을 모방해 제대로 작동하는 `clone` 메서드를 구현하여 `protected`로 두고 `CloneNotSupportedException`을 던질 수 있게 선언한다.

  - 위와 같은 방식을 사용해 `Cloneable` 구현 여부를 하위 클래스에서 선택할 수 있도록 한다. 

- 다른 방법으로, `clone`을 동작하지 않게 구현하고 하위 클래스에서 재정의하지 못하게 하는 것이다.

  ~~~java
  @Override
  protected final Object clone() throws CloneNotSupportedException {
  		throw new CloneNotSupportedException();
  }
  ~~~

### clone 메서드의 동기화

- `Cloneable`을 구현한 스레드 안전 클래스 작성 시, `clone` 메서드 역시 적절한 동기화 작업 필요
- `Object`의 `clone` 메서드는 동기화를 신경쓰지 않았기 때문에, `super.clone` 호출 외에 다른 할 일이 없더라도 `clone` 재정의 및 동기화 작업이 필요하다.



## 요약

- `Cloneable`을 구현하는 모든 클래스는 `clone` 재정의 필요!
- 접근 제한자는 `public`, 반환 타입은 클래스 자신으로!
- 가장 먼저 `super.clone` 호출, 이후 필요한 필드 전부를 적절히 수정!
  - 객체 내부 '깊은 구조'에 숨은 모든 가변 객체 복사, 복제본이 가진 객체 참조는 모두 복사된 객체를 가리키게 함!
  - 내부 복사는 주로 `clone`의 재귀적 호출, 기본 타입 필드나 불변 객체 참조만 갖는 클래스라면 아무 필드 수정 필요 없음!
    - 단, 일련번호, 고유ID는 타입이나 불변 여부 상관 없이 수정 필요!



## 복사 생성자, 복사 팩터리

- `Cloneable`을 이미 구현한 클래스의 확장이라면 `clone`을 잘 작동하도록 구현하는 것이 필요하지만, 그렇지 않은 상황에서는 복사 생성자나 복사 팩터리라는 더 나은 객체 복사 방식이 있다.
- 두 방식은 `Cloneable/clone` 방식과 달리, 언어 모순적이고 위험천만한 객체 생성 매커니즘 사용을 하지 않는다. 또 엉성하게 문서화된 규약에 기대지 않고, 정상적인 `final` 필드 용법과도 충돌하지 않으며, 불필요한 검사 예외를 던지지 않고, 형변환도 필요치 않다.
- 두 방식은 해당 클래스가 구현한 '인터페이스' 타입의 인스턴스를 인수로 받을 수 있다.
  - 인터페이스 기반 복사 생성자와 복사 팩터리는 각각 '변환 생성자', '변환 팩터리'라 불린다.
  - 원본의 구현 타입에 얽매이지 않고 복제본의 타입을 직접 선택하는 것이 가능해진다. 

### 복사 생성자

- 단순히 자신과 같은 클래스의 인스턴스를 인수로 받는 생성자

~~~java
public Younghwan(Younghwan younghwani) {....};
~~~

### 복사 팩터리

- 복사 생성자를 모방한 정적 팩터리

~~~java
public static Younghwan newInstance(Younghwan younghwani) {....};
~~~



## 핵심 정리

- 앞선 내용들을 보았을 때, 새로운 인터페이스 생성 시 절대 `Cloneable`을 확장해서 사용하지 말자!
- 새로운 클래스도 마찬가지로 이를 구현하지 말자!
- `final` 클래스라면 예외적으로 `Cloneable` 구현에 위험이 적지만, 성능 최적화를 고려 후 사용하라!
- 기본적으로 복제 기능은 생성자와 팩터리를 이용하라!
- 예외적으로 배열만은 `clone` 메서드 방식이 가장 좋다.



