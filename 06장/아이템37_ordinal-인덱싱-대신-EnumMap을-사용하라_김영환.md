# Item37. ordinal 인덱싱 대신 EnumMap을 사용하라!

## ordinal()의 사용과 EnumMap

- 배열이나 리스트에서 원소를 꺼낼 때 ordinal 메서드로 인덱스을 얻어 사용하는 코드가 있다.

~~~java
public class Plant {
    enum LifeCycle { ANNUAL, PERENNIAL,BIENNIAL }

    final String name;
    final LifeCycle lifeCycle;

    Plant(String name, LifeCycle lifeCycle) {
        this.name = name;
        this.lifeCycle = lifeCycle;
    }

    @Override
    public String toString() {
        return name;
    }
}
~~~

- 정원에 심은 식물들을 배열 하나로 관리한다. 이들을 생애주기(LifeCycle)를 기준으로 묶어보자.
  - 생애주기별로 총 3개의 집합을 만들 것이다. 정원을 돌며 각 식물을 해당 집합에 넣는다.
    - 이 때, 집합들을 배열 하나에 넣고, 생애주기의 ordinal 값을 그 배열의 인덱스로 사용하려 할 수 있다.
      - 하지 말자!!!



### ordinal()을 배열 인덱스로 사용 - 하지 말 것

~~~java
Set<Plant>[] plantsByLifeCycle = new Set[Plant.LifeCycle.values().length];

for (int i = 0; i < plantsByLifeCycle.length; i++) {
  plantsByLifeCycle[i] = new HashSet<>();
}

for (Plant p : garden) {
  plantsByLifeCycle[p.lifeCycle.ordinal()].add(p);
}

// 결과
for (int i = 0; i < plantsByLifeCycle.length; i++) {
  System.out.printf("%s: %s%n", Plant.LifeCycle.values()[i], plantsByLifeCycle[i]);
}
~~~

- 동작은 한다. 하지만 문제가 한가득이다.
- 배열은 제네릭과 호환되지 않는다.
  - 비검사 형변환을 수행해야 하고 깔끔히 컴파일되지 않을 것이다.
- 배열은 각 인덱스의 의미를 모른다. 그러므로 직접 출력 결과에 레이블을 추가해야 한다.
- 정확한 정수값을 사용한다는 것을 직접 보증해야 한다. 정수는 열거 타입과 달리 타입 안전이 보장되지 않는다.
  - 잘못된 동작도 수행하거나, ArrayIndexOutOfBoundsException 던진다.



### 해결책

- 이 예제에서 배열은 실질적으로 열거 타입 매핑을 담당한다.
- 이 매핑 역할을 Map에 넘긴다.
  - 열거 타입을 키로 사용하도록 설계한 아주 빠른 Map 구현제인 EnumMap을 사용하면 될 것이다.

~~~java
Map<Plant.LifeCycle, Set<Plant>> plantByLifeCycle = new EnumMap<>(Plant.LifeCycle.class);

for (Plant.LifeCycle lc : Plant.LifeCycle.values()) {
		plantByLifeCycle.put(lc, new HashSet<>());
}

for (Plant p : garden) {
		plantByLifeCycle.get(p.lifeCycle).add(p);
}

System.out.println(plantByLifeCycle);
~~~

- 더 짧고 명료하고 안전하다. 성능도 기존 예제와 비등하다.
  - EnumMap의 성능이 ordinal을 사용한 배열과 비등한 이유는 그 내부 로직이 배열을 사용하기 때문이다.
    - 내부 구현 방식을 안으로 숨겨 Map의 타입 안전성과 배열의 성능 모두를 얻었다.
- EnumMap의 생성자가 받는 키 타입의 Class 객체는 한정적 타입 토큰으로, 런타임 제네릭 타입 정보를 제공한다.



### 스트림을 사용해 더 간단 명료하게 구성

~~~java
System.out.println(Arrays.stream(garden)
				.collect(groupingBy(p -> p.lifeCycle, 
						() -> new EnumMap<>(LifeCycle.class), toSet())));
~~~

- 이 예제처럼 단순한 경우 최적화가 굳이 필요하진 않지만, 맵을 빈번히 사용한다면 필요하다.



### 스트림과 EnumMap만 사용했을 때의 차이점

#### EnumMap만 사용하는 경우

- 언제나 식물의 생애주기당 하나씩의 중첩 맵을 생성한다.

#### 스트림을 사용하는 경우

- 생애주기에 해당하는 식물이 있을 때만 그 생애주기 맵을 생성해 사용한다.





## 두 열거 타입 값 매핑을 위해 ordinal을 (두 번이나) 사용한 배열

