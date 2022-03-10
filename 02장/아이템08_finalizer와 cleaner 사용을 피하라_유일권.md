# ITEM 08. **finalizer와 cleaner 사용을 피하라.**

자바에는 `finalizer`, `cleaner` 두 종류의 소멸자가 있다.  

이 두 소멸자는 위험하여 **사용을 피하는 것**이 좋다. 이유는 다음과 같다.

1. 즉시 수행 된다는 보장이 없다.
    - 얼마나 신속히 수행할지는 가비지 컬렉터 알고리즘에 달렸다.
    - `finalizer`는  어떤 스레드가 수행할지 명시하지 않아 문제를 해결할 해법이 없다.  


    - `cleaner`는 그래도 자신이 수행할 스레드를 제어할 수 있어 조금 양호하다. 하지만 여전히 즉각 수행될 것이라는 보장은 없다.
2. 수행 여부조차 보장하지 않는다.
    - 수행 여부를 보장하지 않아, 상태를 영구적으로 수정하는 작업에선 두 소멸자에 의존해서는 안 된다.
    - `System.gc`, `System.runFinalization`메서드에 현혹되지 말자. 실행 가능성은 높아질 수 있지만, 보장해주지는 않는다.
    - `System.runFinalizersOnExit`, `Runtime.runFinalizersOnExit`은 실행은 보장해주지만, 심각한 결함이 있다.  


3. 심각한 성능 문제도 동반한다. 
   - `finalizer`가 가비지 컬렉터의 효율을 떨어뜨린다.
   - `cleaner`도 클래스의 모든 인스턴스를 수거하는 형태로 사용하면 거의 동일


4. `finalizer`는 실행중 발생한 `Exception`마저 무시하며, `finalizer` 공격에 노출되어 심각한 보안 문제를 일으킬 수도 있다.
   - `finalizer`는 실행중 예외가 발생하면 경고조차 출력하지 않는다.
   - 생성자나 직렬화 과정에서 예외가 발생하면, 이 생성되다 만 객체에서 악의적인 `finalizer`를 수행될 수 있게 한다. 이를 통해 가비지 컬렉터가 수집을 못하게 막을 수 있다.
   - 객체 생성을 막으려면 생성자에서 예외를 던지는 것으로 충분하지만,  `finalizer`가 있다면 다르다.
   - ** `final`이 아는 클래스를  `finalizer` 공격으로부터 방어하려면 아무일도 하지 않는  `finalizer` 메서드를 만들로  `final`로 선언하자.

<br>

## **두 소멸자를 대신해줄 방법**

**`AutoCloseable`을 구현해주고, 클라이언트에서 인스턴스를 다 쓰고 나면 `close` 메서드를 호출해준다.** (예외가 발생하여도 제대로 종료되도록 try-with-resources를 사용해야 한다.)  

위 방법을 사용할시 각 인스턴스는 자신이 닫혔는지 추적하는 것이 좋다. 다시 말해, `close`메서드에서 이 객체는 더 이상 유효하지 않음을 필드에 기록하고, 다른 메서드는 이 필드를 검사해서 닫힌 후 불렀다면 `IllegalStateException`을 던지게 한다.

<br>

## **두 소멸자의 쓰임새**

1. 자원 소유자가 `close` 메서드를 호출하지 않는 것에 대한 안전망 역할이다.
    - ex) 자바 라이브러리의 일부(`FileInputStream`, `FileOutputStream`, `ThreadPoolExecutor`)
2. 네이티브 피어와 연결된 객체에서 활용된다.
    - 네이티브 피어는 일반 자바 객체가 네이티브 메서드를 통해 기능을 위임한 네이티브 객체를 뜻한다.
    - 네이티브 피어는 자바 객체가 아니므로 가비지 콜렉터는 존재를 알지 못해 자바 피어를 회수 할 때 네이티브 객체까지 회수하지 못한다.
    - 단, 성능 저하를 감수할 수 있고, 네이티브 피어가 심각한  자원을 가지고 있지 않을때만 사용하고, 그렇지 않을경우 `close`메서드를 사용한다.


## `cleaner`의 사용법

```java
//Room.java
import java.lang.ref.Cleaner;

public class Room implements AutoCloseable {

      private static final Cleaner cleaner = Cleaner.create();

      private static class State implements Runnable {
      int numJunkpiles;

      State(int numJunkpiles) {
	      this.numJunkpiles = numJunkpiles;
      }

      @Override
      public void run() {
	      System.out.println("방 청소");
	      numJunkpiles = 0;
      }
     }
     private final State state;
     private final Cleaner.Cleanable cleanable;

     public Room(int numJunkpiles) {
      state = new State(numJunkpiles);
      cleanable = cleaner.register(this, state);
      }

      @Override
      public void close() throws Exception {
      cleanable.clean();
      }
}

//Adult.java
public class Adult {
    public static void main(String[] args) throws Exception {
        try (Room myRoom = new Room(7)) {
            System.out.println("안녕~");
        }
    }
}
/* Output
안녕~
방 청소
*/

//Teenager.java
public class Teenager {
    public static void main(String[] args) throws Exception {
        new Room(99);
        System.out.println("아무렴~");
        System.gc();    //방 청소가 출력될지 보장 할수는 없다.
    }
}
/* Output
아무렴~
방 청소
*/
```

## **결론**

아주 특별한 경우 외에는 `finalizer`, `cleaner`는 사용하지 말자.