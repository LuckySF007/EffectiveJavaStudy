# 아이템16.public 클래스에서는 public 필드가 아닌 접근자 메서드를 사용하라

인스턴스 필드들을 모아놓는 일 외에는 목적없는 클래스를 작성할 때가 있다.

```java
class Point {
    public double x;
    public double y;
}
```

이런 클래스는 데이터 필드에 직접 접근이 가능하기 때문에 캡슐화의 이점을 제공하지 못한다.

API를 수정하지 않고는 내부 표현 수정 불가, 불변식 보장 x, 스레드 안전하지 않음

→ 필드를 private으로 바꾸고 getter 메서드를 제공한다.

하지만 package-private 클래스나 private 중첩 클래스라면 데이터 필드를 노출해도 하등의 문제가 없다. (getter 방식보다 깔끔)

```java
class Point {
    private double x;
    private double y;

    public Point(double x, double y) {
        this.x = x;
        this.y = y;
    }

    public double getX() {
        return x;
    }

    public double getY() {
        return y;
    }

    public void setX(double x) {
        this.x = x;
    }

    public void setY(double y) {
        this.y = y;
    }
}
```

<br>

## **public 클래스의 불변인 필드를 직접 노출하는 경우**

- API를 변경하지 않고는 표현식을 바꿀 수 없고, 필드를 읽을 때 부수작업 수행이 불가하다는 단점
- 불변식은 보장 가능

```java
public final class Time {
    private static final int HOURS_PER_DAY = 24;
    private static final int MINUTES_PER_HOUR = 60;

    public final int hour;
    public final int minute;

    public Time(int hour, int minute) {
        if (hour < 0 || hour >= HOURS_PER_DAY) {
            throw new IllegalArgumentException(" 시간 : " + hour);
        }
        if (minute < 0 || minute >= MINUTES_PER_HOUR) {
            throw new IllegalArgumentException(" 분 : " + minute);
        }
        this.hour = hour;
        this.minute = minute;
    }
}
```

<br>

## **결론**

public 클래스는 절대 가변 필드를 직접적으로 노출해서는 안된다.

불변 필드라면 노출해도 되지만 완전히 안심할 수는 없다.

package-private 클래스나 private 중첩 클래스에는 불변이든 가변이든 필드를 노출하는게 나을 때도 있다.