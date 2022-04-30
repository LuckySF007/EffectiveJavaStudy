# Item89. 인스턴스 수를 통제해야 한다면 readResolve보다는 열거 타입을 사용하라!

## 싱글턴 직렬화의 문제점

- 싱글턴 패턴을 사용해 바깥에서 생성자를 호출하지 못하도록 하여 인스턴스가 오직 하나만 만들어짐을 보장할 수 있다.
- 하지만 아이템 3에서 이야기했든 선언에 implements Serializable을 추가하는 순간 싱글턴이 아니게 된다.
  - 기본 직렬화를 사용하지 않았고, 명시적인 readObject 메서드를 제공했더라도, 초기화 시 별개의 인스턴스를 반환하기 때문이다.





## readResolve 메서드

- readResolve 기능을 이용하면 readObject가 만들어낸 인스턴스를 다른 것으로 대체할 수 있다.
- 역직렬화한 객체의 클래스가 readResolve 메서드를 적절히 정의해뒀다면, 역직렬화 후  새 객체를 인수로 이 메서드가 호출된다. 이 메서드가 반환한 객체 참조가 새로 생성된 객체를 대신해 반환된다. (새로 생성된 객체 참조는 가비지 컬렉션 대상이 됨)

~~~java
public class Elvis implements Serializable {
	  // transient가 아닌 참조 필드를 가지고 있기에 공격당할 여지가 있다. 
    public static final Elvis INSTANCE = new Elvis();
    private Elvis() { }
  
    private String[] favoriteSongs = {"Hound Dog", "Heartbreak Hotel"};

    public void leaveTheBuilding() {
        // ...
    }

    private Object readResolve() { // 인스턴스 통제를 위한 readResolve 메서드 - 개선의 여지가 있다!!!
        return INSTANCE; // 진짜 Elvis를 반환(가짜는 가비지 컬렉터로)
    }
}
~~~

- 위의 예시에서 readResolve 메서드는 역직렬화한 객체는 무시하고 초기화 때 만들어진 Elvis 인스턴스를 반환하고 있다.
  - 따라서 Elvis 인스턴스의 직렬화 형태는 아무런 실 데이터를 가질 이유가 없고, 모든 인스턴스 필드를 transient로 선언해야 한다.
- **readResolve를 인스턴스 통제 목적으로 사용한다면 객체 참조 타입 인스턴스 필드는 모두 transient로 선언해야 한다.**
  - 그렇지 않으면 아이템 88에서 살펴본 MutablePeriod 공격처럼 readResolve 메서드 수행 전, 역직렬화된 객체 참조를 공격할 여지가 남는다.



## 역직렬화 공격 방법

- 싱글턴이 transient가 아닌 참조 필드를 가지고 있다면, 그 필드의 내용은 readResolve 메서드가 실행되기 전에 역직렬화된다.

- 잘 조작된 스트림을 사용해 해당 참조 필드 내용이 역직렬화되는 시점에 그 인스턴스 참조를 훔쳐올 수 있을 것이다.

- non-transient 참조 필드를 훔쳐오는 Stealer(도둑) 클래스 예시

  ~~~java
  public class ElvisStealer implements Serializable {
      static Elvis impersonator;
      private Elvis payload;
  
      private Object readResolve() {
  				// resolve되기 전의 Elvis 인스턴스의 참조를 저장한다. 
          impersonator = payload;
        	
  				// favoriteSongs 필드에 맞는 타입의 객체를 반환한다. 
          return new String[]{"A Fool Such as I"};
      }
  
      private static final long serialVersionUID = 0;
  }
  ~~~

  - 직렬화된 스트림에서 싱글턴의 비휘발성 필드를 도둑의 인스턴스로 교체하고, 인스턴스 필드는 도둑이 숨길 직렬화된 싱글턴을 참조한다.
    - 즉, 싱글턴은 도둑을 참조하고, 도둑은 싱글턴을 참조하게 된다.

- 직렬화의 허점을 이용해 싱글턴 객체를 2개 생성하는 예시

  ~~~java
  public class ElvisImpersonator {
      // 진짜 Elvis 인스턴스로는 만들어질 수 없는 바이트 스트림
  	private static final byte[] serializedForm = {
          // ... 괴이한 byte 배열 내용은 생략
      };
      public static void main(String[] args) {
          // ElvisStealer.impersonator를 초기화한 다음,
          // 진짜 Elvis(즉, Elvis.INSTANCE)를 반환한다.
          Elvis elvis = (Elvis) deserialize(serializedForm);
          Elvis impersonator = ElvisStealer.impersonator;
  
          elvis.printFavorites();
          impersonator.printFavorites();
      }
  }
  ~~~

  - favoriteSongs 필드를 transient로 선언해 문제를 고칠 수 있다. 하지만 Elvis를 원소 하나짜리 열거 타입으로 바꾸는 편이 더 나을 것이다.
  - ElvisStealer 예제에서 살펴본 것처럼 readResolve 메서드를 사용해 '순간적으로' 만들어진 역직렬화된 인스턴스에 접근 못하게 하는 방법은 깨지기도 쉽고, 신경도 많이 써야 하는 방법이다.





## 싱글턴을 보장하는 열거 타입

- 직렬화 가능한 인스턴스 통제 클래스를 열거 타입을 이용해 구현하면 선언한 상수 외의 다른 객체는 존재하지 않음을 자바가 보장해준다.

- 다만 공격자가 AccessibleObject.setAccessible 같은 특권 메서드를 가로채 공격한다면, 그런 공격자에게는 모든 방어가 무력화된다.

- Elvis를 열거 타입으로 구현한 예시

  ~~~java
  public enum Elvis {
      INSTANCE;
      private String[] favoriteSongs = {"Hound Dog", "Heartbreak Hotel"};
  
      public void printFavorites() {
          System.out.printtn(Arrays.toString(favoriteSongs));
      }
  }
  ~~~

  - 열거 타입을 이용하면 된다고 해서, 인스턴스 통제를 위해 readResolve를 사용하는 방식이 완전히 쓸모없는 것은 아니다.
    - 직렬화 가능 인스턴스 통제 클래스 작성 시, 컴파일 타임에는 어떤 인스턴스들이 있는지 알 수 없는 상황에서는 열거 타입 사용이 불가하다.





## readResolve 메서드의 접근성

- final 클래스에서 readResolve 메서드는 private 이어야 한다.
- final이 아닌 클래스
  - private : 하위 클래스에서 사용 불가
  - package-private : 같은 패키지에 속한 하위 클래스에서만 사용 가능
  - protected, public : 이를 재정의하지 않은 모든 하위 클래서에서 사용 가능 (하위 클래스에서 재정의하지 않았다면, 하위 클래스의 인스턴스 역직렬화 시, 상위 클래스의 인스턴스를 생성한다. 이 과정에서 ClassCastException이 발생할 수 있다.)





## 핵심 정리

- 불변식을 지키기 위해 인스턴스를 통제해야 한다면 가능한 한 열거 타입을 사용하자.
- 여의치 않은 상황에서 직렬화와 인스턴스 통제가 모두 필요한 경우, readResolve 메서드를 작성해 넣자.
  - 그 클래스에서 모든 참조 타입 인스턴스 필드는 transient로 선연해야 한다.

