# Item 9 - try-finally 보다는 try-with-resources를 사용하라

자바 라이브러리 중에는 close 메서드를 호출해 직접 닫아주어야 하는 자원이 많다.

`InputStream`, `OutputStream`, `java.sql.Connection` 등이 있다.

자원 닫기는 클라이언트가 놓치기 쉬워서 예측할 수 없는 성능 문제로 이어지기도 하니 주의해야한다.

이런 자원 중 상당수가 안전망으로 finalizer를 활용하고는 있지만, 앞 Item 8 에서도 설명했듯, finalizer는 그리 믿을만 하지 못하다.

보통 나는 지금까지 close 를 사용하면 아래 코드처럼 사용하였다.

```java
BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
String s = br.readLine();
br.close();
```

그렇지만 위처럼 사용하면, 문제가 생길 수 있다.

`BufferedReader` 사용 중 `IOException` 이 발생하게 되면, 메서드가 종료되므로 `close`가 호출되지 않고 스트림이 메모리에 남아있게 되기 때문에, 예외가 발생되더라도 자원을 닫을 수 있도록 전통적으로는 try-finally 문을 사용해서 `close` 처리를 해주었다.

```java
BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
try {
    return br.readLine();
} finally {
    br.close();
}
```

이렇게 코드를 작성해주면 `IOException`이 발생하게 되어도 `finally` 에서 close를 사용해 자원 사용을 종료하게 된다.

### 그런데 만약, close를 여러번 호출해야 하는 상항이 오면?

```java
static void copy(String src, String dst) throws IOException {
	InputStream in = new FileInputStream(src);
	try {
		OutputStream out = new FileOutputStream(dst);
		try {
			byte[] buf = new byte[BUFFER_SIZE];
			int n;
			while ((n = in.read(buf)) >= 0)
				out.write(buf, 0, n);
		} finally {
			out.close();
	} finally {
		in.close();
	}
}
```

위처럼 코드가 너무 지저분해지는 문제가 생긴다.

게다가 `out.close()`에서 예외가 발생하면 `in.close()`가 호출되지 않을 수 있고, 이는 메모리 누수를 발생시킬 수 있다.

그렇기 때문에 java 7 이상에서 사용할 수 있는 `try-with-resources` 를 사용하도록 하자.

이 구조를 사용하려면 해당 자원이 AutoCloseable 인터페이스를 구현해야 한다.

책의 예시를 보면

```java
import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.IOException;
import java.io.InputStream;
import java.io.OutputStream;

public class FileCopyExample {
    private static final int BUFFER_SIZE = 1024;

    static void copy(String src, String dst) throws IOException {
        try (InputStream in = new FileInputStream(src);
             OutputStream out = new FileOutputStream(dst)) {
            byte[] buf = new byte[BUFFER_SIZE];
            int n;
            while ((n = in.read(buf)) >= 0) {
                out.write(buf, 0, n);
            }
        }
    }

    public static void main(String[] args) {
        try {
            copy("source.txt", "destination.txt");
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

`FileInputStream`과 `FileOutputStream`은 모두 `AutoCloseable` 인터페이스를 구현하고 있기 때문에`try-with-resources` 구문에서 사용할 수 있다.

> try-with-resources 구문
`try-with-resources` 구문은 `try` 블록이 끝난 후 자동으로 `close()` 메서드를 호출해주는 구문
> 

이렇게 작성하는 것이 짧고 읽기 수월하기도 하고 문제를 진단하기에도 훨씬 좋다.

만약 위의 예시에서 readLine과 close에서 예외가 발생한다고 하면

- close 에서 발생한 예외는 숨겨지고 readLine 에서 발생한 예외가 기록된다.
    - 이처럼 실전에서는 프로그래머에게 보여줄 예외 하나만 보존되고 여러 개의 다른 예외가 숨겨질수도 있다.
    - 숨겨진 예외들은 버려지지않고 스택 추적 내역에 숨겨졌다(suppressed)는 꼬리표를 달고 출력된다.

또한 try-with-resources 에서도 catch 절을 쓸 수 있기 때문에 try 문을 더 중첩하지 않고도 다수의 예외를 처리할 수 있다.

```java
static String firstLineOfFile(String path, String defaultVal) {
	try (BufferedReader br = new BufferedReader(
			new FileReader(path))) {
		return br.readLine();
	} catch (IOException e) {
		return defaultVal;
	}
}
```