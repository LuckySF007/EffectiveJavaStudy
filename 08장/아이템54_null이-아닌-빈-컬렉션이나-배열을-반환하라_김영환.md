# Item54. null이 아닌, 빈 컬렉션이나 배열을 반환하라!

## null을 반환하는 경우

~~~java
private final List<Cheese> cheesesInStock = ...;

public List<Cheese> getCheeses() {
		return cheesesInStock.isEmpty() ? null : new ArrayList<>(cheesesInStock);
}
~~~

- 재고가 없다고 해서 특별히 null을 반환할 필요는 없다.
- 이 코드처럼 null을 반환하면, 클라이언트는 null 처리를 추가로 해줘야만 한다.
  - null 처리 방어 코드 필요





## 빈 컬렉션 또는 빈 배열을 반환하는 경우

### 효율성 측면 고려

- 빈 컨테이너를 할당하는 데도 비용이 드니 null을 반환하는 것이 낫다는 주장
  - 두 가지 면에서 틀린 주장이다.
  - 첫째, 분석 결과 빈 컨테이너의 할당이 성능 저하의 주범이라 확인되지 않는 한, 이 정도의 성능 차이는 신경 쓸 수준이 아니다.
  - 둘째, 빈 컬렉션 및 배열은 굳이 새로 할당하지 않고 반환 가능하다.



### 빈 컬렉션 반환의 예

~~~java
public List<Cheese> getCheeses() {
		return new ArrayList<>(cheesesInStock);
}
~~~

- 사용 패턴에 따라 빈 컬렉션 할당에 성능 누수가 있을 수도 있다.
  - 매번 똑같은 빈 '불변' 컬렉션을 반환해 해결 가능하다.

~~~java
public List<Cheese> getCheeses() {
		return cheesesInStock.isEmpty() ? Collections.emptyList() : new ArrayList<>(cheesesInStock);
}
~~~

- 최적화된 반환 코드다.



### 빈 배열 반환의 예

- 길이가 0인 배열을 반환하자.

~~~java
public Cheese[] getCheeses() {
		return cheesesInStock.toArray(new Cheese[0]);
}
~~~

- 이 방식 또한 성능을 떨어뜨릴 수 있으니, 길이가 0인 배열을 미리 선언해두고 반환하자. 길이가 0인 배열은 모두 불변이다.

~~~java
private static final Cheese[] EMPTY_CHEESE_ARRAY = new Cheese[0];

public Cheese[] getCheeses() {
		return cheesesInStock.toArray(EMPTY_CHEESE_ARRAY);
}
~~~

- 최적화된 반환 코드다.





## 핵심 정리

- null이 아닌, 빈 배열이나 컬렉션을 반환하라.
- null을 반환하는 API는 사용하기 어렵고 오류 처리 코드도 늘어난다. 또 null을 반환한다고 해서 성능이 좋을 것도 아니다.

