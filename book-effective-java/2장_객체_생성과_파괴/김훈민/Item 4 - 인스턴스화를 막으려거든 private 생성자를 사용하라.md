# Item 4 - 인스턴스화를 막으려거든 private 생성자를 사용하라

정적 메서드와 정적 필드만을 담은 클래스는 때때로 유용하게 쓰일 때가 있다.

## 1. 기본 타입 값이나 배열 관련 메서드들을 모아놓을 수 있다.

`java.lang.Math` 와 `java.util.Arrays` 처럼 기본 타입 값이나 배열 관련 메서드들을 모아놓을 수 있다.

`java.lang.Math` 클래스는 수학적 계산을 위한 정적 메서드를 모아놓은 유틸리티 클래스이고 이 클래스를 이용하면 인스턴스를 생성할 필요 없이 수학적 연산을 수행할 수 있다.

`Math` 클래스의 모든 메서드는 정적(`static`) 메서드로 선언되어 있으며 생성자가 `private`로 선언되어 있어 인스턴스를 생성할 수 없다.

## 2. 특정 인터페이스를 구현하는 객체를 생성해주는 정적 메서드를 모아놓을 수 있다.

`java.util.Collection` 을 예시로 들고 있는데 `java.util.Math` 와 마찬가지로 `static`으로만 구성되어 있고, `private` 생성자가 존재하는 것을 확인할 수 있다.

## 3. final 클래스와 관련한 메서드들을 모아놓을 수 있다.

final 클래스는 상속해서 하위 클래스에 메서드를 넣는 것은 불가능하다.

`StringUtils` 을 예시로 살펴보면 `StringUtils` 클래스는 `String` 클래스와 관련된 여러 유틸리티 메서드를 제공하는 유틸리티 클래스이다.

이 클래스는 `final`로 선언하여 상속을 불가능하게 하고, 인스턴스 생성을 방지하기 위해 생성자를 `private`으로 선언하고있다.

코드로 보면

```java
public final class StringUtils {

    // private constructor to prevent instantiation
    private StringUtils() {
        throw new AssertionError("Cannot instantiate StringUtils");
    }

    // Checks if a string is null or empty
    public static boolean isNullOrEmpty(String str) {
        return str == null || str.isEmpty();
    }

    // Reverses a string
    public static String reverse(String str) {
        if (str == null) {
            return null;
        }
        return new StringBuilder(str).reverse().toString();
    }

    // Converts a string to uppercase
    public static String toUpperCase(String str) {
        if (str == null) {
            return null;
        }
        return str.toUpperCase();
    }

    // Converts a string to lowercase
    public static String toLowerCase(String str) {
        if (str == null) {
            return null;
        }
        return str.toLowerCase();
    }

    // Trims whitespace from both ends of a string
    public static String trim(String str) {
        if (str == null) {
            return null;
        }
        return str.trim();
    }
}
```

위처럼 되어 있는것을 확인할 수 있다.

즉,`StringUtils` 클래스는 `String` 클래스와 관련된 유틸리티 메서드들을 모아놓은 유틸리티 클래스이고, `StringUtils` 클래스는 `final`로 선언되어 상속할 수 없다.

생성자도 `private`으로 선언되어 있어 인스턴스를 생성할 수 없도록 하고 있다.

또한 모든 메서드는 정적 메서드로 선언되어, 인스턴스를 생성하지 않고도 사용할 수 있다.

그렇다면 인스턴스화를 막기 위해 다른 방법은 없을까?

## 1. 생성자를 명시하지 않으면?

컴파일러가 자동으로 public 으로 매개변수를 받지 않는 기본 생성자를 만들어주기 때문에 안된다.

## 2. 추상클래스로 만들어서 막기?

하위 클래스를 만들어 인스턴스화 하면 되기도 하고, 사용자는 상속해서 쓰라는 뜻으로 오해할 수 있으니 더 큰 문제이다.

그렇기 때문에 우리는

## private 생성자를 이용해서 클래스의 인스턴스화를 막자!!

코드로는

```java
public class UtilClass {

    // 유틸성 클래스로 인해 인스턴스화 방지
    private UtilClass() {
        throw new AssertionError();
    }

    // 나머지 코드는 생략
}
```

이렇게 사용하면 된다.