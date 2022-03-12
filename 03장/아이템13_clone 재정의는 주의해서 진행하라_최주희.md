# 아이템13.clone 재정의는 주의해서 진행하라

clone 메서드가 선언된 곳이 Cloneable이 아닌 Object이며, 접근 제한자가 protected 이기 때문에

Cloneable을 구현하는 것만으로는 외부 객체에서 clone 메서드를 호출할 수 없다.

이런 문제점에도 불구하고 Cloneable 바식은 널리 쓰인다. 이에 대해 알아보자.

<br>

## Cloneable 인터페이스

- Object의 protected 메서드인 clone의 동작 방식을 결정
- Cloneable을 구현한 클래스의 인스턴스에서 clone을 호출하면 그 객체의 필드들을 하나하나 복사한 객체를 반환, 그렇지 않은 클래스의 인스턴스에서 호출하면 CloneNotSupportedException 을 던짐

<br>

## clone 메서드의 허술한 일반 규약

- x.clone() != x 는 참이어야한다. → 복사한 객체는 원본 객체와 독립적
- x.clone().getClass() == x.getClass() 는 참이어야한다. → 복사한 객체와 원본 객체는 같은 클래스
- x.clone().equals(x) 는 일반적으로 참이지만, 필수는 아니다.

<br>

## 생성자 연쇄의 문제점

어떤 클래스가 clone()을 super.clone() 이 아닌, 생성자를 호출해 얻은 인스턴스를 반환하도록 재정의했다하더라도 문제가 없다. 그러나, 이 클래스의 하위 클래스에서 clone()을 호출할때, 상위 클래스가 new 키워드로 생성한 객체를 반환하기 때문에 하위 클래스 타입이 아닌 상위 클래스의 타입 객체를 반환하게 된다. 

이를 다음과 같이 해결하자.

```java
//불변 객체의 경우
@Override
public PhoneNumber clone() {
    PhoneNumber clone;
    try {
        clone = (PhoneNumber) super.clone();
    } catch (CloneNotSupportedException e) {
        throw new AssertionError();  //일어날 수 없는 일이다.
    }
    return clone;
}
```

 클래스의 모든 필드가 기본타입이거나 불변이면 super.clone()을 호출하고 공변 반환 타이핑을 하여 클라이언트가 형변환하지 않아도 되게끔 하고 결과를 반환한다.

try-catch 블록을 이용하여 Object의 clone()이 checked exception인 CloneNotSupportedException을 던지는 것을 unchecked exception 으로 처리한다.

## 가변 객체 참조

- 가변 객체를 참조할 때 clone()이 단순히 super.clone()을 반환한다면?
    
    만약, 복제본 객체가 원본 객체의 배열 필드를 참조하게 된다면 둘 중 하나만 수정하더라도 다른 하나도 수정되어 불변식을 해치게 된다. → 이상하게 동작 , NullPointerException 발생
    
- clone 메서드는 생성자와 같은 효과기 때문에 원본 객체에 해를 끼치지 않으면서 복제 객체의 불변식을 보장해야함.

→ 필드의 clone()을 재귀적으로 호출하는 방법이 있다.

```java
@Override
public Stack clone() {
    try {
        Stack clone = (Stack) super.clone();
        clone.elements = elements.clone();  //배열을 재귀적으로 호출
        return clone;
    } catch(CloneNotSupportedException e) {
        throw new AssertionError();
    }
}
```

그러나 여기에도 문제점이 있다.

- 만약 가변 참조 필드가 final 이라면 새로운 값을 할당할 수 없어서 위 방식을 사용할 수 없음
- Cloneable 아키텍처는 ‘가변 객체를 참조하는 필드는 final로 선언하라’는 일반 용법과 충돌함
- 결국 복제를 위하여 일부 필드에서 final을 제거해야만하는 상황

<br>

## clone() 재귀 호출이 부족하면 deepCopy()

clone()을 재귀적으로 호출하는 것만으로 충분하지 않을때가 있다.

만약 배열안에 가변참조 객체가 있다면? (ex.해시테이블 내부는 버킷들의 배열이고, 각 버킷은 키-값 쌍을 담는 연결리스트의 첫번째 엔트리를 참조)

이럴때는 배열을 순회하며 각각 깊은복사(deep copy)를 수행한다. 

```java
public class HashTable implements Cloneable {
  private Entry[] buckets = ...;
  private static class Entry {
    final Object key;
    Object value;
    Entry next;
    //생략 ..
  }

  //이 엔트리가 가리키는 연결리스트를 재귀적으로 복사 
  Entry deepCopy() {
    return new Entry(key, value, next == null ? null : next.deepCopy());
  }

  @Override public HashTable clone() {
    try {
      HashTable result = (HashTable) super.clone();
      result.buckets = new Entry[buckets.length];
      for (int i = 0; i < buckets.length; i++)
        if (buckets[i] != null)
          result.buckets[i] = buckets[i].deepCopy();
      return result;
    } catch (CloneNotSupportedException e) {
      throw new AssertionError();
    }
  }
}
```

그러나 여기에도 문제점이 있다.

재귀호출 때문에 연결리스트의 원소수만큼 스택 프레임을 소비하기 때문에 크기가 크다면 스택 오버플로우가 발생할 수 있다. → deepCopy를 재귀 호출 대신 반복자로 순회

```java
Entry deepCopy() {
  Entry result = new Entry(key, value, next);
  for (Entry p = result; p.next != null; p = p.next)
    p.next = new Entry(p.next.key, p.next.value, p.next.next);
  return result;
}
```

<br>

## 주의사항

- clone() 에서는 재정의될 수 있는 메서드를 호출하지 않아야한다. 그렇게 되면 하위 클래스는 복제 과정에서 자신의 상태를 교정할 기회를 잃어 원본과 복제본과 상태가 달라질 가능성이 크기 때문이다.
- public 인 clone()에서는 throws 절을 없애야 사용하기 쉽다.

<br>

## 요약

- Cloneable을 구현하는 모든 클래스는 clone을 재정의해야한다.
- 접근 제한자는 public으로 , 반환 타입은 클래스 자신으로 변경한다.
- super.clone을 호출한 후에 필요한 필드를 전부 적절하게 수정한다. (가변 참조 객체는 깊은 복사)

<br>

## 복사 생성자와 복사 팩터리

웬만하면 복사생성자(자신과 같은 클래스의 인스턴스를 인수로 받는 생성자)와 복사 팩터리를 사용하자.

- 충돌이 적고 예외를 던지지도 않으며 형변환도 필요가 없다.
- 해당 클래스가 구현한 인터페이스 타입의 인스턴스를 인수로 받을 수 있다.
- 범용 컬렉션 구현체는 Collection이나 Map 타입을 받는 생성자를 제공하는데, 원본 구현 타입에 얽매이지 않고 복제본의 타입을 선택할 수 있다. 
ex) HashSet 객체 s 를 TreeSet 타입으로 복제 가능 → `new TreeSet<>(s)`

```java
//복사 생성자
public Stack(Stack s) {
    this.elements = s.elements.clone();
    this.size = s.size;
}

//복사 팩터리
public static Stack newInstance(Stack s) {
    return new Stack(s.elements, s.size);
}
```

<br>

## 결론

배열만이 clone()을 제대로 사용하는 유일한 예로, 나머지는 모두 복사 생성자와 복사 팩터리를 이용하는 것이 더 좋다.