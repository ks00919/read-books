# Item 16 - public 클래스에서는 public 필드가 아닌 접근자 메서드를 사용하라

## 아무 목적도 없는 퇴보한 public class를 만들지 말자

```java
class Point {
    public double x;
    public double y;
}
```

- 데이터 필드에 직접 접근할 수 있으니 캡슐화의 이점을 제공하지 못한다.
- API를 수정하지 않고는 내부 표현을 바꿀 수 없고, 불변식을 보장할 수 없다.
- 외부에서 필드에 접근할 때 부수 작업을 수행할 수도 없다.

## 접근자와 변경자 메서드를 활용해 데이터를 캡슐화 하자

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

public class에서라면 이렇게 하자.

- 패키지 바깥에서 접근할 수 있는 클래스라면 접근자를 제공함으로써, 클래스 내부 표현 방식을 언제든 바꿀 수 있도록 하자.

## public 클래스의 필드가 불변이어도 노출하지 말자

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

- API를 변경하지 않고는 표현 방식을 바꿀 수 없다
- 필드를 읽을 때 부수 작업을 수행할 수 없다는 단점이 있지만, 불변식은 보장할 수 있다.