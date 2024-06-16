# Item 22 - 인터페이스는 타입을 정의하는 용도로만 사용하라

- 인터페이스는 자신을 구현한 클래스의 인스턴스를 참조할 수 있는 타입 역할을 한다.
    - 즉, 클래스가 어떤 인터페이스를 구현한다는 것은 자신의 인스턴스로 무엇을 할 수 있는지 클라이언트에게 알려주는 것이다.
- **인터페이스는 오직 이 용도로만 사용해야 한다.**

위 지침에 맞지 않는 예시가 있다.

```java
public interface PhysicalConstants {

    static final double AVOGADROS_NUMBER   = 6.022_140_857e23;
    
    static final double BOLTZMANN_CONSTANT = 1.380_648_52e-23;
    
    static final double ELECTRON_MASS      = 9.109_383_56e-31;
    
}
```

- 상수 인터페이스
    - 메서드 없이 static final 필드로만 가득찬 인터페이스
- 상수 인터페이스 안티패턴
    - 인터페이스를 잘못 사용한 예시이다.
    - 내부 구현을 클래스의 API로 노출하는 행위이다.
    - 사용자에게 혼란을 준다
    - 상수들을 더는 쓰지 않게 되더라도 바이너리 호환성을 위해 여전히 상수 인터페이스를 구현하고 있어야 한다.

### 그렇다면 어떻게 상수를 공개해야할까?

좋은 예시로 Integer의 MIN_VALUE와 MAX_VALUE가 있다.

```java
package effectivejava.chapter4.item22.constantutilityclass;

// 코드 22-2 상수 유틸리티 클래스 (140쪽)
public class PhysicalConstants {
  private PhysicalConstants() { }  // 인스턴스화 방지

  // 아보가드로 수 (1/몰)
  public static final double AVOGADROS_NUMBER = 6.022_140_857e23;

  // 볼츠만 상수 (J/K)
  public static final double BOLTZMANN_CONST  = 1.380_648_52e-23;

  // 전자 질량 (kg)
  public static final double ELECTRON_MASS    = 9.109_383_56e-31;
}
```

인스턴스화를 막고 유틸리티 클래스로 제공하자!