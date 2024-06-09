# Item 1 - 생성자 대신 정적 팩토리 메서드를 고려하라

보통 우리가 클래스를 인스턴스화 해서 사용할때는 생성자를 이용해서 만든다.

```java
public class Car {
    private String model;
    private String color;

    // 일반 생성자
    public Car(String model, String color) {
        this.model = model;
        this.color = color;
    }

    // Getter 메서드
    public String getModel() {
        return model;
    }

    public String getColor() {
        return color;
    }

    public static void main(String[] args) {
        // 일반 생성자를 사용하여 객체 생성
        Car car = new Car("Sedan", "Red");
        System.out.println("Model: " + car.getModel());
        System.out.println("Color: " + car.getColor());
    }
}

```

그렇지만, 정적 팩토리 메서드를 이용하면 얻을 수 있는 장점이 있기 때문에 상황을 고려해서 생성자로만 만들지 말고 정적 팩토리 메서드를 이용해서 만들기도 하면 좋다.

정적 팩토리 메서드라고 해서 어렵게 생각했는데, 위 코드에 정적 팩토리 메서드를 추가하면 아래와 같아진다.

```java
package item1;

public class Car {
    private String model;
    private String color;

    // 일반 생성자
    public Car(String model, String color) {
        this.model = model;
        this.color = color;
    }

    // 정적 팩터리 메서드
    public static Car createCar(String model, String color) {
        return new Car(model, color);
    }

    // Getter 메서드
    public String getModel() {
        return model;
    }

    public String getColor() {
        return color;
    }

    public static void main(String[] args) {
        // 일반 생성자를 사용하여 객체 생성
        Car car = new Car("Sedan", "Red");
        System.out.println("Model: " + car.getModel());
        System.out.println("Color: " + car.getColor());

        // 정적 팩터리 메서드를 사용하여 객체 생성
        Car car2 = Car.createCar("SUV", "Blue");
        System.out.println("Model: " + car2.getModel());
        System.out.println("Color: " + car2.getColor());
    }
}

```

위처럼 `new` 키워드 대신 클래스의 `createCar` 이라는 정적 메서드를 사용해서 객체를 생성할 수 있다.

이로 인해서 얻을 수 있는 장점은 아래 5가지와 같다.

## 1. 이름을 가질 수 있다.

그냥 생성자를 사용하면

```java
public class Car {
    private String model;
    private String color;
    private int year;

    public Car(String model, String color, int year) {
        this.model = model;
        this.color = color;
        this.year = year;
    }
}
```

아래 코드와 같이 생성할 수 있다.

```java
Car car = new Car("Model S", "Red", 2021);
```

여기까지는 내가 아는 내용 그대로이지만, 만약 어떤 객체가 생성되는지 더 구체적으로 명시하고 싶다면 생성자로 생성하는 것보다 정적 팩토리 메서드를 이용하면 이름으로 명확하게 목적을 알 수 있으니 좋다.

정적 팩토리 메서드를 사용하면

```java
public class Car {
    private String model;
    private String color;
    private int year;

    private Car(String model, String color, int year) {
        this.model = model;
        this.color = color;
        this.year = year;
    }

    public static Car createWithModelAndColor(String model, String color) {
        return new Car(model, color, 2021);
    }

    public static Car createWithModelAndYear(String model, int year) {
        return new Car(model, "Unknown", year);
    }

    public static Car createComplete(String model, String color, int year) {
        return new Car(model, color, year);
    }
}

```

위 코드에서 객체를 생성할때

```java
Car car1 = Car.createWithModelAndColor("Model S", "Red");
Car car2 = Car.createWithModelAndYear("Model X", 2022);
Car car3 = Car.createComplete("Model 3", "Blue", 2021);
```

조금 더 명확하게 객체를 생성할 수 있게 된다.

또한 같은 타입을 파라미터로 받는 생성자를 정의할 수 있게 되는데, 예를 들어서,

```java
public Car(String model){
		this.model = model;
}

public Car(String color){
		this.color = color;
}
```

위와 같이 같은 타입을 파라미터로 받는 생성자는 정의할 수 없지만, 정적 팩토리 메서드를 이용하면

```java
public static Car withModel(String model){
		Car car = new Car();
		car.model = model;
		return car;
}

public static Car withColor(String color){
		Car car = new Car();
		car.color = color;
		return car;
}
```

위처럼 가능하게 할 수 있다.

## 2. 호출될 때마다 인스턴스를 새로 생성하지는 않아도 된다.

싱글턴과도 비슷한 느낌이 있는데, 만약에 같은 속성을 가진 car 객체가 이미 존재한다면, 같은 속성의 car 객체를 또 만들 필요는 없으므로 그냥 동일한 객체를 반환하도록 하는 것이 이득이다.

