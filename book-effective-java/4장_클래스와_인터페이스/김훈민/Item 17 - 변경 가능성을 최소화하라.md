# Item 17 - 변경 가능성을 최소화하라

## 불변 클래스?

- 그 인스턴스의 내부 값을 수정할 수 없는 클래스이다.
- 불변 인스턴스에 간직된 정보는 고정되어 객체가 파괴되는 순간까지 절대 달라지지 않는다.
- 자바 플랫폼 라이브러리에도 다음과 같은 불변 클래스가 있다.
    - String, 기본 타입의 박싱된 클래스들, BigInteger 등..
    - 이 클래스들을 불변으로 설계한 건 이유가 있다.
- 불변 클래스는 가변 클래스보다 설계하고 구현하고 사용하기 쉽다.
- 오류가 생길 여지도 적고 안전하다

## 클래스를 불변으로 만들려면 다음 다섯가지 규칙을 따르자

- 객체의 상태를 변경하는 메서드를 제공하지 않음 (예. setter)
- 클래스를 확장할 수 없도록 함
    - 하위 클래스에서 객체가 변경될 수 있기 때문
    - final 키워드를 사용하자
- 모든 필드를 final 로 선언
    - 여러 스레드에서 접근해도 값이 바뀌지 않는다
    - 다중 스레드 환경에서도 안전하다
- 모든 필드를 private 로 선언
- 자신 외에는 내부의 가변 컴포넌트에 접근할 수 없도록 함.
    - 클라이언트에서 가변 객체를 참조하는 필드의 참조를 얻으면 안된다
    - 서버에서도 접근자 메서드로 가변 객체를 참조하는 필드값을 반환하면 안된다
    - 생성자, 접근자, readObject 메서드 모두에서 방어적 복사를 수행해야한다

## 불변 복소수 클래스 예시

```java
package effectivejava.chapter4.item17;

// 코드 17-1 불변 복소수 클래스 (106-107쪽)
public final class Complex {
    private final double re;
    private final double im;

    public static final Complex ZERO = new Complex(0, 0);
    public static final Complex ONE  = new Complex(1, 0);
    public static final Complex I    = new Complex(0, 1);

    public Complex(double re, double im) {
        this.re = re;
        this.im = im;
    }

    public double realPart()      { return re; }
    public double imaginaryPart() { return im; }

    public Complex plus(Complex c) {
        return new Complex(re + c.re, im + c.im);
    }

    public Complex minus(Complex c) {
        return new Complex(re - c.re, im - c.im);
    }

    public Complex times(Complex c) {
        return new Complex(re * c.re - im * c.im,
                re * c.im + im * c.re);
    }

    public Complex dividedBy(Complex c) {
        double tmp = c.re * c.re + c.im * c.im;
        return new Complex((re * c.re + im * c.im) / tmp,
                (im * c.re - re * c.im) / tmp);
    }
    
    @Override public boolean equals(Object o) {
        if (o == this)
            return true;
        if (!(o instanceof Complex))
            return false;
        Complex c = (Complex) o;

        // == 대신 compare를 사용하는 이유는 63쪽을 확인하라.
        return Double.compare(c.re, re) == 0
                && Double.compare(c.im, im) == 0;
    }
    @Override public int hashCode() {
        return 31 * Double.hashCode(re) + Double.hashCode(im);
    }

    @Override public String toString() {
        return "(" + re + " + " + im + "i)";
    }
}
```

위 코드를 보면 위의 다섯가지 규칙을 따르는 것을 확인할 수 있다.

## 불변 객체의 장점

- 불변 객체는 단순하다
- 불변 객체는 생성된 시점의 상태를 파괴될 때까지 간직한다.
- 스레드 안전하여 따로 동기화할 필요 없다
- 안심하고 공유할 수 있다
- 불변 객체는 자유롭게 공유할 수 있음은 물론, 불변 객체끼리는 내부 데이터를 공유할 수 있다.
- 불변 객체는 그 자체로 실패 원자성을 제공한다

## 불변 객체의 단점

- 값이 다르면 반드시 독립된 객체로 만들어야한다
    - 값의 가짓수가 많으면 큰 비용이 든다.
    - 이 문제를 해결하고 싶다면 다단계 연산을 제공하는 가변 동반 클래스를 사용하자
    - 예시로는 StringBuilder와 StringBuffer가 있다.

## 불변 클래스를 만드는 다른 설계 방법 몇가지

위에서 클래스가 불변임을 보장하려면 자신을 상속하지 못하게 해야한다고 했다.

- 가장 쉬운 방법은 final 클래스로 선언하는 것

그렇지만, 더 유연한 방법이 있다.

- 모든 생성자를 private 혹은 package-private로 만들고 public 정적 팩토리를 제공하는 방법이다.

위 예시 코드를 정적 팩토리 방식으로 리팩토링하면

```java
package effectivejava.chapter4.item17;

// 코드 17-1 불변 복소수 클래스 (106-107쪽)
public final class Complex {
    private final double re;
    private final double im;

    public static final Complex ZERO = new Complex(0, 0);
    public static final Complex ONE  = new Complex(1, 0);
    public static final Complex I    = new Complex(0, 1);

    public Complex(double re, double im) {
        this.re = re;
        this.im = im;
    }

    public double realPart()      { return re; }
    public double imaginaryPart() { return im; }

    public Complex plus(Complex c) {
        return new Complex(re + c.re, im + c.im);
    }

    // 코드 17-2 정적 팩터리(private 생성자와 함께 사용해야 한다.) (110-111쪽)
    public static Complex valueOf(double re, double im) {
        return new Complex(re, im);
    }

    public Complex minus(Complex c) {
        return new Complex(re - c.re, im - c.im);
    }

    public Complex times(Complex c) {
        return new Complex(re * c.re - im * c.im,
                re * c.im + im * c.re);
    }

    public Complex dividedBy(Complex c) {
        double tmp = c.re * c.re + c.im * c.im;
        return new Complex((re * c.re + im * c.im) / tmp,
                (im * c.re - re * c.im) / tmp);
    }

    @Override public boolean equals(Object o) {
        if (o == this)
            return true;
        if (!(o instanceof Complex))
            return false;
        Complex c = (Complex) o;

        // == 대신 compare를 사용하는 이유는 63쪽을 확인하라.
        return Double.compare(c.re, re) == 0
                && Double.compare(c.im, im) == 0;
    }
    @Override public int hashCode() {
        return 31 * Double.hashCode(re) + Double.hashCode(im);
    }

    @Override public String toString() {
        return "(" + re + " + " + im + "i)";
    }
}
```

이렇게 할 수 있다.

## 정리

- 게터가 있다고 해서 무조건 세터를 만들지는 말자
    - **클래스는 꼭 필요한 경우가 아니라면 불변이어야 한다**
- 불변으로 만들 수 없는 클래스라도 변경할 수 있는 부분을 최소한으로 줄이자
- 다른 합당한 이유가 없다면 모든 필드는 private final 이어야 한다.
- 생성자는 불변식 설정이 모두 완료된, 초기화가 완벽히 끝난 상태의 객체를 생성해야 한다