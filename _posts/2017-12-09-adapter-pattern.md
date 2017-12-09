---
title: "[pattern] Adapter Pattern"
date: 2017-12-09 01:23:45
categories:
- Pattern
---

# Adapter Pattern
약간의 차이가 있는 클래스들을 인터페이스를 통해 똑같은 기능으로 사용하는 방법

# 코드
```java
public static void main(String[] args) {
    MyEngine engine = new MyEngine();

    engine.add(1);
    engine.add(2);
    engine.add(3);
    engine.printAll();
    engine.delete(1);
    engine.printAll();
    engine.print(1);
    engine.printAll();
}
```
```java
public static void main(String[] args) {
    AEngine aEngine = new AEngine();

    engine.add(1);
    engine.add(2);
    engine.add(3);
    for (int i = 0; i < engine.getSize(); i++) {
        engine.print(i);
    }
    System.out.println();
    engine.delete(1);
    for (int i = 0; i < engine.getSize(); i++) {
        engine.print(i);
    }
    System.out.println();
    engine.print(1);
    for (int i = 0; i < engine.getSize(); i++) {
        engine.print(i);
    }
    System.out.println();
}
```
* 기존에 사용하던 엔진에서 다른 엔진을 사용하려면 그에 맞게 코드를 수정할 필요가 있을것이다.
* 이대로라면 유지보수가 불편해질것이다.

* 그래서 다음과 같이 인터페이스를 만들고 그것을 구현한다.

```java
interface EngineInterface {
    public void print(int index);
    public void printAll();
    public void add(int data);
    public void delete(int index);
    public int getSize();
}

class AdapterEngine implements EngineInterface {
    AEngine engine;

    public AdapterEngine(AEngine engine) {
        this.engine = engine;
    }

    @Override
    public void print(int index) {
        engine.print(index);
    }

    @Override
    public void printAll() {
        for (int i = 0; i < engine.getSize(); i++) {
            engine.print(i);
        }
        System.out.println();
    }

    @Override
    public void add(int data) {
        engine.add(data);
    }

    @Override
    public void delete(int index) {
        engine.delete(index);
    }

    @Override
    public int getSize() {
        return engine.getSize();
    }
}
```
* 두 클래스가 제공하는 모든 기능이 들어있음을 알 수 있다.

* 결과적으론 다음과 같이 변경되지 않은 코드를 사용할 수 있다.

```java
public static void main(String [] args) {
    AEngine aEngine = new AEngine();
    EngineInterface engine = new AdapterEngine(aEngine);

    engine.add(1);
    engine.add(2);
    engine.add(3);
    engine.printAll();
    engine.delete(1);
    engine.printAll();
    engine.print(1);
    engine.printAll();
}
```