```java
import java.util.HashMap;
import java.util.Map;

public class Car {
    private String model;
    private String color;
    private int year;
    private int horsepower;

    // 이미 생성된 Car 객체들을 저장할 캐시
    private static final Map<String, Car> cache = new HashMap<>();

    // 생성자는 private으로 해서 함부로 재생성할 수 없도록 막는다.
    private Car(String model, String color, int year, int horsepower) {
        this.model = model;
        this.color = color;
        this.year = year;
        this.horsepower = horsepower;
    }

    // 정적 팩토리 메서드
    public static Car createWithModelAndColor(String model, String color) {
        String key = model + color;
        // 만약 같은 속성의 객체가 만들어져 있다면
        if (cache.containsKey(key)) {
            return cache.get(key); // 이미 생성된 객체를 반환
        } else {
            Car newCar = new Car(model, color, 2021, 150); // default year and horsepower
            cache.put(key, newCar); // 새로 생성된 객체를 캐시에 저장
            return newCar;
        }
    }

    // 다른 정적 팩토리 메서드 예시
    public static Car createWithModelAndYear(String model, int year) {
        return new Car(model, "Unknown", year, 150); // 기본 color와 horsepower
    }

    public static Car createComplete(String model, String color, int year, int horsepower) {
        return new Car(model, color, year, horsepower);
    }
}

```

위 코드처럼 HashMap에 이미 같은 속성의 객체가 있다면 해당 객체를 반환하는 것으로 정적 팩토리 메서드를 통해서 인스턴스를 새로 생성하지 않을 수 있다.

## 3. 반환 타입의 하위 타입 객체를 반환할 수 있는 능력이 있다.

위와 같은 Car 클래스를 상속받는 ElectricCar와

```java
public class ElectricCar extends Car {
    private int batteryCapacity;

    public ElectricCar(String model, String color, int year, int horsepower, int batteryCapacity) {
        super(model, color, year, horsepower);
        this.batteryCapacity = batteryCapacity;
    }
}
```

GasCar 가 있다고 했을 때

```java
public class GasCar extends Car {
    private int fuelCapacity;

    public GasCar(String model, String color, int year, int horsepower, int fuelCapacity) {
        super(model, color, year, horsepower);
        this.fuelCapacity = fuelCapacity;
    }
}
```

Car 클래스에 아래와 같은 메서드를 추가할 수 있다.

```java
    public static Car createGasCar(String model, String color, int year, int horsepower, int fuelCapacity) {
        return new GasCar(model, color, year, horsepower, fuelCapacity);
    }

    public static Car createElectricCar(String model, String color, int year, int horsepower, int batteryCapacity) {
        return new ElectricCar(model, color, year, horsepower, batteryCapacity);
    }
```

그러면 다양한 하위 타입의 객체를 아래 코드처럼 생성할 수 있게 된다.

```java
public class Main {
    public static void main(String[] args) {
        Car gasCar = Car.createGasCar("Model X", "Blue", 2022, 400, 50);
        Car electricCar = Car.createElectricCar("Model S", "Red", 2022, 500, 100);

        System.out.println(gasCar);
        System.out.println(electricCar);
    }
}

```

라는 것까지는 이해했는데 어떤 부분에서 이게 장점이 되는지 잘 와닿지 않아서 더 고민을 해보았는데

Car 라는 인터페이스가 있다고 하고

```java
public interface Car {
    String getModel();
    String getColor();
    int getYear();
    int getHorsepower();
    String toString();
}
```

이 Car 라는 인터페이스를 구현하는 두개의 클래스가 있다고 하고

```java
public class GasCar implements Car {
    private String model;
    private String color;
    private int year;
    private int horsepower;
    private int fuelCapacity;

    public GasCar(String model, String color, int year, int horsepower, int fuelCapacity) {
        this.model = model;
        this.color = color;
        this.year = year;
        this.horsepower = horsepower;
        this.fuelCapacity = fuelCapacity;
    }

    @Override
    public String getModel() { return model; }
    @Override
    public String getColor() { return color; }
    @Override
    public int getYear() { return year; }
    @Override
    public int getHorsepower() { return horsepower; }
}

```

```java
public class ElectricCar implements Car {
    private String model;
    private String color;
    private int year;
    private int horsepower;
    private int batteryCapacity;

    public ElectricCar(String model, String color, int year, int horsepower, int batteryCapacity) {
        this.model = model;
        this.color = color;
        this.year = year;
        this.horsepower = horsepower;
        this.batteryCapacity = batteryCapacity;
    }

    @Override
    public String getModel() { return model; }
    @Override
    public String getColor() { return color; }
    @Override
    public int getYear() { return year; }
    @Override
    public int getHorsepower() { return horsepower; }
}

```

