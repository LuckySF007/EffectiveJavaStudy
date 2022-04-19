# 아이템81.wait와 notify보다는 동시성 유틸리티를 사용하라

## **동시성 유틸리티**

- 자바5에 도입된 고수준의 동시성 유틸리티 덕분에 자바의 wait와 notify [아이템 50] 를 사용해야 할 이유가 줄었다.
- wait와 notify는 올바르게 사용하기 까다로우니 고수준 동시성 유틸리티를 사용하자.
- java.util.concurrent의 고수준 유틸리티의 범주 : 실행자 프레임워크 [아이템 80], 동시성 컬렉션(concurrent collection), 동기화 장치(sychronized)

<br>

## **동시성 컬렉션**

- 표준 컬렉션 인터페이스에 동시성을 가미해 구현한 고성능 컬렉션
- 높은 동시성을 위해 동기화를 각자 내부에서 수행 [아이템 79]
- 동시성 컬렉션에서 동시성을 무력화하는 것은 불가능, 외부에서 락을 추가로 사용하면 오히려 속도가 느려짐
- 동시성 무력화가 불가능하므로 여러 메서드를 원자적으로 묶어 호출하는 것도 불가능 → 여러 동작을 하나의 원자적 동작으로 묶는 상태 의존적 수정 메서드 추가
    - 유용하여 자바8에서 일반 컬렉션 인터페이스들에도 디폴트 메서드로 추가됨. ex) Map의 putIfAbsent(key, value)
- 동기화한 컬렉션이 아닌 동시성이 뛰어나며 속도도 빠른 동시성 컬렉션을 사용하자. 즉, Collections.synchronizedMap 보다는 ConcurrentHashMap을 사용하는게 훨씬 좋다.
- 컬렉션 인터페이스 중 작업이 성공적으로 완료될 때까지 기다리도록 확장한 것들도 있다. ex) Queue를 확장한 BlockingQueue → 대부분의 실행자 서비스 구현자에서 작업큐로 사용됨

<br>

## **동기화 장치**

다른 스레드를 기다릴 수 있게하여 서로 작업을 조율할 수 있게함. 

CountDownLatch와 Semaphore 이 자주 쓰임.

1) CountDownLatch

- 일회성 장벽으로 하나 이상의 스레드가 다른 하나 이상의 스레드 작업이 끝날때까지 기다리게함
- 생성자에서 받는 int값으로 래치의 countDown 메서드를 몇번 호출해야 대기 중인 스레드들을 깨우는지 결정

```java
//동시 실행 시간을 재는 간단한 프레임워크
public static long time(Executor executor, int concurrency, Runnable action) throws InterruptedException {
    CountDownLatch ready = new CountDownLatch(concurrency);
    CountDownLatch start = new CountDownLatch(1);
    CountDownLatch done = new CountDownLatch(concurrency);

    for (int i = 0; i < concurrency; i++) {
      executor.execute(() -> {
        ready.countDown(); // 타이머에게 준비가 됐음을 알린다.
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
```

executor는 concurrency 매개변수로 지정한 동시성 수준만큼의 스레들르 생성할 수 있어야 한다.

그렇지 않으면 메서드가 결코 끝나지 않는 스레드 기아 교착 상태가 될 것이다.

<br>

## wait와 notify

어쩔 수 없이 레거시 코드를 다룰 때는 wait와 notify를 사용해야하는 상황도 있다.

1) wait

```java
synchronized (obj) {
    while (조건이 충족되지 않았다) {
        obj.wait(); // 락을 놓고, 깨어나면 다시 잡는다.
    }

    ... // 조건이 충족됐을 때의 동작을 수행한다.
}
```

- wait 메서드를 사용할때는 반드시 대기 반복문(wait loop) 관용구를 사용하고 반복문 밖에서는 절대로 호출하지 말자.
- 대기 전에 조건을 검사하여 wait를 건너뛰게 한 것은 응답 불가 상태를 예방하기 위함이다.
- 대기 후에 조건을 검사하여 조건이 충족되지 않았다면 다시 대기하게 하는 것은 안전 실패를 막기 위함이다.
- 조건이 만족되지 않아도 스레드가 깨어날 수 있는 상황이 몇가지 있다.
    - notify를 호출하여 대기 중인 스레드가 깨어나는 사이에 다른 스레드가 락을 거는 경우
    - 조건이 만족되지 않았지만 실수 혹은 악의적으로 notify를 호출하는 경우
    - 대기 중인 스레드 중 일부만 조건을 충족해도 notifyAll로 모든 스레드를 깨우는 경우
    - 대기 중인 스레드가 드물게 notify 없이 깨어나는 경우. 허위 각성(spurious wakeup)이라고 한다.
    

<br>

2) notify

일반적으로 notify() 보다는 notifyAll() 을 사용하라.

실수나 악의적인 wait을 호출하는 공격으로부터 보호할 수 있다.

<br>

## **결론**

- 코드를 새로 작성한다면 java.util.concurrent를 사용하라
- 레거시 코드를 유지보수한다면 wait는 항상 while문 안에서 호출하자.
- notify보다는 notifyAll을 사용하라. 혹시 notify를 사용해야한다면 응답 불가 상태에 빠지지 않도록 조심하자.