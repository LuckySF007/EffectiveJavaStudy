# 아이템37.ordinal 인덱싱 대신 EnumMap을 사용하라

ordinal 메서드로 인덱스를 얻는 코드가 있다.

```java
Set<Plant>[] plantsByLifeCycle = new Set[Plant.LifeCycle.values().length];

for (int i = 0; i < plantsByLifeCycle.length; i++) {
    plantsByLifeCycle[i] = new HashSet<>();
}

for (Plant p : garden) {
    plantsByLifeCycle[p.lifeCycle.ordinal()].add(p);
}

// 인덱스의 의미를 알 수 없어 직접 레이블을 달아 데이터 확인 작업 필요
for (int i = 0; i < plantsByLifeCycle.length; i++) {
    System.out.printf("%s: %s%n", 
					Plant.LifeCycle.values()[i], plantsByLifeCycle[i]);
}
```

- 위 코드는 문제가 많음
- 배열은 제네릭과 호환되지 않아 비검사 형변환을 수행해야 함 (깔끔하지 않은 컴파일)
- 배열은 각 인덱스의 의미를 몰라 출력 시 직접 레이블을 달아줘야함
- 정확한 정숫값을 사용한다는 것을 개발자가 직접 보증해야한다. (정수는 열거타입과 달리 타입 안전하지 않음)
    - 잘못된 값 → 잘못된 동작 or ArrayIndexOutOfBoundsException 발생

<br>

## **EnumMap 으로 해결**

열거 타입을 키로 사용하도록 설계한 Map의 구현체 EnumMap을 사용하자.

위 코드를 EnumMap으로 바꿔서 아래와 같이 변경할 수 있다.

```java
Map<Plant.LifeCycle, Set<Plant>> plantByLifeCycle = new EnumMap<>(Plant.LifeCycle.class);

for (Plant.LifeCycle lc : Plant.LifeCycle.values()) {
    plantByLifeCycle.put(lc, new HashSet<>());
}

for (Plant p : garden) {
    plantByLifeCycle.get(p.lifeCycle).add(p);
}

System.out.println(plantByLifeCycle);
```

- 더 짧고 가독성이 좋고 안전하며 성능도 비등하다.
- 안전하지 않은 형변환 없음
- 맵의 키인 열거 타입이 자체로 출력용 문자열을 제공하므로 레이블을 달 필요도 없음
- 배열 인덱스 계산 과정에서 오류날 가능성도 없음
- EnumMap 생성자의 키 타입의 Class 객체는 한정적 타입 토큰 (런타임 제네릭 타입 정보 제공)

<br>

위 코드를 스트림을 사용하여 작성할 수도 있다.

```java
//Collectors.groupingBy 메서드는 mapFactory 매개변수에 원하는 맵 구현체를 명시 호출 가능
System.out.println(Arrays.stream(garden)
	.collect(groupingBy(p -> p.lifeCycle,
		() -> new EnumMap<>(LifeCycle.class), toSet())));
```

<br>

## **두 열거 타입 값들을 매핑하기 위해서 ordinal을 두번이나 쓴 배열들의 배열**

두 개의 열거 타입을 억지로 매핑하기 위해 ordinal을 두번이나 쓴 잘못된 방법이 있다.

```java
public enum Phase {
    SOLID,
    LIQUID,
    GAS;

    public enum Transition {
        MELT,
        FREEZE,
        BOIL,
        CONDENSE,
        SUBLIME,
        DEPOSIT;

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
```

- 컴파일러가 ordinal과 배열 인덱스의 관계를 알 도리가 없음
- 표를 수정하지 못하거나 표를 수정하다가 잘못 수정하면 런타임 오류 발생
    - ArrayIndexOutOfBoundsException 이나 NullPointerException 발생
    - 운 안좋으면 예외도 발생하지 않고 이상하게 동작
- 표의 크기는 상태 가짓수가 늘어나면 제곱해서 커지고, null로 채워지는 칸도 많아짐

<br>

EnumMap을 사용해서 바꿔라. 맵 2개를 중첩하면 된다.

```java
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

        private static final Map<Phase, Map<Phase, Transition>> m =
                Stream.of(values()) // enum 타입 두 개를 매핑한 필드 리스트
                        .collect(
                                groupingBy(
                                        t -> t.from,
                                        () -> new EnumMap<>(Phase.class),
                                        toMap(
                                                t -> t.to,
                                                t -> t,
                                                (x, y) -> y,
                                                () -> new EnumMap<>(Phase.class)
                                        ))
                        );

        public static Transition from(Phase from, Phase to) {
            return m.get(from).get(to);
        }
    }
}
```

- Map의 Map을 초기화하기 위해 수집기(java.util.stream.Collector) 2개를 차례로 사용함
- 첫 번째 수집기인 groupingBy에서는 전이를 이전 상태를 기준으로 묶음
- 두 번째 수집기인 toMap 에서는 이후 상태를 전이에 대응시키는 EnumMap을 생성. 두 번째 수집기의 병합 함수인 (x, y) -> y는 선언만 하고 실제로는 쓰이지 않음. 이는 단지 EnumMap을 얻으려면 MapFactory가 필요하고 수집기들은 점층적 팩터리(telescoping factory)를 제공하기 때문이다.
    
    
     
    

<br>

## **새로운 상태를 추가할 때**

1) 배열로 만든 코드에서 새로운 상태를 추가할 때

- 새로운 상수를 Phase에 1개, Phase.Transition에 2개를 추가해야함
- 표 배열 길이를 제곱만큼 커지고 표 배열 원소도 수정해줘야함
- 잘못된 순서로 원소를 나열하거나 원소 수를 맞추지 못할 경우 런타임 오류 발생

2) EnumMap로 만든 코드에서 새로운 상태를 추가할 때

- 새로운 상수를 Phase에 1개, Phase.Transition에 추가할 전이1(상태1, 상태2), 전이2(상태2, 상태1) 만 추가해주면 된다.
    - ex) Phase에 PLASMA 추가 Phase.Transition에 IONIZE(GAS, PLASMA), DEIONIZE(PLASMA, GAS) 추가
- 더 이상 수정할 것 없음 → 잘못 수정할 가능성이 극히 작아 안전하고 명확하고 유지보수하기에 좋음

<br>

## **결론**

배열의 인덱스를 얻기 위해 ordinal을 쓰는 것은 좋지 않으니 EnumMap을 사용하라.

다차원 관계는 EnumMap을 중첩해서 사용하라.