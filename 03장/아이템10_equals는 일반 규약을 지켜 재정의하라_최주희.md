# 아이템10.equals는 일반 규약을 지켜 재정의하라

Object 클래스의 equals 메서드는 재정의하지않고 그냥두면 그 클래스는 오직 자기 자신과만 같게 된다.

## **equals 재정의를 추천하지 않는 상황**

1) 각 인스턴스가 본질적으로 고유할때

값을 표현하는게 아니라 동작하는 개체를 표현하는 클래스일때 ex)Thread 클래스

2) 인스턴스의 논리적 동치성을 검사할 일이 없을때

클라이언트가 논리적 동치성 검사 방식을 원하지 않거나 필요하지 않으면 재정의할 필요가 없다.

3) 상위 클래스에서 재정의한 equals가 하위 클래스에도 들어맞을때

ex) Set 구현체는 AbstractSet로부터, List 구현체들은 AbstractList로부터, Map 구현체들은 AbstractMap으로부터 equals를 상속받아 그대로 사용

4) 클래스가 private이거나 package-private(default)이고 equals 메서드를 호출할 일이 없을때

<br>

## equals 재정의를 해야하는 상황

객체 식별성이 필요할때이다.

상위 클래스의 equals가 논리적 동치성을 비교하도록 재정의되지 않았을때이다. 

equals가 논리적 동치성을 비교하도록 재정의해놓으면, 값을 비교할 수 있을뿐만 아니라

Set이나 Map의 키로 사용할 수 있다.

<br>

## equals 규약

동치관계를 만족시키기위한 규약이 있다.

1) 반사성

객체는 자기자신과 같아야 한다. x.equals(x)는 true (x ≠ null)

2) 대칭성 : 

두 객체는 서로에 대한 동치여부에 대해 똑같이 답해야한다. x.equals(y) == y.equals(x) 여야한다.

아래는 대칭성 위반 코드다.

```java
public final class CaseInsensitiveString {
    private final String s;

    public CaseInsensitiveString(String s) {
        this.s = Objects.requireNonNull(s);
    }

    //대칭성 위배
    @Override
    public boolean equals(Object o) {
        if (o instanceof CaseInsensitiveString)
            return s.equalsIgnoreCase(((CaseInsensitiveString) o).s);
        if (o instanceof String)  //한 방향으로만 작동
            return s.equalsIgnoreCase((String) o);
        return false;
    }
}
```

```java
CaseInsensitiveString hello = new CaseInsensitiveString("Hello~");
String helloString = "Hello~";

hello.equals(helloString) //true
helloString.equals(hello) //false
```

CaseInsensitiveString의 equals는 일반 String을 알고 있지만 그 반대는 아니기 때문에 대칭성을 위반한다.

equals 규약을 어기면 그 객체를 사용하는 다른 객체들이 어떻게 반응할지 알 수 없다.

이 문제를 해결하기 위해서는 CaseInsensitiveString의 equals를 String과도 연동하겠다는 꿈을 버려야한다.

3) 추이성 

첫번째 객체가 두번째 객체와 같고, 두번째 객체와 세번째 객체가 같으면 첫번째 객체와 세번째 객체도 같아야한다. x.equals(y)가 true,  y.equals(z)가 true이면, x.equals(z)도 true 여야한다. 

추이성 문제는 상위 클래스에는 없는 새로운 필드를 하위 클래스에 추가할때 발생한다.

```java
public class Point {
    private final int x;
    private final int y;

    public Point(int x, int y) {
        this.x = x;
        this.y = y;
    }

    @Override
    public boolean equals(Object o) {
        if (!(o instanceof Point))
            return false;
        Point p = (Point)o;
        return p.x == x && p.y == y;
    }
}
```

```java
public class ColorPoint extends Point {
    private final Color color;

    public ColorPoint(int x, int y, Color color) {
        super(x, y);
        this.color = color;
    }

    //추이성 위반
    @Override
    public boolean equals(Object o) {
        if (!(o instanceof Point))
            return false;

        //o가 일반 Point면 색상을 무시하고 비교
        if (!(o instanceof ColorPoint))
            return o.equals(this);

        //o가 ColorPoint면 색상까지 비교
        return super.equals(o) && ((ColorPoint) o).color == color;
    }
}
```

위 방식은 추이성을 깨버리는 코드이다.

```java
ColorPoint p1 = new ColorPoint(1, 2, Color.RED);
Point p2 = new Point(1, 2);
ColorPoint p3 = new ColorPoint(1, 2, Color.BLUE);

p1.equals(p2); //true, 색상 무시
p2.equals(p3); //true, 색상 무시
p1.equals(p3); //false, 색상까지 비교
```

