# Item36. 비트 필드 대신 EnumSet을 사용하라!

## 비트 필드

- 열거한 값들이 주로 집합으로 사용될 경우, 예전에는 각 상수에 서로 다른 2의 거틉제곱 값을 할당한 **정수 열거 패턴**을 사용했다.

~~~java
public class Text {
    public static final int BOLD = 1 << 0; // 첫 번째 비트 // 1 // 이진수 : 0001
    public static final int ITALIC = 1 << 1; // 두 번째 비트 // 2 // 이진수 : 0010
    public static final int UNDERLINE = 1 << 2; // 세 번째 비트 // 4 // 이진수 : 0100
    public static final int STRIKETHROUGH = 1 << 3; // 네 번째 비트 // 8 이진수 : 1000

    public void applyStyles(int styles) {....}
}
~~~

- 위의 비트 필드 열거 상수 예제는 구닥다리 기법이다.

~~~java
text.applyStyles(BOLD | ITALIC); // 01 | 10 -> 11
~~~

- 다음과 같이 비트별 OR 사용을 통해 여러 상수를 하나의 집합으로 나타내는 것이 가능하다.
- 이렇게 만들어진 집합을 **비트 필드**라고 한다.



### 비트 필드의 장점

- 비트 필드를 사용하면 비트별 연산을 사용해 합집합과 교집합 같은 집합 연산을 효율적으로 수행할 수 있다.



### 비트 필드의 단점

- 하지만 비트 필드는 정수 열거 상수의 단점을 그대로 지니고 있다. 
- 비트 값이 그대로 출력 시, 단순 정수 열거 상수 사용보다 해석이 훨씬 어려워진다.
- 비트 필드 하나에 녹아 있는 모든 원소를 순회하기도 까다롭다.
- 최대 몇 비트가 필요한지 API 작성 시 미리 예측해야 한다.
  - 예측한 수치를 바탕으로 적절한 타입(보통 int, long)을 선택해야 한다.





## EnumSet

- 정수 열거 패턴을 완전 대체하는 것이 가능하다.
- 열거 타입을 사용해 열거 타입 상수 값으로 구성된 집합을 효과적으로 표현해준다.
- Set 인터페이스를 완벽히 구현하며, 타입 안전하고, 다른 어떤 Set 구현체와도 함께 사용하는 것이 가능하다.
- EnumSet 내부는 비트필드로 구현되었고, 원소가 64개 이하면 long 변수 하나로 표현하여 비트 필드에 비견되는 성능을 보인다.
- removeAll, retainAll 같은 대량 작업은 비트를 효율적으로 처리할 수 있는 산술 연산으로 구현.
- 내부가 비트필드로 구현된 것 뿐이기에, 클라이언트는 비트를 직접 다룰 때 흔히 겪는 오류에서 해방된다.



### EnumSet 예제

~~~java
public class Text {

    public enum Style { BOLD, ITALIC, UNDERLINE, STRIKETHROUGH }

    public static void applyStyles(Set<Style> styles) {
        //...
    }

    public static void main(String[] args) {
        // ...
        text.applyStyles(EnumSet.of(Style.BOLD, Style.ITALIC, Style.UNDERLINE));
    }
}
~~~

- EnumSet이 집합 생성을 위해 제공하는 정적 팩터리 중 of 메서드를 사용한다.
  - EnumSet 인스턴스로 변환해 건넨다.





## 핵심 정리

- 열거할 수 있는 타입을 한데 모아 집합 형태로 사용한다고 해도 비트 필드를 사용할 이유는 없다.
- EnumSet 클래스는 비트 필드 수준의 명료함, 성능을 보인다. 또 열거 타입의 장점까지 선사한다.
- EnumSet의 단점으로는 불변 EnumSet을 만들 수 없다는 것이 있다.
  - 이는 자바 11까지도 수정되지 않음. 옮긴이의 생각으로는 자바 개발팀은 불변 EnumSet이 그리 필요하지 않다고 보는 것 같다고 함.
  - 명확성, 성능에서 조금 손해를 볼 수 있지만 Collections.unmodifiableSet으로 EnumSet을 감싸서 사용하면 된다.

