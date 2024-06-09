# Item 5 - 자원을 직접 명시하지 말고 의존 객체 주입을 사용하라

많은 클래스가 하나 이상의 자원에 의존한다.

예를 들어, 맞춤법 검사기는 사전(dictionary)에 의존하는데, 이런 클래스를 정적 유틸리티 클래스로 구현한 모습을 종종 볼 수 있다.

### 정적 유틸리티를 잘못 사용한 예시

이렇게 하면 유연하지 않고 테스트하기도 어렵다.

```java
public class SpellChecker {
    private static final Lexicon dictionary = ...;

    private SpellChecker() {} // 객체 생성 방지

    public static boolean isValid(String word) { ... }
    public static List<String> suggestions (String typo) { ... }
}
```

위 코드에서 `SpellChecker` 클래스는 정적 메서드만을 가지고 있으며, `dictionary` 객체는 정적 필드로 초기화된다. 이로 인해 `dictionary` 객체를 주입할 방법이 없고, 클래스 내부에서만 직접 초기화된 인스턴스를 사용하게 된다.

더 자세하게 설명하자면, 정적 유틸리티 클래스로 구현 하면 몇가지 단점이 있다.

1. 의존성 주입이 어렵다

의존성 주입은 크게 생성자 주입(Constructor Injection), 세터 주입(Setter Injection), 그리고 필드 주입(Field Injection) 이렇게 3가지가 있는데 어떤 방법으로도 의존성을 주입할 수 없다.

생성자를 private 로 숨겨놓았기 때문에 인스턴스화가 방지되고 상태를 유지할 수 없기 때문이다.

1. 테스트가 어렵다

정적 유틸리티 클래스는 정적 메서드를 사용하기 때문에, 테스트 시 객체의 상태를 설정하거나 변경하는 것이 어렵다.

### 싱글턴을 잘못 사용한 예시

이 방식도 정적 유틸리티 클래스의 방식과 같은 단점을 가지고 있다.

```java
public class SpellChecker {
    private final Lexicon dictionary = ...;

    private SpellChecker() {}
    public static SpellChecker INSTANCE = new SpellChecker(...);

    public boolean isValid(String word) { ... }
    public List<String> suggestions (String typo) { ... }
}
```

두 방식 모두 사전을 하나만 사용한다면 괜찮지만, 실전에서는 사전이 언어별로 따로 있기도 하고, 특수 어휘용 사전을 별도로 두기도 하며 테스트용 사전도 필요할 수 있기 때문에 사전 하나로 모든 쓰임새에 대응 할 수 있기를 바라는 것은 잘못된 생각이다.

그렇다면 어떻게 해야할까?

### 의존 객체 주입을 통해 리팩토링한 코드

```java
public class SpellChecker {
    private final Lexicon dictionary;

    public SpellChecker(Lexicon dictionary) {
        this.dictionary = Objects.requireNonNull(dictionary);
    }

    public boolean isValid(String word) { ... }
    public List<String> suggestions(String typo) { ... }
}
```

인스턴스를 생성할 때 생성자에 필요한 자원을 넘겨주는 방식으로, 맞춤법 검사기를 생성할 때 의존 객체인 사전을 주입해주면 된다.

일단 이 의존 객체 주입 패턴은 아주 단순해서 직관적으로 이해하기 쉽고

자원이 몇 개든, 의존 관계가 어떻든 상관없이 잘 작동한다.

또한 불변을 보장하여 같은 자원을 사용하려는 여러 클라이언트가 의존 객체들을 안심하고 공유할 수 있게 한다.

이 의존 객체 주입 패턴은 생성자, 정적 팩토리, 빌더 모두에 똑같이 응용할 수 있다.

클라이언트에서는

```java
Lexicon lexicon = ...; // Lexicon 객체 생성
SpellChecker spellChecker = new SpellChecker(lexicon); // 주입
```

위와 같이 사용하면 된다.

이 패턴의 변형으로 생성자에 자원 팩토리를 넘겨주는 방식도 있다.

자바 8에서 소개한 `Supplier<T>` 인터페이스가 팩토리를 표현한 완벽한 예시라고 한다.

`Supplier<T>`를 입력으로 받는 메서드는 일반적으로 한정적 와일드카드 타입을 사용해 팩토리의 타입 매개변수를 제한해야 한다.

코드로 작성하면

```java
public class SpellChecker {
    private final Lexicon dictionary;

    // 생성자에서 Supplier<Lexicon>을 받음
    public SpellChecker(Supplier<? extends Lexicon> dictionarySupplier) {
        this.dictionary = dictionarySupplier.get();
    }

    public boolean isValid(String word) {
        return dictionary.contains(word);
    }

    public List<String> suggestions(String typo) {
        return dictionary.suggest(typo);
    }
}
```

한정적 와일드카드 타입을 사용해서 (`upplier<? extends Lexicon>`)  `SpellChecker`는 `Lexicon`의 하위 타입까지 받을 수 있다.

클라이언트에서는

```java
public class Main {
    public static void main(String[] args) {
        Supplier<AdvancedLexicon> advancedDictionarySupplier = AdvancedLexicon::new;
        SpellChecker spellChecker = new SpellChecker(advancedDictionarySupplier);

        boolean isValid = spellChecker.isValid("hello");
        System.out.println("Is valid: " + isValid);
    }
}
```

이렇게 사용할 수 있다.

의존 객체 주입이 유연성과 테스트 용이성을 개선해주긴 하지만, 의존성이 복잡한 큰 프로젝트에서는 코드를 어지럽게 만들기 때문에 대거, 주스, 스프링같은 의존 객체 주입 프레임워크를 사용하면 이런 어지러움을 해결 할 수 있다.