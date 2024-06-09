# Item 7 - 다 쓴 객체 참조를 해제하라

자바는 가비지 컬렉터가 있어서 메모리 관리를 안해도 될 것처럼 보이지만 그렇다고 아예 신경을 끄면 안된다.

스택을 간단히 구현한 예제 코드를 보자

```java
public class Stack {
    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public Stack() {
        elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }

    public void push(Object e) {
        ensureCapacity();
        elements[size++] = e;
    }

    public Object pop() {
        if (size == 0)
            throw new EmptyStackException();
        return elements[--size];
    }

    /**
     * 원소를 위한 공간을 적어도 하나 이상 확보한다.
     * 배열 크기를 늘려야 할 때마다 대략 두 배씩 늘린다.
     */
    private void ensureCapacity() {
        if (elements.length == size)
            elements = Arrays.copyOf(elements, 2 * size + 1);
    }

}
```

겉보기에는 특별한 문제가 없어보이지만 메모리 누수 문제가 있다.

이 스택을 사용하는 프로그램을 오래 실행하다보면 점차 가비지 컬렉션 활동과 메모리 사용량이 늘어나 결국 성능 저하를 일으킨다.

그런데 이 코드에서 메모리 누수는 어디에서 일어날까?

스택이 커졌다가 줄어들 때 스택에서 꺼내진 객체들을 가비지 컬렉터가 회수하지 않기 때문에 일어난다.

위 코드에서는 `elements[--size]` 와 같이 반환값을 해놓으면 실제 값은 삭제되지 않고 인덱스만 한칸씩 이동하기 때문에 메모리 누수가 발생한다고 생각하면 된다.

이 스택이 객체들의 다 쓴 참조(obsolete reference)를 여전히 가지고 있기 때문인데, 여기서 말하는 다 쓴 참조란 문자 그대로 앞으로 다시 쓰지 않을 참조를 뜻한다.

이 코드에서는 elements 배열의 size 를 넘어서는 인덱스를 가졌던 참조들이 해당 된다.

가비지 컬렉션 언어에서는 메모리 누수를 찾기가 아주 까다롭다. 객체 참조 하나를 살려두면 GC는 그 객체뿐만 아니라 그 객체가 참조하는 모든 객체를 회수해가지 못하고 성능에 영향을 줄 수 있다.

### 해결 방안

해결 방안은 해당 참조를 다 썼을 때 null 처리하면 된다.

pop 메서드를 제대로 리팩토링하면 아래 코드와 같다.

```java
public Object pop() {
	if (size == 0) {
		return new EmptyStackException();
	}
	Object result = elements[--size];
	elements[size] = null; // 다 쓴 참조 해제
	return result;
```

다 쓴 참조를 null 처리하면 다른 이점도 따라오는데, 실수로 null 처리한 참조를 사용하려고 하면 프로그램은`NullPointException`을 던지며 조기 종료 시키게 된다.

그렇지만, 모든 객체를 다 쓰자마자 일일이 null 처리하려고 하는데, 그럴 필요가 없다.

### 객체 참조를 null 처리하는 일은 예외적인 경우여야 한다.

참조해제의 가장 좋은 방법은 참조를 담은 변수를 유효 범위 밖으로 밀어내는 것이고 우리가 변수의 범위를 최소가 되게 정의했다면 이 일은 자연스럽게 일어난다.

그렇다면 언제 null 처리를 해야할까?

stack 클래스 처럼 자기 메모리를 직접 관리하는 클래스라면 프로그래머는 항시 메모리 누수에 주의해야한다.

또한 캐시 역시 메모리 누수를 일으키는 주범이다.

이를 해결하기 위해서는 WeakHashMap을 사용해 캐시를 만들면 된다.

다 쓴 엔트리는 그 즉시 자동으로 제거될 것이다.

또는 ScheduledThreadPoolExecutor 같은 백그라운드 스레드를 활용해서 쓰지 않는 엔트리를 청소해주면 된다.

마지막으로 리스너 혹은 콜백도 메모리 누수를 일으킬 수 있다.

클라이언트가 콜백을 등록만하고 명확히 해지하지 않는다면 뭔가 조치해주지 않는 한 콜백은 계속 쌓여가게 될 것이고, 이럴 때 콜백을 약한 참조로 저장하면 가비지 컬렉터가 즉시 수거해가기 때문에 이를 잘 사용하도록 하자.

### 추가)

일반 HashMap 같은 경우에는

```java
public static void main(String[] args) {
        Map<Integer, String> map = new HashMap<>();

        Integer key1 = 1000;
        Integer key2 = 2000;

        map.put(key1, "test a");
        map.put(key2, "test b");

        key1 = null;

        System.gc();

        map.entrySet().stream().forEach(el -> System.out.println(el));
    }
```

이와 같은 코드에서 결과가

```java
2000=test b
1000=test a
```

위와 같이 나오게 되지만, WeakHashMap 을 사용하면

```java
public static void main(String[] args) {
        Map<Integer, String> map = new WeakHashMap<>();

        Integer key1 = 1000;
        Integer key2 = 2000;

        map.put(key1, "test a");
        map.put(key2, "test b");

        key1 = null;

        System.gc();

        map.entrySet().stream().forEach(el -> System.out.println(el));
    }
```

아래처럼 test a 가 제거된 것을 확인할 수 있다.

```java
2000=test b
```