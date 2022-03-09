# Item11. equals를 재정의하려거든 hashCode도 재정의하라!

- `equals`를 재정의한 클래스 모두에서 `hashCode`도 재정의해야 한다.
- 그렇지 않으면 `hashCode` 일반 규약을 어기게 되어 `HashMap`, `HashSet`과 같은 컬렉션의 원소 사용 시 문제가 발생한다.



## HashCode 관련 일반 규약

1. `equals` 비교에 사용되는 정보가 변경되지 않았다면, 애플리케이션이 실행되는 동안 그 객체의 `hashCode` 메서드는 몇 번을 호출해도 일관되게 항상 같은 값을 반환해야 한다.
   단, 애플리게이션을 다시 실행한다면 이 값이 달라져도 상관없다.
2. `equals(Object)`가 두 객체를 같다고 판단했다면, 두 객체의 `hashCode`는 똑같은 값을 반환해야 한다.
3. `equals(Object)`가 두 객체를 다르다고 판단했더라도, 두 객체의 `hashCode`가 서로 다른 값을 반환할 필요는 없다.
   단, 다른 객체에 대해서는 다른 값을 반환해야 해시테이블의 성능이 좋아진다.



### 논리적으로 같은 객체는 같은 해시코드를 반환해야 한다.

- `equals`는 물리적으로 다른 두 객체를 논리적으로는 같다고 할 수 있다.
- 하지만 `Object`의 기본 `hashCode` 메서드는 이 둘은 서로 다르다고 판단해 규약과 달리 서로 다른 값을 반환한다.
  이러한 경우, `hashCode`를 재정의하지 않는다면 논리적 동치인 두 객체가 서로 다른 해시코드를 반환해 두 번째 규약을 지키지 못하게 되고, 올바른 동작이 일어나지 않게 된다.



## 좋은 hashCode 작성

- 좋은 해시 함수란? 서로 다른 인스턴스에 다른 해시코드를 반환하는 함수 -> `hashCode`의 세 번째 규약
- 이상적인 해시 함수는 주어진 (서로 다른) 인스턴스들을 `32비트` 정수 범위에 균일하게 분배한다.



### hashCode 작성 요령

1. `int` 변수 `result`를 선언 -> 값 `c`로 초기화
   이때 `c`는 해당 객체의 첫번째 핵심 필드를 <u>단계 2.1 방식</u>으로 계산한 해시코드

   > 핵심 필드 : equals 비교에 사용되는 필드

2. 해당 객체의 나머지 핵심 필드 `f` 각각에 대해 다음 작업을 수행한다.

   1. 해당 필드의 해시코드 `c`를 계산한다.

      1. 기본 타입 필드라면, `Type.hashCode(f)`를 수행한다. (Type: 해당 기본 타입의 박싱 클래스)
      2. 참조 타입 필드고 이 클래스의 `equals` 메서드가 이 필드의 `equals`를 재귀적으로 호출해 비교한다면, 이 필드의 `hashCode`를 재귀적으로 호출한다. 필드값이 `null`이면 0을 사용함. (계산이 복잡해질 것 같다면, 이 필드의 표준형 만들기 -> 표준형의 `hashCode`를 호출하기)
      3. 필드가 배열이라면, 핵심 원소 각각을 별도 필드처럼 다룬다. 이상의 규칙을 재귀적으로 적용해 각 핵심 원소의 해시코드 계산 후 <u>단계 2.2 방식</u>으로 갱신한다. 배열에 핵심 원소 하나도 없다면 단순히 상수(0 추천) 사용함. 모든 원소가 핵심 원소라면 `Arrays.hashCode` 를 사용함.

   2. <u>단계 2.1</u>에서 계산한 해시코드 `c`로 `result`를 갱신한다.

      ~~~java
      result = 31 * result + c;
      ~~~

3. `result`를 반환한다.



### 주의사항

