# 아이템60.정확한 답이 필요하다면 float와 double은 피하라

float와 double 연산은 정확하지 않다. (특히 금융 관련 계산과는 맞지 않음)

```java
double result = 1.03 - 0.42;
System.out.println(result); // 0.6100000000000001
```

위와 같은 단순한 계산에도 오차가 발생한다. → float와 double은 부동소수점 방식을 사용하기 때문

> **부동소수점 방식**
부동소수점 방식은 수를 정수와 실수로 구분하지 않고 지수부와 가수부로 구분한다.
소수점이 고정되지 않기 때문에 더 폭넓은 범위를 표현할 수 있지만, 필연적으로 오차가 발생하여 근사값을 구한다.
> 

<br>

## 해결 방법

1) BigDecimal

double 타입을 BigDecimal 을 사용하면 실수 연산을 오차없이 사용할 수 있다.

BigDecimal은 기본 타입보다 쓰기가 불편하고 훨씬 느리다는 단점이 있다.

```java
public static void main(String[] args) {
	final BigDecimal TEN_CENTS = new BigDecimal(".10");
	int itemsBought = 0;
	BigDecimal funds = new BigDecimal("1.00");

	for(BigDecimal price = TEN_CENTS; funds.compareTo(price) >= 0; price = price.add(TEN_CENTS)) {
		funds = funds.subtract(price);
		itemsBought++;
	}
	System.out.println(itemsBought + "개 구입");
	System.out.println("잔돈(달러): " + funds); 
}
```

<br>

2) int 혹은 long

BigDecimal의 대안으로 int 나 long을 쓸 수도 있다.

이 방법은 연산도 정확하고 편리하고 빠르다.

그러나 다룰 수 있는 값의 크기가 제한되고, 소수점을 직접 관리해야한다.

ex) 0.1 달러 = 10 센트

```java
public static void main(String[] args) {
	int itemsBought = 0;
	int funds = 100;
	for(int price = 10; funds >= price; price+=10) {
		funds -= price;
		itemsBought++;
	}
	System.out.println(itemsBought + "개 구입");
	System.out.println("잔돈(센트): " + funds); 
}
```

<br>

## 결론

정확한 답이 필요할 때는 float나 double은 피하라.

소수점 추적은 시스템에 맡기고 성능 저하나 불편함은 신경쓰지 않는다면 BigDecimal을 사용하라. (반올림 완벽히 제어 가능)

반면, 소수점을 직접 추적할 수 있고 성능이 중요하고 숫자가 너무 크지 않다면 int나 long을 사용하라.

숫자를 아홉자리 십진수로 표현할 수 있다면 int, 열여덟자리 십진수로 표현할 수 있다면 long을 사용하라.

이를 넘어가면 BigDecimal을 사용해야함