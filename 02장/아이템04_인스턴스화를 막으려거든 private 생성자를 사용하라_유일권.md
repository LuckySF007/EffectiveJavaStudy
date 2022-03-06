# ITEM 04. **인스턴스화를 막으려거든 private 생성자를 사용하라.**

## **정적 멤버만 담은 클래스**
- 객체 지향적이지 않지만, 나름의 쓰임새가 존재한다.
- `java.lang.Math` 와 `java.util.Arrays` 처럼 기본 타입 값, 배열관련 메서드를 모아둘 수 있다.
-  `java.util.Collections` 처럼 특정 인터페이스를 구현하는 객체를 생성해주는 정적 메서드를 모아놓을 수도 있다.
-  `final` 클래스와 관련한 메서드들을 모아놓을때도 사용한다.


## **인스턴스화 막는 법**
- **추상 클래스로 만드는 것으로는 인스턴스화를 막을 수 없다.**(하위 클래스를 활용하여 인스턴스화 가능)
- private 생성자를 추가하여 클래스의 인스턴스화를 막을 수 있다.
- 생성자가 private이므로 상속도 불가능하다.
```java
public class UtilityClass {
    //기본 생성자가 만들어지는 것을 막는다(인스턴스화 방지용)
    private UtilityClass(){
        throw new AssertionError();
    }
    ... //나머지 코드는 생략
}
```
