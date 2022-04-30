# Item88. readObject 메서드는 방어적으로 작성하라!

## 불변클래스의 직렬화 문제

- 아이템 50에서 불변인 날짜 범위 클래스를 만드는 데 가변인 Date 필드를 이용해서, 불변식을 지키고 유지하기 위해 생성자와 접근자에서 Date 객체를 방어적으로 복사하느라 코드가 상당히 길어졌다.

  ~~~java
  public final class Period {
      private final Date start;
      private final Date end;
  
      /**
       * @param  start 시작 시각
       * @param  end 종료 시각; 시작 시각보다 뒤여야 한다.
       * @throws IllegalArgumentException 시작 시각이 종료 시각보다 늦을 때 발생한다.
       * @throws NullPointerException start나 end가 null이면 발생한다.
       */
      public Period(Date start, Date end) {
          this.start = new Date(start.getTime()); // 새로운 객체로 방어적 복사
          this.end = new Date(end.getTime());
          if (this.start.compareTo(this.end) > 0) {
              throw new IllegalArgumentException(start + " after " + end);
          }
      }
  
      public Date start() { return new Date(start.getTime()); }
      public Date end() { return new Date(end.getTime()); }
      public String toString() { return start + " - " + end; }
    
      // ... 나머지 코드는 생략
  }
  ~~~

- 이러한 클래스를 직렬화하기로 결정했다고 해보자.

  - Period 객체의 물리적 표현이 논리적 표현과 부합한다. 고로 기본 직렬화 형태를 사용해도 나쁘지 않다.
    - 그럼, implements Serializable만 추가하면 될 것인데, 과연 그럴까?

  - 이 클래스는  implements Serializable만 추가한다면, 주요한 불변식을 더는 보장하지 못하게 된다.
    - 문제는 readObject 메서드가 실질적으로 또 다른 public 생성자이기 때문이다. 따라서 다른 생성자와 동일한 수준의 주의를 기울여야 한다. readObject 메서드에서도 인수가 유효한지 검사해야 하고, 필요하다면 매개변수를 방어적으로 복사해야 한다.






## readObject

- 매개변수로 바이트 스트림을 받는 생성자다.

- 보통의 경우 바이트 스트림은 정상적으로 생성된 인스턴스를 직렬화해 만들어진다.

- 불변식을 깨뜨릴 의도로 임의 생성한 바이트 스트림을 건네면, 정상적인 생성자로는 만들어낼 수 없는 객체를 생성해내는  문제가 생긴다.

  ~~~java
  public class BogusPeriod {
      // 진짜 Period 인스턴스에서는 만들어질 수 없는 비정상적 바이트 스트림
      private static final byte[] serializedForm = {
          (byte)0xac, (byte)0xed, 0x00, 0x05, 0x73, 0x72, 0x00, 0x06,
          0x50, 0x65, 0x72, 0x69, 0x6f, 0x64, 0x40, 0x7e, (byte)0xf8,
          0x2b, 0x4f, 0x46, (byte)0xc0, (byte)0xf4, 0x02, 0x00, 0x02,
          0x4c, 0x00, 0x03, 0x65, 0x6e, 0x64, 0x74, 0x00, 0x10, 0x4c,
          0x6a, 0x61, 0x76, 0x61, 0x2f, 0x75, 0x74, 0x69, 0x6c, 0x2f,
          0x44, 0x61, 0x74, 0x65, 0x3b, 0x4c, 0x00, 0x05, 0x73, 0x74,
          0x61, 0x72, 0x74, 0x71, 0x00, 0x7e, 0x00, 0x01, 0x78, 0x70,
          0x73, 0x72, 0x00, 0x0e, 0x6a, 0x61, 0x76, 0x61, 0x2e, 0x75,
          0x74, 0x69, 0x6c, 0x2e, 0x44, 0x61, 0x74, 0x65, 0x68, 0x6a,
          (byte)0x81, 0x01, 0x4b, 0x59, 0x74, 0x19, 0x03, 0x00, 0x00,
          0x78, 0x70, 0x77, 0x08, 0x00, 0x00, 0x00, 0x66, (byte)0xdf,
          0x6e, 0x1e, 0x00, 0x78, 0x73, 0x71, 0x00, 0x7e, 0x00, 0x03,
          0x77, 0x08, 0x00, 0x00, 0x00, (byte)0xd5, 0x17, 0x69, 0x22,
          0x00, 0x78
      };
  
      public static void main(String[] args) {
          Period p = (Period) deserialize(serializedForm);
          System.out.println(p);
      }
      
      static Object deserialize(byte[] sf) {
          try {
              return new ObjectInputStream(new ByteArrayInputStream(sf)).readObject();
          } catch (IOException | ClassNotFoundException e) {
              throw new IllegalArgumentException(e);
          }
      }
  }
  ~~~

  - 이 괴기한 프로그램을 수행하면 종료 시각이 시작 시각보다 앞서는 Period 생성이 가능하다. 
    - 어떻게 종료 시각이 시작한 시각보다 앞서겠는가? 이건 잘못된 것이다.
  - Period를 직렬화할 수 있도록 선언한 것만으로 클래스의 불변식을 깨뜨리는 객체를 만들 수 있다는 것을 확인했다.





## 불변식 보장을 위한 방법

- **유효성 검사**

  ~~~java
  private void readObject(ObjectInputStream s) throws IOException, ClassNotFoundException {
      s.defaultReadObject(); // 기본 직렬화를 수행한다.
      if (start.compareTo(end) > 0) { // 유효성 검사를 수행한다.
          throw new InvalidObjectException(start + " 가 " + end + " 보다 늦다.");
      }
  }
  ~~~

  - 이러한 문제 해결을 위해 Period의 readObject 메서드가 defaultReadObject를 호출한 후 역직렬화된 객체의 유효성을 검사하면 된다. 유효성 검사 실패 시 InvalidObjectException을 던져 잘못 역직렬화가 일어나는 것을 막자.



