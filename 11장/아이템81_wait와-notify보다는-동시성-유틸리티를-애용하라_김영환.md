# Item81. wait와 notify보다는 동시성 유틸리티를 애용하라!

## Intro

- 이 책의 초판에서는 wait와 notify를 올바르게 사용하는 방법을 안내했지만, 지금은 wait와 notify를 사용해야 할 이유가 많이 줄었다.
- 자바 5에서 도입된 고수준의 동시성 유틸리티가 wait, notify로 하드코딩해야 했던 전형적인 일들을 대신 처리해준다.
- **wait와 notify는 올바르게 사용하기가 아주 까다로우니 고수준 동시성 유틸리티를 사용하자.**





## java.util.concurrent의 고수준 유틸리티

1. 실행자 프레임워크(아이템 80)
2. 동시성 컬렉션(concurrent collection)
3. 동기화 장치(synchronizer)



### 1. 실행자 프레임워크

- 아이템 80에서 자세히 다뤘으니, 참고하자.



### 2. 동시성 컬렉션

- List, Queue, Map 같은 표준 컬렉션 인터페이스에 동시성을 가미해 구현한 고성능 컬렉션.
- 높은 동시성에 도달하기 위해 동기화를 각자의 내부에서 수행한다.
- **동시성 컬렉션에서 동시성을 무력화하는 건 불가능하며, 외부에서 락을 추가로 사용하면 오히려 속도가 느려진다.**
  - 동시성 컬렉션에서 동시성을 무력화하지 못하므로 여러 메서드를 원자적으로 묶어 호출하는 일 역시 불가능하다.

#### 상태 의존적 메서드

- 여러 기본 동작을 하나의 원자적 동작으로 묶고 싶었고, 이를 위해 '상태 의존적 수정' 메서드들이 추가되었다.

#### 예시

- putIfAbsent(key, value)

  ~~~java
  putIfAbsent("key", "value"); // 키에 매핑된 값이 없을 때만 새 값을 넣고, null을 반환한다. 있다면 기존 값을 반환한다.
  ~~~

  - 이 메서드 덕에 스레드 안전한 정규화 맵(canonicalizing map)을 쉽게 구현할 수 있다.

- **ConcurrentHashMap**은 동시성이 뛰어나며 속도도 무척 빠르다. 동시성 컬렉션은 동기화한 컬렉션을 낡은 유산으로 만들어버렸다.

  - 대표적 예로, 이제는 **Collections.synchronizedMap보다는 ConcurrentHashMap을 사용하는 게 훨씬 좋다.**
  - 동기화된 맵을 동시성 맵으로 교체하는 것만으로 성능은 극적으로 개선된다.
- **BlockingQueue**는 일부 작업이 성공적으로 완료될 때까지 기다리도록(즉, 차단되도록) 확장된 컬렉션 인터페이스 중 하나다.

  - BlockingQueue에 추가된 메서드 중 take는 규의 첫 원소를 꺼내는데, 만약 큐가 비었다면 새로운 원소가 추가될 때까지 기다린다.
  - 이런 특성 덕에 BlockingQueue는 작업 큐(생산자-소비자 큐)로 쓰기에 적합하다.
  - ThreadPoolExecutor를 포함한 대부분의 실행자 서비스 구현체에서 이 BlockingQueue를 사용한다.



### 3. 동기화 장치

- 동기화 장치는 스레드가 다른 스레드를 기다릴 수 있게 하여, 서로 작업을 조율할 수 있게 해준다.
- 가장 자주 쓰이는 동기화 장치는 CountDownLatch와 Semaphore다. CyclicBarrier와 Exchanger도 있는데, 앞서 두 장치보단 적게 쓰인다.
- 가장 강력한 동기화 장치는 Phaser다.

#### CountDownLatch

- 일회성 장벽으로, 하나 이상의 스레드가 또 다른 하나 이상의 스레드 작업이 끝날 때까지 기다리게 한다.
- 생성 시 int 값을 받는다. 이 값이 countDown 메서드를 몇 번 호출해야 대기 중인 스레드를 깨우는지 결정한다.

~~~java
public static long time(Executor executor, int concurrency, Runnable action) throws InterruptedException {
  CountDownLatch ready = new CountDownLatch(concurrency);
  CountDownLatch start = new CountDownLatch(1);
  CountDownLatch done = new CountDownLatch(concurrency);

  for (int i = 0; i < concurrency; i++) {
    executor.execute(() -> {
      ready.countDown(); // 타이머에게 준비를 끝냈음을 알린다.
      try {
        // 모든 작업자 스레드가 준비될 때까지 기다린다.
        start.await();
        action.run();
      } catch (InterruptedException e) {
        Thread.currentThread().interrupt();
      } finally {
        // 타이머에게 작업을 마쳤음을 알린다.
        done.countDown();
      }
    });
  }

  ready.await(); // 모든 작업자가 준비될 때까지 기다린다.
  long startNanos = System.nanoTime();
  start.countDown(); // 작업자들을 깨운다.
  done.await(); // 모든 작업자가 일을 끝마치기를 기다린다.
  return System.nanoTime() - startNanos;
}
~~~

