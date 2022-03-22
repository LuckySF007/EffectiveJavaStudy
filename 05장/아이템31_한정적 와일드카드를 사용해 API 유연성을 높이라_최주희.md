# 아이템31.한정적 와일드카드를 사용해 API 유연성을 높이라

매개변수화 타입은 불공변 → List<Type1>은 List<Type2>의 하위타입도 상위타입도 아니다.

때로는 불공변 방식보다 유연한 무언가가 필요함

```java
public class Stack<E> {
    public Stack();
    public void push(E e);
    public E pop();
    public boolean isEmpty();
}
```

```java
//일련의 원소를 스택에 넣는 메서드 pushAll
//와일드카드 타입을 사용하지 않음 -> 결함
public void pushAll(Iterable<E> src) {
    for(E e : src)
        push(e);
}
```

위 메서드는 결함이 있다.

src의 원소타입이 스택의 원소 타입과 일치하면 잘 작동하지만,

만약 Stack<Number> 원소에 Integer 타입의 src를 넣으면 어떻게 될까?

Integer는 Number의 하위타입이기 때문에 잘 동작할 것 같지만 오류가 뜬다.

매개변수화 타입이 불공변이기 때문이다.

→ 한정적 와일드카드 타입 사용하기

<br>

## **한정적 와일드카드 타입**

위 pushAll 메서드의 입력 매개변수 타입은 E의 Iterable이 아니라 E의 하위타입의 Iterable 이어야한다.

즉 , Iterable<? extends E> 이렇게 사용해야한다.

```java
//와일드카드 타입 적용
public void pushAll(Iterable<? extends E> src) {
    for(E e : src)
        push(e);
}
```

popAll메서드는? Stack안의 모든 원소를 주어진 컬렉션으로 옮겨 담아야하기 때문에 E의 상위 타입의 Collection이어야한다. 즉, Collection<? super E> 이렇게 사용해야한다.

```java
//와일드카드 타입 적용
public void popAll(Collection<? super E> dst) {
    while(!isEmpty())
        dst.add(pop());
}
```

**유연성을 극대화하기 위해서는 원소의 생산자나 소비자용 입력 매개변수에 와일드카드 타입을 사용해라.**

(PECS 공식 : T가 생산자면 <? extends T>, T가 소비자면 <? super T> )

제대로 사용하면 클라이언트는 와일드카드 타입이 쓰였다는 것도 모른다.

클라이언트가 와일드카드 타입을 신경써야한다면 API에 문제가 있을 가능성이 크다.

<br>

 

## 자바7

자바7이하에서는 위의 변경이 소용이 없다.

명시적 타입인수를 사용해서 타입을 알려줘야한다.

```java
Set<Integer> integers = Set.of(1,3,5);
Set<Double> doubles = Set.of(2.0, 4.0, 6.0);

//명시적 타입인수
Set<Number> numbers = Union.<Number>(integers,doubles);
```

<br>

## Comparable 과 Comparator

Comparable과 Comparator은 언제나 소비자이기 때문에

<E> 보다는 <? super E>를 사용하는것이 낫다.

<br>

## 타입 매개변수와 와일드 카드

```java
public static <E> void swap(List<E> list, int i, int j);
public static void swap(List<?> list, int i, int j);
```

메서드 선언에 타입 매개변수가 한번만 나오면 와일드 카드로 대체하라 (두번째)

이 때 비한정적 타입 매개변수면 비한정적 와일드카드로 바꾸고, 한정적 타입 매개변수면 한정적 와일드카드로.

단 두번째 swap은 문제가 있다. 비한정적 와일드카드 List<?>은 null만 넣을 수 있기 때문이다. 

→ 실제타입을 알려줄 도우미 메서드 사용으로 해결 (도우미 메서드는 제너릭 메서드여야함)

```java
public static void swap(List<?> list, int i, int j) {
	swapHelper(list, i, j);
}

//도우미 메서드가 리스트의 타입이 항상 E인것을 알기 때문에 안전함을 알고 있다.
private static <E> void swapHelper(List<E> list, int i, int j) {
	list.set(i, list.set(j, list.get(i)));
}
```

<br>

## 결론

조금 복잡하더라도 와일드카드 타입을 적용하면 api가 훨씬 유연해진다.

PECS 공식을 기억하자. (생산자는 extends, 소비자는 super 사용)