- **방어적 복사**

  ~~~java
  public class MutablePeriod {
     public final Period period; // Period 인스턴스
     public final Date start; // 시작 시각 필드 - 외부 접근 불가
     public final Date end; // 종료 시각 필드 - 외부 접근 불가
  
     public MutablePeriod() {
         try {
             ByteArrayOutputStream bos = new ByteArrayOutputStream();
             ObjectOutputStream out = new ObjectOutputStream(bos);
  
             // 유효한 Period 인스턴스를 직렬화한다.
             out.writeObject(new Period(new Date(), new Date()));
  
             /*
              * bos 값에 악의 적인 이전 객체 참조(내부 Date 필드로의 참조)를 추가한다.
              */
             byte[] ref = { 0x71, 0, 0x7e, 0, 5 }; // 악의적인 바이트스트림
             bos.write(ref);                       // start 필드
             ref[4] = 4;                           // 악의적인 바이트스트림
             bos.write(ref);                       // end 필드
  
             // Period 역직렬화 후 Period Date 참조를 훔친다.
             ObjectInputStream in = new ObjectInputStream(
               new ByteArrayInputStream(bos.toByteArray()));
             period = (Period) in.readObject();
             start  = (Date) in.readObject();
             end    = (Date) in.readObject();
         } catch (IOException | ClassNotFoundException e) {
             throw new AssertionError(e);
         }
     }
  }
  ~~~

  ~~~java
  public static void main(String[] args) {
      MutablePeriod mp = new MutablePeriod();
      Period p = mp.period;
      Date pEnd = mp.end;
  
    	// 시간을 되돌린다.
      pEnd.setYear(78);
      System.out.println(p);
          
      pEnd.setYear(69);      // 60년대로 회귀
      System.out.println(p);
  }
  ~~~

  - Period 인스턴스는 불변식을 유지한 채 생성되었지만, 의도적으로 내부 값을 수정할 수 있다. 이처럼 변경할 수 있는 Period 인스턴스를 획득한 공격자는 이 인스턴스가 불변이라 가정하는 클래스에 넘겨 엄청난 문제를 일으킬 수 있다.

  - 이 문제의 근원은 Period의 readObject 메서드가 방어적 복사를 충분히 하지 않은 데 있다. 

    - **객체를 역직렬화할 때는 클라이언트가 소유해서는 안 되는 객체 참조를 갖는 필드를 모두 반드시 방어적으로 복사해야 한다.**
    - 따라서 readObject에서는 불변 클래스 안의 모든 private 가변 요소를 방어적으로 복사해야 한다.

    ~~~java
    private void readObject(ObjectInputStream s) throws IOException, ClassNotFoundException {
        s.defaultReadObject();
    
        // 가변 요소들을 방어적으로 복사한다.
        start = new Date(start.getTime());
        end   = new Date(end.getTime());
    
        // 불변식을 만족하는지 검사한다.
        if (start.compareTo(end) > 0) {
            throw new InvalidObjectException(start + "가 " + end + "보다 늦다.");
        }
    }
    ~~~

    - **final 필드는 방어적 복사가 불가능**하니, 이 readObject 메서드를 사용하려면 start와 end 필드에서 final 한정자를 제거해야 한다. 아쉬운 일이나 앞서 살펴본 공격 위험에 노출되는 것보다야 낫다.





## 기본 readObject 사용 여부 판단

- **transient 필드를 제외한 모든 필드의 값을 매개변수로 받아 유효성 검사 없이 필드에 대입하는 public 생성자를 추가해도 괜찮다**면, 사용 가능!
  - 괜찮지 않다면, 커스텀 readObject 메서드를 만들어 (생성자에서 수행했어야 할) 모든 유효성 검사와 방어적 복사를 수행해야 한다. 혹은 직렬화 프록시 패턴을 사용하는 방법도 있는데, 이는 역직렬화를 안전하게 만드는 데 필요한 노력을 상당히 경감해주므로 적극 권장한다.

- final이 아닌 직렬화 가능 클래스라면, 마치 생성자처럼 readObject 메서드도 재정의 가능 메서드를 (직접적으로든 간접적으로든) 호출해서는 안 된다. 이 규칙을 어겼는데 해당 메서드가 재정의된다면, 하위 클래스의 상태가 완전히 역직렬화되기 전 하위 클래스에서 재정의된 메서드가 실행되어 프로그램 오작동을 유발한다.





## 핵심 정리

- readObject 메서드 작성 시 언제나 public 생성자를 작성하는 자세로 임하자.
- readObject는 어떤 바이트 스트림이 넘어오더라도 유효한 인스턴스를 만들어내야 한다.
- 바이트 스트림이 진짜 직렬화된 인스턴스라 가정하지 말자.

### 안전한 readObject 메서드 작성 지침 요약

~~~
(1) private이어야 하는 객체 참조 필드는 각 필드가 가리키는 객체를 방어적으로 복사하라. (불변 클래스 내의 가변 요소)
(2) 모든 불변식을 검사하여 어긋나는 게 발견되면 InvalidObjectException을 던진다. (방어적 복사 이후 불변식 검사)
(3) 역직렬화 후 객체 그래프 전체의 유효성을 검사해야 한다면 ObjectInputValidation 인터페이스를 사용하라.
(4) 직접적이든 간접적이든, 재정의할 수 있는 메서드는 호출하지 말자.
~~~

