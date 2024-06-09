# Item 11 - equals를 재정의하려거든 hashCode도 재정의하라

equals를 재정의한 클래스 모두에서 hashCode도 재정의해야 한다.

그렇지 않으면 hashCode 일반 규약을 어기게 되어 해당 클래스의 인스턴스를 HashMap이나 HashSet 같은 컬렉션의 원소로 사용할 때 문제를 일으킬 수 있다.

Object 명세에서 발췌한 규약은 아래와 같다.

1. equals 비교에 사용되는 정보가 변경되지 않았다면, 애플리케이션이 실행되는 동안 그 객체의 hashCode 메서드는 몇 번을 호출해도 일관되게 항상 같은 값을 반환해야한다. 단, 어플리케이션을 다시 실행한다면 이 값이 달라져도 상관없다.
2. equals(Object)가 두 객체를 같다고 판단했다면, 두 객체의 hashCode는 똑같은 값을 반환해야 한다.
3. equals(Object)가 두 객체를 다르다고 판단했더라도, 두 객체의 hashCode가 서로 다른 값을 반환할 필요는 없다. 단, 다른 객체에 대해서는 다른 값을 반환해야 해시테이블의 성능이 좋아진다.

위 규약중에서 두번째 규약이 hashCode 재정의를 잘못했을 때 문제가 되는 조항이다.

**즉, 논리적으로 같은 객체는 같은 해시코드를 반환해야 한다.**

그러니까, 논리적으로 같은 객체는 equals를 재정의해서 같다고 표현해주었다면 hashCode도 같다고 표현해주어야 한다.

## 이상적인 hashCode 메서드 짜기

이상적인 해쉬 함수는 서로 다른 인스턴스들을 32비트 정수 범위에 균일하게 분배해야 한다.

다음은 좋은 hashCode를 작성하는 간단한 요령이다.

1. int 변수 result를 선언한 후 값 c로 초기화한다. 
    
    이때 c는 해당 객체의 첫번째 핵심 필드를 다음 소개하는 2.a 방식으로 계산한 해시코드이다. 
    
    참고로 여기 소개되는 핵심 필드는 equals 비교에 사용되는 필드이다. 
    
    **equals에서 사용되지 않는 필드는 반드시 hashCode에서도 제외해야한다.** 
    
    이를 어기면 hashCode의 두번째 규약을 어기기 때문이다.
    
2. 해당 객체의 나머지 핵심 필드 f에 대해 각각 다음 작업을 수행한다.
    
    a. 필드의 해시코드 c를 계산한다.
    
    - 기본 필드라면 Type.hashCode(f)를 수행한다. 이때 Type은 기본 타입에 매핑되는 래퍼 타입이다.
    - 참조 필드이면서 클래스의 equals가 이 필드의 equals를 재귀적으로 호출해 비교한다면 이 필드의 hashCode가 재귀적으로 호출한다. 만약 필드의 값이 `null`이면 0을 반환한다.
    - 배열이라면 핵심 원소 각각을 별도의 필드로 나눈다. 별도로 나눈 필드를 위 규칙을 적용하여 해시코드를 계산한 후 다음 소개되는 2.b 방식으로 갱신한다. 모든 원소가 핵심이라면 `Arrays.hashCode`를 이용한다.
    
    b. 2.a로 계산한 해시코드 c로 result를 갱신한다.
    
    ```java
    result = 31 * result + c
    ```
    
3. result를 반환한다.

굳이 31을 곱한 이유는 31이 홀수이면서 소수이기 때문이다. 요즘 VM들은 최적화를 잘 해주기 때문에 31을 곱하는 행위는 i << 5 - 1 이 되어 속도를 최적화 할 수 있는 듯하다.

위 규칙을 지켜 만든 코드는 다음과 같다.

```java
@Override public int hashCode() {
        int result = Short.hashCode(areaCode);
        result = 31 * result + Short.hashCode(prefix);
        result = 31 * result + Short.hashCode(lineNum);
        return result;
    }
```

만약 클래스가 불변이고, 해시코드를 계산하는 비용이 크다면, 매번 새로 계산하기 보다는 캐싱하는 방식을 고려해야 한다.

이 타입의 객체가 주로 해시의 키로 사용될 것 같다면 인스턴스가 만들어질 때 해시코드를 계산해두어야 하고, 해시의 키로 사용되지 않는 경우라면 hashCode가 처음 불릴 때 계산하는 지연 초기화 전략을 이용하면 된다.

필드를 지연 초기화 하려면 쓰레드 안정성까지 고려해야 하기 때문에 아래 코드를 참고하자.

```java
    @Override public int hashCode() {
        int result = hashCode;
        if (result == 0) {
            result = Short.hashCode(areaCode);
            result = 31 * result + Short.hashCode(prefix);
            result = 31 * result + Short.hashCode(lineNum);
            hashCode = result;
        }
        return result;
    }
```

주의할점은 다음과 같다

1. 성능을 높인답시고 해시코드를 계산할 때 핵심 필드를 생략해서는 안된다.
2. hashCode가 반환하는 값의 생성 규칙을 API 사용자에게 자세히 공표하지 말자
    1. 그래야 클라이언트가 이 값에 의지하지 않게 되고, 추후에 계산 방식을 바꿀 수도 있다.