이 클래스들을 CarFactory 라는 클래스를 통해서 제공한다고 하면

```java
public class CarFactory {
    public static Car createGasCar(String model, String color, int year, int horsepower, int fuelCapacity) {
        return new GasCar(model, color, year, horsepower, fuelCapacity);
    }

    public static Car createElectricCar(String model, String color, int year, int horsepower, int batteryCapacity) {
        return new ElectricCar(model, color, year, horsepower, batteryCapacity);
    }
}
```

사용할때

```java
public class Main {
    public static void main(String[] args) {
        Car gasCar = CarFactory.createGasCar("Model X", "Blue", 2022, 400, 50);
        Car electricCar = CarFactory.createElectricCar("Model S", "Red", 2022, 500, 100);

        System.out.println(gasCar);
        System.out.println(electricCar);
    }
}
```

이렇게 사용할 수 있다.

이렇게 했을때 클라이언트 코드는 `Car` 인터페이스를 통해서만 객체에 접근하므로, 실제 구현 클래스(`GasCar`, `ElectricCar`)에 대한 의존성이 없다.

또한 개발자는 `CarFactory` 클래스의 정적 팩토리 메서드를 통해서만 접근할 수 있으며 구현 클래스를 찾아보지 않아도 된다.

그리고 사용할때 `CarFactory`의 메서드를 통해 `Car` 인터페이스만 사용하게 되므로, API는 더 단순하고 명확해진다.

책에서는 이 부분에서 `java.util.Collections` 를 예시로 들었다.

그리고 `Car`의 새로운 구현체가 추가되더라도 새로운 구현체는 `CarFactory` 클래스에 추가하면 되니, 코드도 더 간단하게 작성할 수 있게 된다!

## 4. 입력 매개변수에 따라 매번 다른 클래스의 객체를 반환할 수 있다.

책에서는 `EnumSet` 클래스로 예시를 들고 있는데, OpenJDK 에서는 `EnumSet` 클래스의 원소의 수에 따라 두 가지 하위 클래스 중 하나의 인스턴스를 반환한다고 한다.

64개 이상이면 원소들을 `long` 변수 하나로 관리하는 `RegularEnumSet` 인스턴스를 반환하고

65개 이상이면 원소들을 `long` 배열로 관리하는 `JumboEnumSet` 인스턴스를 반환한다고 설명하고 있다.

우리가 `EnumSet` 을 사용할때 이런 부분까지 고려하거나 인지하고 사용하지는 않고, 그냥 `EnumSet`에 원소들을 추가하기만 하면 알아서 다른 인스턴스를 반환해준다.

이렇게 했을 때의 장점은 클라이언트는 위 두 클래스의 자세한 정보를 몰라도 된다는 것, 성능을 더 개선한 다른 클래스가 추가된다고 하더라도 기존의 코드를 변경할 필요가 없다는 점 등이 있다.

## 5. 정적 팩터리 메서드를 작성하는 시점에는 반환할 객체의 클래스가 존재하지 않아도 된다.

이부분도 이해하기 쉽지 않았는데, 유연성에 대한 설명이라고 이해했다.

위에 작성했던 내용대로 `Car` 라는 인터페이스를 구현하는 `GasCar` 와 `ElectricCar` 가 있다고 했을 때, 추가적으로 `HybridCar` 구현체가 생기면

```java
public class CarFactory {
    public static Car createCar(String type, String model, String color, int year, int horsepower, int capacity1, int capacity2) {
        if (type.equalsIgnoreCase("Gas")) {
            return new GasCar(model, color, year, horsepower, capacity1);
        } else if (type.equalsIgnoreCase("Electric")) {
            return new ElectricCar(model, color, year, horsepower, capacity1);
        } else if (type.equalsIgnoreCase("Hybrid")) {
            return new HybridCar(model, color, year, horsepower, capacity1, capacity2);
        } else {
            throw new IllegalArgumentException("Unknown car type: " + type);
        }
    }
}
```

코드가 크게 변경될 필요 없이 `Hybrid` 부분만 추가하면 된다.

사용하는 `Client` 코드도 크게 변경될 필요가 없다.

