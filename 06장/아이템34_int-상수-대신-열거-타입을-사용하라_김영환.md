# Item34. int 상수 대신 열거 타입을 사용하라!

> 열거 타입
>
> 일정 개수의 상수 값을 정의한 다음, 그 외의 값은 허용하지 않는 타입




## 정수 열거 패턴

~~~java
public static final int APPLE_FUJI = 0;
public static final int APPLE_PIPPIN = 1;
public static final int APPLE_GRANNY_SMITH = 2;
~~~

- static final int로 상수를 정의해 사용하는 방법이다.
- 단점이 많은 방법이다.
  - 타입 안전을 보장할 방법이 없다.
  - 표현력이 좋지 않다.
  - 컴파일러는 상수값을 통해 값을 비교하므로 다른 이상한(사용하고자 한 값이 아닌) 값과 비교해도 아무런 경고도 없다.
  - 컴파일 시, 상수 값이 클라이언트 파일에 그대로 심어진다.
    - 즉, 상수 값 변경 시 반드시 재 컴파일이 필요하다는 얘기다.
  - 정수형 상수를 문자열(APPLE_FUJI과 같은 값)로 출력하는 것이 까다롭다. 



## 문자열 열거 패턴

- 정수 대신 문자열 상수를 사용하는 변형 패턴이다.
- 상수의 의미 출력이라는 의미는 좋지만, 문자열 상수 이름 그대로 하드코딩하게 만들 수 있어 나쁘다.
  - 하드코딩 시, 문자열 오타가 있다면 컴파일러는 확인할 방법이 없고, 이는 런타임 에러로 이어진다.
- 문자열 비교를 사용하므로 성능 저하가 뒤따른다.



## 열거 타입(Enum type)

~~~java
public enum Apple { FUJI, PIPPIN, GRANNY_SMITH }
public enum Orange { NAVEL, TEMPLE, BLOOD}
~~~

- 가장 단순한 형태의 열거 타입 예제다.



### 자바의 열거 타입

- 자바 열거 타입은 완전한 형태의 클래스다.
  - 상수 하나당 자신의 인스턴스를 만들어 public static final 필드로 공개한다.
- 열거 타입 자체는 밖에서 접근할 수 있는 생성자 제공을 하지 않는다.
  - 사실상 final 이다.
- 클라이언트는 직접 인스턴스를 생성, 확장하는 것이 불가능하다.
  - 열거 타입 선언으로 만들어진 인스턴스는 딱 하나씩만 존재한다.
    - 싱글턴으로 구현되어 있다.



### 열거 타입의 장점

- 컴파일타임 타입 안전성을 제공한다.
  - Apple 열거 타입을 매개변수로 받는 메서드 선언 시, 그 매개변수의 참조는 무조건 Apple의 세 가지 값 중 하나다. null 값도 올 수 없다.
  - 다른 타입이라면 컴파일 에러가 난다.
- 각자의 이름 공간이 있다.
  - 이름이 같은 상수라도 공존이 가능하다.
  - 열거 타입에 새로운 상수 추가, 순서 변경이 있어도 재 컴파일이 필요 없다.
- toString 메서드를 통해 문자열을 반환해준다.
- 임의의 메서드나 필드 추가가 가능하다. 임의의 인터페이스 구현도 가능하다.



### 열거 타입에 메서드나 필드를 추가하는 경우

- 열거 타입은 단순하게는 just 상수 모음이다. 하지만 실재로는 클래스이니 고차원의 추상 개념 하나를 완벽히 표현하는 것이 가능하다.

~~~java
public enum Planet {
		MERCURY(3.302e+23,2.439e6),
		VENUS(4.869e+24,6.052e6),
		EARTH(5.975e+24, 6.378e6),
		MARS(6.419e+23,3.393e6),
		JUPITER(1.899e+27,7.149e7),
		SATURN(5.685e+26,6.027e7),
		URAUS(8.683e+25,2.556e7),
		NEPTUNE(1.024e+26,2.477e7);

		private final double mass;
		private final double radius;
		private final double surfaceGravity;
  
  	// 생성자, getter, setter 생략...
  
		public double surfaceWeight(double mass) {
			return mass * surfaceGravity;
		}
}
~~~

- 열거 타입 상수 각각을 특정 테이터와 연결지으려면 생성자에서 데이터를 받아 인스턴스 필트에 저장하면 된다.
- 열거 타입은 불변이다. 그래서 모든 필드는 final 이어야 한다.
- 필드를 public으로 선언해도 되나, private으로 두고 public 접근자 메서드를 두는 것이 좋은 방법이다.



### values 메서드

- 자신 안에 정의된 상수들의 값을 배열에 담아 반환하는 정적 메서드다.
- 값들은 선언된 순서로 저장된다.



