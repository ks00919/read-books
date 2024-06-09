# Item 2 - 생성자에 매개변수가 많다면 빌더를 고려하라

앞에서 설명한 정적 팩토리 메서드와 생성자는 선택적 매개변수가 많을 때, 적절히 대응하기가 어렵다는 단점이 있다. 그래서 이런 경우에는 점층적 생성자 패턴, 자바 빈즈 패턴, 빌더 패턴 을 사용해서 대응할 수 있다.

## 1. 점층적 생성자 패턴(telescoping constructor pattern)

점층적 생성자 패턴은 지금 까지 내가 사용했던 방식(…)이다.

```java
public class NutritionFacts {
	private final int servingSize;  // 필수
	private final int servings;     // 필수
	private final int calories;     // 선택
	private final int fat;          // 선택
	private final int sodium;       // 선택
	private final int carbohydrate; // 선택

	public NutritionFacts(int servingSize, int servings) {
		this(servingSize, servings, 0);
	}

	public NutritionFacts(int servingSize, int servings, int calories) {
		this(servingSize, servings, calories, 0);
	}

	public NutritionFacts(int servingSize, int servings, int calories, int fat) {
		this(servingSize, servings, calories, fat, 0);
	}

	public NutritionFacts(int servingSize, int servings, int calories, int fat, int sodium) {
		this(servingSize, servings, calories, fat, 0);
	}

		public NutritionFacts(int servingSize, int servings, int calories, int fat, int sodium, int carbohydrate) {
		this.servingSize = servingSize;
		this.servings = servings;
		this.calories = calories;
		this.fat = fat;
		this.sodium = sodium;
		this.carbohydrate = carbohydrate;
	}
}
```

책에서는 위 코드로 점층적 생성자 패턴을 설명하고 있다.

코드를 보면 이해할 수 있듯, 필수 매개변수만 받는 생성자, 필수 매개변수와 선택 매개변수 1개를 받는 생성자.. 등등 생성자의 종류를 늘려가는 방식이다.

이렇게 점층적 생성자 패턴으로 대응하게 되면, 매개변수 개수가 많아졌을 때, 클라이언트 코드를 작성하거나 읽기 어려워진다는 단점이 있다. 심지어 클라이언트가 실수로 매개변수 순서를 바꿔서 코드를 작성해도 컴파일러는 알아차리지 못한다..

## 2. 자바 빈즈 패턴(java beans pattern)

자바빈즈 패턴은 매개변수가 없는 생성자로 객체를 만든 후, `setter` 를 호출해서 원하는 매개변수의 값을 설정하는 방식이다.

```java
public class NutritionFacts {
	private int servingSize = -1;  // 필수
	private int servings = -1;     // 필수
	private int calories = 0;
	private int fat = 0;
	private int sodium = 0;
	private int carbohydrate = 0;

	public NutritionFacts() {}

	public void setServingSize(int val) { servingSize = val; }
	public void setServings(int val) { servings = val; }
	public void setCalories(int val) { calories = val; }
	public void setFat(int val) { fat = val; }
	public void setSodium(int val) { sodium = val; }
	public void setCarbohydrate(int val) {carbohydrate = val; }
}
```

위 코드에서 볼 수 있듯 점층적 생성자 패턴보다는 더 읽기 쉬운 코드가 되긴했다.

그렇지만 자바 빈즈 패턴에는 심각한 단점이 있다.

1. 객체 하나를 만들려면 메서드를 여러 개 호출해야 한다
2. 객체가 완전히 생성되기 전까지는 일관성(consistency)이 무너진 상태가 된다.

그렇기 때문에 쓰레드 안정성을 얻을 수가 없다.

만약 객체가 완전히 생성되기전에 이 객체를 다른 쓰레드가 사용하려고 한다면 이는 버그를 일으킬수있으니까..

코드로 보면

```java
public class Main {
    public static void main(String[] args) {
        Car car = new Car();

        Thread threadA = new Thread(() -> {
            car.setModel("Model S");
            car.setColor("Red");
            car.setYear(2021);
            car.setHorsepower(300);
        });

        Thread threadB = new Thread(() -> {
            System.out.println(car);
        });

        threadA.start();
        threadB.start();
    }
}
```

이런 상태에서 문제가 생길수 있다.

그래서 프로그래머가 freeze 메서드를 통해서 얼리고 얼리기 전에는 사용할 수 없도록 한다고 하는데 이 부분이 이해가 안되어서 찾아보았다.

