# Item 20 - 추상 클래스보다는 인터페이스를 우선하라

## 인터페이스와 추상 클래스

- 공통점
    - 다중 구현 메커니즘
    - 인스턴스 메서드를 구현 형태로 제공한다
- 차이점
    - 추상 클래스가 정의한 타입을 구현하는 클래스는 반드시 추상 클래스의 하위 클래스가 되어야 한다.
        - 단일 상속만 지원하는 자바에서는 새로운 타입을 정의하는데 큰 제약을 안게 된다.
    - 인터페이스가 선언한 메서드를 모두 정의한 클래스는 어떤 클래스를 상속했든 같은 타입으로 취급된다.

⇒ 인터페이스를 사용하자

## 인터페이스의 장점

1. 기존 클래스에도 손쉽게 새로운 인터페이스를 구현해넣을 수 있다.
2. 인터페이스는 믹스인 정의에 안성맞춤이다.
3. 인터페이스로는 계층구조가 없는 타입 프레임워크를 만들 수 있다.
4. 인터페이스는 기능을 향상시키는 안전하고 강력한 수단이 된다.

## 인터페이스의 디폴트 메서드

인터페이스의 메서드 중 구현 방법이 명백한 것 ⇒ 디폴트 메서드로 제공하자

- 물론 디폴트 메서드에도 제약은 있다.
    - ex. equals, hashCode 등은 디폴트 메서드로 제공해서는 안된다.
    - 인터페이스는 인스턴스 필드를 가질 수 없으며 public이 아닌 정적 멤버도 가질 수 없다.
    - 내가 만들지 않은 인터페이스에는 디폴트 메서드를 추가할 수 없다.

## 인터페이스와 추상클래스의 장점을 모두 취하는 방법?

- 인터페이스와 추상 골격 구현 클래스를 함께 제공하는 식
- 인터페이스로는 타입을 정의하고 필요하면 디폴트 메서드 몇 개도 함께 제공한다.
- 골격 구현 클래스는 나머지 메서드들까지 구현한다.
- 디자인 패턴으로는 템플릿 메서드 패턴이다.

```java
package effectivejava.chapter4.item20;
import java.util.*;

// 코드 20-1 골격 구현을 사용해 완성한 구체 클래스 (133쪽)
public class IntArrays {
    static List<Integer> intArrayAsList(int[] a) {
        Objects.requireNonNull(a);

        // 다이아몬드 연산자를 이렇게 사용하는 건 자바 9부터 가능하다.
        // 더 낮은 버전을 사용한다면 <Integer>로 수정하자.
        return new AbstractList<>() {
            @Override public Integer get(int i) {
                return a[i];  // 오토박싱(아이템 6)
            }

            @Override public Integer set(int i, Integer val) {
                int oldVal = a[i];
                a[i] = val;     // 오토언박싱
                return oldVal;  // 오토박싱
            }

            @Override public int size() {
                return a.length;
            }
        };
    }

    public static void main(String[] args) {
        int[] a = new int[10];
        for (int i = 0; i < a.length; i++)
            a[i] = i;

        List<Integer> list = intArrayAsList(a);
        Collections.shuffle(list);
        System.out.println(list);
    }
}
```

```java
package effectivejava.chapter4.item20;
import java.util.*;

// 코드 20-2 골격 구현 클래스 (134-135쪽)
public abstract class AbstractMapEntry<K,V>
        implements Map.Entry<K,V> {
    // 변경 가능한 엔트리는 이 메서드를 반드시 재정의해야 한다.
    @Override public V setValue(V value) {
        throw new UnsupportedOperationException();
    }
    
    // Map.Entry.equals의 일반 규약을 구현한다.
    @Override public boolean equals(Object o) {
        if (o == this)
            return true;
        if (!(o instanceof Map.Entry))
            return false;
        Map.Entry<?,?> e = (Map.Entry) o;
        return Objects.equals(e.getKey(),   getKey())
                && Objects.equals(e.getValue(), getValue());
    }

    // Map.Entry.hashCode의 일반 규약을 구현한다.
    @Override public int hashCode() {
        return Objects.hashCode(getKey())
                ^ Objects.hashCode(getValue());
    }

    @Override public String toString() {
        return getKey() + "=" + getValue();
    }
}
```

## 결론

- 일반적으로 다중 구현용 타입으로는 인터페이스가 적합하다.
- 복잡한 인터페이스라면 구현하는 수고를 덜기 위해서 골격 구현을 함께 제공하는 방법을 고려하자