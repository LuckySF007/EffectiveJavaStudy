# Item31. 한정적 와일드카드를 사용해 API 유연성을 높이라!

## 불공변

- 매개변수화 타입은 불공변이다.
  - 즉, 서로 다른 타입 Type1, Type2가 있을 때, List<Type1>, List<Type2>는 서로 상하 관계가 없다.
    - 예를 들어서 List<String>가 List<Object>의 하위 타입이 아니라고 말하는 것이다.
      - 이게 말이 되는 이유? List<String>은 List<Object>가 하는 일을 제대로 수행 못한다. -> 리스코프 치환 원칙 위반



### 불공변의 문제점

~~~java
public class Stack<E> {
    private List<E> elements;
    private int size = 0;

    public Stack() {
        this.elements = new ArrayList<>();
    }

    public void push(E o) {
        elements.add(o);
        size++;
    }

    public <E> E pop() {
        if (size == 0) {
            throw new EmptyStackException();
        }
        E element = (E)elements.get(size);
        elements.remove(size--);
        return element;
    }

    public void pushAll(Iterable<E> src) {
        for (E e : src) {
            push(e);
        }
    }

    public boolean isEmpty() {
        return elements.size() == 0;
    }
}
~~~
- 이 메서드는 깨끗이 컴파일된다. 하지만 완벽하진 않다. 

~~~java
public static void main(String[] args) {
    Stack<Number> stack = new Stack<>();
    Iterable<Integer> integers = Arrays.asList(1);
    stack.pushAll(integers);
}
~~~

- 논리적으로 잘 동작해야 할 것 같다. 하지만 실제로는 오류 메시지가 뜬다.
  - 오류 메시지의 이유? 매개변수화 타입이 불공변이기 때문이다.
- 이러한 문제의 해결책으로 한정적 와일드카드 타입이라는 특별한 매개변수화 타입을 지원한다.



## 한정적 와일드카드 타입

- pushAll의 입력 매개변수 타입은 E의 Iterable이 아니라 E의 하위타입의 Iterable 이어야 한다.
  - 이는 와일드카드 타입으로 Iterable<? extends E>로 표현 가능하다.
- 아래 와일드카드 타입을 적절히 사용한 생산자와 소비자의 예를 볼 수 있다.
  - 유연성을 극대화하려면 원소의 생산자나 소비자용 입력 매개변수에 와일드카드 타입을 사용하면 된다.
  - 입력 매개변수가 생산자, 소비자 역할을 동시에 하는 경우, 와일드카드 타입을 쓴다고 좋은 점이 생기진 않는다.




### producer

~~~java
public void pushAll(Iterable<? extends E> src) {
		for (E e : src) {
			push(e);
		}
}
~~~
- 입력된 파라미터를 컬렉션의 원소로 옮긴다.		



### consumer

~~~java
public void popAll(Collection<? super E> dst) {
		while (!isEmpty()) {
			dst.add(pop());
		}
}
~~~
- 컬렉션 인스턴스의 원소를 변수에 옮긴다.



### 펙스(PECS)

- **producer - extends**
- **consumer - super**

- 나프탈린과 와들러는 이를 겟풋원칙이라 부름.

- 이 공식을 제대로 이용한다면 클래스 사용자는 와일드카드 타입이 쓰였다는 사실조차 모를 것이다.
  - 받아들여야 할 매개변수와 거절해야할 매개변수를 잘 구분해 작업이 이뤄진다.
  - **클래스 사용자가 와일드카드 타입을 신경 써야 한다면 그 API는 문제가 있을 가능성이 크다.**
    - 반환 타입에 한정적 와일드카드 타입 사용 시, 컴파일러가 올바른 타입을 추론하지 못해 문제가 생길 가능성이 있다.
    - 컴파일러가 올바른 타입 추론을 못하면 명시적 타입 인수를 사용해 타입을 알려주자.

- Comparable, Comparator는 언제나 소비자이다. 
  - Comparable을 예로 들어보면, Comparable<E>는 E를 소비해 선후관계를 뜻하는 정수를 반환한다.
  - 소비자는 언제나 <E> 대신 <? super E>를 사용하는 편이 낫다.
  - 그러니 Comparable<? super E>, Comparator<? super E>와 같은 형태를 사용하자.



##  타입 매개변수 vs 와일드카드

~~~java
public static <E> void swap(List<E> list, int i, int j);
public static void swap(List<?> list, int i, int j);
~~~

- 메서드 선언에 타입 매개변수가 한 번만 나오면 와일드카드로 대체하라.
  - 비한정적 타입 매개변수 -> 비한정적 와일드카드(?)
  - 한정적 타입 매개변수 -> 한정적 와일드카드(<T extends ...>)



## Swap

~~~java
public static void swapHelper(List<?> list, int i, int j) {
    list.set(i, list.set(j, list.get(i))); 
}
~~~

- 이 코드를 컴파일하면 오류 메시지가 나온다.
- 방금 꺼낸 원소를 리스트에 다시 사용하는 것이 안된다. 
  - 이유? 리스트의 타입이 List<?>로 선언되어 있다. 
    - List<?>는 null 외의 어떠한 값도 넣을 수 없다.

- private 도우미 메서드를 활용한다.

  ~~~java
  public static void swap(List<?> list, int i, int j) {
    	swapHelper(list, i, j);
  }
  
  // 와일드카드 타입을 실제 타입으로 바꿔주는 private 도우미 메서드
  public static <E> void swapHelper(List<E> list, int i, int j) {
      list.set(i, list.set(j, list.get(i))); 
  }
  ~~~

  - swapHelper는 리스트가 List<E>임을 알고 있다.
    - 즉 해당 메서드에서 꺼낸 값 타입이 항상 E, E 타입 값을 리스트에 넣어도 안전함.
  - 도우미를 통해 클라이언트는 복잡한 swapHelper의 존재를 모른 채 그 혜택을 누릴 수 있다.



## 핵심 정리

- 조금 복잡하더라도 와일드카드 타입을 적용하면 API가 훨씬 유연해진다.
  - 널리 쓰일 라이브러리 작성 시 반드시 와일드카드 타입을 적절히 사용해야 한다.
- PECS 공식을 기억하자.
  - Producer - Extends, Consumer - Super
- Comparable, Comparator는 모두 소비자다.
  - <? super E>를 사용한다.



