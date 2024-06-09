# Item 8 - finalizer와 cleaner 사용을 피하라

자바에는 두가지의 객체 소멸자가 있다.

## finalizer

- 예측할 수 없고 상황에 따라 위험할 수 있어 일반적으로 불필요하다.
- 자바 9부터 deprecated API로 지정하고 그 대안으로 cleaner 를 소개하고 있다.

## cleaner

- finalizer보다는 덜 위험하지만, 여전히 예측할 수 없고 느리며 불필요하다

## 그 이유는?

### 1. finalizer 와 cleaner는 즉시 수행된다는 보장이 없다.

즉, finalizer 와 cleaner 로는 제때 실행되어야 하는 작업은 절대 할 수 없다.

예를들어, 파일 닫기를 finalizer나 cleaner에 맡기면 중대한 오류를 일으킬 수 있다.

시스템이 finalizer나 cleaner 실행을 게을리해서 파일을 계속 열어둔 채 방치한다면 파일을 무한히 열 수 없기 때문에 새로운 파일을 열지 못해 프로그램이 실패할 수 있다.

finalizer나 cleaner를 얼마나 신속해 수행할지는 GC 알고리즘에 달렸다.

자바 언어 명세는 finalizer나 cleaner의 수행 시점, 수행 여부도 보장하지 않기 때문에 상태를 영구적으로 수정하는 작업에서는 절대 finalizer나 cleaner에 의존해서는 안된다.

### 2. finalizer 와 cleaner 는 심각한 성능 문제를 동반한다.

AutoCloseable 객체를 생성하고 GC가 수거하기 까지 12ns 가 걸렸지만

finalizer를 사용하니 550ns가 걸렸다.

### 3. finalizer를 사용한 클래스는 finalizer 공격에 노출되어 심각한 보안 문제를 일으킬 수도 있다.

## 그렇기 때문에, AutoCloseable을 사용하자

AutoCloseable 을 구현해주고 클라이언트에서 인스턴스를 다 쓰고 나면 close 메서드를 호출하면 된다.

```java
public class Resource implements AutoCloseable {
    public void doSomething() {
        System.out.println("Doing something with the resource");
    }

    @Override
    public void close() {
        System.out.println("Resource has been closed");
    }

    public static void main(String[] args) {
        // Using the resource with try-with-resources
        try (Resource resource = new Resource()) {
            resource.doSomething();
        }
    }
}
```

그리고 위 코드처럼 try-with-resources를 사용해서 예외가 발생되어도 제대로 종료되도록 해야한다.