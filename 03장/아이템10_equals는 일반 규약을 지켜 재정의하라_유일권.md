# ITEM 10. **equals는 일반 규약을 지켜 재정의하라.**

`Object` 클래스의 `equals` 메서드는 재정의하지 않을경우 오직 자기 자신과만 같게 된다.

## equals 재정의를 하지 않아도 되는 경우

1) 각 인스턴스가 본질적으로 고유한 경우
    - `Thread` 클래스와 같이 값을 표현하는게 아닌 동작하는 개체를 표현하는 클래스인 경우  
<br>
1) 인스턴스의 논리적 동치성을 검사할 일이 없는 경우
   - 클라이언트가 논리적 동치성 검사 방식을 원하지 않거나, 검사할 필요가 없다면 재정의할 필요가 없다.  
<br>
3) 상위 클래스에서 재정의한 `equals`가 하위 클래스에도 들어맞을때
    - `Set` &rarr; `AbstractSet`에서, `List` &rarr; `AbstractList`에서, `Map` &rarr; `AbstractMap`에서 상속받아  사용  
<br>
4) 클래스가 `private`, `package-private`이며 `equals` 메서드를 호출할 일이 없는 경우
<br>

## equals 재정의를 해야하는 상황

논리적 동치성을 확인해야하나 상위 클래스의 equals가 논리적 동치성을 비교하도록 정의되지 않았을때이다. 

equals가 논리적 동치성을 비교하도록 재정의하면, 값을 비교할 수 있고, Set이나 Map의 키로 사용될 수 있다.

<br>

## **equals 규약**

동치관계를 만족시키기위한 규약이 있다.

1) **반사성**

    - 객체는 자기자신과 같아야 한다. x가 null이 아니면 x.equals(x)는 true다.  
<br>
2) **대칭성**
    - 두 객체는 서로에 대한 동치여부에 대해 똑같이 답해야한다. `x.equals(y) == y.equals(x)`를 만족해야한다. 아래 예시를 보자  
<br>

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
        if (o instanceof String)  //한 방향으로만 작동한다!
            return s.equalsIgnoreCase((String) o);
        return false;
    }
    ... //나머지 코드 생략
}

//비교 시도
CaseInsensitiveString cis = new CaseInsensitiveString("Polish");
String s = "polish";

cis.equals(s) //true
s.equals(cis) //false

