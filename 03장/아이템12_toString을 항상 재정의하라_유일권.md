# ITEM 12. **toString을 항상 재정의하라.**

`Object`의 기본 `toString` 메서드는 단순히 **클래스 이름@16진수로_표시한_해시코드**를 반환한다.  
`toString`의 규약은 "모든 하위 클래스에서 이 메서드를 재정의하라"고 한다.  
잘 구현한 클래스는 경우 사용하기 용이하고, 디버깅에 강점을 가지게 된다.

`toString`은 그 객체가 가진 주요 정보 모두 반환하는게 좋다. 하지만 적합하지 않다면, 요약 정보를 담아도 좋다.

`toString`을 구현할 경우 포맷을 명시하든 아니든 의도를 명확히 밝혀야한다.  
포맷을 명시하려면 아주 정확하게, 앞으로의 유지보수에도 항상 이 포맷을 써야하기 때문에 신중히 포맷을 정해야한다.

포멧 명시 여부와 상관없이 **`toString`이 반환한 값에 포함된 정보를 얻어 올수 있는 API를 제공하자**.  
제공하지 않을 경우 `toString`반환 값에서 파싱해야 하므로 성능이 나빠진다.

아래 코드는 포맷을 명시한 경우와, 아닌경우 두가지 경우이다.

```java
/*
 * 포멧을 명시한 경우
 * 전화번호의 문자열 표현을 반환합니다.
 * 이 문자열은 XXX-YYY-ZZZZ 형태의 12글자로 구성됩니다.
 * XXX는 지역코드, YYY는 프리픽스, ZZZZ는 가입자 번호다.
 * 각각의 대문자는 10진수 숫자 하나를 나타낸다.
*/
@Override
public String toString() {
    return String.format("%03d-%03d-%04d", areaCode, prefix, lineNum);
}

/*
 * 포멧을 명시하지 않은 경우
 * 상세 형식은 정해지지 않았으며 향후 변경될 수 있다.
 *
 * "[약물 #9: 유형=사랑, 냄새=테레빈유, 겉모습=먹물]"
*/
@Override
public String toString() { ... }
```

<br>
정적 유틸리티 클래스는 `toString`을 제공할 이유가 없고, `Enum`클래스는 자바가 `toString`을 완벽하게 제공하므로 재정의 할 필요가 없다.

하위 클래스들이 공유해야할 문자열 표현이 있는 추상 클래스는 재정의해줘야 한다. 예를 들어 대다수의 컬렉션 구현체는 추상 컬렉션 클래스들의 toString 메서드를 상속받아 사용하므로 재정의 해줘야 한다.

<br>

## 결론

모든 구체 클래스에서 `Object`의 `toString`을 재정의해서 사용하자. 단 상위 클래스에서 알맞게 재정의한 경우는 예외다.  
이때 해당 객체에 관한 명확하고 유용한 정보를 좋은 포맷으로 반환해야한다.

알맞게 `toString` 재정의한 클래스는 사용하기에도 유용하고 디버깅도 용이하다.
