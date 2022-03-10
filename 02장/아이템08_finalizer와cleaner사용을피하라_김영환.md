# Item08. finalizer와 cleaner 사용을 피하라!

- 자바는 기본적으로 두 가지 객체 소멸자를 제공한다.



## finalizer

- 예측할 수 없고, 상황에 따라 위험할 수 있어 일반적으로 불필요하다.

- 나름 몇가지 쓰임이 있긴 하나 기본적으로 '<u>쓰지 말아야</u>' 한다.



## cleaner

- 자바 9에서 finalizer가 deprecated 되고 cleaner가 생김.
- 자신을 수행할 스레드를 제어할 수 있다는 점에서는 finalizer 보다 낫다. 하지만 여전히 백그라운드에서 수행되며 가비지 컬렉터의 통제 하에 있어 언제 수행될지 미지수이다.



## finalizer, cleaner를 사용하지 말아야 하는 이유

- finalizer와 마찬가지로 즉시 수행된다는 보장이 없다. 즉 객체에 접근할 수 없게 된 이후 finalizer, cleaner가 언제 실행되는지 알 수 없다.
  - 제때 실행되어야 하는 작업은 절대 할 수 없다.
- 예를 들어 파일 닫기를 맡길 경우, 시스템이 동시에 열 수 있는 파일의 수는 한정적인데 언제 파일을 닫을지 미지수이니 새 파일을 여는데 실패하는 오류를 발생시킬 수 있다.
- 두 소멸자의 사용은 단지 수행 시점이 문제가 되는 것이 아니다. 수행 여부조차 확답할 수 없다. 그렇기 때문에 상태를 영구적으로 수정하는 작업에서는 절대로 finalizer, cleaner에 의존하지 않도록 하자!
- 두 소멸자의 실행을 도와주는 메서드에 현혹되지 말자!
  - System.gc, System.runFinalization과 같이 소멸자의 실행 가능성을 높여주는 메서드가 존재한다. 하지만 여전히 실행의 보장은 되지 않는다.
  - 실행을 보장해주는 메서드가 2개 존재한다. System.runFinalizersOnExit, Runtime.runFinalizersOnExit 이다. 하지만 이 두 메서드 모두 심각한 결함이 존재해 deprecated 되었다.
- finalizer의 경우, finalizer 동작 중 발생한 예외가 무시되고, 그 순간 종료되는 문제가 있다. 이로 인해 객체가 망가질 수 있고, 다른 스레드에서 객체에 접근 시 어떤 동작을 할지 예측할 수 없게 된다.
  - 그나마 cleaner는 자신의 스레드를 통제하기에 위와 같은 문제는 발생하지 않으나, 가비지 컬렉터에 의존하고 있는 것은 동일하기에 사용을 지양한다.
- 두 소멸자는 심각한 성능 문제도 동반한다. 소멸자의 사용이 가비지 컬렉터의 효율을 떨어뜨려 결국 성능의 저하를 유발한다.
- finalizer를 사용한 클래스는 finalizer 공격에 노출되어 심각한 보안 문제를 겪을 수 있다. 그 방법은 생성자나 직렬화 과정에서 에러를 발생시켜 생성되다가 중단된 객체에서 악의적인 하위 클래스의 finalizer가 수행될 수 있게 하는 것이다.
  - 위와 같은 경우 가비지 컬렉터의 대상에서 벗어나게 한다.
  - 하위 클래스 문제는 final 클래스 사용으로 막을 수 있다. final 클래스는 하위 클래스를 갖지 못하기 때문이다.
  - final이 아닌 클래스는 아무 일도 하지 않는 finalize 메서드를 만들고 final로 선언해 finalizer 공격으로부터 보호할 수 있다.

### 예제

~~~java
class Positive {

  Integer value = 0;

  public Positive(Integer value) {
    if (value <= 0) {
      throw new IllegalArgumentException("Value must be positive");
    }
    this.value = value;
  }

  @Override
  public String toString() {
    return value.toString();
  }
}

class AttackPositive extends Positive {
  static Positive positive;

  public AttackPositive(Integer value) {
    super(value);
  }

  @Override
  protected void finalize() throws Throwable {
    System.out.println("finalizer!");
    positive = this; // gc되지 않음
  }

  public static void main(String[] args) {
    try {
      new AttackPositive(-1);
    } catch (Exception e) {
      System.out.println(e.getMessage());
    }

    System.gc();
    System.runFinalization();

    if (positive != null) {
      System.out.println("Positive object : " + positive + " created!");
    }
  }
}
~~~



## finalizer나 cleaner를 대신해줄 묘안책

- **AutoCloseable**을 구현해주고, 인스턴스를 다 사용하고 나면 **close** 메서드를 호출한다.
- 일반적으로는 예외가 발생해도 제대로 종료되도록 **try-catch-resources**를 사용한다.



## 그럼 cleaner나 finalizer는 어디에 쓰는가?

1. 자원의 소유자가 close 메서드를 호출하지 않는 것에 대한 안전망

   - 두 소멸자가 호출될 것이란 보장은 없지만 클라이언트가 잊은 자원 회수를 뒤늦게라도 해준다는 것에서 의미를 가진다. (안하는 것보단...)

2. 네이티브 피어와 연결된 객체에서 사용한다.

   - > 네이티브 피어
     >
     > 일반 자바 객체가 네이티브 메서드를 통해 기능을 위임한 네이티브 객체를 말한다.
     
   - 네이티브 피어는 자바 객체가 아니니 가비지 컬렉터의 영향을 받지 않는다. 이 경우 finalizer, cleaner가 나서서 가비지 컬렉터의 역할을 대신해 자원을 회수할 수 있을 것이다.

### cleaner를 안전망으로 활용하는 예

~~~java
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

public class main {  
  public static void main(String[] args) throws Exception {  
      try (Room myRoom = new Room(7)) { // try catch resources
	      System.out.println("안녕~");
	  }
	  new Room(99); // 언제나 '방 청소'가 출력될까?....
	  System.out.println("아무렴~");
  }
}

// Result
// 안녕
// 방 청소
// 아무렴~
~~~



## 정리

1. cleaner(Java8-> finalizer)는 안전망 역할 or 중요하지 않은 네이티브 자원 회수용으로만 사용하자!
2. 혹시라도 사용한다면 불확실성, 성능 저하에 대해 충분히 고민하고 주의해 사용하자!
3. 그냥 웬만하면 지양하는 방향도 좋을듯...(사설)
