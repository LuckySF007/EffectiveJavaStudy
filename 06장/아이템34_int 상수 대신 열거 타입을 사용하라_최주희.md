# 아이템34.int 상수 대신 열거 타입을 사용하라

열거 타입은 고정된 상수의 집합으로, 그 외의 값은 허용하지 않는 타입이다. ex) 계절

열거 타입이 자바에 추가되기 전엔 정수 상수를 열거하는 취약한 방법 사용

```java
public static final int APPLE_FUJI = 0;
public static final int APPLE_PIPPIN = 1;
public static final int APPLE_GRANNY_SMITH = 2;

public static final int ORANGE_NAVEL = 0;
public static final int ORANGE_TEMPLE = 1;
public static final int ORANGE_BLOOD = 2;
```

위와 같은 방식은 상당히 취약

- 타입 안전 보장 x
- 표현력 좋지 않음. 컴파일러가 이해하는 값은 정수이므로 오렌지를 건너야할 메서드에 사과를 보내고 동등연산자로 비교하더라도 컴파일러는 틀린것을 모름
- 상수의 값이 바뀌면 클라이언트도 다시 컴파일해야함
- 문자열로 출력하기가 까다로움

<br>

## 열거 타입(enum type)

위의 단점들을 해소시켜주는게 자바의 열거 타입이다.

```java
public enum Apple { FUJI, PIPPIN, GRANNY_SMITH }
public enum Orange { NAVEL, TEMPLE, BLOOD}
```

생긴건 다른 언어의 열거 타입과 비슷해보이지만, 자바의 열거 타입은 다르다.

- 완전한 형태의 클래스이다. (단순히 정수 값인 다른언어의 열거타입보다 강력)
- 열거 타입 자체는 클래스이며, 상수 하나당 자신의 인스턴스로 구성됨
- 싱글톤으로 구현되어 있음 (public static final)
- 컴파일타임 타입 안전성 제공 (특정 열거 타입을 받아야하는 자리에 다른 타입의 값을 넘기려 하면 컴파일 오류)
- 열거 타입에 새로운 상수를 추가하거나 순서를 바꿔도 다시 컴파일 하지 않아도됨
- 열거 타입에 임의의 메서드나 필드를 추가할 수 있으며 인터페이스 구현할수도 있음
- Object 메서드들, Comparable, Serializable 을 높은 품질로 구현해놓음

<br>

## 열거 타입에 메서드나 필드 추가하기

단순하게 상수 모음뿐일 수도 있찌만, 실제로는 클래스여서 고차원의 추상 개념하나를 표현해낼수 있는것이다.

```java
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
  // 생성자, getter, setter

	public double surfaceWeight(double mass) {
		return mass * surfaceGravity;
	}
}
```

- 열거 타입 상수 각각을 특정 데이터와 연결지으려면 생성자에서 데이터를 받아 인스턴스 필드에 저장하면 됨 (열거 타입은 기본적으로 불변이므로 모든 필드는 final)
- private 나 package-private 로 메서드를 구현하면, 자신을 선언한 클래스 혹은 패키지에서만 사용할 수 있다. (public으로 노출할 필요가 없다면 private, 혹은 package-private으로 선언하라)

<br>

## 열거 타입의 상수 별 메서드 구현

열거 타입의 상수마다 다르게 동작을 구현하고 싶다면?

먼저 switch문으로 분기하는 방법을 보자.

```java
public enum Operation {
  PLUS, MINUS, TIMES, DIVIDE;

  public double apply(double x, double y) {
    switch (this) {
      case PLUS: return x + y;
      case MINUS: return x - y;
      case TIMES: return x * y;
      case DIVIDE: return x / y;
    }
    throw new AssertionError("알 수 없는 연산 :" + this);  //런타임 오류
  }
}
```

위 코드는 잘 동작하지만, 깨지기 쉬운 코드이다.

새로운 상수가 추가될 경우 해당 case문을 추가해야하는 변경가능성이 많은 코드이다.

새로 추가한 연산을 수행하려할때 컴파일은 잘되지만 런타임 오류가 난다.

<br>

