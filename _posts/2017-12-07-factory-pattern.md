---
title: "[pattern] Factory Pattern"
date: 2017-12-07 01:23:45
categories:
- Pattern
---

# Factory Pattern
클래스의 생성을 다른 클래스의에 위임하는 것

# 코드
```java
public class FactoryPatternPractice {
    public static void main(String[] args) {
        Book mathBook = FactoryPattern.createClass("math");
        Book historyBook = FactoryPattern.createClass("history");

        System.out.println(mathBook.getName());
        System.out.println(historyBook.getName());
    }
}

interface Book {
    String getName();
}

class MathBook implements Book {
    @Override
    public String getName() {
        return "MathBook";
    }
}

class HistoryBook implements Book {
    @Override
    public String getName() {
        return "HistoryBook";
    }
}

class FactoryPattern {
    public static Book createClass(String name) {
        switch (name) {
            case "math":
                return new MathBook();
            case "history":
                return new HistoryBook();
            default:
                return null;
        }
    }
}
```

# 장단점
* 객체 생성 코드를 전부 한 객체 또는 메소드에 집어넣으면 코드에 중복되는 내용을 제거할 수 있다.
* 유지보수에도 편하다.
* 인터페이스 바탕으로 프로그래밍을 할 수 있다.
* 생성할 객체의 종류가 달라질 때마다 새로운 클래스를 만들어야 한다. 이것은 불 필요하게 많은 클래스를 만들어낼 수 있다.
