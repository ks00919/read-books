# Item 8 : finalizer와 cleaner 사용을 피하라

### 자바에서 객체를 소멸하는 방법

1. finalizer
2. cleaner

→ 예측할 수 없고 제때 실행되어야하는 작업에 사용할 수 없다.

→ 상태를 영구적으로 수정하는 작업에서는 절대 finalizer와 cleaner에 의존해서는 안된다.

#### 파일이나 스레드 등을 종료해야 할 자원을 담고 있는 객체의 클래스에서 finalizer나 cleaner를 대신하는법

→ AutoCloseable을 구현하고 클라이언트에서 인스턴스를 다 쓰고 나면 close 메서드를 호출한다.

# Item 9 : try-finally보다는 try-with-resource를 사용하라

- 자원이 둘 이상이면 try-finally 방식은 지저분해지고 디버깅이 어려워진다.

```java
//try-with-resource 예시
try(InputStream in = new FileInputStream(src);
		OutputStream out = new FileOutputStream(dst);){
			byte[] buf = new Byte[BUUFER_SIZE];
			int n;
			while((n = in.read(buf)>=0) out.write(buf,0,n)
 		}
```
