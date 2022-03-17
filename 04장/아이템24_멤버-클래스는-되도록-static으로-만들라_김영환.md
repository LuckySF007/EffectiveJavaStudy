# Item24. 멤버 클래스는 되도록 static으로 만들라!



## 중첩 클래스 (Nested class)

- 다른 클래스 안에 정의된 클래스를 말한다. 
- 자신을 감싼 바깥 클래스에서만 쓰여야 한다.
  - 그 외에 쓰임이 있다면 톱레벨 클래스로 만든다.



### 중첩 클래스의 종류

- 정적 멤버 클래스 
- (비정적) 멤버 클래스
- 익명 클래스
- 지역 클래스



### 정적 멤버 클래스

- 다른 클래스 안에 선언되고, 바깥 클래스의 private 멤버에도 접근할 수 있다는 점만 빼면 일반 클래스와 동일.

- 다른 정적 멤버와 똑같은 접근 규칙을 적용받는다.

- private으로 선언 : 바깥 클래스에서만 접근 가능

- 보통 정적 멤버 클래스는 바깥 클래스와 함께 쓰일 때만 유용한 public 도우미 클래스로 쓰인다.

  ~~~java
  public class Calculator {
      public enum Operation {
          PLUS(Integer::sum),
          MINUS((x, y) -> x - y);
  
          private BiFunction<Integer, Integer, Integer> calculate;
  
          Operation(BiFunction<Integer, Integer, Integer> calculate) {
              this.calculate = calculate;
          }
  
          public BiFunction<Integer, Integer, Integer> getFunction() {
              return calculate;
          }
      }
  
      public int Sum(int x, int y) {
          return Operation.PLUS.getFunction().apply(x, y);
      }
  }
  ~~~

  ~~~java
  public static void main(String[] args) {
    	Calculator c = new Calculator();
    	System.out.println(c.Sum(1, 2)); // 3
   	 	System.out.println(Calculator.Operation.PLUS); // PLUS
  }
  ~~~



### 비정적 멤버 클래스

- 구문상 차이는 단지 static 여부의 차이

- 의미상 차이는 의외로 꽤 크다. 

  - 비정적 멤버 클래스의 인스턴스는 바깥 클래스의 인스턴스와 암묵적으로 연결된다.
    - 비정적 멤버 클래스의 인스턴스 메서드에서 정규화된 this를 사용해 바깥 인스턴스 메서드 호출 및 참조를 가져올 수 있다.
    - `클래스명.this` 형태를 사용한다.
  - 개념상 중첩 클래스의 인스턴스가 바깥 인스턴스와 독립적으로 존재 가능하면, 정적 멤버 클래스로 만들어야 한다.
    - 비정적 멤버 클래스는 바깥 인스턴스 없이 생성 불가하다.

- 비정적 멤버 클래스의 인스턴스와 바깥 인스턴스 사이 관계는 멤버 클래스가 인스턴스화될 때 확립되며, 변경 불가하다.

- 비정적 멤버 클래스는 어댑터를 정의할 때 자주 쓰인다.

  - 즉, 어떤 클래스의 인스턴스를 감싸 마치 다른 클래스의 인스턴스처럼 보이게 하는 뷰로 사용한다.

- HashMap의 KeySet 비정적 멤버 클래스 (예시)

  ~~~java
  public class HashMap<K,V> extends AbstractMap<K,V>
      implements Map<K,V>, Cloneable, Serializable {
      ...
      public Set<K> keySet() {
          Set<K> ks = keySet;
          if (ks == null) {
              ks = new KeySet();
              keySet = ks;
          }
          return ks;
      }
      final class KeySet extends AbstractSet<K> {
          public final int size()                 { return size; }
          public final void clear()               { HashMap.this.clear(); }
          public final Iterator<K> iterator()     { return new KeyIterator(); }
          public final boolean contains(Object o) { return containsKey(o); }
          public final boolean remove(Object key) {
              return removeNode(hash(key), key, null, false, true) != null;
          }
          public final Spliterator<K> spliterator() {
              return new KeySpliterator<>(HashMap.this, 0, -1, 0, 0);
          }
          public final void forEach(Consumer<? super K> action) {
              Node<K,V>[] tab;
              if (action == null)
                  throw new NullPointerException();
              if (size > 0 && (tab = table) != null) {
                  int mc = modCount;
                  for (Node<K,V> e : tab) {
                      for (; e != null; e = e.next)
                          action.accept(e.key);
                  }
                  if (modCount != mc)
                      throw new ConcurrentModificationException();
              }
          }
      }
  }
  ~~~

