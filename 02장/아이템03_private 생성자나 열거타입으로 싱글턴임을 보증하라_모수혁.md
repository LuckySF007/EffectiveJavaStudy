# 아이템 03_private 생성자나 열거 타입으로 싱글턴임을 보증하라

### public static 멤버가 final 필드인 방식

```java
public class Elvis {
    
    public static final Elvis INSTANCE = new Elvis();
    
    private Elvis(){};
    
    public void leaveTheBuilding(){};
}
```

private 생성자는 Elvis.INSTANCE를 초기화 할 떄 한번만 호출된다.

예외는 권한이 있는 클라이언트 리플렉션 API인 AccessibleObject.setAccessible을 사용해 private 생성자를 호출 할 수 있다.

이러한 공격을 방어하려면 생성자를 수정하여 두 번째 객체가 생성되려 할 때 예외를 던지면 된다.

### 정적 팩터리 메서드를 사용한 방식

```java
public class Elvis {

    public static final Elvis INSTANCE = new Elvis();

    private Elvis(){};
    
    public static Elvis getInstance(){return INSTANCE;}
    public void leaveTheBuilding(){};
}
```

장점

1. API를 바꾸지 않고 싱글톤으로 변환 가능
2. 원한다면 정적팩터리를 제네릭 싱글턴 팩토리로 만들수 있다
3. Elvis::getInstance를 Supplier<Elvis>로 사용 가능

### 열거 타입 방식의 싱글턴

```java
public enum Elvis {

    INSTANCE;
    
}
```

장점

1. 간단하다
2. 추가 노력 없이 직렬화할 수 있다
3. 리플렉션 공격에서도 제 2의 인스턴스가 생기는걸 막아준다

단, 만드려는 싱글턴이 Enum 외의 클래스를 상속해야 한다면 이 방법을 사용 할 수 없다.

**대부분 상황에서는 원소가 하나뿐인 열거 타입이 싱글턴을 만드는 가장 좋은방법이다**