위의 방법말고 상수마다 동작을 다르게 하기 위한 더 좋은 방법이 있다.

열거 타입에 추상 메서드를 선언하고 각 상수별 클래스 몸체에서 자신에 맞게 재정의해주는 것이다.

```java
public enum Operation {
  PLUS {public double apply(double x, double y) { return x + y; }},
  MINUS {public double apply(double x, double y) { return x - y; }},
  TIMES {public double apply(double x, double y) { return x * y; }},
  DIVIDE {public double apply(double x, double y) { return x / y; }};

  public abstract double apply(double x, double y);  //추상 메서드 
}
```

추상메서드이기 때문에 재정의하지 않으면 컴파일 오류를 내준다.

<br>

상수별 메서드 구현을 상수별 데이터와 결합할 수도 있다.

생성자를 이용하여 상수별 데이터와 결합하여 편리하게 사용했다.

```java
public enum Operation {
	PLUS("+") {public double apply(double x, double y) { return x + y; }},
	MINUS("-") {public double apply(double x, double y) { return x - y; }},
	TIMES("*") {public double apply(double x, double y) { return x * y; }},
	DIVIDE("/") {public double apply(double x, double y) { return x / y; }};
	public abstract double apply(double x, double y);

	private final String symbol;

	Operation(String symbol) { this.symbol = symbol; }

	@Override public String toString() { return symbol; }
}
```

위 코드와 toString()을 잘 이용하면 다음과 같은 아주 편리한 코드를 만들 수 있다.

```java
for (Operation op : Operation.values()) {
    System.out.printf("%f %s %f = %f%n", x, op, y, op.apply(x, y));
}

/* 출력
2.000000 PLUS 4.000000 = 6.000000
2.000000 MINUS 4.000000 = -2.000000
2.000000 TIMES 4.000000 = 8.000000
2.000000 DIVIDE 4.000000 = 0.500000
*/
```

<br><br>

## 전략 열거 타입 패턴

상수별 메서드 구현에는 열거 타입 상수끼리 코드를 공유하기 어렵다는 단점이 있다.

공유하기 위해서 값에 따라 분기하여 코드를 공유하는 열거 타입을 구현해보았다.

```java
 enum PayrollDay {
	 MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY,
	 SATURDAY, SUNDAY;

   private static final int MINS_PER_SHIFT = 8 * 60;

	 int pay(int minutesWorked, int payRate) {
     int basePay = mintesWorkd * payRate;
     int overtimePay;

     //case문을 날짜별로 두어 계산 수행 
     switch(this) {
       case SATURDAY : case SUNDAY : 
         overtimePay = basePay / 2;
         break;
       default:
         overtimePay = minutesWorked <= MINS_PER_SHIFT ? 0 :
                (minutesWorked - MINS_PER_SHIFT) * payRate / 2;
     }
		 return basePay + overtimePay;
	 }
}
```

위 코드는 변경사항이 많은 위험한 코드이다. 

위 코드를 전략 열거 타입 패턴으로 바꿔보겠다.

중첩 열거 타입을 사용하고 생성자에서 이 열거타입 상수중 적절한 것을 선택하도록 한다.

계산을 그 전략 열거 타입에 위임하여 switch 문이나 상수별 메서드 구현이 필요없게 된다. (안전,유연) 

```java
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

   //전략 열거 타입 
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
```

<br>

## 열거 타입을 언제 사용할까?

필요한 원소를 컴파일 타임에 다 알 수 있는 상수집합이면 항상 열거 타입을 사용하자.

열거 타입에 정의된 상수 개수가 영원히 고정 불변일 필요는 없다.

<br>

## 결론

- 열거 타입은 정수 상수보다 안전하고 가독성이 좋고 강력하다.
- 열거 타입의 각 상수를 특정 데이터와 연결짓거나 상수별로 동작을 다르게 할 때 명시적 생성자나 메서드가 쓰인다.
- 하나의 메서드가 상수별로 다르게 동작해야할 때는 switch 문 대신에 상수별 메서드 구현을 사용하자.(추상메서드 재정의)
- 열거 타입 상수 일부가 같은 동작을 공유한다면 전략 열거 타입 패턴을 사용하자.