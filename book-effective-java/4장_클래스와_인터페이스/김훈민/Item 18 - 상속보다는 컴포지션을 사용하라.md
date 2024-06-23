# Item 18 - 상속보다는 컴포지션을 사용하라

## 상속은 코드를 재사용하는 강력한 수단이지만, 항상 최선은 아니다.

인터페이스 상속을 말하는 것이 아닌 클래스가 다른 클래스를 확장하는 구현 상속을 의미한다.

### 그 이유는 메서드 호출과 달리 상속은 캡슐화를 깨뜨리기 때문이다.

- 상위 클래스의 구현이 변경되면 하위 클래스의 동작에 이상이 생길 수 있다.
    - 상위 클래스는 릴리스마다 내부 구현이 달라질 수 있다.
    - 그 여파로 코드 한 줄 건드리지 않은 하위 클래스가 오동작할 수 있다.
- 상위클래스 설계자가 확장을 충분히 고려하고 문서화도 제대로 해두지 않으면 하위 클래스는 상위 클래스가 변경되면 같이 변경되어야 한다.

## 상속을 잘못 사용한 예시

```java
public class InstrumentedHashSet<E> extends HashSet<E> {
	private int addCount = 0;

	public InstrumentedHashSet(int initCap, float loadFactor) {
		super(initCap, loadFactor);
	}

	@Override 
  public boolean add(E e) {
		addCount++;
		return super.add(e);
	}

	@Override 
  public boolean addAll(Collection<? extends E> c) {
		addCount += c.size();
		return super.addAll(c);
	}

	public int getAddCount() {
		return addCount;
	}
}
```

아래의 코드를 통해 이 클래스의 인스턴스에 addAll 메서드로 원소 3개를 더했다고 해보자

```java
InstrumentedHashSet<String> s = new InstrumentedHashSet<>();
s.addAll(List.of("틱", "탁탁", "펑"));
```

이제 getAddCount 메서드를 호출하면 3을 반환할거라 생각하겠지만 실제로는 6을 반환하게 된다.

## 어디가 잘못된 걸까?

HashSet의 addAll 메서드가 add를 사용해 구현되었기 때문이다.

그런데 이 add는 InstrumentedHashSet에서 Override했기 때문에 InstrumentedHashSet의 add가 호출되고 그 add는 super.add를 다시 호출하게 되어 결국 addCount에는 값이 중복되어 더해지게 되어 최종값이 6이 되는 것이다.

## 그렇다면 어떻게 해결해야할까?

이 예제의 경우에는, addAll 메서드를 재정의하지 않거나, 다른 식의 재정의를 통해서 문제를 해결할 수 있다.

1. addAll 메서드를 재정의하지 않는 경우
    1. 당장은 제대로 동작하지만, HashSet의 addAll이 add메서드를 이용해 구현했음을 가정했기 때문에 이 해법이 가능한 것이다.
    2. 다음 릴리스에도 이는 보장될지 알수가 없어서 깨지기 쉽다.
2. addAll 메서드를 다른 식으로 재정의 하는 경우 (주어진 컬렉션을 순회하며 원소 하나당 add 메서드를 한 번만 호출하는 경우)
    1. 이 방식은 HashSet의 addAll을 더 이상 호출하지 않으니 addAll이 add를 사용하는지와 상관없이 결과가 제대로 나온다는 점에서는 조금 더 나은 해법이긴 하다
    2. 그러나, 여전히 문제가 있다.
        1. 상위 클래스의 메서드 동작을 다시 구현하는 방식은 어렵다
        2. 시간도 더 많이 든다.
        3. 오류를 내거나 성능을 떨어뜨릴 수도 있다.
        4. 만약 private 필드를 상위 클래스에서 사용하고 있다면 구현 자체가 불가능하다

## 그렇다면 재정의를 하지 않고 새로운 메서드를 추가하면 되지 않을까?

이 방식이 훨씬 안전하긴하지만, 위험이 없지는 않다.

만약 상위클래스에서 새 메서드가 추가되었는데 운 없게도 하필 우리가 추가한 메서드와 시그니처가 같고 반환 타입은 다르다면..? ⇒ 컴파일조차 되지 않을 것이다.

## 그렇다면 어떻게?

기존 클래스를 확장하지 말고, 새로운 클래스를 만들고 private 필드로 기존 클래스의 인스턴스를 참조하게 하자!

⇒ 이러한 방식을 기존 클래스가 새로운 클래스의 구성 요소로 쓰인다는 뜻에서 컴포지션이라고 한다.

```java
package effectivejava.chapter4.item18;
import java.util.*;

// 코드 18-2 래퍼 클래스 - 상속 대신 컴포지션을 사용했다. (117-118쪽)
public class InstrumentedSet<E> extends ForwardingSet<E> {
    private int addCount = 0;

    public InstrumentedSet(Set<E> s) {
        super(s);
    }

    @Override public boolean add(E e) {
        addCount++;
        return super.add(e);
    }
    @Override public boolean addAll(Collection<? extends E> c) {
        addCount += c.size();
        return super.addAll(c);
    }
    public int getAddCount() {
        return addCount;
    }

    public static void main(String[] args) {
        InstrumentedSet<String> s = new InstrumentedSet<>(new HashSet<>());
        s.addAll(List.of("틱", "탁탁", "펑"));
        System.out.println(s.getAddCount());
    }
}
```

```java
package effectivejava.chapter4.item18;
import java.util.*;

// 코드 18-3 재사용할 수 있는 전달 클래스 (118쪽)
public class ForwardingSet<E> implements Set<E> {
    private final Set<E> s;
    public ForwardingSet(Set<E> s) { this.s = s; }

    public void clear()               { s.clear();            }
    public boolean contains(Object o) { return s.contains(o); }
    public boolean isEmpty()          { return s.isEmpty();   }
    public int size()                 { return s.size();      }
    public Iterator<E> iterator()     { return s.iterator();  }
    public boolean add(E e)           { return s.add(e);      }
    public boolean remove(Object o)   { return s.remove(o);   }
    public boolean containsAll(Collection<?> c)
                                   { return s.containsAll(c); }
    public boolean addAll(Collection<? extends E> c)
                                   { return s.addAll(c);      }
    public boolean removeAll(Collection<?> c)
                                   { return s.removeAll(c);   }
    public boolean retainAll(Collection<?> c)
                                   { return s.retainAll(c);   }
    public Object[] toArray()          { return s.toArray();  }
    public <T> T[] toArray(T[] a)      { return s.toArray(a); }
    @Override public boolean equals(Object o)
                                       { return s.equals(o);  }
    @Override public int hashCode()    { return s.hashCode(); }
    @Override public String toString() { return s.toString(); }
}
```

이렇게 컴포지션을 이용할 수 있다.

## 상속은 그럼 어떨때?

- 상속은 하위 클래스가 상위 클래스의 진짜 하위 타입인 상황에서만 쓰여야 한다.
- 클래스 B가 클래스 A와 is-a 관계일 때만 상속해야한다.