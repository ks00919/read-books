# Item 3 : private 생성자나 열거 타입으로 싱글턴임을 보증하라

> 싱글턴 : 인스턴스를 오직 하나만 생성할 수 있는 클래스

### 싱글턴을 만드는 방식

- 두 방식 모두 생성자를 private으로 감추고 인스턴스에 접근할 수 있도록 public static 멤버를 마련해둔다.

1. public static 멤버가 final 필드인 방식
   - 리플렉션 API을 사용해 private 생성자를 호출할 수 있다.
     -> 생성자를 수정하여 두번째 객체가 생성되려 할 때 예외를 던진다.
2. 정적 팩터리 메서드를 public static 멤버로 제공
   - 추후 싱글턴이 아니게 변경이 용이하다.
   - 정적 팩터리를 제네릭 싱글턴 팩터리로 만들 수 있다.
   - 정적 팩터리의 메소드 참조를 공급자로 사용할 수 있다.

```java
//1
public static final Elvis INSTANCE = new Evis();

//2
private static final Elvis INSTANCE = new Evis();

public static final getInstance(){
    return INSTANCE;
}
```

또 다른 방식 3. 원소가 하나인 열거타입을 사용한다. - 복잡한 직렬화 상황이나 리플렉션 공격에서도 제 2인스턴스가 생기는 일을 완벽히 막아준다. - 만들려는 싱글턴이 Enum 이외 클래스를 상속해야한다면 이 방법은 사용할 수 없다.

```java

public enum Elvis{
    INSTANCE;
    public void leaveTheBuilding(){...}
}

```

# Item 4: 인스턴스화를 막으려거든 private 생성자를 사용하라

- 추상 클래스를 만드는 것으로는 인스턴스화를 막을 수 없다.
- 단순한 정적메서드나 정적 피드만을 담은 클래스를 만들고 싶을 때 private 생성자를 추가면 클래스의 인스턴스화를 막을 수 있다.
