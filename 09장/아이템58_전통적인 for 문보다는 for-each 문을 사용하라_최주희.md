# 아이템58.전통적인 for 문보다는 for-each문을 사용하라

## 전통적인 for문

```java
//컬렉션 순회하기
for (Iterator<Element> i = c.iterator(); i.hasNext()) {
    Element e = i.next();
		// ...
}

//배열 순회하기
for (int i = 0; i < a.length; i++) {
    // ...
}
```

- 위 for문은 while문보다는 낫지만 [아이템 57] 가장 좋은 방법은 아님
- 반복자와 인덱스 변수는 코드를 지저분하게 함 → 우리에게 정말 필요한건 원소들뿐
- 요소 종류가 늘어나면 오류 가능성이 높아짐
- 컬렉션이냐 배열이냐에 따라 코드 형태가 매우 달라짐

<br>

## for-each 문

```java
//컬렉션 순회하기
for(Element e : elements) {
		// ...
}

//배열 순회하기
for(int i : a) {
		// ...
}
```

- for-each문의 정식 이름은 향상된 for문 (enhanced for statement)
- 반복자와 인덱스 변수를 사용하지 않으니 코드가 깔끔해지고 오류가 날 일도 없음
- 하나의 관용구로 컬렉션과 배열을 모두 처리할 수 있어서 어떤 컨테이너를 다루는지 신경 안써도됨
    - 컬렉션과 배열 뿐만 아니라 Iterable 인터페이스를 구현한 객체라면 모두 순회가능

<br>

## 컬렉션을 중첩해 순회해야할 때

```java
enum Suit { CLUB, DIAMOND, HEART, SPADE }
enum Rank { ACE, DEUCE, THREE, FOUR, ... }

static List<Suit> suits = Arrays.asList(Suit.values());
static List<Rank> ranks = Arrays.asList(Rank.values());

List<Card> deck = new ArrayList<>();

for (Iterator<Suit> i = suits.iterator(); i.hasNext(); ) {
    for (Iterator<Rank> j = ranks.iterator(); j.hasNext(); ) {
        // i.next() 메서드가 '숫자(suit) 하나당' 한 번씩만 불려야 하는데
        // 카드(Rank) 하나당 불리고 있다.
        deck.add(new Card(i.next(), j.next()));
    }
}
```

위 코드는 i.next() 때문에 NoSuchElementException 을 던지는 버그가 있는 코드다.

아래와 같이 고치더라도 보기 좋지 않다.

```java
for (Iterator<Suit> i = suits.iterator(); i.hasNext(); ) {
    Suit suit = i.next();
    for (Iterator<Rank> j = ranks.iterator(); j.hasNext(); ) {
        deck.add(new Card(suit, j.next()));
    }
}
```

이런 문제는 for-each문을 중첩하는 것으로 간단히 해결되고 간결해진다.

```java
//컬렉션이나 배열의 중첩 반복을 위한 권장 관용구 - for-each
for(Suit suit : suits) {
	for(Rank rank : ranks) {
		deck.add(new Card(suit, rank));
	}
}
```

<br>

## for-each문을 사용할 수 없는 상황

- 파괴적인 필터링
    - 컬렉션을 순회하면서 선택된 원소를 제거해야한다면 반복자의 remove를 호출해야한다.
    - 자바8부터는 Collection의 removeIf로 명시적으로 순회하는 일을 피할 수 있다.
- 변형 : 리스트나 배열을 순회하면서 그 원소 값 일부 혹은 전체를 교체해야한다면 리스트의 반복자나 배열의 인덱스를 사용해야함
- 병렬 반복 : 여러 컬렉션을 병렬로 순회해야한다면 각각의 반복자와 인덱스 변수를 사용해 엄격하고 명시적으로 제어해야함

<br>

## 결론

전통적인 for문보다 for-each문이 더 간결하고 명료하고 유연하고 버그 가능성이 적다.

성능 저하도 없으므로 웬만하면 for-each문을 사용하자.