//올바른 정렬코드
public boolean equals(Object o) {
    return o instanceof CaseInsensitiveString &&
        ((CaseInsensitiveString) o).s.equalsIgnoreCase(s);
}
```

`CaseInsensitiveString`의 `equals`는 `String`을 알고 있지만 그 반대는 아니기 때문에 대칭성을 위반하게 된다.  
이 문제를 해결하기 위해서는 `CaseInsensitiveString`의 `equals`를 String과도 연동을 포기해야 한다.  

<br>

3) **추이성**

    - x, y, z가 모두 `null`이 아닐경우 `x.equals(y)`가 `true`이고,  `y.equals(z)`가 `true`이면, `x.equals(z`)도 `true` 여야한다.  

    - 상위 클래스에는 없는 새로운 필드를 하위 클래스에 추가할때 의 예시를 보자

```java
//2차원 점 표현 클래스
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

//2차원 점, 색 표현 클래스
public class ColorPoint extends Point {
    private final Color color;

    public ColorPoint(int x, int y, Color color) {
        super(x, y);
        this.color = color;
    }

    //대칭성은 지키지만 추이성 위반
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

//추이상 위반 확인
ColorPoint p1 = new ColorPoint(1, 2, Color.RED);
Point p2 = new Point(1, 2);
ColorPoint p3 = new ColorPoint(1, 2, Color.BLUE);

p1.equals(p2); //true
p2.equals(p3); //true
p1.equals(p3); //false (색까지 고려해 false)
```

### 해결방법

equals 안의 instanceof 검사를 getClass 검사로 바꾸면 추이성은 지킬 수 있다. 단 리스코프 치환 원칙(상위 타입의 자료형은 하위 타입으로 변환되어도 문제없이 잘 작동해야한다.)에는 어긋난다.

```java
if (o == null || getClass() != o.getClass()) return false;
```

Point 클래스와 하위 클래스인 ColorPoint 클래스는 getClass가 다르기 때문에 올바르게 작동하지 않는다. → 구체 클래스의 하위 클래스에서 값을 추가하면서 equals 규약을 만족시킬 방법은 없다.

하지만 우회 방법이 있다.  
**상속 대신 컴포지션을 사용하라**(Item 18) 를 고려해보자. ColorPoint가 Point를 상속하는 대신 private 필드로 둔다.  
이후 비교할때 각각을 비교한다.  

```java
public class ColorPoint {
    private final Point point;
    private final Color color;

    public ColorPoint(int x, int y, Color color) {
        point = new Point(x, y);
        this.color = Objects.requireNonNull(color);
    }
    //ColorPoint의 Point 뷰 반환
    public Point asPoint() {
        return point;
    }

    @Override
    public boolean equals(Object o) {
        if (!(o instanceof ColorPoint))
            return false;
        ColorPoint cp = (ColorPoint) o;
        return cp.point.equals(point) && cp.color.equals(color);
    }
}
```

4) **일관성**

    - 두 객체가 같다면 앞으로도 영원히 같아야한다. 그러기 위해 `equals`의 판단에 신뢰할 수 없는 자원이 끼어들게 해서는 안된다. 이런 문제를 피하기 위해 `equals`는 항시 메모리에 존재하는 객체만을 사용한 결정적 계산만 수행해야한다.

5) **null-아님**

모든 객체가 `null`과 같지 않아야 한다. `instanceof` 는 `null`이면 `false`를 반환하기 때문에 `equals`에서 `instanceof` 를 사용하여 타입검사 동시 `null` 검사도 같이 진행이 가능하다.  

<br>

## 양질의 equals 메서드 구현 방법

1) == 연산자를 사용하여 입력이 자기 자신의 참조인지 확인

2) `instanceof` 연산자로 입력이 올바른 타입인지 확인

3) 입력을 올바른 타입으로 형변환

4) 입력 객체와 자기 자신의 대응되는 핵심 필드들이 모두 일치하는지 하나씩 검사 - 하나라도 다르면 false 반환

   - `float`, `double`은 정적 메서드 `compare`을 사용하여 비교(특수한 부동소수 값 비교 때문에 `compare`을 사용)
   - `float`와 `double`을 제외한 기본 타입 비교 : == 연산자 
   - 참조 타입 필드 : 각각의 equals 메서드
   - 다를 가능성이 더 높거나 비교하는 비용이 저렴한 필드를 우선 비교해 equals의 성능을 향상 시키자
   - 객체 전체의 상태를 대표하는 필드가 있다면 제일 우선 비교하자.  

<br>

## equals를 다 구현했다면 확인해야할 것

`equals` 규약을 다 지키고 있는지 확인한다.  
특히 **대칭적, 추이성, 일관적**인지 확인한다.

<br>

## 주의사항

- `equals`를 재정의할땐 `hashcode`도 반드시 재정의하자(Item 11)
- 너무 복잡하게 해결하려 들지말자. 필드들의 동치성만 검사해도 규약을 어렵지 않게 지킬 수 있다.
- `Object` 외 타입을 매개변수로 받는 `equals` 메서드는 선언하지말자. 이 경우 재정의가 아니라 다중정의한 것이다.  
이 때, `@Override` 애너테이션을 붙이면 컴파일 오류를 알려주기 때문에 꼭 사용하자.

<br>

## **결론**

꼭 필요한 경우가 아니면 equals 를 재정의하지 말자.

만약 재정의해야 할 때는 규약을 확실히 지켜가면서 비교하자.