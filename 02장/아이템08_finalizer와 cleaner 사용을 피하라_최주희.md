# 아이템08.finalizer와 cleaner 사용을 피하라

## finalizer와 cleaner

자바가 제공하는 두가지 객체 소멸자다. 

- finalizer는 예측할 수없고 위험할 수 있어 일반적으로 불필요하여 자바9 이후 deprecataed 되었다.
- 그 대안인 cleaner는 finalizer보단 덜하지만 여전히 예측할 수 없고 느려서 불필요하다.
- c++의 파괴자(destructor) 와는 다른 개념이다. 
c++에서 파괴자는 비메모리 자원을 회수하는 용도르 쓰이지만,
자바에서는 try-with-resources나 try-finally를 사용하여 해결한다.

<br>

## finalizer와 cleaner의 부작용

- finalizer와 cleaner는 즉시 수행된다는 보장이 없어 제때 실행되어야 하는 작업은 절대 할 수 없다.
ex) 파일 닫기를 finalizer와 cleaner에 맡기면 오류가 날 수 있다. 시스템이 동시에 열수 있는 파일의 개수에 한계가 있는데, finalizer나 cleaner 실행을 게을리하여 파일을 계속 열어둔다면 새 파일을 열지 못할 수 있기 때문이다.
- 가비지 컬렉터 구현마다 천차만별의 수행속도를 가진다.
- 자바 언어 명세는 finalizer는 cleaner의 수행여부조차 보장하지 않는다. 상태를 영구적으로 수정하는 작업에서는 절대 finalizer나 cleaner에 의존해서는 안된다.
- finalizer 동작 중 발생한 예외는 무시되고 처리할 작업이 남았어도 즉시 종료해버린다. 잡지 못한 예외 때문에 객체가 훼손되고 다른 스레드가 이를 사용하려한다면 어떻게 동작할지 예측할 수 없다.
- AutoCloseable 객체를 생성하고 try-with-resources로 자신을 닫도록 하는 방법에 비해 finalizer는 50배나 느릴 수 있다.
- 보안에도 심각하다. 생성자나 직렬화 과정에서 예외가 발생하면, 생성되다만 객체에서 악의적인 하위 클래스의 finalizer가 수행될 수 있기 때문. 객체 생성을 막기위해 생성자에서 예외를 던지는 것도 finalizer가 있다면 불가능하다. 
→ final이 아닌 클래스들은 이 공격을 방어하기 위해 아무일도 하지 않는 finalize 메서드를 만들고 final로 선언하자.

<br>

## finalizer와 cleaner의 대체

AutoCloseable을 구현해주고, 객체를 다 쓰고나면 close 메서드를 호출해주면 된다.

일반적으로 예외가 발생하도 종료되도록 try-with-resources를 사용해야한다.

close메서드에서 이 객체는 더이상 유효하지 않음을 필드에 기록하고, 다른 메서드는 이 필드를 검사해서

객체가 닫힌후에 불렸다면 IllegalStateException을 던지는것이 좋다.

<br>

## finalizer와 cleaner의 쓰임새

1) 자원의 소유자가 close 메서드를 호출하지 않는 것에 대비

늦게라도 자원회수를 하는 것이 안하는것보다 낫기 때문에 안전망 역할을 하도록 쓰인다.

ex) FileInputStream, FileOutputStream, ThreadPoolExecutor

2) 네이티브 피어(native peer) 와 연결된 객체

네이티브 피어는 일반 자바객체가 네이티브 메서드를 통해 기능을 위임한 네이티브 객체를 말한다.

네이티브 피어는 자바 객체가 아니어서 가비지 컬렉터가 존재를 알지 못한다.

그래서 자바 피어를 회수할때 네이티브 객체까지 회수하지 못하기 때문에 finalizer나 claenr가 처리하기 적당하다. (성능 저하를 감수해야하며 네이티브 피어가 심각한 자원을 가지고 있을때만 해당)

<br>

## cleaner 사용 예제

```java
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
      try (Room myRoom = new Room(7)) {
	      System.out.println("안녕~");
	  }
	  new Room(99);
	  System.out.println("아무렴~");
  }
}
```

State는 Runnable을 구현하고, 그 안의 run 메서드는 cleanable에 의해 딱 한번 호출된다.

close 메서드에서 cleanable의 clean을 호출하면 이 메서드 안에서 run을 호출한다.

혹은 바라건대 가비지 컬렉터가 Room을 회수할때까지 close를 호출하지 않는다면, cleaner가 run을 호출할것이다.

그러나 위의 cleaner는 단지 안전망으로 쓰였기 때문에 클라이언트가 모든 Room 생성을 try-with-resources 블록으로 감싸는것이 더 좋다.

<br>

## 결론

cleaner는 안전망 역할이나 중요하지 않은 네이티브 자원 회수용으로만 사용하자. (불확실성과 성능 주의)