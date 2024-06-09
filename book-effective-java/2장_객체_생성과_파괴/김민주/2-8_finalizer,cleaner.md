## 아이템 8. finalizer와 cleaner 사용을 피하라.

- 자바의 객체 소멸자
- finalizer는 예측할 수 없고, 상황에 따라 위험할 수 있다.
- finalizer의 대안인 cleaner은 finalizer보다는 덜 위험하지만 예측할 수 없고, 느리다.

### 위험성

- 즉시 수행됨이 보장되지 않기 때문에 제때 실행되어야 하는 작업에 사용할 수 없다.
  - 자원 회수가 멋대로 지연되기 때문에 심각한 오류를 발생시킬 수 있다.
- finalizer 스레드의 우선순위가 낮고 스레드를 제어할 수 있더라도 백그라운드에서 수행되며 가비지 컬렉터의 통제하에 있으니 즉각 수행될 수 없다.
- finalizer 동작 중 발생한 예외는 무시되며, 처리할 작업이 남았더라도 그 순간 종료된다.
  - 작업이 완료되지 않은 객체가 남아 다른 스레드가 사용할 수 있는 위험이 있다.
  - stack trace를 출력하지 않는다.
- finalizer과 cleaner는 가비지 컬렉터의 효율을 떨어뜨려 성능 문제를 동반한다.
- finalizer 공격에 노출되어 심각한 보안 문제를 일으킬 수 있다.
  - 생성자나 직렬화 과정에서 예외가 발생하면, 이 생성되다 만 객체에서 악의적인 하위 클래스의 finalizer가 수행될 수 있다.

### 해결책

- `AutoCloseable`을 구현하고 클라이언트에서 인스턴스 사용 후 close 메서드를 호출한다.
  - `close` 메서드에서 객체가 유효함을 필드에 기록하고 추적하는 것이 좋다.

### finalizer와 cleaner의 적절한 용도

- 클라이언트가 close 메서드를 호출하지 않는 것에 대비하는 것이다.
- native peer(일반 자바 객체가 네이티브 메서드를 통해 기능을 위임한 네이티브 객체)와 연결된 객체

  - 네이티브 피어는 자바 객체가 아니기 때문에 가비지 컬렉터가 회수할 수 없다.
  - 성능 저하를 감당할 수 있고 네이티브 피어가 심각한 자원을 가지고 있지 않을 때에만 사용한다.
  - 네이티브 피어가 사용하는 자원을 즉시 회수해야할 때는 `close`를 사용한다.

- cleaner를 안정망으로 사용하는 `AutoCloseAble` 클래스

```java
public class Room implements AutoCloseable {
  private static final Cleaner cleaner = Cleaner.create();

  // 청소가 필요한 자원
  // State 클래스에서 Room을 참조하면 순환참조가 생겨서 가비지 컬렉터가 Room 인스턴스를 회수하지 않는다 !!!!!
  private static class State implements Runnable {
    int numJunkPiles; // Room 안의 쓰레기 수

    State(int numJunkPiles) {
      this.numJunkPiles = numJunkPiles;
    }

    // close 메서드나 cleaner가 호출하는 메서드
    @Override
    public void run() {
      numJunkPiles = 0;
    }
  }

  // 방의 상태, cleanable과 공유
  private final State state;
  // cleanable 객체, 수거 대상이 되면 방을 청소
  private final Cleaner.Cleanable cleanable;

  public Room(int numJunkPiles) {
    state = new State(numJunkPiles);
    cleanable = cleaner.register(this, state);
  }

  @Override

}
```