```java
public class Main {
    public static void main(String[] args) {
        Car gasCar = CarFactory.createCar("Gas", "Model X", "Blue", 2022, 400, 50, 0);
        Car electricCar = CarFactory.createCar("Electric", "Model S", "Red", 2022, 500, 100, 0);
        Car hybridCar = CarFactory.createCar("Hybrid", "Model Y", "Green", 2023, 450, 40, 60);

        System.out.println(gasCar);
        System.out.println(electricCar);
        System.out.println(hybridCar);
    }
}
```

만약 책에서 말한 내용처럼 정적 팩토리 메서드를 작성할때는 `HybridCar` 가 구현되어 있지 않아서

```java
public class CarFactory {
    public static Car createCar(String type, String model, String color, int year, int horsepower, int capacity1, int capacity2) {
        if (type.equalsIgnoreCase("Gas")) {
            return new GasCar(model, color, year, horsepower, capacity1);
        } else if (type.equalsIgnoreCase("Electric")) {
            return new ElectricCar(model, color, year, horsepower, capacity1);
        } else {
            throw new IllegalArgumentException("Unknown car type: " + type);
        }
    }
}
```

`CarFactory` 가 위와 같은 모습이라고 해도, 어차피 `else` 에서 잡히니까 클라이언트 코드는 변경될 필요가 없을 것이고, `HybridCar` 가 구현되더라도

```java
else if (type.equalsIgnoreCase("Hybrid")) {
            return new HybridCar(model, color, year, horsepower, capacity1, capacity2);
        } 
```

이 부분만 새로 추가하면 될 것이다.

책에서는 `JDBC`를 예시로 들고 있는데, 잘 이해가 되지 않아서 자세히 찾아 보았는데

우리가 `JDBC` 를 사용할때

```java
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.SQLException;

public class DatabaseExample {
    public static void main(String[] args) {
        try {
            // 데이터베이스 URL, 사용자 이름, 비밀번호를 사용하여 데이터베이스에 연결
            Connection conn = DriverManager.getConnection("jdbc:mysql://localhost:3306/mydatabase", "user", "password");

            // 데이터베이스 작업 수행

            // 연결 종료
            conn.close();
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
}

```

위와 같이 사용하게 된다.

여기에서 `DriverManager` 클래스는 정적 팩토리 메서드인 `getConnection` 을 사용하여 `Connection` 객체를 반환하는데 이 `Connection` 객체는 인터페이스 타입이다.

여기에서 이 `Connection` 객체는 `getConnection` 메서드를 통해서 MySQL 드라이버나, Oracle 드라이버 등  url 에 따라 다른 객체를 받게 되며, 인터페이스 타입이기 때문에 모든 것들을 받을 수 있다.

그리고 이러한 `JDBC`는 서비스 제공자 프레임워크의 대표격이라고 한다..

책에 나온 내용을 그대로 이해하지는 못했고 내가 생각했을때 이런 장점을 설명하고 싶은 것 같은데 맞는지 모르겠다.

그리고 단점도 있는데 단점으로는

## 1. 상속을 하려면 public이나 protected 생성자가 필요하니 정적 팩토리 메서드만 제공하면 하위 클래스를 만들 수 없다.

위의 예시를 보면

```java
public class Car {
    private String model;
    private String color;
    private int year;
    private int horsepower;

    private Car(String model, String color, int year, int horsepower) {
        this.model = model;
        this.color = color;
        this.year = year;
        this.horsepower = horsepower;
    }

    public static Car createWithModelAndColor(String model, String color) {
        return new Car(model, color, 2021, 150);
    }

    public static Car createWithModelAndYear(String model, int year) {
        return new Car(model, "Unknown", year, 150);
    }

    public static Car createComplete(String model, String color, int year, int horsepower) {
        return new Car(model, color, year, horsepower);
    }
}
```

생성자를 이렇게 `private` 로 했을때, 외부에서 이 클래스를 상속할수없다.

만약 상속하게 하고 싶다면 생성자를 `protected` 나 `public` 으로 해야한다.

그래서 `Collections Framework` 의 `Utilities` 구현 클래스들은 상속할 수 없다. 그러나 어떻게 보면 상속보다 컴포지션을 사용하도록 유도하기도 하고, 불변 타입으로 만들려면 이 제약을 지켜야하기 때문에 장점으로 받아들일수도 있다고 한다.

## 2. 정적 팩토리 메서드는 프로그래머가 찾기 어렵다

생성자처럼 API 설명에 명확하게 드러나지 않으니(생성자는 `javaDoc` 이 알아서 모아서 설명해준다), 사용자는 정적 팩토리 메서드 방식 클래스를 인스턴스화할 방법을 알아내야하기때문에, API 문서를 잘 써놓고 메서드 이름도 최대한 convention 을 지켜서 문제를 완화해야한다고 한다.

convention 은 책에 설명되어 있다.