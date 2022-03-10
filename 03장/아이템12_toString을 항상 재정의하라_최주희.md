# 아이템12.toString을 항상 재정의하라

Object의 기본 toString 메서드가 우리가 작성한 클래스에서 적합한 문자열을 반환하는 경우는 거의 없다.

단순히 `클래스 이름@16진수로_표시한_해시코드`  를 반환한다.

toString은 유익한 정보를 반환할 수 있도록 하위 클래스에서 재정의해야한다. (디버깅에도 용이)

toString은 그 객체가 가진 주요 정보 모두를 반환하는게 좋다.

```java
/*
 * 전화번호의 문자열 표현을 반환합니다.
 * 이 문자열은 XXX-YYYY-ZZZZ 형태의 11글자로 구성됩니다.
 * XXX는 지역코드, YYYY는 접두사, ZZZZ는 가입자 번호입니다.
*/
@Override
public String toString() {
    return String.format("%03d-%04d-%04d", areaCode, prefix, lineNum);
}
```

toString 시에 포맷을 명시하든 아니든 의도를 명확히 밝혀야한다.

포맷을 명시하려면 아주 정확하게, 앞으로의 유지보수에도 항상 이 포맷을 써야하기 때문에 신중히 포맷을 정해야한다.

포맷을 명시하지 않느다면 보통 다음과 같이 작성한다.

```java
@Override
public String toString() {
    return "PhoneNumber{" +
                "areaCode='" + areaCode + '\'' +
                ", prefix='" + prefix + '\'' +
                ", lineNum='" + lineNum + '\'' +
                '}';
}
```

정적 유틸리티 클래스는 toString을 제공할 이유가 없다.

Enum도 자바가 toString을 다음과 같이 제공하기 때문에 재정의할 필요가 없다.

```java
public abstract class Enum<E extends Enum<E>> implements Comparable<E>, Serializable {
    private final String name;

    public String toString() {
        return name;
    }
}
```

하위 클래스들이 공유해야할 문자열 표현이 있는 추상 클래스는 재정의해줘야 한다.

ex) 대다수의 컬렉션 구현체는 추상 컬렉션 클래스들의 toString 메서드를 상속해씀

<br>

## 결론

모든 구체 클래스에서 Object의 toString을 재정의해서 사용하자.

상위 클래스에서 알맞게 재정의한 경우는 예외다.

알맞게 재정의한 toString은 사용하기에도 유용하고 디버깅하기 쉽게한다.

해당 객체에 관한 명확하고 유용한 정보를 좋은 형태로 반환해야한다.