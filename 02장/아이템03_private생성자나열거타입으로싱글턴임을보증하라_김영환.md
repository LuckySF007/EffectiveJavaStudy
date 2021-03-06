#  Item03. private 생성자나 열거 타입으로 싱글턴임을 보증하라!



## 싱글턴

인스턴스를 오직 하나만 생성할 수 있는 클래스

예 : 무상태 객체, 설계상 유일해야 하는 시스템 컴포넌트 등



### 구성 방법 1

생성자는 private으로 두고, 유일한 인스턴스 접근 수단으로 public static 멤버를 두는 구성

#### 권한이 있는 클라이언트

클라이언트가 private에 접근할 권한을 가진 경우, 싱글턴이 깨질 수 있는 위험이 존재한다. 이 경우 두번째 객체가 생성되려 할 때 예외를 던지도록 구성해 해결할 수 있다.



### 구성 방법 2

정적 팩터리 메서드를 public static 멤버로 제공하는 구성

API의 구성을 바꾸지 않고도 싱글턴 유무의 변경이 가능하다는 장점 존재.

정적 팩토리를 제네릭 싱글턴 팩터리로 만들 수 있다는 장점 존재.

정잭 팩터리의 메서드 참조를 공급자로 사용할 수 있다는 장점 존재.



### 구성 방법 3

원소가 하나인 열거 타입을 선언하는 구성

public 방식과 비슷하지만 더 간결, 추가 노력 없이 직렬화 가능함.

대부분의 상황에서 원소가 하나뿐인 열거 타입이 싱글턴을 만드는 가장 좋은 방법임. (Enum외의 클래서 사용 시 해당 구성 사용 불가)



### 직렬화

단순히 Serializable 구현한다고 되는 것이 아님.

모든 인스턴스 필드를 transient라고 선언, readResolve 메서드 제공 필요 -> 역직렬화 시 새로운 인스턴스가 생성되는 현상 방지를 위해





