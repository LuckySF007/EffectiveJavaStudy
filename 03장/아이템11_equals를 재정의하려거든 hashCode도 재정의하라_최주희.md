# 아이템11.equals를 재정의하려거든 hashCode도 재정의하라

equals를 재정의한 클래스 모두에서 hashCode도 재정의해야한다.

그렇지 않으면 HashMap이나 HashSet 같은 컬렉션 원소로 사용할때 문제가 발생한다.

## Object 명세 규약

- equals 비교에 사용되는 정보가 변경되지 않았으면, hashCode 를 호출할때 항상 같은 값을 반환해야한다.
- equals(Object)가 두 객체가 같다고 판단했다면, 두 객체의 hashCode 반환 값이 같아야한다. 즉, 논리적으로 같은 객체는 같은 해시코드를 반환해야한다.
- equals(Object)가 두 객체를 다르다고 판단했더라도, 두 객체의 hasCode 반환 값이 다를 필요는 없다. 
단, 다른 객체에 대해서는 다른 값을 반환해야 해시테이블 성능이 좋아진다.

```java
Map<Student, String> m = new HashMap<>();
m.put(new Student(12345), "김코딩");
System.out.println(m.get(12345));  //null 출력
```

논리적으로 같은 객체라고 판단이 되어도, hashCode를 재정의하지 않았기 때문에 서로 다른 해시코드를 반환하여서 null을 반환한다. 그래서 hashCode 메서드를 재정의해야한다.

<br>

## 좋은 hashCode 작성법

```java
//사용 금지
@Override public int hashCode() { return 42; }
```

위 hashCode 메서드는 모든 객체에게 똑같은 값만 내어주기 때문에 해시테이블이 마치 연결리스트처럼 동작하여 평균 수행 시간이 O(1) → O(n)으로 느려진다.

이상적인 해시함수는 다른 객체들을 32비트 정수 범위에 균일하게 분배해야한다.

1) int 변수를 선언한 후 값을 초기화 (이 때 값은 해당 객체의 첫번째 핵심 필드를 2.a 방식으로 계산한 해시코드)

2) 해당 객체의 나머지 핵심 필드 각각에 대해 다음 작업 수행 (equals 비교에 사용되지 않은 필드는 반드시 제외)

a) 해당 필드의 해시코드 계산

- 기본 타입 필드 : Type.hashCode(f)를 수행 (Type은 기본 타입 박싱 클래스)
- 참조 타입 필드면서 이 클래스의 equals 메서드가 이 필드의 equals를 호출해 비교 : 필드의 표준형을 만들어 표준형의 hashCode를 호출. 필드 값이 null이면 0을 사용
- 배열 : 핵심 원소 각각을 별도 필드처럼 해시코드를 계산한 다음 2.b 방식으로 갱신. 배열에 핵심 원소가 없다면 상수 0을 사용. 모든 원소가 핵심 원소이면 Arrays.hashCode 사용

b) 2.a에서 계산한 해시코드 c로 변수를 갱신 `result = 31 * result + c;`

- 31 을 곱하는 이유 : 31은 홀수이며 소수이기 때문이다.
    - 곱할 숫자가 짝수고 오버플로가 발생하면 2를 곱하면 시프트 연산과 같은 결과를 주기 때문에 정보를 잃게 된다.
    - 31은 시프트연산과 뺄셈으로 대체해 최적화 가능. `(i<<5)-i`

3) int 변수 반환

```java
@Override public int hashCode() {
	int result = Short.hashCode(areaCode);
	result = 31 * result + Short.hashCode(prefix);
	result = 31 * result + Short.hashCode(lineNum);
	return result;
}

//한줄짜리 -> 성능이 조금 더 느리다.
@Override public int hashCode() {
	return Objects.hash(lineNum, prefix, areaCode);
}
```

클래스가 불변이고 해시코드 계산 비용이 크다면 매번 새로 계산하는것보다 캐싱을 고려해야한다.

- 객체가 주로 해시 키로 사용될것같다면 : 인스턴스가 생성될때 해시코드를 계산
- 해시 키로 사용되지 않는다면 : hashCode가 처음 호출될때 계산하는 지연 초기화를 하자. (지연 초기화 시에 그 클래스를 스레드 안전하게 만들도록 해야한다)

```java
private int hashCode; //자동으로 0으로 초기화된다.

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
```

<br>

## 주의사항

hashCode의 반환값 생성 규칙을 API 사용자에게 자세히 공표하지 말자.

클라이언트가 이 값에 의지하지 않고 추후에 계산 방식을 바꿀수도 있기 때문이다.

<br>

## 결론

equals를 재정의할때는 hashCode 도 반드시 재정의하자.

재정의한 hashCode는 Object의 API 문서의 일반 규약을 따라야하며,

서로 다른 인스턴스라면 되도록 해시코드도 서로 다르게 구현해야한다.