코드로 확인하면

```java
public class Car {
    private String model;
    private String color;
    private int year;
    private int horsepower;
    private boolean frozen = false; // freeze 상태인지 체크

    public Car() {
    }

    public void setModel(String model) {
        checkFrozen();
        this.model = model;
    }

    public void setColor(String color) {
        checkFrozen();
        this.color = color;
    }

    public void setYear(int year) {
        checkFrozen();
        this.year = year;
    }

    public void setHorsepower(int horsepower) {
        checkFrozen();
        this.horsepower = horsepower;
    }

    // freeze Method
    public void freeze() {
        this.frozen = true;
    }

    // 객체가 freeze 상태인지 체크하는 메서드
    private void checkFrozen() {
        if (frozen) {
            throw new IllegalStateException("Cannot modify a frozen object");
        }
    }
}
```

이런 상태에서 다중 쓰레드 환경에서 사용하려면

```java
public class Main {
    public static void main(String[] args) {
        Car car = new Car();

        // 객체를 설정하는 첫번째 쓰레드
        Thread threadA = new Thread(() -> {
            try {
                car.setModel("Model S");
                car.setColor("Red");
                car.setYear(2021);
                car.setHorsepower(300);
                car.freeze();
                System.out.println("Thread A: Car is fully initialized and frozen.");
            } catch (Exception e) {
                System.out.println("Thread A: " + e.getMessage());
            }
        });

        // 객체를 사용하려는 두번째 쓰레드
        Thread threadB = new Thread(() -> {
            try {
                car.checkFrozenForAccess(); // 객체가 동결 상태인지 확인
                System.out.println("Thread B: " + car);
            } catch (IllegalStateException e) {
                System.out.println("Thread B: " + e.getMessage());
            }
        });

        threadA.start();
        try {
            Thread.sleep(100);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        threadB.start();
    }
}

```

이런식으로 해서 쓰레드 안정성을 보장한다는 의미로 이해했다.

그렇지만 어찌되었든 수동으로 freeze 를 해주어야 하기도 하고, 객체 사용 전에 개발자가 freeze 를 확실히 호출했는지 컴파일러가 확인해줄 수 없기 때문에 런타임 에러가 발생할 확률이 높다.

그렇기 때문에 빌더 패턴을 사용하는 것이 좋다.

## 3. 빌더 패턴

클라이언트는 필요한 객체를 직접 만드는 대신, 필수 매개변수만으로 만들어진 빌더 객체를 얻은 다음에 빌더 객체가 제공하는 일종의 세터 메서드들로 원하는 선택 매개변수들을 설정하고, 매개변수가 없는 build 메서드를 활용하여 우리에게 필요한 객체를 얻는 방식이다.

그리고 빌더는 생성할 클래스 안에 정적 멤버 클래스로 만들어두는 것이 보통이라고 한다.

```java
public class NutritionFacts {
	private final int servingSize;
	private final int servings;
	private final int calories;
	private final int fat;
	private final int sodium;
	private final int carbohydrate;

	public static class Builder {
		private final int servingSize;  // 필수
		private final int servings;     // 필수
		private int calories = 0;
		private int fat = 0;
		private int sodium = 0;
		private int carbohydrate = 0;

		public Builder(int servingSize, int servings) {
			this,servingSize = serginsSize;
			this.servings = servings;
		}

		public Builder fat(int val) {
			fat = val;
			return this;
		}

		public Builder sodium(int val) {
			sodium = val;
			return this;
		}

		public Builder carbohydrate(int val) {
			carbohydrate = val;
			return this;
		}

		public NutritionFacts build() {
			return new NutritionFacts(this);
		}
	}

	private NutirionFacts(Builder builder) {
		servingSize = builder.servingSize;
		servings = builder.servings;
		calories = builder.calories;
		fat = builder.fat;
		sodium = builder.fat;
		carbohydrate = builder.carbohydrate;
	}
}
```

위 코드에서 알 수 있듯이 NutritionFacts 클래스는 불변하다… 라고 하는데 나는 잘 이해가 되지 않아서 찾아보았다.

불변하다고 하는 이유는 크게 3가지이다.

1. 모든 필드가 `final`로 선언되어 있다.
2. 필드 값이 생성자를 통해서만 초기화된다.
3. `setter` 메서드가 없어서 생성 후에는 필드 값을 변경할 수 없다.

