# 아이템35.ordinal 메서드 대신 인스턴스 필드를 사용하라

- 대부분의 열거 타입 상수는 자연스럽게 하나의 정숫값에 대응됨
- 모든 열거 타입의 ordinal 메서드 : 해당 상수가 그 열거 타입에서 몇번쨰 위치인지 반환

```java
public enum Number {
	ONE, TWO, THREE, FOUR

	pbulic int convertInt() {
		return ordinal() + 1;
	}
}
```

위 코드는 유지보수에 안좋다. 상수 선언 순서를 바꾸면 converInt가 오동작하고,

이미 사용중인 정수와 값이 같은 상수는 추가할 방법이 없다. 그리고 중간에 비워놓지도 못한다.

해결책은 열거 타입 상수에 연결된 값은 ordinal 메서드로 얻지 말고 인스턴스 필드에 저장해라.

```java
puplic enum Number {
	ONE(1), TWO(2), THREE(3), FOUR(4)

	private final int number;

	public Number(int number) {
		this.number = number;
	}

	public int getNumber() {
		return number;
	}
}
```

<br>

## 결론

Enum의 API 문서에 보면 ordinal 메서드는 EnumSet과 EnumMap과 같이 열거 타입 기반의

범용 자료구조에 쓸 목적으로만 설계되었다고 쓰여 있다. 그게 아니면 절대 쓰지마라.