- `hashCode`를 다 구현했다면 이 메서드가 동치인 인스턴스에 대해 똑같은 해시코드를 반환할지 자문해보자.
  (단위 테스트 작성, `AutoValue` 프레임워크 사용했다면 건너뛰어도 좋음)
- 파생 필드는 해시코드 계산에서 제외해도 된다.
  - 즉, 다른 필드로부터 계산해낼 수 있는 필드는 모두 무시해도 된다.
- `equals` 비교에 사용되지 않은 필드는 '반드시' 제외해야 한다.
  - 그렇지 않다면, `hashCode` 규약 두 번째를 어기게 될 위험 존재
- 성능을 높인답시고 해시코드를 계산할 때 핵심 필드를 생략해서는 안 된다.
  - 속도는 빨라지겠지만, 해시 품질이 나빠져 해시테이블의 성능을 심각하게 떨어뜨릴 수도 있다.

- `hashCode`가 반환하는 값의 생성 규칙을 API 사용자에게 자세히 공표하지 말자.
  - 그래야 클라이언트가 이 값에 의지하지 않게 되고, 추후에 계산 방식을 바꿀 수 있다.



## 전형적인 hashCode 메서드

~~~ java
@Override public int hashCode() {
	int result = Short.hashCode(areaCode);
	result = 31 * result + Short.hashCode(prefix);
	result = 31 * result + Short.hashCode(lineNum);
	return result;
}

public static void main(String[] args) {
	Map<PhoneNumber, String> m = new HashMap<>();
	m.put(**new PhoneNumber**(707, 867, 5309), "Younghwani");

	System.out.println(**m.get(new PhoneNumber(707, 867, 5309)**));
	// hashCode를 재정의 했으므로 "Younghwani" 가 나온다
}
~~~

- `31 * result` : 곱하는 순서에 따라 `result` 값이 달라지게 한다.
- 31을 사용하는 이유 : 31이 소수(prime)이고 홀수이기 때문이다.



## Object 클래스의 hash 메서드

~~~java
@Override public int hashCode() {
  return Objects.hash(lineNum, prefix, areaCode);
}
~~~

- 이 메서드를 활용하면 앞서의 요령대로 구현한 코드와 비슷한 수준의 `hashCode` 함수를 단 한줄로 작성 가능하다.
- 하지만 속도는 더 느리다.
  입력 인수를 담기 위한 `배열 생성`, 입력 중 기본 타입이 있다면 `박싱&언박싱`을 거쳐야하기 때문이다.
- `hash` 메서드는 성능에 민감하지 않은 상황에서만 사용하자.



## 클래스가 불변이고 해시코드 계산 비용이 큰 경우

- 매번 새로 계산하는 것 보다는 캐싱을 이용하는 방식을 고려하자
- 클래스의 객체가 주로 해시의 키로 사용되는 경우 -> 인스턴스 생성 시 해시코드 계산해두기
- 해시 키로 사용되지 않는 경우 -> `hashCode`가 처음 불릴 때 계산하는 지연 초기화(lazy initialization) 전략 사용 고려
  - 필드를 지연 초기화시키려면 그 클래스를 thread safety 하게 만들도록 신경 써야 한다.

### 해시코드를 지연 초기화하는 hashCode 메서드

~~~java
private int hashCode; // 자동으로 0으로 초기화

@Override public int hashCode() {
	int result = hashCode;
	if(result == 0){
		result = Short.hashCode(areaCode);
		result = 31 * result + Short.hashCode(prefix);
		result = 31 * result + Short.hashCode(lineNum);
		hashCode = result;
	}
	return result;
}
~~~



## 핵심 정리

- `equals`를 재정의할 때는 `hashCode`도 반드시 재정의해야 한다.
- 재정의한 `hashCode`는 `hashCode` 일반 규약을 따라야 하며, 서로 다른 인스턴스라면 서로 다른 해시코드를 반환하도록 해야 한다.
- `AutoValue` 프레임워크나 `IDE` 사용으로 손쉽게 equals, hashCode 메서드를 만들어 사용하자.



