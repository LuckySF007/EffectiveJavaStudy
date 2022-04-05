# 아이템56.공개된 API 요소에는 항상 문서화 주석을 작성하라

## 자바독 (Javadoc)

- API를 쓸모 있게 하기 위해 문서화가 잘 되어있어야함
- API 문서화 유틸리티
- 소스코드 파일에서 문서화 주석(자바독 주석)이라는 특수한 형태로 기술된 설명을 추려 API 문서로 변환해준다.
- 자바 프로그래머라면 자바독 주석을 작성하는 규칙을 알아야함

<br>

## Javadoc 주석 유의사항

- API를 올바로 문서화하기 위해선 공개된 모든 클래스, 인터페이스, 메서드, 필드 선언에 문서화 주석을 달아야함
    - 주석을 달지 않으면 그저 공개된 API 요소들의 선언만을 나열
    - 공개 클래스는 기본 생성자에 주석을 달 수 있는 방법이 없으니 절대 기본 생성자를 사용하지마라.
    - 유지보수까지 고려한다면 공개되지 않은 것들에도 문서화 주석을 달아야할 것이다.
- 메서드용 문서화 주석에는 해당 메서드와 클라이언트 사이의 규약을 명료하게 기술해야함
    - 상속용으로 설계된 API가 아닌 이상 메서드가 어떻게 동작하는지가 아닌 무엇을 하는지를 기술해라
    - 클라이언트가 해당 메서드를 호출하기 위한 전제조건 모두 나열해라
    - 메서드가 성공적으로 수행된 후에 만족해야할 사후조건 모두 나열해라
    - 부작용(사후조건으로 명확히 나타나진 않지만 시스템의 상태에 변화를 가져오는 것)을 문서화해라
    

<br>

## Javadoc 주석 태그

```java
public interface List<E> extends Collection<E> {
    // Positional Access Operations

    /**
     * Returns the element at the specified position in this list.
     *
     * @param index index of the element to return
     * @return the element at the specified position in this list
     * @throws IndexOutOfBoundsException if the index is out of range
     *         ({@code index < 0 || index >= size()})
     */
    E get(int index);

}
```

1) @param

- 해당 매개변수가 뜻하는 값을 설명하는 명사구
- 모든 매개변수에 설명을 달아야함
- 마침표 x

2) @return

- 반환값을 설명하는 명사구
- 반환값이 void가 아니라면 달아야함
- 마침표 x

3) @throws

- if로 시작해 해당 예외를 던지는 조건을 설명하는 절
- 발생할 가능성이 있는 모든 예외에 달아야함
- 마침표 x

4) {@code}

- 주석 내의 HTML 요소나 다른 자바독 태그를 무시함
- 감싼 내용을 코드용 폰트로 렌더링
- 여러줄로 된 코드 예시를 넣으려면 <pre>{@code ...코드... }</pre> 형태로 쓰면 된다.

5) @implSpec

- 클래스를 상속용으로 설계할 때는 자기 사용 패턴에 대해서 문서에 남겨야함 [아이템 15]
- 자기 사용 패턴은 자바8에 추가된 @implSpec으로 문서화
- 해당 메서드와 하위 클래스 사이의 계약을 설명
- 하위 클래스들이 그 메서드를 상속하거나 super 키워드로 호출할 때 그 메서드가 어떻게 동작하는지를 명확히 알려주도록 해야함
- 자바 11까지도 자바독 명령줄에서 -tag "implSpec:a:Implementation Requirements:" 스위치를 켜주지 않으면 @implSpec 태그를 무시해 버린다.

```java
public interface List<E> extends Collection<E> {

    /**
     * Returns true< if this list contains no elements.
     * @implSpec
     * This implementation returns {@code this size ==0}.
     *
     * @return true if this list contains no elements
     */
    boolean isEmpty();
}
```

6) {@literal}

- HTML 마크업이나 자바독 태그를 무시하게 해줌
- {@code} 와 비슷하지만 코드 폰트로 렌더링하지는 않는다.

```java
* A geometric series converges if {@literal |r| < 1}.
```

7) {@index}

- 자바9부터 자바독이 생성한 HTML 문서에 검색(색인) 가능
- 클래스,메서드,필드 같은 API 요소의 색인은 자동으로 만들어짐
- {@index} 로 API에서 중요한 용어를 추가로 색인 가능하게 할 수 있음

<br>

## Javadoc 주석 규약

1) 각 문서화 주석의 첫번째 문장은 해당 요소의 요약 설명

