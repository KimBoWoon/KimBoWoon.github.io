---
title: "[Effective Java] Builder Pattern"
date: 2017-09-13 16:30:00
categories:
- Effective Java
---

# Builder Pattern
정적 팩토리 메소드나 생성자는 같은 문제를 가지고 있다. 선택적 인자가 많은 상황에 잘 적응하지 못한다는 것이다. 이럴때에 점층적 생성자 패턴을 적용하기는 하지만 필요없는 인자를 넣어 줘야 할 때도 있고 클라이언트 코드를 작성하기도 어려워진다. 그리고 가독성도 많이 떨어진다. 다른 방법으로 자바빈 패턴이있다. 일단 객체를 생성한 뒤에 세터를 사용해 객체에 값을 넣어주는 방법이다. 하지만 객체를 만들면서 초기화 작업을 하지 못하므로 객체의 일관성이 일시적으로 깨질 수 있다는 것이다. 빌더 패턴을 사용하면 이 문제를 해결 할 수 있다.

## Code
```java
class NutritionFacts {
    private final int servingSize;
    private final int servings;
    private final int calories;
    private final int fat;
    private final int sodium;
    private final int carbohydrate;

    public static class Builder {
        // 필수인자
        private final int servingSize;
        private final int servings;
        // 선택적 인자 - 기본값으로 초기화
        private int calories = 0;
        private int fat = 0;
        private int sodium = 0;
        private int carbohydrate = 0;

        public Builder(int servingSize, int servings) {
            this.servingSize = servingSize;
            this.servings = servings;
        }

        public Builder calories(int val) {
            calories = val;
            return this;
        }

        public Builder fat(int val) {
            fat = val;
            return this;
        }

        public Builder sodium(int val) {
            sodium = val;
            return this;
        }

        public Builder carbohydrate(int val) {
            carbohydrate = val;
            return this;
        }

        public NutritionFacts build() {
            return new NutritionFacts(this);
        }
    }

    private NutritionFacts(Builder builder) {
        servingSize = builder.servingSize;
        servings = builder.servings;
        calories = builder.calories;
        fat = builder.fat;
        sodium = builder.sodium;
        carbohydrate = builder.carbohydrate;
    }
}

public class BuilderPattern {
    public static void main(String[] args) {
        NutritionFacts n = new NutritionFacts.Builder(0, 1).calories(1).build();
    }
}

```
NutritionFacts 객체는 변경이 불가능하고 모든 인자의 기본값이 한곳에 모여 있다. 그리고 빌더에 정의된 설정 매소드는 빌더 객체 자신을 반환하므로, 설정 매소드를 호출하는 코드는 쭉 이어서 쓸 수 있다. 또 작성하기도 쉽고 가독성도 뛰어나다.

## 빌더 패턴 단점
객체를 만들려면 빌더 객체를 생성해야 한다는 것이다.  빌더 객체를 만드는 오버헤드가 문제가 안될 수 도있지만 성능이 중요한 상황에서는 고려해야한다.
