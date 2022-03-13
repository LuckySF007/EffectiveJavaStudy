# Item14. Comparable을 구현할지 고려하라!

- `Comparable` 인터페이스의 유일무이한 메서드 -> `compareTo`

- `compareTo`는 단순 동치성 비교에 더해 순서까지 비교할 수 있으며 제네릭하다.

- `Comparable`을 구현했다는 것은 그 클래스의 인스턴스들에는 자연적인 순서(`natural order`)가 있음을 뜻한다. 

- `Comparable`을 구현한 객체는 다음처럼 손쉬운 정렬이 가능하다.

  ~~~java
  Arrays.sort(a);
  ~~~

- 사실상 자바 플랫폼 라이브러리의 모든 값 클래스와 열거 타입이 `Comparable`을 구현했다.



## compareTo의 일반 규약

> compareTo 개요
>
> 객체를 비교한다. 비교하려는 객체가 파라미터로 주어진 객체보다 작으면 음의 정수를, 같으면 0을, 크면 양의 정수를 반환한다.
> 비교할 수 없는 경우 ClassCastException을 던진다.

- `Comparable`을 구현한 클래스는 모든 `x`, `y`에 대해 `sgn(x.compareTo(y))==-sgn(y.compareTo(x))`여야 한다. 
- `Comparable`을 구현한 클래스는 추이성을 보장해야 한다. 즉, `x.compareTo(y)>0 && y.compareTo(z)>0`이면 `x.compareTo(z)>0`이다.
- `Comparable`을 구현한 클래스는 모든 `z`에 대해 `x.compareTo(y)==0`이면 `sgn(x.compareTo(z))==sgn(y.compareTo(z))`다.

- 필수는 아니지만 지켜야하는 권고가 있다. `x.compareTo(y)==0`이면 `x.equals(y)`여야 한다.
  이 권고를 지키지 않는 경우 "주의: 이 클래스의 순서는 equals 메서드와 일관되지 않다."라는 것을 명시하면 좋을 것이다.

### 알아두기

- `equals` 메서드와 달리, `compareTo`는 타입이 다른 객체를 신경 쓰지 않아도 된다.
- 타입이 다른 객체가 주어진다면 간단히 `ClassCaseException`을 던져 해결해도 좋다.

- `compareTo` 규약을 지키지 못하면 비교를 활용하는 클래스와 어울리지 못한다.
  비교를 활용하는 클래스의 예로는 `TreeSet`, `TreeMap`, `Collections`, `Arrays`가 있다.

### 주의사항

- `compareTo` 메서드로 수행하는 동치성 검사도 `equals` 규약과 똑같이 반사성, 대칭성, 추이성을 충족해야 한다.
- `Comparable`을 구현한 클래스를 확장해 값 컴포넌트를 추가하고 싶다면, 확장 대신 독립된 클래스를 만들고, 이 클래스에 원래 클래스의 인스턴스를 가리키는 필드를 두어 우회해 문제를 해결할 수 있다. 그 다음 내부 인스턴스를 반환하는 '뷰' 메서드를 제공한다.
- 마지막 규약에서 권고하는 내용처럼 `compareTo`와 `equals`가 일관되는 것이 좋다. 
  예로 `BigDecimal`은 두 메서드가 일관되지 않아, `equals`로 비교하는 `HashSet`과 `compareTo`로 비교하는 `TreeSet`에 원소를 넣었을 때의 결과값이 다르게 된다. 



## compareTo 작성 요령

- `equals` 작성 요령과 비슷하다. 몇가지 차이점만 유의하자.
- `Comparable`은 타입을 인수로 받는 제네릭 인터페이스다. 고로 `compareTo` 메서드의 인수 타입은 컴파일 타임에 정해진다. 이는 입력 인수 타입을 확인하거나 형변환의 필요가 없다는 것이다.
  - 인수 타입이 잘못된 경우 컴파일 에러
  - `null`을 인수로 넣으면 `NullPointerException`

- `compareTo` 메서드는 각 필드가 동치인지를 비교하는 것이 아니라 그 순서를 비교한다.
  - 객체 참조 필드 비교 시 `compareTo` 메서드를 재귀 호출한다.
  - `Comparable`을 구현하지 않은 필드나 표준이 아닌 순서로 비교 시 `Comparator`를 사용한다.
- 과거 `compareTo` 메서드는 관계 연산자인 `<, >, =`을 사용하는 방식이었다. 하지만 이는 거추장스럽고 오류를 유발한다. 현재는 기본 타입 비교에 박싱된 기본타입이 제공하는 정적 메서드 `compare`를 이용하는 것이 좋다.



## Comparator 비교자 활용

- Java8에서 `Comparator` 인터페이스가 일련의 비교자 생성 메서드와 팀을 꾸려 메서드 연쇄 방식으로 비교자를 생성할 수 있게 되었다.
- 필드 하나하나를 `compare` 메서드로 비교하기 힘든 경우, `Comparator`를 활용해 해결할 수 있다. 

~~~java
import java.util.Comparator;

public class Student implements Comparable<Student> {
    private static final Comparator<Student> COMPARATOR =
            Comparator.comparingInt((Student student) -> student.grade)
                    .thenComparing((Student student) -> student.name)
                    .thenComparingInt((Student student) -> student.age);

    private int grade;
    private String name;
    private int age;

    @Override
    public int compareTo(Student o) {
        return COMPARATOR.compare(this, o);
    }
}
~~~

> comparingInt
>
> 객체 참조를 int 타입 키에 매핑하는 키 추출 함수를 인수로 받아, 그 키를 기준으로 순서를 결정하는 비교자를 반환하는 정적 메서드

- 위 예제에서는 `grade`가 같은 경우를 대비해, `name`, `age`를 또다른 기준으로 하고 있다. 이때 두번 `thenComparingInt`를 통해 비교자를 추가 생성한다.

> thenComparingInt
>
> Comparator의 인스턴스 메서드로, int 키 추출자 함수를 입력받아 다시 비교자를 반환한다.



## "값의 차"를 기준으로 비교

~~~java
static Comparator<Student> hashCodeOrder = new Comparator<>() {
    @Override
    public int compare(Student o1, Student o2) {
        return o1.hashCode() - o2.hashCode();
    }
};
~~~

- 이 방식은 정수 오버플로우를 일으키거나 IEEE 754 부동소수점 계산 방식에 따른 오류를 낼 수 있다.

- 그러므로 정적 `compare` 메서드를 활용한 비교자를 이용하자.

  ~~~java
  static Comparator<Student> hashCodeOrder = new Comparator<>() {
      @Override
      public int compare(Student o1, Student o2) {
          return Integer.compare(o1.hashCode(), o2.hashCode());
      }
  };
  ~~~

- 비교자 생성 메서드를 활용한 비교자를 이용해도 된다.

  ~~~java
  private static final Comparator<Student> hashCodeOrder = Comparator.comparingInt(Object::hashCode);
  ~~~

  

## 핵심 정리

- 순서를 고려해야 하는 값 클래스를 작성한다면 꼭 `Comparable` 인터페이스를 구현하여, 그 인스턴스들을 쉽게 정렬하고, 검색하고, 비교 기능을 제공하는 컬렉션과 어우러지도록 해야 한다.
- `compareTo` 메서드에서 필드의 값을 비교할 때 `<`와 `>` 연산자는 쓰지 말아야 한다.
- 비교 연산자 대신 박신된 기본 타입 클래스가 제공하는 정적 `compare` 메서드나 `Comparator` 인터페이스가 제공하는 비교자 생성 메서드를 사용하자.