- 반드시 대상의 기능을 고유하게 기술해야함 (한 클래스나 인터페이스 안에서 요약설명이 똑같은 멤버 혹은 생성자가 둘 이상이면 안됨)
- 특히, 다중정의된 메서드들의 설명이 같은 문장으로 시작하지 않도록 조심
- 마침표까지가 첫문장이 되므로 마침표에 주의 → 의도치 않은 마침표 사용시에는 {@literal}로 감싸주자
- 자바10부터는 {@summary} 라는 요약 설명 전용 태그 추가
- 메서드와 생성자의 요약 설명 : 주어가 없는 동사구, 2인칭 문장(return)이 아닌 3인칭 문장(returns) 사용 (한글은 차이 없음)
- 클래스,인터페이스,필드의 요약 설명 : 대상을 설명하는 명사절
- 제네릭 : 제네릭 타입이나 제네릭 메서드를 문서화할때는 모든 타입 매개변수에 주석 달아야함

```java
/**
 * An object that maps keys to values.  A map cannot contain duplicate keys; 
 * ...
 * @param <K> the type of keys maintained by this map
 * @param <V> the type of mapped values
 */
public interface Map<K,V> { ... }
```

- 열거 타입 : 열거 타입 자체, 열거 타입의 public 메서드, 상수들에도 주석을 달아야함

```java
/**
 * An instrument section of a symphony orchestra.
 */
public enum OrchestraSection {
    /** Woodwinds, such as flute, clarinet, and oboe. */
    WOODWIND,

    /** Brass instruments, such as french horn and trumpet. */
    BRASS,

    /** Percussion instruments, such as timpani and cymbals. */
    PERCUSSION,

    /** Stringed instruments, such as violin and cello. */
    STRING;
}
```

- 애너테이션 타입 : 애너테이션 타입 자체, 멤버들에도 모두 주석을 달아야함 (필드 설명은 명사구)

```java
/**
 * Indicates that the annotated method is a test method that
 * must throw the designated exception to pass.
 */
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTest {
    /**
     * The exception that the annotated test method must throw
     * in order to pass. (The test is permitted to throw any
     * subtype of the type described by this class object.)
     */
    Class<? extends Throwable> value();
}
```

- 패키지 설명 문서화 주석은 package-info.java 파일에 작성
    - 패키지 선언을 반드시 포함
    - 패키지 선언 관련 애너테이션 추가 포함 가능
- 자바9부터 지원하는 모듈 관련 설명은 module-info.java 파일에 작성

<br>

2) API 문서화에서 자주 누락되는 설명 2가지

- 스레드 안전성 : 클래스 혹은 정적 메서드가 스레드 안전하든 그렇지 않든, 스레드 안전 수준을 반드시 API 설명에 포함해야함 [아이템 82]
- 직렬화 가능성 : 직렬화할 수 있는 클래스면 직렬화 형태도 기술해야함 [아이템 87]

3) 자바독은 메서드 주석을 상속시킬 수 있음

- 문서화 주석이 없는 API 요소를 발견하면 자바독이 가장 가까운 문서화 주석을 찾음 (상위 클래스보다 그 클래스가 구현한 인터페이스를 먼저 찾음)
- {@inheritDoc} 태그로 상위 타입의 문서화 주석 일부 상속 가능

<br>

## 문서화 주석 주의사항

- 여러 클래스가 상호작용하는 복잡한 API 경우 아키텍처 설명도 필요할 때가 있음
- 이 때는 관련 클래스나 패키지의 문서화 주석에서 그 문서의 링크를 제공해주면 좋음

<br>

## 자바독 문서를 올바르게 작성했는지 확인하는 기능

- 자바 7에서는 CLI에서 -Xdoclint 스위치를 켜주면 기능이 활성화, 자바 8부터는 기본으로 작동
- 체크 스타일(checkstyle) 같은 IDE 플러그인을 사용하면 더 완벽하게 검사됨
- 자바독이 생성한 HTML 파일을 HTML 유효성 검사기로 돌리면 문서화 주석의 오류를 한층 더 줄일 수 있음 (HTML 유효성 검사기는 잘못 사용한 HTML 태그를 찾아줌)
- 로컬에 내려받아 사용할 수 있는 설치형 검사기
- 웹에서 바로 사용할 수 있는 W3C 마크업 검사 서비스[W3C-validator]
- 자바 9와 10의 자바독은 기본적으로 HTML 4.01 문서를 생성, 명령줄에서 -html5 스위치를 켜면 HTML5 버전으로 만들어줌
- 자바독 유틸리가 생성한 웹페이지를 읽어보는 것이 제일 정확

<br>

## 결론

- 문서화 주석은 API를 문서화하는 가장 좋은 방법
- 공개 API라면 빠짐없이 설명을 달아라 (표준 규약 일관되게 지키면서)
- 문서화 주석에 임의의 HTML 태그 사용 가능(but HTML 메타문자는 특별하게 취급해야함)