구체 클래스를 확장해 새로운 값을 추가하면서 equals 규약을 만족시킬 방법은 존재하지 않는다.

equals 안의 instanceof 검사를 getClass 검사로 바꾸면 추이성은 지킬 수 있다.

```java
if (o == null || getClass() != o.getClass()) return false;
```

그러나 리스코프 치환 원칙에는 어긋난다.

> 리스코프 치환 원칙 : 상위 타입의 자료형은 하위 타입으로 변환되어도 문제없이 잘 작동해야한다.
> 

따라서 그 타입의 모든 메서드가 그 하위타입에서도 똑같이 잘 작동해야한다.

그런데, Point 클래스와 하위 클래스인 ColorPoint 클래스는 getClass가 다르기 때문에 올바르게 작동하지 않는다. → 구체 클래스의 하위 클래스에서 값을 추가하면서 equals 규약을 만족시킬 방법은 없다.

<br>

그러나 equals 규약을 지키면서 값을 추가할 방법이 있다. 상속 대신 합성을 사용하는 것이다.

ColorPoint가 Point를 상속하는 대신에 private 필드로 두어라.

```java
public class ColorPoint {
    private final Point point;
    private final Color color;

    public ColorPoint(int x, int y, Color color) {
        point = new Point(x, y);
        this.color = Objects.requireNonNull(color);
    }

    public Point asPoint() {  //Point 뷰를 반환
        return point;
    }

    //equals를 재정의할 때는 상위/하위 클래스를 따질 필요 없이 컴포지션 클래스 내부의 멤버들을 비    교하기만 하면 된다.
    @Override
    public boolean equals(Object o) {
        if (!(o instanceof ColorPoint))
            return false;
        ColorPoint cp = (ColorPoint) o;
        return cp.point.equals(point) && cp.color.equals(color);
    }
}
```

4) 일관성

두 객체가 같다면 앞으로도 영원히 같아야한다. 

equals의 판단에 신뢰할 수 없는 자원이 끼어들게 해서는 안된다. 

이런 문제를 피하기 위해 equals는 항시 메모리에 존재하는 객체만을 사용한 결정적 계산만 수행해야한다.

5) null-아님

모든 객체가 null과 같지 않아야 한다.

instanceof 는 null이면 false를 반환하기 때문에 equals에서 instanceof 를 이용하여 묵시적 null 검사를 하는것이 낫다.

<br>

## 양질의 equals 메서드 구현 방법

1) == 연산자를 사용하여 입력이 자기 자신의 참조인지 확인 - 성능 최적화

2) instanceof 연산자로 입력이 올바른 타입인지 확인 - 같은 인터페이스를 구현한 클래스끼리 비교하기 위해서는 equals 에서 클래스가 아닌 해당 인터페이스를 사용해야한다. ex) Set,List,Map 같은 컬렉션 인터페이스

3) 입력을 올바른 타입으로 형변환

4) 입력 객체와 자기 자신의 대응되는 핵심 필드들이 모두 일치하는지 하나씩 검사 - 하나라도 다르면 false 반환

- float : Float.compare(float, float)
- double : Double.compare(double, double)
- float와 double을 제외한 기본 타입 비교 : == 연산자 (float와 double은 특수한 부동소수 값을 다뤄야해서)
- 참조 타입 필드 : 각각의 equals 메서드
- 다를 가능성이 더 크거나 비교하는 비용이 싼 필드를 먼저 비교하여 equals의 성능을 높이자.
- 객체 전체의 상태를 대표하는 필드가 있다면 먼저 비교하자.

<br>

## equals를 다 구현했다면 확인해야할 것

대칭적인지, 추이성이 있는지, 일관적인지 확인해보자.

<br>

## 주의사항

- equals를 재정의할땐 hashcode도 반드시 재정의하자 [아이템11]
- 너무 복잡하게 해결하려 들지말자. 필드들의 동치성만 검사해도 규약을 어렵지 않게 지킬 수 있다.
- Object 외 타입을 매개변수로 받는 equals 메서드는 선언하지말자. 
입력 타입이 아닌 Object가 아니면 재정의가 아니라 다중정의한 것이다.
이 때, @Override 애너테이션을 붙이면 컴파일 오류를 알려주기 때문에 습관적으로 붙이자.

<br>

## 결론

꼭 필요한 경우가 아니면 equals 를 재정의하지 말자.

재정의해야할때는 그 클래스의 핵심필드를 빠짐없이 규약을 지켜가면서 비교하자.