### 메서드 타입

- 열거 타입을 선언한 클래스나 패키지에서만 유용한 기능은 private이나 package-private 메서드로 구현한다.
  - 자신을 선언한 곳에서만 구현된 열거 타입 상수를 사용할 수 있게 된다.
- 널리 쓰이는 열거 타입은 톱레벨 클래스로 만든다.
  - 특정 톱레벨 클래스에서만 사용되는 경우, 해당 클래스의 멤버 클래스로 만든다.



### 상수별 클래스 몸체(class body)와 데이터를 사용한 열거 타입

- 같은 이름의 메서드를 상수별로 다르게 동작하게 구성하고 싶은 경우 switch-case 문을 사용할 수도 있다.
  - 하지만 그리 좋은 방법은 아니다.
  - 알 수 없는 연산에 대한 고려를 해주어야 하기 때문이다. - 새로운 상수 추가 시 case 추가해줘야 하는데 안하면 런타임 에러
- 열거 타입은 상수별로 다르게 동작하는 코드를 구현하는 더 나은 수단을 제공한다.
  - apply라는 추상 메서드를 선언해 각 상수별로 원하는 동작을 재정의할 수 있다.

~~~java
public enum Operation {
    PLUS("+") {
	    public douvle apply(double x, double y) { return x + y };
    },
    MINUS("-") {
      public douvle apply(double x, double y) { return x - y };
    },
    TIMES("*") {
      public douvle apply(double x, double y) { return x * y };
    },
    DIVIDE("/") {
      public douvle apply(double x, double y) { return x / y };
    };
	
    private final String symbol;

    Operation(String symbol) {this.symbol = symbol;}

    @Override public String toString() {return symbol;}
    public abstract double apply(double x, double y);
}
~~~



## 전략 열거 타입 패턴

~~~java
enum PayrollDay {  
    MONDAY(WEEKDAY),  
    TUESDAY(WEEKDAY),  
    WEDNESDAY(WEEKDAY),  
    THURSDAY(WEEKDAY),  
    FRIDAY(WEEKDAY),  
    SATURDAY(WEEKEND),  
    SUNDAY(WEEKEND);  

    private final PayType payType;  

    PayrollDay(PayType payType) {  
        this.payType = payType;  
    }  

    int pay(int minutesWorked, int payRate) {  
        return payType.pay(minutesWorked, payRate);  
    }  

    enum PayType {  
        WEEKDAY {  
          int overtimePay(int minutesWorked, int payRate) {  
            return minutesWorked <= MINS_PER_SHIFT ? 0 : (minutesWorked - MINS_PER_SHIFT) * payRate / 2;  
          }  
      },  

      WEEKEND {  
          int overtimePay(int minutesWorked, int payRate) {  
            return minutesWorked * payRate / 2  
          }  
      };  

      abstract int overtimePay(int minutesWorked, int payRate);  
      private static final int MINS_PER_SHIFT = 8 * 60;  

      int pay(int minutesWorked, int payRate) {  
          int basePay = minutesWorked & payRate;  
          return basePay + overtimePay(minutesWorked, payRate);  
      }  
   }  
}
~~~

- 새로운 상수를 추가할 때 잔업수당 '전략'을 선택하도록 한다.
- 잔업수당 계산을 private 중첩 열거 타입(PayType)으로 옮긴다.
  - PayrollDay 생성자에서 PayType을 적절히 선택해 사용한다.
- 이 패턴은 switch 문이나 상수별 메서드 구현이 필요없고, 더 복잡하지만 안전한 방법이다.
  - 기존 열거 타입에 상수별 동작을 혼합해 넣을 때는 switch 문이 좋은 선택이 될 순 있음.



## 열거 타입을 사용하는 경우

- 필요한 원소를 컴파일타임에 다 알 수 있는 상수 집합이라면 항상 열거 타입을 사용하자.
  - 열거 타입에 정의된 상수 개수가 영원히 고정일 필요는 없다.






## 핵심 정리

- 열거 타입은 확실히 정수 상수보다 뛰어나다.
  - 더 읽기 쉽고 안전하고 강력하다.
- 대다수 열거 타입이 명시적 생성자나 메서드 없이 쓰인다.
  - 하지만 각 상수를 특정 데이터와 연결짓거나 상수마다 다르게 동작하게 할 때는 필요하다.
- 드물게는 하나의 메서드가 상수별로 다르게 동작해야 할 때도 있다.
  - 이런 열거 타입의 경우 switch 문 대신 상수별 메서드 구현을 사용한다.
- 열거 타입 상수 일부가 같은 동작 공유 시 전략 열거 타입 패턴을 사용하자.

