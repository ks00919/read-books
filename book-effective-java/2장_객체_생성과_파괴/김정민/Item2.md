# Item 2 : 생성자에 매개 변수가 많다면 빌더를 고려하라

- 정적 팩터리와 생성자는 선택적 매개변수가 많을 때 적절히 대응하기 힘들다.

1. 점층적 생성자 패턴

- 선택 매개변수를 전부 받는 생성자까지 늘려가는 방식이다.

```java

public NutritionFacts(int servingSize, int servies){..}
public NutritionFacts(int servingSize, int servies,int colories){..}

```

- 매개변수가 많아지면 클라이언트 코드를 작성하거나 읽기 어렵다.

2. 자바빈즈 패턴

- 매개변수가 없는 생성자로 객체를 만든 후 세터 메서드를 호출해 원하는 매개변수의 값을 설정한다.
- 객체 하나를 만들려면 여러 세터 메서드를 호출해야한다.
- 객체 하나를 완성하기 전에 일관성이 무너진 상태에 놓인다.

3. 빌더 패턴

```java
public class NutrtionFacts {
    private final int servingSize;
    private final int servings;
    private final int cal;
    private final int fat;

    public static class Builder{
        //필수 매개변수
        private final int servingSize;
        private final int servings;

        //선택 매개변수
        private final int cal = 0;
        private final int fat = 0;

        public Builder(int servingSize,int servings){
            this.servingSize = servingSize;
            this.servings = servings;
        }

        public Builder cal(int val){
            cal = val;
            return this;
        }

        public NutrtionFacts build(){
            return new NutrtionFacts(this);
        }
    }

    private NutrtionFacts(Builder builder){
        servingSize = builder.servingSize;servings = builder.servings;
        cal = builder.cal;
        fat = builder.fat;
    }


}
```

- 계층적으로 설계된 클래스와 함께 쓰기 좋다.
- 생성자나 정적 팩터리가 처리해야 할 매개변수가 많다면 빌더 패턴을 선택하는게 낫다.
- 빌더 생성비용이 크지는 않지만 성능에 민감한 상황에서는 문제가 발생할 수 있다.
