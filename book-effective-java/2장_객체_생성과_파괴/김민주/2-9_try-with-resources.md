## try-finally보다는 try-with-resources를 사용하라

```java
try (InputStream in = new FileInputStream(src);
    OutputStream out = new FileOutputStream(dst)) {
        byte[] buf = new byte[BUFFER_SIZE];
        ...
} catch (IOException e) {
    ...
}
```

- 자원이 `AutoCloseable`인터페이스를 구현하면 사용할 수 있다.
- 자원 닫기를 클라이언트가 놓칠 때 예상할 수 없는 성능 문제가 발생한다.
- 여러 자원을 닫아야 할 때 간결한 코드와 쉬운 디버깅을 제공한다.
  - 예외는 finally 구문에서도 발생하기 때문에 finally 구문에서 발생된 예외때문에 첫 번째 예외의 스택 추적 내역이 남지 않을 수 있다.
