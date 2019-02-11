---
title: "[Effective Java] 태그 달린 클래스 대신 클래스 계층을 활용하라"
date: 2019-02-11 14:50:33
categories:
- Effective Java
---

# 태그 클래스
하나의 클래스의 두 가지 이상의 기능이 있는 클래스를 말한다. 하지만 이와 같은 클래스는 오히려 가독성을 떨어트릴 뿐이다.
```java
class Figure {
    enum Shape { RECTANGLE, CIRCLE };
    
    // 어떤 모양인지 나타내는 태그 필드
    final Shape shape;
    
    // 태그가 RECTANGLE일 때만 사용되는 필드들
    double length;
    double width;
    
    // 태그가 CIRCLE일 때만 사용되는 필드들
    double radius;
    
    // 원을 만드는 생성자
    Figure(double radius) {
        shape = Shape.CIRCLE;
        this.radius = radius;
    }
    
    // 사각형을 만드는 생성자
    Figure(double length, double width) {
        shape = Shape.RECTANGLE;
        this.length = length;
        this.width = width;
    }
    
    double area() {
        switch (shape) {
            case RECTANGLE:
                return length * width;
            case CIRCLE:
                return Math.PI * (radius * radius);
            default:
                throw new AssertionError();
        }
    }
}
```

보이는 봐와 같이 enum과 switch문이 들어가 있고 다양한 생성자와 필드가 만들어 진걸 볼 수 있다.
이런 코드는 가독성을 떨어뜨릴고 객체가 만들어 질 때 필요하지 않은 필드까지 초기화 시키니 메모리 요구도 늘어난다.
그리고 오류 발생 가능성도 더 높고 도형이 추가 될 때 마다 이 클래스는 점점 불어 날것이다.

# 수정
위와 같은 클래스를 다음과 같이 고칠 수 있다.
```java
abstract class Figure {
    abstract double area();
}

class Circle extends Figure {
    final double radius;
    
    Circle(double radius) {
        this.radius = radius;
    }
    
    double area() {
        return Math.PI *(radius * radius);
    }
}

class Rectangle extends Figure {
    final double length;
    final double width;
    
    Rectangle(double length, double width) {
        this.length = length;
        this.width = width;
    }
    
    double area() {
        return length * width;
    }
}
```

위에 있는 태그 기반 클래스 보다 더 읽기 쉽고 불 필요한 코드가 전부 사라졌음을 볼 수 있다. 또한 태그 기반 클래스의 단점을 모두 해결하였다.
