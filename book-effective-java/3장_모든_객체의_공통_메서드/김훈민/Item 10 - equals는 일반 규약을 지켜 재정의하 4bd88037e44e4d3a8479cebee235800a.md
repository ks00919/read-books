# Item 10 - equals는 일반 규약을 지켜 재정의하라

equals 메서드는 재정의하기 쉬워 보이지만 자칫하면 끔찍한 결과를 초래하기 때문에 아래 상황 중 하나에 해당된다면 재정의 하지 않는 것이 최선이다.

## 1. 각 인스턴스가 본질적으로 고유하다

값을 표현하는 것이 아니라 동작하는 개체를 표현하는 클래스가 여기에 해당한다.

예시로는 Thread가 있다.

Object의 equals 메서드는 이러한 클래스에 딱 맞게 구현되었다.

두 개의 서로 다른 스레드가 동일한 작업을 수행하더라도, 이들은 서로 다른 실행 경로와 리소스를 가지므로 동일한 객체로 간주해서는 안된다. 그렇기 때문에 이러한 경우에는 equals 메서드를 재정의 하지 않는 것이 좋다.

## 2. 인스턴스의 논리적 동치성(logical equality)을 검사할 일이 없다.

equals를 재정의하여 각 인스턴스가 같은 정규표현식을 나타내는지 검사할 수 있지만, 그럴 일이 없다면 굳이 재정의 할 필요가 없다.

## 3. 상위 클래스에서 재정의한 equals가 하위 클래스에도 딱 들어맞는다.

대부분의 Set 구현체는 AbstractSet이 구현한 equals를 상속 받는 것처럼 같은 Set인지 구분하기 위해서 사이즈, 내부 값들을 비교하면 되기 때문에 굳이 중간에 바꾸지 않고 그대로 쓴다.

## 4. 클래스가 private이거나 package-private이고 equals 메서드를 호출할 일이 없다.

equals 자체를 호출하는 일을 막고 싶다면 아래 코드를 사용해서 구현하자.

```java
@Override
public boolean equals (Object object) {
	throw new AssertionError();
}
```

그렇다면 equals를 재정의할때는 언제일까? 객체 식별성(두 객체가 물리적으로 같은가?)이 아니라 논리적 동치성을 확인해야 하는데, 상위 클래스의 equals가 논리적 동치성을 비교하도록 재정의되지 않았을 때다.

equals 메서드를 재정의할때는 반드시 일반 규약을 따라야 하는데, 아래는 Object 명세에 적힌 규약이다.

> equals 메서드는 동치관계(equivalence relation)을 구현하며, 다음을 만족해야한다.
null이 아닌 모든 참조 값 x, y, z 에 대해서

1. 반사성(reflexivity) : x.equals(x) == true
2. 대칭성(symmetry) : x.equals(y) == true 이면 y.equals(x) == true 이다.
3. 추이성(transitivity) : x.equals(y) == true 이고 y.equals(z) == true 이면, x.equals(z)도 true 이다.
4. 일관성(consistency) : x.equals(y)를 반복해서 호출하면 항상 true 이거나 항상 false 여야한다.
5. null-아님 : null이 아닌 모든 참조 값 x에 대해, x.equals(null)은 false이다.
> 

equals 규약을 어기면 그 객체를 사용하는 다른 객체들이 어떻게 반응할지 알수없기 때문에 주의해서 사용하도록 하자.

책에서는 각 단계별로 예시를 들며 equals 메서드를 재정의 할때 왜 일반 규약을 반드시 따라야 하는지 설명하고 있다.

그렇다면 양질의 equals 메서드 구현 방법은 무엇일까?

1. == 연산자를 사용해 입력이 자기 자신의 참조인지 확인한다.
    1. 성능 최적화를 위한 단계로, 비교 작업이 복잡한 상황일때 값어치를 하는 단계이다.
2. instanceof 연산자로 입력이 올바른 타입인지 확인한다.
    1. 여기서 말하는 올바른 타입은 equals가 정의된 클래스를 의미한다.
    2. 가끔은 그 클래스가 구현한 특정 인터페이스가 될 수도 있다.
3. 입력을 올바른 타입으로 형변환한다.
    1. 위에서 instanceof 검사를 했기 때문에 100% 성공이 보장된다.
4. 입력 객체와 자기 자신의 대응되는 핵심 필드들이 모두 일치하는지 하나씩 검사한다

위 단계를 통해서 equals 메서드를 다 구현했다면 세 가지만 자문해보자.

1. 대칭적인가?
2. 추이성이 있는가?
3. 일관적인가?

그 이후에 단위 테스트도 작성해 돌려보도록하자.

이 단계들을 거친 이후에 완성된 equals 메서드이다.

```java
package effectivejava.chapter3.item10;

// 코드 10-6 전형적인 equals 메서드의 예 (64쪽)
public final class PhoneNumber {
    private final short areaCode, prefix, lineNum;

    public PhoneNumber(int areaCode, int prefix, int lineNum) {
        this.areaCode = rangeCheck(areaCode, 999, "지역코드");
        this.prefix   = rangeCheck(prefix,   999, "프리픽스");
        this.lineNum  = rangeCheck(lineNum, 9999, "가입자 번호");
    }

    private static short rangeCheck(int val, int max, String arg) {
        if (val < 0 || val > max)
            throw new IllegalArgumentException(arg + ": " + val);
        return (short) val;
    }

    @Override public boolean equals(Object o) {
        if (o == this)
            return true;
        if (!(o instanceof PhoneNumber))
            return false;
        PhoneNumber pn = (PhoneNumber)o;
        return pn.lineNum == lineNum && pn.prefix == prefix
                && pn.areaCode == areaCode;
    }

    // 나머지 코드는 생략 - hashCode 메서드는 꼭 필요하다(아이템 11)!
}
```

그리고 equals 를 재정의할 땐 반드시 hashCode도 재정의하자.

아래는 eqauls 메서드를 재정의할때 주의해야할 주의점이다.

1. float 와 double 을 제외한 기본 타입 필드는 == 연산자로 비교하자.
2. float 와 double은 Float.compare() , Double.compare() 로 비교하자.
3. 참조 타입 필드는 각각의 equals 메서드로 비교하자.
4. 어떤 필드를 먼저 비교하느냐가 equals의 성능을 좌우하기도 한다
    1. 다를 가능성이 더 크거나, 비교하는 비용이 싼 필드를 먼저 비교하자
5. 동기화용 락(lock) 필드 같이 객체의 논리적 상태와 관련없는 필드는 비교하면 안된다.
6. Object 외의 타입을 매개변수로 받는 equals 메서드는 선언하지 말자.