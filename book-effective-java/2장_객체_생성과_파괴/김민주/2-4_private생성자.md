## 아이템 4. 인스턴스화를 막으려거든 `private` 생성자를 사용하라

```java
public class UtilityClass {
    private UtilityClass() {
        throw new AssertionError();
    }
}
```

- 정적 메서드와 정적 클래스만을 담은 클래스의 인스턴스화를 막기 위해서는 `private` 생성자를 사용해야 한다.
  - 생성자를 명시하지 않으면 컴파일러가 자동으로 기본 생성자를 만들어 준다.
- 상위 클래스를 호출할 수 없어 상속을 막는 효과도 있다.