- 멤머 클래스 바깥에서 인스턴스에 접근할 일이 없다면 무조건 static을 붙여서 정적 멤버 클래스로 만들자.

  - static 생략 시 숨은 외부 참조를 갖게 된다. 이는 시공간의 낭비다.
  - 가비지 컬렉터가 바깥 클래스의 인스턴스 회수를 못하게 되어 메모리 누수가 생길 수 있다.

- private 정적 멤버 클래스는 흔히 바깥 클래스가 표현하는 객체의 한 부분을 나태낼 때 사용한다.

  - 예) Map 인스턴스
    - 키-값 상을 표현하는 엔트리 객체들을 가지고 있다.
    - 모든 엔트리가 맵과 연관되었지만 메서드들을 직접 사용하지는 않는다.
    - 따라서 엔트리를 비정적 멤버 클래스로 표현하는 것은 낭비다. private 정적 멤버 클래스가 알맞다.


- 멤버 클래스가 공개된 클래스의 public이나 protected 멤버라면 정적이냐 아니냐는 두 배로 중요해진다. 
  - 멤버 클래스 역시 공개 API가 된다. 혹시라도 향후 배포에서 static을 붙이면 하위 호환성이 깨짐에 유의하자.



### 익명 클래스

- 이름이 없는 클래스이며, 바깥 클래스의 멤버가 아니다.
- 쓰이는 시점에 선언 및 인스턴스 생성이 이뤄진다.
- 어디서든 만들어질 수 있으나, 비정적인 문맥에서 사용될 경우 바깥 클래스의 인스턴스 참조도 가능하다.
- 상수 변수 이외에 정적 멤버를 가질 수 없다.
- 익명 클래스는 응용에 제약이 많다.
  - 선언 시점에서만 인스턴스 생성이 가능하며, instanceof 검사나 클래스 이름이 필요한 작업은 당연히 불가능하다.
  - 여러 인터페이스 구현 안되고, 인터페이스 구현과 동시에 다른 클래스 상속은 불가하다.
  - 클라이언트는 익명 클래스가 상위 타입에서 상속한 멤버 외에는 호출할 수 없다.
  - 익명 클래스는 가독성을 위해 짧게(10줄 이하로) 구성하는 것이 좋다.

- 람다 지원 전에 많이 사용했으나, 현재는 그 자리를 대부분 람다에 물려준 상황이다.



### 지역 클래스

- 네 가지 중첩 클래스 중 가장 드물게 사용된다. 
- 지역 변수 선언이 가능한 곳이면 어디서든 선언 가능하다. 유효 범위도 지역 변수와 동일하다.
- 멤버 클래스처럼 이름이 있고, 반복해서 사용하는 것이 가능하다.
- 익명 클래스처럼 비정적 문맥에서만 인스턴스 참조가 가능하며, 정적 멤버는 가질 수 없고, 가독성 위해 짧게 구성해야 한다.



## 핵심 정리

- 중첩 클래스의 종류는 네 가지가 있다. 각각의 쓰임은 다르다.
- 메서드 밖에서도 사용해야 하는 경우, 메서드 안에 정의하기에 너무 긴 경우, 멤버 클래스로 구성한다.
- 멤버 클래스의 인스턴스 각각이 바깥 인스턴스 참조 시 -> 비정적 멤버 클래스
  - 그렇지 않은 경우 -> 정적 멤버 클래스

- 중첩 클래스가 한 메서드 안에서만 쓰이면서 인스턴스 생성 지점이 한 곳이고, 해당 타입으로 쓰기에 적합한 클래스나 인터페이스가 이미 있는 경우, 익명 클래스를 사용한다.
  - 그렇지 않은 경우 -> 지역 클래스를 사용한다.



