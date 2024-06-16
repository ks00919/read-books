# Item 13 - clone 재정의는 주의해서 진행하라

### Cloneable은 의도한 목적을 제대로 이루지 못했다.

- clone 메서드가 선언된 곳이 cloneable이 아닌 Object이고
- 접근 제한자가 protected 이다.
- 그래서 Cloneable 을 구현하는 것만으로는 외부 객체에서 clone 을 호출할 수 없다.
    - Cloneable 인터페이스는 빈 인터페이스라서 clone 메서드를 따로 오버라이딩 해주어야 한다.

### 그렇다면 Cloneable은 무슨 일을 할까?

- Object의 protected 메서드인 clone의 동작 방식을 결정한다.
- Cloneable을 구현한 클래스의 인스턴스에서 clone 을 호출하면 그 객체의 필드들을 하나하나 복제한 객체를 반환한다.
- Cloneable을 구현하지 않은 클래스의 인스턴스에서 호출하면 CloneNotSupportedException을 던진다.

### Clone 메서드의 일반 규약은 허술하다

Object 명세에서 가져온 다음 내용을 보자

- 어떤 객체 x 에 대해서 다음 식은 참이다

```java
 x.clone() != x
```

- 또한 다음식도 참이다

```java
x.clone().getClass() == x.getClass()
```

- 다음식도 일반적으로는 참이다

```java
x.clone().equals(x)
```

그런데, 위 내용 모두 반드시 만족해야 하는 것은 아니라고 한다…

### 클래스의 필드가 불변이라면

필드가 불변이라면 clone 메서드를 아래 코드와 같이 구현하면 된다.

```java
    @Override public PhoneNumber clone() {
        try {
            return (PhoneNumber) super.clone();
        } catch (CloneNotSupportedException e) {
            throw new AssertionError();  // 일어날 수 없는 일이다.
        }
    }
```

그렇지만, 쓸데없는 복사를 지양한다는 관점에서 보면 불변 클래스는 굳이 clone 메서드를 제공하지 않는 것이 좋다.

### 그런데 클래스가 가변 객체를 참조한다면?

다음과 같은 클래스를 복제할 수 있도록 만든다고 했을 때

```java
public class Stack {
    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public Stack() {
        this.elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }

    public void push(Object e) {
        ensureCapacity();
        elements[size++] = e;
    }
    
    public Object pop() {
        if (size == 0)
            throw new EmptyStackException();
        Object result = elements[--size];
        elements[size] = null; // 다 쓴 참조 해제
        return result;
    }
    
    // 원소를 위한 공간을 적어도 하나 이상 확보한다.
    private void ensureCapacity() {
        if (elements.length == size)
            elements = Arrays.copyOf(elements, 2 * size + 1);
    }
}
```

clone 메서드를 단순히 super.clone을 반환하도록 하면 반환된 Stack 인스턴스의 size 필드는 올바른 값을 갖겠지만, elements 필드는 원본 Stack 인스턴스와 똑같은 배열을 참조하게 되어, 원본이나 복제본 중 하나를 수정하면 다른 하나도 수정되어 불변식을 해치게 될 것이다.

만약 Stack 클래스의 하나뿐인 생성자를 호출한다면 이런일은 일어나지 않을 것이다.

**clone 메서드는 사실상 생성자와 같은 효과를 내기 때문에, clone 은 원본 객체에 아무런 해를 끼치지 않는 동시에 복제된 객체의 불변식을 보장해야 한다.**

우리가 기대하는 clone 메서드는 스택 내부 정보도 다 복사해야 한다.

elements 배열의 clone 을 아래 코드처럼 재귀적으로 호출하면 된다.

```java
    // 코드 13-2 가변 상태를 참조하는 클래스용 clone 메서드
    @Override public Stack clone() {
        try {
            Stack result = (Stack) super.clone();
            result.elements = elements.clone();
            return result;
        } catch (CloneNotSupportedException e) {
            throw new AssertionError();
        }
    }
```

### clone() 을 재귀적으로 호출하는 것만으로는 충분하지 않을 때도 있다

해시테이블용 clone을 생각해보자

복제본은 자신만의 버킷 배열을 갖지만, 이 배열은 원본과 같은 연결 리스트를 참조하여 원본과 복제본 모두 예기치 않게 동작할 수 있다.

그럴때는 deep copy를 이용한다.

```java
    public Entry deepCopy() {
        return new Entry(key, value, next == null ? null : next.deepCopy());
    }
    
    @Override
    public CustomHashTable clone() {
        try {
            CustomHashTable result = (CustomHashTable) super.clone();
            result.buckets = new Entry[buckets.length];

            for (int i = 0; i < buckets.length; i++) {
                if (buckets[i] != null)
                    result.buckets[i] = buckets[i].deepCopy();
            }

            return result;
        } catch (final CloneNotSupportedException e) {
            throw new AssertionError();
        }
    }
```

이렇게 해주면 잘 작동하지만, 연결 리스트의 원소 수가 너무 많으면 스택 오버플로우를 일으킬 수 있기 때문에 deepCopy를 재귀 호출 대신 반복자를 써서 순회하는 방향으로 수정하는 것이 좋다.

```java
    public Entry deepCopy() {
        Entry result = new Entry(key, value, next);
        for (Entry p = result; p.next != null; p = p.next) {
            p.next = new Entry(p.next.key, p.next.value, p.next.next);
        }
        return result;
    }
```

### 고수준 API를 이용한 복제

- put 등의 메서드를 호출해서 똑같이 만들어주면 된다.
- 코드가 깔끔하지만 상대적으로 느리다

### 주의할 점

- 생성자에서 재정의될 수 있는 메서드를 호출하면 안되는 것처럼 clone도 마찬가지이다
    - clone이 하위 클래스에서 재정의한 메서드를 호출하면 하위 클래스는 복제과정에서 원본과 복제본의 상태가 달라질 수 있다.
- clone 메서드는 public 으로 하고, throws 절은 없애야 한다
    - 그래야 사용하기 편하기 때문이다.

### 요약

- Cloneable을 구현하는 모든 클래스는 clone을 재정의해야 한다.
- 접근자는 public, 반환 타입은 클래스 자신으로 변경한다.

배열은 clone 메서드 방식이 깔끔하지만 대부분은 복사 생성자와 복새 팩토리를 사용하자.

## 복사 생성자와 복사 팩토리가 더 나은 객체 복사 방식을 제공한다.

복사 생성자와 복사 팩토리를 코드로 보면 아래와 같다.

- 복사 생성자

```java
public class Stack {
    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public Stack(Stack s) {
        this.elements = s.elements.clone();
        this.size = s.size;
    }
}
```

- 복사 팩토리

```java
public class Stack {
    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public static Stack newInstance(Stack s) {
        return new Stack(s.elements, s.size);
    }
}
```