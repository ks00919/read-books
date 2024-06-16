# Item 12 : toString을 항상 재정의하라

- toString을 잘 구현한 클래스는 디버깅하기 쉽다.
- 그 객체가 가진 주요 정보를 모두 반환하는게 좋다.

# Item 13 : clone 재정의는 주의해서 진행하라

- clone 메서드가 선언된 곳은 Clonable이 아닌 Object이고 , 그마저도 protected이다.
  -> 단순히 Clonable만 구현해서는 안되고 Clonable 인터페이스를 구현 하고 clone 메서드를 오버라이딩 후 사용해야한다.
  -> 하지만 여러 문제가 생긴다.
  -> Clonable 인터페이스를 구현한 클래스가 불변 객체가 아닌 가변 객체를 참조할 경우 복제 이후 문제가 생긴다
  -> 해결법으로 가변 객체에 clone을 재귀적으로 호출한다.
  -> 하지만 이 경우에도 가변 객체의 필드가 final일 경우는 통하지 않는다.

- 복사 생성자와 복사 팩터리를 사용하자.
- 배열은 clone을 사용하자.

- 새로운 인터페이스를 만들 때는 Clonable을 확장해서는 안되며, 이를 구현해서도 안된다.

# Item 14 : Comparable을 구현할지 고려하라

- compareTo는 Object의 메서드가 아니다. Comparable 인터페이스의 메서드이다.
- Comparable을 구현한 객체들의 배열은 Arrays.sort()로 정렬할 수 있다.

1. 두 객체 참조의 순서를 변경해 비교해도 예상한 결과가 나와야한다.
2. 첫 번째가 두 번째보다 크고 두 번째가 세 번째보다 크다면 첫 번째는 세 번째보다 커야한다.
3. 크기가 같은 객체들 끼리는 어떤 객체와 비교하더라도 항상 같아야 한다.
4. 필수는 아니지만 compareTo 메서드로 수행한 동치성 테스트가 equals의 결과와 일관되어야한다.

- compareTo 메서드에서 필드 값을 비교할 때 "<"와 ">" 연산자는 쓰지 말아야한다.
- Comparator 인터페이스가 가지고 있는 비교자 생성 메서드를 사용할 수 있다.

```java
private static final Comparator<PhoneNumber> COMPARATOR = comparingInt((PhoneNumber) pn->pn.areaCode).thenComparingInt(pn->pn.prefix).
themComparingInt(pn->pn.lineNum);

public int compareTo(PhoneNumber pn){
    return COMPARATOR.compare(this,pn);
}
```
