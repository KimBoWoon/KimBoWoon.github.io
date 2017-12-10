---
title: "[pattern] Strategy Pattern"
date: 2017-12-11 01:23:45
categories:
- Pattern
---

# Strategy Pattern
필요한 알고리즘을 캡슐화 시켜서 상황에 맞게 사용하는 방법

# 코드
```java
public class StrategyPatternPractice {
    public static void main(String[] args) {
        Tiger tiger = new Tiger("Tiger", new Walk());
        Bird bird = new Bird("Bird", new Fly());

        tiger.doMove();
        bird.doMove();
    }
}
```
* 알고리즘 인스턴스를 만들어 생성자로 넘겨준다

```java
interface Move {
    String move();
}
```
* 캡슐화 알고리즘을 만들기 위해 인터페이스를 정의

```java
class Fly implements Move {
    @Override
    public String move() {
        return "Fly";
    }
}

class Walk implements Move {
    @Override
    public String move() {
        return "Walk";
    }
}
```
* 인터페이스를 상속 받아 각각의 알고리즘을 구현한다

```java
class Tiger {
    private String name;
    private Move move;

    public Tiger(String name, Move move) {
        this.name = name;
        this.move = move;
    }

    public void doMove() {
        System.out.println(getName() + " " + move.move());
    }

    public String getName() {
        return name;
    }
}

class Bird {
    private String name;
    private Move move;

    public Bird(String name, Move move) {
        this.name = name;
        this.move = move;
    }

    public void doMove() {
        System.out.println(getName() + " " + move.move());
    }

    public String getName() {
        return name;
    }
}
```
* 해당하는 클래스 마다 생성자에서 인터페이스로 캐스팅시켜서 할당한다