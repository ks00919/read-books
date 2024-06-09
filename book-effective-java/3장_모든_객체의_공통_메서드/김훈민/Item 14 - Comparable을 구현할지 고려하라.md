# Item 14 - Comparable을 구현할지 고려하라

comparable 을 구현한 객체들의 배열은 다음과 같이 손쉽게 정렬할 수 있다.

```java
Arrays.sort(a);
```

comparable 을 구현하면 이 인터페이스를 활용하는 수많은 제네릭 알고리즘과 컬렉션의 힘을 누릴 수 있기 때문에, 알파벳, 숫자, 연대 같이 순서가 명확한 값 클래스를 작성한다면 반드시 Comparable 인터페이스를 구현하자.

### compareTo 메서드에서 관계 연산자 < 와 > 은 추천하지 않는다

자바 7 부터는 박싱된 기본 타입 클래스들에 새로 추가된 정적 메서드 compare을 이용하도록 하자.

클래스에 핵심 필드가 여러개라면 어느 것을 먼저 비교하느냐가 중요해진다. 다음의 코드를 보자.

```java
    // 코드 14-2 기본 타입 필드가 여럿일 때의 비교자 (91쪽)
    public int compareTo(PhoneNumber pn) {
        int result = Short.compare(areaCode, pn.areaCode);
        if (result == 0)  {
            result = Short.compare(prefix, pn.prefix);
            if (result == 0)
                result = Short.compare(lineNum, pn.lineNum);
        }
        return result;
    }
```

그리고 자바 8에서는 Comparator 인터페이스가 일련의 비교자 생성 메서드와 팀을 꾸려 메서드 연쇄 방식으로 비교자를 생성할 수 있게 되었다.

그런데 이 방식을 이용하면 약간의 성능 저하가 있다. 10% 정도 느려진다고 한다.

```java
    // 코드 14-3 비교자 생성 메서드를 활용한 비교자 (92쪽)
    private static final Comparator<PhoneNumber> COMPARATOR =
            comparingInt((PhoneNumber pn) -> pn.areaCode)
                    .thenComparingInt(pn -> pn.prefix)
                    .thenComparingInt(pn -> pn.lineNum);

    public int compareTo(PhoneNumber pn) {
        return COMPARATOR.compare(this, pn);
    }

```

### 해시코드의 값의 차를 비교해야한다면?

나는 지금까지 값의 차를 비교하는 방식으로 compare을 구현해왔다.

```java
static Comparator<Object> hashCodeOrder = new Comparator<>() {
    @Override
    public int compare(Object o1, Object o2) {
        return o1.hashCode() - o2.hashCode();
    }
};
```

그런데 이 방식은 정수 오버플로우를 일으키거나 부동소수점 계산 방식에 따른 오류를 낼 수 있기 때문에 아래 두 방식 중 하나를 사용하자

1. 정적 compare 메서드를 활용한 비교자

```java
static Comparator<Object> hashCodeOrder = new Comparator<>() {
    @Override
    public int compare(Object o1, Object o2) {
        return Integer.compare(o1.hashCode(), o2.hashCode());
    }
};
```

1. 비교자 생성 메서드를 활용한 비교자

```java
static Comparator<Object> hashCodeOrder = Comparator.comparingInt(o -> o.hashCode());
```