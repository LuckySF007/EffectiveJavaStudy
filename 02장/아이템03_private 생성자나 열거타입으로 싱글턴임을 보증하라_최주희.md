# 아이템03.private 생성자나 열거 타입으로 싱글턴임을 보증하라

### 싱글턴(singleton)

- 인스턴스를 오직 하나만 생성할 수 있는 클래스
- 무상태 객체, 설계상 유일해야하는 시스템 컴포넌트
- 클래스를 싱글턴으로 만들면 이를 사용하는 클라이언트를 테스트하기가 어려워질 수 있다.
싱글턴 인스턴스를 mock 구현으로 대체할 수 없기 때문

### public static final 필드 방식의 싱글턴

```java
public class Elvis {
    public static final Elvis INSTANCE = new Elvis();
    private Elvis() { ... }

    public void leaveTheBuilding() { ... }
}
```

- private 생성자는 public static final 필드인 Elvis.INSTANCE를 초기화할때 한번만 호출됨.
- public 이나 protected 생성자가 없어서 한번 호출될 때의 그 인스턴스 하나뿐임이 보장되는것.
- 리플렉션 api를 이용하여 AccessibleObject.setAccessible(true)로 private 생성자 호출되나 두번째 생성 시에 예외를 던지게 하여 공격 방어
- 장점 : 싱글턴임이 api에 명백히 드러나고 간결

<br>

### 정적 팩터리 방식의 싱글턴

```java
public class Elvis {
    private static final Elvis INSTANCE = new Elvis();
    private Elvis() { ... }
    public static Elvis getInstance() { return INSTANCE; }

    public void leaveTheBuilding() { ... }
}
```

- Elvis.getInstance() 는 항상 같은 객체 참조를 반환
- 이 방식도 리플렉션 api에 의해 예외 상황 발생 가능
- 장점 :
    - api를 바꾸지 않고도 싱글턴이 아니게 변경 가능
    - 정적 팩터리를 제네릭 싱글턴 팩터리로 만들 수 있음
    - 정적 팩터리의 메서드 참조를 공급자(supplier)로 사용 가능 (Elvis::getInstance → Supplier<Elvis>)
    

```java
private Object readResolve() {
    return INSTANCE;
}
```

- public 필드 방식과 정적 팩터리 방식의 싱글턴 클래스는 직렬화하기 위해서 단순히 Serializable을 구현한다고 되지 않는다.
- 모든 인스턴스 필드에 trasient 를 선언하고, 싱글턴임을 보장할 readResolve() 메서드를 제공해야함 → 그렇지 않으면, 역직렬화시에 새로운 인스턴스가 만들어진다

<br>

### 열거 타입 방식의 싱글턴

```java
public enum Elvis {
    INSTANCE;

    public void leaveTheBuilding() { ... }
}
```

- 더 간결하고 바로 직렬화 가능
- 리플렉션 공격에서도 다른 인스턴스가 생기지 않음
- 대부분 상황에서 원소가 하나뿐인 열거타입이 싱글턴을 만드는 가장 좋은 방법
- Enum 외의 클래스를 상속해야 한다면 이 방법은 사용 불가