라고한다.

그리고 빌더의 세터 메서드들은 this 를 통해서 자기 자신을 반환하기 때문에 연쇄적으로 호출할 수 있다.

이런 방식을 플루언트 API(Fluent API) 혹은 메서드 연쇄(Method Chaining)이라고 한다.

위 코드처럼 빌더 패턴을 사용하면

```java
NutritionFacts cocaCola = new NutritionFacts.Builder(240, 8)
	.calories(100)
	.sodium(35)
	.carbohydrate(27)
	.build();
```

이렇게 깔끔하게 사용할 수 있다.

또한 빌더 패턴은 계층적으로 설계된 클래스와 함께 쓰기에 좋다.

## 빌더 패턴 - 계층적인 클래스에서의 사용

책에서는 피자로 예시를 들고 있다.

```java
public abstract class Pizza {
	public enum Topping { HAM, MUSHROOM, ONION, PEPPER, SAUSAGE }
	final Set<Topping> toppings;

	abstract static class Builder<T extends Builder<T>> {
		EnumSet<Topping> toppings = EnumSet.noneOf(Topping.class);
		public T addTopping(Topping topping) {
			toppings.add(Objects.requireNonNull(topping));
			return self();
		}

		abstarct Pizza build();

		protected abstract T self(); // 하위 클래스에서 이 메서드를 오버라이드 해서 this를 반환
	}

	Pizza(Builder<?> builder) {
		toppings = builder.toppings.clone();
	}
}
```

여기에서 T는 제네릭 타입에서 자기자신을 의미한다.

`Builder<T>` 이면 자기자신을 반환하는것을 보장하는 것이라고 생각하면 되고 이를 통해서 메서드 체이닝을 할때 타입 안정성을 구현할 수 있다.

`addTopping` 메서드는 T 타입을 반환한다. 즉, 호출한 인스턴스의 자기 자신을 반환하도록 만들고 이를 통해 `builder.addTopping(HAM).addTopping(ONION)` 이 코드처럼 메서드 체이닝을 사용할 수 있다.

이 피자의 하위 클래스로는 2가지의 피자가 있다.

1. 일반적인 뉴옥 피자 ⇒ `size` 매개변수를 필수로 받는다
2. 칼초네 피자 ⇒ 소스를 안에 넣을지 선택(`sauceInside`)하는 매개변수를 필수로 받는다

```java
public class NyPizza extends Pizza {
	public enum Size { SMALL, MEDIUM, LARGE }
	private final Size size;

	public static class Builder extends Pizza.Builder<Builder> {
		private final Size size;

		public Builder(Size size) {
			this.size = Objects.requireNonNull(size);
		}

		@Override
		public NyPizza build() {
			return new NyPizza(this);
		}

		@Override
		protected Builder self() {
			return this;		
		}

		public NyPizza(Builder builder) {
			super(builder);
			size = builder.size;
		}
	}
}
```

```java
public class Calzone extends Pizza {
	private final boolean sauceInside;

	public static class Builder extends Pizza.Builder<Builder> {
		private boolean sauceInsice = false;

		public Builder sauceInside() {
			sauceInsdie = true;
			return this;
		}
		
		@Override
		public Calzone build() {
			return new Calzone(this);
		}

		@Override
		protected Builder self() {
			return this;
		}

		private Calzone(Builder builder) {
			super(builder);
			sauceInside = builder.sauceInside;
		}
	}
}
```

클라이언트 코드에서는

```java
NyPizza pizza = new NyPizza.Builder(SMALL)
	.addTopping(SAUSAGE)
	.addTopping(ONION)
	.build();

Calzone pizza = new Calzone.Builder()
	.addTopping(HAM)
	.sauceInside()
	.build();
```

위처럼 사용할 수 있다.

위에서 설명했듯 제네릭 타입이 T이기 때문에 `NyPizza.Builder` 에서는 `NyPizza`를, `Calzone.Builder` 에서는 `Calzone`을 반환한다.

## 빌더 패턴의 단점?

객체를 만들려면, 그에 앞서 빌더부터 만들어야 해서 빌더 생성 비용이 들어간다는 단점이 있다.

빌더 생성 비용이 크지는 않지만 성능에 민감한 상황에서는 문제가 될 수 있으니 고려하자.

또한 점층적 생성자 패턴보다는 코드가 장황해서 매개변수가 4개 이상이어야 이득이다.