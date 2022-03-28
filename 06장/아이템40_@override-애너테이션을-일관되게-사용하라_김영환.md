# Item40. @Override 애너테이션을 일관되게 사용하라!

## @Override

- 자바가 기본 제공하는 애너테이션 중 가장 중요도 높음.
- 메서드 선언에만 달 수 있으며, @Override가 달렸다는 것은 상위 타입 메서드를 재정의했음을 뜻한다.
- @Override를 일관되게 사용하면 여러 가지 악명 높은 버그들을 예방해준다.



### Non @Override 애너테이션

~~~java
public class Bigram {
    private final char first;
    private final char second;

    public Bigram(char first, char second) {
        this.first = first;
        this.second = second;
    }

    public boolean equals(Bigram b) {
        return b.first == first && b.second == second;
    }

    public int hashCode() {
        return 31 * first + second;
    }

    public static void main(String[] args) {
        Set<Bigram> s = new HashSet<>();
        for (int i = 0; i < 10; i++) {
            for (char ch = 'a'; ch <= 'z'; ch++) 
              	s.add(new Bigram(ch, ch));
        }
        System.out.println(s.size()); //260
    }
}
~~~

- 26 출력을 기대했으나 실제로는 260이 출력된다.
  - equals 재정의 시 hashCode 재정의를 해야 하는데 이를 잊은 경우다.
  - 또 해당 경우는 재정의가 아닌 오버로딩을 한 예제다.
    - 즉, 상속한 equals 재정의가 아닌 새로운 equals 메서드를 정의했다는 얘기다.
- 문제의 해결을 위해 @Override 애너테이션을 사용해 재정의한다는 의도를 명시할 수 있다.



### @Override 애너테이션

~~~java
@Override public boolean equals(Bigram b) {
    return b.first == first && b.second == second;
}
~~~

- 이 예제는 에러가 난다.
  - 이유는 재정의하려는 메서드의 매개변수 타입이 다르기 때문이다.
  - Object 타입을 사용해야 한다.

~~~java
@Override public boolean equals(Object bigram) {
    if(!(o instanceof Bigram)) {
        return false;
    }
    Bigram b = (Bigram) bigram;
    return b.first == first && b.second == second;
}
~~~

- 위와 같이 변경해 Object.equals 메서드를 재정의할 수 있다.





## 핵심 정리

- 재정의한 모든 메서드에 @Override 애너테이션을 의식적으로 달면 실수했을 때 컴파일러가 바로 알려줄 것이다.
- 예외는 단 한가지로, 구체 클래스에서 상위 클래스의 추상 메서드를 재정의한 경우이다. 
  - 이러한 경우 이 애너테이션을 달지 않아도 된다.
  - 단다고 해서 문제되는 것은 없다.
