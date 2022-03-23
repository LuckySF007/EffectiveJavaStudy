# 아이템36.비트 필드 대신 EnumSet을 사용하라

열거한 값들이 집합으로 사용될 경우, 예전에는 각 상수에 서로 다른 2의 거듭제곱 값을 할당한 정수 열거 패턴을 사용해왔다.

```java
//비트 필드 열거 상수 - 구닥다리 기법!
public class Text {
    public static final int BOLD = 1 << 0;
    public static final int ITALIC = 1 << 1;
    public static final int UNDERLINE = 1 << 2;
    public static final int STRIKETHROUGH = 1 << 3;

    //매개변수는 0개 이상의 상수를 비트별 OR한 값
    public void applyStyles(int styles) {
        // ...
    }
}
```

```java
text.applyStyles(BOLD | UNDERLINE); //여러 상수를 하나의 집합에 모으는 방법 (비트 필드)
```

비드 필드 값 하나로는 단순한 정수 열거 상수를 출력할때보다 값의 의미를 해석하기 어려움

그리고, 최대 몇 비트가 필요한지를 API 작성 시 미리 예측하여 적절한 타입을 선언해야하는 어려움

<br>

## EnumSet

위 문제점의 해결방안이 EnumSet 이다.

이 클래스는 열거 타입 상수값으로 구성된 집합을 효과적으로 표현해준다.

- Set 인터페이스 완벽 구현 가능 (타입 안전)
- 내부가 비트 벡터로 구성되어 있어서 원소가 64개 이하라면 대부분 long 변수 하나로 표현하여 비트 필드와 비슷한 성능

아까 위의 코드를 아래와 같이 깔끔하게 바꿀 수 있다.

```java
public class Text {
    public enum Style {
        BOLD, ITALIC, UNDERLINE, STRIKETHOUGH
    }

		//웬만하면 EnumSet을 넘겨라
    //매개변수로 Set을 받은 이유는 이왕이면 인터페이스로 받는게 일반적으로 좋은 습관이기 때문
		public void applyStyles(Set<Style> styles) {
		     //생략
		}
}
```

```java
text.applyStyles(EnumSet.of(Style.BOLD, Style.ITALIC));
```

<br>

## 결론

열거할 수 있는 타입을 한데모아 집합 형태로 사용할때는 EnumSet 클래스를 사용하자.

EnumSet은 불변 EnumSet을 만들 수 없다는 유일한 단점을 가지고 있다.

이는 Collections.unmodifiableSet으로 EnumSet을 감싸 사용할 수 있다.