- time 메서드에 넘겨진 실행자(executor)는 concurrency 매개변수로 지정한 동시성 수준만큼의 스레드를 생성할 수 있어야 한다.
  - 그렇지 못하면 이 메서드는 결코 끝나지 않을 것이고, 이런 상태를 **스레드 기아 교착상태(thread starvation deadlock)**라 한다.
- **시간 간격을 잴 때는 항상 System.currentTimeMillis가 아닌 System.nanoTime을 사용하자.**
  - 더 정확하고, 정밀하며, 시스템의 실시간 시계의 시간 보정에 영향을 받지 않는다.

#### Semaphore

- Semaphore는 특정 리소스에 접근하는 것을 제한하도록 지원하는 동기화 장치다.

~~~java
Semaphore semaphore = new Semaphore(3); // 접근 가능한 스레드 개수를 3개로 제한한다.
~~~

- Semaphore의 acquire로 세마포어를 하나 사용할 수 있다.
- 자원을 다 사용했다면 반드시 release 해줘야 한다. 그렇지 않으면 다른 스레드들이 접근할 수 없게 된다.





## wait와 notify

- 새로운 코드라면 언제나 wait와 notify가 아닌 동시성 유틸리티를 쓰자.
- 하지만 어쩔 수 없이 레거시 코드를 다뤄야 하는 경우도 있으니 알아둬야 한다.



### wait

- 스레드가 어떤 조건이 충족되기를 기다리게 할 때 사용한다.
- 락 객체의 wait 메서드는 반드시 그 객체를 잠근 동기화 영역 안에서 호출해야 한다.
- wait 메서드를 사용하는 표준 방식

  ~~~java
  synchronized (obj) {
      while (<조건이 충족되지 않았다>) {
          obj.wait(); // 락을 놓고, 깨어나면 다시 잡는다.
      }
  
      ... // 조건이 충족됐을 때의 동작을 수행한다.
  }
  ~~~

- **wait 메서드를 사용할 때는 반드시 대기 반복문(wait loop) 관용구를 사용하라. 반복문 밖에서는 절대로 호출하지 말자.**
  - 이 반복문은 wait 호출 전후로 조건이 만족하는지를 검사하는 역할을 한다.
- 대기 전 조건을 검사하여 조건이 충족되었다면 wait 하지 않게 하자. 이는 응답 불가 상태를 예방하는 조치다.
- 대기 후에 조건을 검사하여 조건이 충족되지 않았다면 다시 대기하게 하자. 조건이 충족되지 않아도 스레드가 깨어날 수 있는 상황이 만들어질 수 있기 때문이다. 이는 안전 실패를 예방하는 조치다.
  - 다음이 조건이 충족되지 않아도 스레드를 깨울 수 있는 상황이다.
    - 스레드가 notify를 호출한 다음 대기 중이던 스레드가 깨어나는 사이에 다른 스레드가 락을 얻어 그 락이 보호하는 상태를 변경한다.
    - 조건이 만족되지 않았음에도 다른 스레드가 실수로 혹은 악의적으로 notify를 호출한다.
    - 깨우는 스레드의 지나친 관대함으로, 대기 중인 스레드 중 일부만 조건이 충족되어도 notifyAll을 호출해 모든 스레드를 깨울 수도 있다.
    - 대기 중인 스레드가 드물게 notify 없이도 깨어나는 경우가 있다. 이를 허위 각성(spurious wakeup)이라 부른다.



### notify, notifyAll

- notify는 스레드 하나만 깨운다.
- notifyAll은 모든 스레드를 깨운다.
- 일반적으로 언제나 notifyAll을 사용하라는 게 합리적이고 안전하다. 깨어나야 하는 모든 스레드가 깨어남을 보장하니 항상 정확한 결과를 얻을 것이다.
  - 다른 스레드가 깨어나더라도, 조건 만족 여부를 다시 비교하여 충족되지 않았으면 다시 대기할 것이니 정확성에 영향을 주지 않는다.

- notifyAll을 사용하면 관련 없는 스레드가 실수 혹은 악의적으로 wait를 호출하는 공격으로부터 보호 가능하다.





## 핵심 정리

- **코드를 새로 작성한다면 wait와 notify를 쓸 이유가 거의(어쩌면 전혀) 없다.**
- 이들을 사용하는 레거시 코드를 유지보수하는 경우, wait는 항상 표준 관용구에 따라 while문 안에서 호출하도록 하자.
- 일반적으로 notify보다는 notifyAll을 사용해야 한다.
- 혹시라도 notify를 사용하는 경우, 응답 불가 상태에 빠지지 않도록 각별한 주의가 필요하다.