- 두 열거 타입을 강제로 매핑하기 위해 사용한 잘못된 방법이다.

~~~java
public enum Phase {
    SOLID, LIQUID, GAS;

    public enum Transition {
        MELT, FREEZE, BOIL, CONDENSE, SUBLIME, DEPOSIT;

        private static final Transition[][] TRANSITIONS = {
          	{null, MELT, SUBLIME},
          	{FREEZE, null, BOIL},
          	{DEPOSIT, CONDENSE, null}
        };

        public static Transition from(Phase from, Phase to) {
            return TRANSITIONS[from.ordinal()][to.ordinal()];
        }
    }
}
~~~

- 잘못된 방식을 사용해 두 가지 상태(Phase)를 전이(Transition) 및 매핑하도록 구현한 예제다.
- 멋져 보이지만 속지말자. 앞서 설명한 것과 같이 컴파일러는 ordinal과 배열 인덱스 관계를 알 수 없다.
  - 즉, Phase나 Phase.Transition 열거 타입을 수정하고, TRANSITIONS를 수정하지 않거나 실수한다면 런타임 에러가 난다.
  - IndexOutOfBoundsException, NullPointerException을 던질 가능성도 있다.
  - 상태의 가짓수가 늘어갈수록, TRANSITIONS 표의 null 값도 증가해 비효율적이다.
- 이 경우도 마찬가지로 그냥 EnumMap을 사용해 해결하면 된다.

~~~java
public enum Phase {
    SOLID, LIQUID, GAS;

    public enum Transition {
        MELT(SOLID, LIQUID), FREEZE(LIQUID, SOLID),
        BOIL(LIQUID, GAS), CONDENSE(GAS, LIQUID),
        SUBLIME(SOLID, GAS), DEPOSIT(GAS, SOLID);

        private final Phase from;
        private final Phase to;

        Transition(Phase from, Phase to) {
            this.from = from;
            this.to = to;
        }

      	// 상전이 맵을 초기화한다.
        private static final Map<Phase, Map<Phase, Transition>> 
          m = Stream.of(values()).collect(groupingBy(t -> t.from,
              () -> new EnumMap<>(Phase.class),
              toMap(t -> t.to, t -> t, (x, y) -> y,
                	() -> new EnumMap<>(Phase.class))));

        public static Transition from(Phase from, Phase to) {
            return m.get(from).get(to);
        }
    }
}
~~~

- 상전이 맵 초기화 코드는 복잡하다.

- Map<Phase, Map<Phase, Transition>> : 이전 상태에서 '이후 상태에서 전이로의 맵'에 대응시키는 맵
- 첫 번째 수집기인 groupingBy에서는 전이를 이전 상태 기준으로 묶는다.
- 두 번째 수집기인 toMap에서는 이후 상태 전이에 대응시키는 EnumMap 생성을 한다.
  - 병합 함수인 (x, y) -> y는 선언만 하고 실제로 쓰이진 않는다.
    - 이는 단지 EnumMap을 얻으려면 맵 팩터리가 필요하고, 수집기들은 점층적 팩터리를 제공하기 때문이다.



### 새로운 상태 추가하기

- EnumMap 사용 시 새로운 상태를 추가하는 것은 쉬운 일이다.

~~~java
public enum Phase {
    SOLID, LIQUID, GAS,
    // PLASMA 추가
    PLASMA;

    public enum Transition {
        MELT(SOLID, LIQUID), FREEZE(LIQUID, SOLID),
        BOIL(LIQUID, GAS), CONDENSE(GAS, LIQUID),
        SUBLIME(SOLID, GAS), DEPOSIT(GAS, SOLID),
        IONIZE(GAS, PLASMA), DEIONIZE(PLASMA, GAS); // 전이 상태 추가

        // ... 나머지는 동일하니 생략
    }
}
~~~

- 기체에서 플라스마로 변하는 이온화와 반대인 탈이온화를 추가했다.
  - 배열로 만든 코드였다면 Phase, Phase.Transition, 상전이표 변경 등의 복잡한 작업이 필요하고, 잘못 나열하면 문제를 일으킨다.
  - 하지만 EnumMap의 사용을 통해 손쉬운 상태 추가를 할 수 있다.





## 핵심 정리

- 배열의 인덱스를 얻기 위해 ordinal을 쓰는 것은 일반적으로 좋지 않으니, 대신 EnumMap을 사용하라.
- 다차원 관계는 EnumMap<..., EnumMap<...>>으로 표현하라.
- "애플리케이션 프로그래머는 Enum.ordinal을 웬만해서는 사용하지 말아야 한다."는 일반 원칙의 특수 사례다.

