# Item60. 정확한 답이 필요하다면 float과 double은 피하라!

## Intro

- float과 double 타입은 과학과 공학 계산용으로 설계되었다.
- 이진 부동소수점 연산에 쓰이며, 넓은 범위의 수를 빠르게 정밀한 '**근사치**'로 계산하도록 세힘하게 설계되었다.
- 따라서 정확한 결과가 필요할 때는 사용하면 안 된다.
- **float과 double 타입은 특히 금융 관련 계산과는 맞지 않는다.**





## 계산 문제 예시

~~~java
System.out.println(1.03-0.42); // 0.6100000000000001
~~~

- 이렇듯 간단한 실수 계산도 근사치를 출력해 정확한 계산이 힘들다.
- '반올림하면 되겠지'라고 생각할 수 있다. 하지만 반올림해도 틀린 답이 나오는 경우도 있다.





## 부동소수점

### 고정소수점

- 소수점의 위치를 고정시키는 방식이다.
- 소수점의 위치가 고정되어 있기 때문에 오차 없는 정확한 연산이 가능하다.
- 다만 표현의 범위가 제한적이다.



### 부동소수점

- 수를 정수/실수로 구분하는 것이 아니라 지수부/가수부를 나눠 표현하는 방식이다.
  - 지수부에서는 어디 까지가 소수점 범위인지 파악하고, 가수부에는 수를 저장한다.
- 소수점이 고정되지 않기에 넓은 범위 표현이 가능하다.
- 하지만 필연적으로 근사값 계산을 할 수밖에 없기에 오차가 발생한다.





## 정확한 계산을 위한 해결책

- 정확한 계산이 필요하다면 `BigDecimal`, `int`, `long`을 사용하자.
- double 타입을 BigDecimal 타입으로 교체만 해줘도 정확한 결과값을 출력한다. 하지만 BigDecimal 사용에 단점도 존재하는데, 그건 바로 기본 타입보다 쓰기가 훨씬 불편하고, 느리다는 점이다.
- BigDecimal의 대안으로 int 혹은 long 타입을 사용할 수 있다. 
  - 이러한 경우 다룰 수 있는 값의 크기 제한, 소수점을 직접 관리해야 하는 불편함을 감수해야 한다.





## 핵심 정리

- 정확한 답이 필요한 계산에는 float이나 double을 피하라.
- 소수점 추적은 시스템에 맡기고, 코딩 시의 불편함이나 성능 저하를 신경쓰지 않겠다면 BigDecimal을 사용하자.
- BigDecimal은 8가지 반올림을 제공한다. 이를 통해 완벽한 반올림을 제어할 수 있다.
- 성능이 중요하고, 소수점을 직접 추적할 수 있고, 숫자가 너무 크지 않다면 int나 long을 사용하자.
  - int는 숫자를 9자리로, long은 숫자를 18자리로 관리할 수 있을 때 사용한다. 넘어가면 BigDecimal 사용 밖에 대안이 없다.
