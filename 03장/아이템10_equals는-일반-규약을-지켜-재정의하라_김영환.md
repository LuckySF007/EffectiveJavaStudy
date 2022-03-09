# Item10. equals는 일반 규약을 지켜 재정의하라!

- equals 메서드 재정의 시 함정에 빠져 문제를 초래할 수 있다.
- 문제 회피의 가장 쉬운 길 -> 아예 재정의 하지 않기.



## equals를 재정의 하지 않는 것이 최선인 경우

1. **각 인스턴스가 본질적으로 고유한 경우**

   - 값을 표현하는 것이 아니라 동작하는 개체를 표현하는 클래스

2. **인스턴스의 '논리적 동치성'을 검사할 일이 없는 경우**

   > 논리적 동치
   >
   > 두 개의 명제 p, q의 쌍방조건이 항진명제이면, 두 명제 p, q는 논리적 동치라 한다.

3. **상위 클래스에서 재정의한 equals가 하위 클래스에도 딱 들어맞는 경우**

   - 대부분의 Set, List, Map 구현체들은 AbstractSet, AbstractList, AbstractMap으로부터 equals를 상속받아 그대로 사용함.

4. **클래스가 private이거나 package-private이고, equals 메서드를 호출할 일이 없는 경우**

   - 실수로라도 equals가 호출되는 것을 막으려면

     ~~~java
     @Override
     public boolean equals (Object object) {
     	throw new AssertionError();
     }
     ~~~



## equals를 재정의해야 하는 경우

- 객체 식별성(물리적 동일 여부)이 아니라 논리적 동치성을 확인해야 하는데, 상위 클래스의 `equals`가 논리적 동치성을 비교하도록 재정의되지 않은 경우
- 주로 값 클래스(Integer, String)이 이에 해당된다.
- 클라이언트는 주로 값의 객체가 같은 지가 아니라 그 값이 같은지를 비교하고 싶을 것이다. `equals` 메서드를 논리적 동치성 확인이 가능하도록 재정의하면 이러한 니즈를 충족시킬 수 있다.
- 논리적 동치성을 확인 가능하게 `equals` 메서드 재정의 시, 인스턴스는 `Map`의 키, `Set`의 원소로 사용이 가능하다.
- `Enum`은 값 클래스이나 같은 인스턴스가 둘 이상 생성되지 않으니 재정의가 필요없다. (이 경우 `Object`의 `equals` 만으로도 논리적 동치성까지 확인 가능하니)



## equals 메서드 재정의 시 지켜야할 일반 규약

1. **반사성** : null이 아닌 모든 참조 값 x에 대해, `x.equals(x)`는 `true`

2. **대칭성** : null이 아닌 모든 참조 값 x, y에 대해, `x.equals(x)`는 `y.equals(x)`와 같다.

3. **추이성** : null이 아닌 모든 참조 값 x, y, z에 대해, `x.equals(y)`가 `true`이고, `y.equals(z)`가 `true`이면 `x.equals(z)`도 `true`이다.

4. **일관성** : null이 아닌 모든 참조 값 x, y에 대해, `x.equals(y)`를 반복해서 호출 시 항상 같은 결과를 출력한다.

5. **null 아님** : null이 아닌 모든 참조 값 x에 대해, `x.equals(null)`은 false



### 반사성

- 객체는 자기 자신과 같아야 한다는 논리
- 이 논리를 만족하지 않는다면 인스턴스를 클래스에 넣은 다음 `contains` 호출 시 없다고 답할 것이다.



### 대칭성

- 두 객체는 서로에 대한 동치 여부가 같다는 논리

- 대소문자를 구분하지 않는 `CaseInsensitiveString` 클래스

  ~~~java
  public class CaseInsensitiveString {
      private final String s;

      public CaseInsensitiveString(String s) {
          this.s = s;
      }

      @Override
      public boolean equals(Object o) {
          if (o instanceof CaseInsensitiveString)
              return s.equalsIgnoreCase(((CaseInsensitiveString) o).s);
          if (o instanceof String)  // String과 비교
              return s.equalsIgnoreCase((String) o);
          return false;
      }
      // ...생략
  }
  ~~~

- `CaseInsensitiveString`의 `equals`는 일반 문자열과도 비교를 시도하고 있다. 이는 잘못된 것이다.

