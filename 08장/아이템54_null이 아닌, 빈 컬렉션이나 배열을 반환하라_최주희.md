# 아이템54.null이 아닌, 빈 컬렉션이나 배열을 반환하라

## 빈 컬렉션 반환

```java
//컬렉션이 비었다면 null을 반환 -> 따라하지 말 것
public List<Cheese> getCheeses() {
	return cheesesInStock.isEmpty() ? null : new ArrayList<>(cheesesInStock);
}
```

만약 위와 같이 코드를 작성하였다면, 클라이언트는 이 null 상황을 처리하는 코드를 추가로 작성해야한다.

```java
List<Cheese> chesses = shop.getCheeses();
if(cheeses != null && chesses.contatins(Cheese.STILTON))
   System.out.println("ok");
```

컬렉션이나 배열이 비었을 때 null을 반환하는 메서드를 사용할 때면 항상 이렇게 방어 코드를 넣어줘야 한다.

만약 방어 코드를 까먹으면 오류가 발생할 수 있다.

null 대신 빈 컬렉션을 반환하자.

```java
public List<Cheese> getCheeses() {
	 return new ArrayList<>(cheesesInStock);
}
```

빈 컬렉션을 새로 할당하지 않고 아래와 같이 빈 불변 컬렉션을 반환 할 수 있다.

```java
//최적화 - 빈 컬렉션을 매번 새로 할당하지 않도록 함.
public List<Cheese> getCheeses() {
	 return cheesesInStock.isEmpty() ? Collections.emptyList() : new ArrayList<>(cheesesInStock);
}
```

<br>

## **빈 배열 반환**

배열을 쓸 때도 절대 null을 반환하지 말고 길이가 0인 배열을 반환하라.

```java
//길이가 0일수도 있는 배열을 반환하는 방법
public Cheese[] getCheeses() {
   return cheesesInStock.toArray(new Cheese[0]);
}
```

매번 새로 할당하는 위 방식이 성능을 떨어트릴 것 같다면, 미리 길이 0짜리 배열을 선언해두고 매번 그것을 반환하도록 하자. (길이 0인 배열은 불변)

```java
//최적화 - 빈 배열을 매번 새로 할당하지 않도록 함.
private static final Cheese[] EMPTY_CHEESE_ARRAY = new Cheese[0];

public Cheese[] getCheeses() {
   return cheesesInStock.toArray(EMPTY_CHEESE_ARRAY);
}
```

toArray에 넘기는 배열을 미리 할당하는건 오히려 성능을 떨어뜨린다.

```java
//나쁜 예 - 배열을 미리 할당하면 성능이 나빠짐.
return cheeseInStock.toArray(new Cheese[cheesesInStock.size()]);
```

<br>

## **결론**

null이 아닌, 빈 배열이나 컬렉션을 반환하라.

null을 반환하는 API는 사용하기 어렵고 오류 처리 코드도 늘어난다. 성능이 더 좋은것도 아니다.