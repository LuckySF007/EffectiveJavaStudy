# 아이템40.@Override 애너테이션을 일관되게 사용하라

## **@Override**

- 자바가 기본으로 제공하는 애너테이션 중 중요한 애너테이션
- 메서드 선언에만 달 수 있는 애너테이션
- 상위 타입의 메서드를 재정의했음을 뜻함 → 버그 예방

만약, equals 메서드를 다음과 같이 재정의하였다면?

```java
public boolean equals(Bigram bigram) {
    return bigram.first == this.first && bigram.second == this.second;
}
```

위의 equals 메서드는 재정의(overriding)한게 아니라 다중정의(overloading) 해버렸다.

Object의 equals 를 재정의하려면 매개변수 타입을 Object로 해야하는데 그렇게 하지 않았다.

실수로 별개인 equals를 새로 정의한 꼴이 되었다.

만약 그냥 위와 같이 코드를 작성했다면, 재정의를 의도했지만 위와 같이 실수를 하더라도 알 수가 없다.

컴파일러가 이 오류를 찾아주기를 원한다면, 재정의하고자 하는 메서드에 @Override 를 붙여야한다. 

이는 버그를 예방할 수 있다. 

```java
@Override
public boolean equals(Object bigram) {
    if(!(o instanceof Bigram)) {
        return false;
    }
    Bigram b = (Bigram) bigram;
    return b.first == this.first &&
        b.second == this.second;
}
```

그러니 상위 클래스의 메서드를 재정의할때는 꼭 @Override 를 붙이자.

상위 클래스의 추상 메서드를 재정의할때는 굳이 안달아도 된다. 

어차피 구현하지 않은 추상메서드가 남아있다면 컴파일러가 알려주기 때문이다.

@Override는 클래스뿐 아니라 인터페이스의 메서드를 재정의할때도 사용할 수 있다.

디폴트 메서드가 생기면서, 인터페이스 메서드를 구현한 메서드에서도 이 애너테이션을 다는 습관을 들이면 좋다.

<br>

## **결론**

재정의한 모든 메서드에 @Override 애너테이션을 의식적으로 달면 실수 시에 컴파일러가 오류를 알려줄 것이다.

구체 클래스에서 상위 클래스의 추상 메서드를 재정의한 경우에만 애너테이션을 달지 않아도 된다. (달아도 괜찮고)