- 다음의 예를 통해 이 예제가 대칭성을 위반하고 있음을 확인할 수 있다.

	~~~java
	CaseInsensitiveString cis = new CaseInsensitiveString("Younghwani");
	String s = "younghwani";
	~~~

  ~~~java
	cis.equals(s); // true
	s.equals(cis); // false
	~~~
	
- 위 예제의 결과, 대칭성이 위반되는 경우를 확인할 수 있다. `equals` 규약을 어긴다면 그 객체를 사용하는 다른 객체들이 어떻게 반응할지 알 수 없을 것이다.



### 추이성

- 첫 번째 객체와 두 번째 객체가 같고, 두 번째 객체와 세 번째 객체가 같다면, 첫 번째 객체와 세 번째 객체도 같아야 한다는 논리

- 주로 상위 클래스에 없는 새로운 필드를 하위 클래스에 추가하는 상황에 발생한다. 이 경우 `equals` 비교에 영향을 주는 정보가 추가되어 문제가 된다.

  ~~~java
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
  ~~~

  ~~~java
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
        
          //Point와 비교 시 색 무시
          if (!(o instanceof ColorPoint))
              return o.equals(this);
          return super.equals(o) && ((ColorPoint) o).color == color;
      }
  }
  ~~~

- `ColorPoint`는 `Point`를 확장해 점에 색을 추가했다. 
  만약 `equals` 메서드를 재정의 하지 않는다면 색 정보를 무시한 채 비교를 하게 된다. 이 경우 `equals` 규약을 어긴 것은 아니나 색 정보를 무시하게 되니 잘못된 경우다. 

- `ColorPoint`에서 재정의한 `equals`의 경우, 추이성을 위반하고 있다.
  
  ~~~java
  ColorPoint p1 = new ColorPoint(1, 2, Color.RED);
  Point p2 = new Point(1, 2);
  ColorPoint p3 = new ColorPoint(1, 2, Color.BLUE);
  
  p1.equals(p2); //true
  p2.equals(p3); //true
  p1.equals(p3); //false
  ~~~
  
  `p1`과 `p2`, `p2`와 `p3` 비교는 색을 무시하고 진행했지만, `p1`과 `p3`는 색 정보도 사용해 비교했기에 이러한 결과가 나오게 된다.

#### 해결방법

  - 객체 지향 언어의 동치 관계에서 나타나는 근본적 문제이기 때문에, 추상화의 이점을 포기하지 않는 한 구체 클래스를 확장해 값을 추가하면서 `equals` 규약을 만족시킬 방법은 존재하지 않는다.

  - `ColorPoint`의 `equals` 안의 `instanceof` 검사를 getClass 메서드를 이용한 로직으로 수정해 같은 클래스 타입일 경우에만 비교해 `true`를 출력하는 방식으로 변경한다면 어떨까?
    `Point`의 하위 클래서는 여전히 `Point`이고 `Point`로 활용 가능해야 한다. 하지만 이 경우 그렇지 못하다. 이는 리스코프 치환 원칙을 위배한다.

    > 리스코프 치환 원칙
    >
    > 어떤 타입에 있어 중요한 속성이라면 그 하위 타입에서도 마찬가지로 중요하다. 따라서 그 타입의 모든 메서드가 하위 타입에서도 똑같이 잘 작동해야 한다.

  - 우회방법 -> **"상속 대신 컴포지션을 사용하라"**

    - `Point`를 상속하는 대신 `Point`를 `ColorPoint`의 `private` 필드로 두고, `ColorPoint`와 같은 위치의 일반 `Point`를 반환하는 뷰 메서드를 `public`으로 추가하는 방식

      ~~~java
      public class ColorPoint {
          private final Point point;
          private final Color color;
      
          public ColorPoint(int x, int y, Color color) {
              point = new Point(x, y);
              this.color = Objects.requireNonNull(color);
          }
        
        	public Point asPoint() { // 이 ColorPoint의 Point 뷰를 반환한다.
            	return point;
          }
      
          @Override
          public boolean equals(Object o) {
              if (!(o instanceof ColorPoint))
                  return false;
              ColorPoint cp = (ColorPoint) o;
              return cp.point.equals(point) && cp.color.equals(color);
          }
      	  // ... 생략
      }
      ~~~




### 일관성

