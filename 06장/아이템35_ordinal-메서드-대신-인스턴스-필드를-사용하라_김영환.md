# Item35. ordinal 메서드 대신 인스턴스 필드를 사용하라!

- 대부분의 열거 타입 상수는 자연스레 하나의 정수값에 대응된다.
  - 모든 열거 타입은 해당 상수가 그 열거 타입의 몇 번째인지 반환하는 ordinal이라는 메서드를 제공한다.
- 열거 타입 상수와 연결된 정수값이 필요한 경우 ordinal 메서드를 사용하고 싶은 유혹이 들 수 있다. 이를 뿌리치자.



## ordinal을 잘못 사용한 예

~~~java
public enum Ensemble {
    SOLO, DUET, TRIO, QUARTET, QUINTET,
    SEXTET, SETPTET, OCTET, NONET, DECTET;

    public int numberOfMusicians() { return ordinal() + 1 }
}
~~~

- 동작은 하지만 유지보수가 아주 어려운 코드다.

  - 상수 선언 순서를 바꾸는 순간 numberOfMusicians 메서드는 오작동한다.

  

## ordinal 사용의 문제

- 이미 사용 중인 정수와 값이 같은 상수는 추가할 방법이 없다.
- 값을 중간에 비워둘 수 없다.
  - 빈 자리에 dummy 상수를 넣어 사용해야 하는데, 코드가 혼잡해진다.



## 해결책

- 열거 타입 상수에 연결된 값은 ordinal 메서드로 얻지 말고, 인스턴스 필드에 저장하자.

~~~java
public enum Ensemble {
    SOLO(1), DUET(2), TRIO(3), QUARTET(4), QUINTET(5),
    SEXTET(6), SETPTET(7), OCTET(8), NONET(9), DECTET(10);

    private final int numberOfMusicians;
    Ensemble(int size) { this.numberOfMusicians = size }
    public int numberOfMusicians() { return numberOfMusicians }
}
~~~

