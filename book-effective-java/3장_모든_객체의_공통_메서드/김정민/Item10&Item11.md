# Item 10 : equals는 일반 규약을 지켜 재정의하라

- 꼭 필요한 경우가 아니라면 equals을 재정의하지 말자 !

- 그렇다면 언제 equals를 재정의해야할까?

→ 객체 식별성이 아니라 논리적 동치성을 확인해야하는데 상위 클래스의 equals가 논리적으로 동치성을 비교하도록 재정의되지 않았을 때이다.

- 재정의해야 할 때는 그 클래스의 핵심 필드 모두를 빠짐 없이 다섯가지 규약을 지켜가며 비교해야한다.

1. 반사성 : null 이 아닌 모든 참조 값 x에 대해, x.equals(x)는 true이다.
   - 객체는 자기 자신과 같아야한다.
2. 대칭성 : null이 아닌 모든 참조 값 x,y에 대해 x.equals(y)가 true면 y.equals(x)도 true다.

   - 두 객체는 서로에 대한 동치 여부에 똑같이 답해야한다.

   ```java
   //대칭성이 위배된 잘못된 코드
   public final class CaseInsensitiveString{
   	 private final String s;
   	 public CaseInsensitiveString(String s){
   		 this.s = Objects.requireNonNull(s);
   	 }

   	 //대칭성 위배!
   	 @Override public boolean equals(Object o){
   		 if(o instanceof CaseInsensitiveString){
   			 return s.equalsIgnoreCase(((CaseInsensitiveString) o).s);
   		 }
   		 if( o instanceof String){
   			 return s.equalsIgnoreCase((String) o);
   			}
   		return false;
   	 }
   }

   //예
   CaseInsensitiveString cis = new CaseInsensitiveString("Polish");
   String s = "polish";

   cis.equals(s); //true
   s.equals(cis); // false String의 equals는 CaseInsensitiveString의 존재를 모르기 때문에

   ```

3. 추이성 : null이 아닌 모든 참조 값 x,y,z에 대해 x.equals(y)가 true이고 y.equals(z)도 true면 x.equals(z)도 true다
   - 첫번째 객체와 두번째 객체가 같고 , 두번째 객체와 세번째 객체가 같다면 첫번째 세번째 객체도 같아야한다.
   - 구체 클래스를 확장해 새로운 값을 추가하면서 equals 규약을 만족시킬 방법은 존재하지 않는다.
     - 상속 대신 컴포지션을 사용하면된다.
     - 추상 클래스의 하위클래스라면 equals 규약을 지키면서 값을 추가할 수 있다.
4. 일관성 : null이 아닌 모든 참조 값 x,y에 대해 , x.equals(y)를 반복해서 호출하면 항상 true를 반환하거나 항상 false를 반환한다.
   - 두 객체가 같다면 앞으로 영원히 같아야한다.
5. null-아님 : null이 아닌 모든 참조 값 x에 대해 , x.equals(null)은 false다.
   - 모든 객체가 null과 같지 않아야한다.

### equals 메서드 구현방법

1. == 연산자를 사용해 자기 자신의 참조인지 확인한다.
2. instanceof 연산자로 입력이 올바른 타입인지 확인한다.
3. 입력을 올바른 타입으로 형변환 한다.
4. 입력 객체와 자기 자신의 대응되는 “핵심”필드들이 모두 일치하는지 하나씩 검사한다.

### 주의사항

1. equals를 재정의할 때는 hashcode도 반드시 재정의하자
2. Object 외의 타입을 매개변수로 받는 equals 메서드는 선언하지 말자

# Item 11 : equals를 재정의하려거든 hashCode도 재정의하라

- equals(Object) 가 두 객체를 같다고 판단했으면, 두 객체의 hashCode는 똑같은 값을 반환해야한다.
- equlas(Object)가 두 객체를 다르다고 판단했더라도, 두 객체의 hashCode가 서로 다른 값을 반환할 필요는 없으나 다른 값을 반환해야 해시테이블 성능이 좋아진다.
  - 똑같은 값을 반환한다면 모든 객체가 해시 테이블 버킷 하나에 담겨 마치 연결리스트 처럼 동작한다.

```java
@Override
public int hashCode(){
	int result = Short.hashCode(areaCode);
	result = 31 * result + Short.hashCode(prefix);
	result = 31 * result + Short.hashCode(lineNum);
	return result;
}

//한줄짜리 hashcode메서드
@Override
public int hashCode(){
	return Objects.hash(lineNum,prefix,areaCode);//위 메서드보다 느림
}
```
