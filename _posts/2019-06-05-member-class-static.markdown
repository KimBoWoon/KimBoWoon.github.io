---
title: "[Effective Java] 멤버 클래스는 가능하면 static으로 선언하라"
date: 2019-06-05 15:48:41
categories:
- Effective Java
---

# 중첩 클래스
중첨 클래스란 클래스안에 있는 또 다른 클래스를 말한다. 중첩 클래스는 네가지 종류가 있다.
정적 멤버 클래스, 비정적 멤버 클래스, 익명 클래스, 지역 클래스가 있다.

## 정적 클래스
이 클래스는 평범한 클래스에 또 다른 static 클래스가 들어있는 것이다. 이 클래스는 바깥 클래스의 모든 멤버 접근이 가능하다.
이 클래스가 유용하게 사용되는 곳은 헬퍼 클래스가 들어가는 곳이다. 계산기를 예로 들어보면 4가지의 연산자를 헬퍼 클래스로 만들수 있다.
그리고 연산자는 Calculator.Operation.PLUS 처럼 접근해서 사용할 수 있다.

## 비정적 멤버 클래스
이 클래스는 클래스 안에 또 다른 클래스가 있는 것이다. 비정적 멤버 클래스 객체와 바깥 객체와의 연결은 비정적 멤버 클래스의 객체가 만들어지는 순간에 확립되고 그 뒤에는 변경할 수 없다.
this를 사용해 바깥 객체에 대한 참조를 사용할 수 있다. 비정적 멤버 클래스는 바깥 클래스 객체 없이는 존재할 수 없다.
바깥 클래스 객체에 접근할 필요가 없는 경우 항상 static를 붙여서 정적 멤버 클래스로 만든다.
이 클래스는 어댑터를 정의할 때 많이 사용된다.

## 익명 클래스
이 클래스는 자바의 함수 객체와 비슷하다. 한 곳에서만 사용되고 그 순간에만 객체 생성이 되기 때문이다. 이 클래스는 제약이 많다.
instanceof, 클래스 이름이 필요한 곳에는 사용할 수 없고 여러 인터페이스를 구현하는 익명 클래스는 선언할 수 없으며 인터페이스를 구현하는 동시에 특정한 클래스를 계승하는 익명 클래스도 만들 수 없다.
이 클래스는 Comparator, Runnable, Thread, TimerTask 등과 같이 함수 객체를 만들어서 사용할 때 많이 쓰인다.

## 지역 클래스
이 클래스는 메소드안에 생성되는 클래스이다. 메소드 내부에서 필요한 기능을 설정하기위해 만드는 클래스이다.
하지만 코드의 가독성이 떨어지고 다른 방법으로도 해결할 수 있기 때문에 많이 사용되지 않는다.

## 예
```java
import java.util.ArrayList;
import java.util.Comparator;

public class NestedClassPractice {
    public static void main(String[] args) {
        // 바깥 클래스를 먼저 만들어야
        // 내부 클래스를 생성할 수 있다.
        // 사용하기위해 메모리에 올라가야 하기 때문
        Outer1 a = new Outer1();
        Outer1.Inner1 b = a.new Inner1();

        a.outerMethod();
        b.innerMethod();

        // 위와 같이 메모리에 올려서 사용할 수 있지만
        // static 키워드가 붙으면 클래스가 로딩될 때 static 메모리 영역에 올라가게 된다
        // 그래서 프로그램이 종료 될 때 까지 사라지지 않는다
        // static은 클래스 이름을 사용해 사용할 수 있다.
        // static 메모리에 할당을 받고 있기 때문
        Outer2 c = new Outer2();
        Outer2.Inner2 d = new Outer2.Inner2();

        c.outerMethod();
        d.innerMethod();
        Outer2.Inner2.staticInnerMethod();

        // 익명 클래스는 함수 객체로 많이 사용된다.
        // 일회용이 목적인 클래스
        // new 키워드가 사용되므로 객체가 생성되고 한 곳에서만 쓰인다
        ArrayList<Integer> arrayList = new ArrayList<>();
        arrayList.add(3);
        arrayList.add(5);
        arrayList.add(1);
        arrayList.add(2);
        arrayList.add(8);
        arrayList.sort(new Comparator<Integer>() {
            @Override
            public int compare(Integer o1, Integer o2) {
                return o1 - o2;
            }
        });

        localClass();
    }


    // 지역 클래스
    // 메소드 안에서 사용되는 클래스
    // 메소드 내부에서 필요한 기능을 설정하기 위해 만드는 클래스
    // 하지만 다른 방법으로 해결할 수 있기 때문에 많이 사용되지 않는다
    private static int a = 1001;

    public static void localClass() {
        class LocalClass {
            private int a = 10;
            private int b = 11;

            public void c() {
                System.out.println(NestedClassPractice.a);
                System.out.println("Local Class");
            }
        }

        LocalClass l = new LocalClass();

        System.out.println(l.a);
        System.out.println(l.b);
        l.c();
    }
}

class Outer1 {
    private int a;

    public void outerMethod() {
        System.out.println("Outer1 Method");
    }

    class Inner1 {
        private int b;

        public void innerMethod() {
            System.out.println("Inner1 Method");
        }
    }
}

class Outer2 {
    private int a;
    private static int b;

    public void outerMethod() {
        System.out.println("Outer2 Method");
    }

    static class Inner2 {
        private int c;
        private static int d;

        public void innerMethod() {
            System.out.println("Inner2 Method");
        }

        public static void staticInnerMethod() {
            System.out.println("Static Inner2 Method");
        }
    }
}
```