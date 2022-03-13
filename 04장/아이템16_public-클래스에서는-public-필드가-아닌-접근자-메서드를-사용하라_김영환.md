# Item16. public 클래스에서는 public 필드가 아닌 접근자 메서드를 사용하라!

## 문제점

~~~java
class Point {
    public double x;
    public double y;
}
~~~

- 위와 같은 클래스는 데이터 필드에 직접 접근이 가능하다. 
  - 캡슐화의 이점을 제공하지 못한다.
  - API 수정 없이는 내부 표현을 바꾸지 못하고, 불변식 보장도 안되고, 외부에서 접근 시 부수 작업 수행도 할 수 없다.
- 문제 해결을 위해 필드를 모두 private으로 바꾸고, public 접근자를 추가한다.

~~~java
class Point {
    private double x;
    private double y;
    
    public Point(double x, double y) {
        this.x = x;
        this.y = y;
    }
    
    public double getX() {return x;}
    public double getY() {return y;}
}
~~~

- 패키지 바깥에서 접근 가능한 클래스라면 접근자를 제공해 클래스 내부 표현 방식의 유연성을 얻을 수 있다.



## 예외

- package-private 클래스 또는 private 중첩 클래스의 경우, 데이터 필드를 노출한다고 해도 문제 없다.
  - 각각 패키지/클래스 내부 구현이기 때문에 그렇다.



## 문제 사례

- java.awt.package 패키지의 Point, Dimension 클래스
  - 이 클래스들은 따라 하지 말자.
  - 이 클래스들은 필드를 직접 노출하고 있다.



## 불변 필드를 노출한 public 클래스

~~~java
public final class Time {
    private static final int HOURS_PER_DAY = 24;
    private static final int MINUTES_PER_HOURS = 60;

    public final int hour;
    public final int minute;

    public Time(int hour, int minute) {
        if (hour < 0 || hour >= HOURS_PER_DAY)
            throw new IllegalArgumentException("시간: " + hour);
        if (minute < 0 || minute >= MINUTES_PER_HOURS)
            throw new IllegalArgumentException(": " + minute);
        this.hour = hour;
        this.minute = minute;
    }
    //..생략
}
~~~

- 위 클래스는 각 인스턴스가 유효한 시간을 표현함을 보장한다.
- API를 변경하지 않고는 내부 표현을 변경하는 것이 불가능하다.
- 외부에서 필드 접근 시 부수적인 작업이 필요하다.



## 핵심 정리

- public 클래스는 절대 가변 필드를 직접 노출해서는 안 된다.
- 불변 필드라면 노출해도 덜 위험하나 완전하다 볼 수 없다.
- package-private 클래스나 private 중첩 클래스는 종종 필드를 노출하는 편이 좋을 때가 존재한다.(깔끔한 코드 작성)
