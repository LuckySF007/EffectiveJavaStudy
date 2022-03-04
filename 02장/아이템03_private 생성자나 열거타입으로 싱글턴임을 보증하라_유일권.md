# ITEM 03. **Private 생성자나 열거 타입으로 싱글턴임을 보증하라.**
<br>

## **싱글턴**
* * *
### **정의**
인스턴스를 오직 하나만 생성할 수 있는 클래스

### **사용 예**
함수와 같은 무상태 객체나 설계상 유일해야 하는 시스템 컴포넌트

## **만드는 방식**
* * *
방식은 총 3가지 이지만 대부분 2가지 중 하나를 사용한다.  
두 방식 모두 생성자는 private으로 숨겨두고, 유일하게 인스턴스에 접근 가능한 수단으로 public static 멤버 하나를 마련해둔다.  
단, 리플렉션 API에 대한 예외는 따로 선언 해주어야 한다.
### **방식 1**
pubic static 멤버가 final로 선언된 방식
```java
public class Elvis {
    public static final Elvis INSTANCE = new Elvis();
    private Elvis() { ... }

    public void leaveTheBuilding() { ... }
}
```
생성자는 처음 final필드인 `Elvis.INSTANCE`를 초기화 할떄 단 한번만 호출된다.  
### **장점**
1. 싱글턴 방식으로 선언되었음이 API에 명확히 들어남
2. 코드의 간결함

### **방식 2**
정적 팩터리 메서드를 통한 방식
```java
public class Elvis {
    private static final Elvis INSTANCE = new Elvis();
    private Elvis() { ... }
    public static Elvis getInstance() { return INSTANCE; }

    public void leaveTheBuilding() { ... }
}
```
`Elvis.getInstance()`는 항상 같은 객체의 참조를 반환하므로 제2의 Elvis 인스턴스는 결코 만들어 지지 않는다.
### **장점**
1. API를 변경하지 않고도 싱글턴이 아니게 변경이 가능하다.
2. 정적 팩터리를 제네릭 싱글턴 팩터리로 변경이 가능하다.
3. 정적 팩터리의 메서드 참조를 공급자로 사용이 가능하다.

### **유의사항**
만약 싱글턴 클래스를 직렬화 하려면  `Serializable`을 구현한다 선언하는 것만으로는 부족하다.  
모든 인스턴스 필드에 `transient`를 선언하고, `readResolve`메서드를 제공해야 역직렬할때마다 새로운 인스턴스가 생성된다.
```java
public Object readResolve(){
    // '진짜' Elvis를 반환하고, 가짜 Elvis는 가비지 컬렉터에 맡긴다.
    return INSTANCE;
}
```
### **방식 3**
원소가 하나인 열거 타입을 선언하는 방식
```java
public enum Elvis {
    INSTANCE;

    public void leaveTheBuilding() { ... }
}
```
public 필드 방식과 비슷하지만, 더 간결하고, 추가 노력없이 직렬화 가능하고, 복잡한 직렬화 상황이나, 리플렉션 공격에도 제2의 인스턴스 생성을 완벽히 막아준다. **대부분의 상황에서는 원소가 하나뿐인 열거 타입이 싱글턴을 만드는 가장 좋은 방법이다.** 단, 만들려는 싱글턴이 Enum 이외의 클래스를 상속해야한다면 이 방법은 사용할 수 없다.