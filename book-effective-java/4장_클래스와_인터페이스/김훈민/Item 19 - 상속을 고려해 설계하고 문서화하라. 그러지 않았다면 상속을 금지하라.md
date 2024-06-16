# Item 19 - 상속을 고려해 설계하고 문서화하라. 그러지 않았다면 상속을 금지하라

### 상속용 클래스는 재정의할 수 있는 메서드들을 내부적으로 어떻게 이용하는지 문서로 남겨야 한다.

- 재정의 가능 메서드의 API 설명에 대해 적시하기
    - 클래스의 API로 공개된 메서드에서 클래스 자신의 또 다른 메서드를 호출할 수도 있음!
    - 어떤 순서로 호출하는지, 각각의 호출 결과가 이어지는 처리에 어떤 영향을 주는지도 담아야함.
    - 재정의 가능 메서드를 호출할 수 있는 모든 상황을 문서로 남겨야 한다
    - API 문서의 메서드 설명 끝에서 Implementation Requirements로 시작하는 절이 메서드의 내부 동작 방식을 설명하는 곳이다.

### 클래스의 내부 동작 과정 중간에 끼어들 수 있는 훅을 잘 선별하여 protected 메서드 형태로 공개해야 할 수도 있다.

- ex) java.util.AbstractList의 removeRange 메서드

### 상속용 클래스를 시험하는 방법은 직접 하위 클래스를 만들어보는 것이 유일하다

- 상속용 클래스를 설계할때 어떤 메서드를 protected로 노출해야할지 결정하는 일은 실제로 하위클래스를 만들어 시험해보는 것이 최선이다.
    - 꼭 필요한 protected 멤버를 놓치면 하위클래스를 작성할 때 그 빈자리가 크다
    - 만약 전혀 쓰이지 않는 protected 멤버가 있다면 private으로 하자.
    - 결국은 만들어서 사용해봐야 알수있다.

### 상속용 클래스의 생성자는 직접적으로든 간접적으로든 재정의 가능 메서드를 호출해서는 안된다.

- 상위 클래스의 생성자가 하위 클래스의 생성자보다 먼저 실행된다.
    - 하위 클래스에서 재정의한 메서드가 하위 클래스의 생성자보다 먼저 호출된다
    - 결국, 의도대로 동작하지 않게 된다.

```java
public class Super {
    // 잘못된 예시 - 생성자가 재정의 가능 메서드를 호출한다.
    public Super() {
        overrideMe();
    }

    public void overriedMe() {

    }
}
```

```java
public final class Sub extends Super {
    // 초기화되지 않은 final 필드, 생성자에서 초기화한다.
    private final Instant instant;

    Sub() {
        instant = Instant.now();
    }

    // 재정의 가능 메서드, 상위 클래스의 생성자가 호출한다.
    @Override public void overrideMe() {
        System.out.println(instant);
    }

    public static void main(String[] args) {
        Sub sub = new Sub();
        sub.overrideMe();
    }
}
```

- 이 프로그램은 instance를 두 번 출력하지 않고, 첫 번째는 null을 출력한다.
- 상위 클래스의 생성자는 하위 클래스의 생성자가 인스턴스 필드를 초기화하기도 전에 overrideMe를 호출하기 때문이다.
- private, final, static 메서드는 재정의가 불가능하니 생성자에서 안심하고 호출해도 된다.

### clone과 **readObject 모두 직접적으로든 간접적으로든 재정의 가능 메서드를 호출해서는 안 된다.**

- readObject의 경우 하위 클래스의 상태가 미처 다 역직렬화되기 전에 재정의한 메서드부터 호출하게 된다.
- clone의 경우 하위 클래스의 clone 메서드가 복제본의 상태를 (올바른 상태로) 수정하기 전에 재정의한 메서드를 호출한다.
- clone이 잘못되면 복제본뿐만아니라 원본 객체에도 피해를 줄 수 있다.

### **Serializable을 구현한 상속용 클래스가 readResolve나 write Replace 메서드를 갖는다면 이 메서드들은 private이 아닌 protected로 선언해야 한다.**

즉, 클래스를 상속용으로 설계하려면 엄청난 노력이 들고 그 클래스에 안기는 제약도 상당하다는 것을 알 수 있다.

그러므로 상속용으로 설계하지 않은 클래스는 상속을 금지하자!!