- 두 객체가 같다면 앞으로도 영원히 같아야 한다는 논리
- 가변 객체의 경우 비교 시점에 따라 서로 다를 수도 혹은 같을 수도 있지만, 불변 객체는 한번 다르면 끝까지 달라야 한다.
- `equals`의 판단에 신뢰할 수 없는 자원이 끼어들게 해서는 안된다.
- `equals`는 항시 메모리에 존재하는 객체만을 사용한 결정적(deterministic) 계산만 수행해야 한다.



### null-아님

- 모든 객체가 `null`과 같지 않아야 한다는 논리
- `NullPointerException`을 던지는 경우도 발생시키지 않아야 한다.



## 양질의 equals 메서드 구현 방법

### 구현 순서

1. `==` 연산자를 사용해 입력이 자기 자신의 참조인지 확인한다.
   - 자기 자신이면 `true`
   - 단순한 성능 최적화용, 비교 작업 복잡한 경우 값어치를 할 것
2. `instanceof` 연산자로 입력이 올바른 타입인지 확인한다.
   - 올바르지 않다면 `false`, 올바른 타입은 보통 equals가 정의된 클래스인 경우
   - 어떤 인터페이스는 자신을 구현한 서로 다른 클래스끼리도 비교할 수 있도록 equals 규약을 수정하기도 한다.
     이런 경우는 equals에서 클래스가 아닌 해당 인터페이스를 사용해야 한다.
     `Set`, `List`, `Map`, `Map.Entry` 등의 컬렉션 인터페이스들이 여기 해당한다.
3. 입력을 올바른 타입으로 형변환한다. 
   - 2단계에서 `instanceof` 검사를 했기에 100% 성공
4. 입력 객체와 자기 자신의 대응되는 '핵심' 필드들이 모두 일치하는지 하나씩 검사한다.
   - 모든 필드 일치 시 `true`, 하나라도 다르면 `false`
   - 2단계에서 인터페이스 사용했으면 입력의 필드 값 가져올 때도 그 인터페이서의 메서드 사용 필요
   - 타입이 클래스라면 접근 권한에 따라 해당 필드에 직접 접근 가능

### 필드 타입 비교 방법

- `float`, `double`을 제외한 기본 타입 필드 : `==`
- 참조 타입 필드 : 각각의 `equals` 메서드
- `float`, `double` 필드 : 각각 `Float.compare(float, float)`, `Double.compare(double, double)`
  - `Float.NaN`, `-0.0f`, 특수한 부동소수 값을 다룰 필요가 있어 `float`과 `double`은 특별이 취급한다.
- 배열 필드 : 원소 각각을 앞의 비교 방법대로 비교, 배열의 모든 원소가 핵심 필드라면 `Arrays.equals` 메서드들 중 하나 사용



### 필드 비교 순서

- 어떤 필드를 먼저 비교하느냐에 따라 `equals`의 성능을 좌우하기도 한다.
- 다를 가능성이 더 크거나 비교 비용이 싼 필드부터 비교한다.
- 핵심 필드로부터 계산해낼 수 있는 파생 필드는 굳이 비교할 필요 없다.
  하지만 파생 필드가 객체 전체의 상태를 대표하는 경우, 그 필드를 먼저 비교하는 것이 효율적이다.



### equals를 다 구현했다면 대칭적인가? 추이성이 있는가? 인관적인가? 자문해보자!

- 단위 테스트를 돌려보거나, `AutoValue`를 이용해 `equals` 작성을 하자.



## 주의사항

- `equals`를 재정의할 땐 `hashCode`도 반드시 재정의하자
- 너무 복잡하게 해결하려 들지 말자
- `equals` 매개변수의 입력타입은 반드시 `Object`로 구성한다.



## 구글이 만든 AutoValue 프레임워크 or IDE의 equals, hashCode 자동완성 기능

- 사람이 직접 작성하는 수고를 대신해준다. 
- 사람이 부주의해 야기할 실수를 저지르지 않는다.



## 핵심 정리

- 꼭 필요한 경우가 아니면 equals를 재정의하지 말자!
  - 많은 경우에 Object의 equals 메서드는 클라이언트가 원하는 비교를 정확히 수행해준다.
- 재정의해야 할 때는 그 클래스의 핵심 필드 전체를 빠짐없이, 다섯 가지 규약을 확실히 지켜가며 비교해야 한다.















