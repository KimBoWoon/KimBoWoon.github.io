---
title: "[Effective Java] 불필요한 객체는 만들지 말라"
date: 2017-09-27 01:23:45
categories:
- Effective Java
---

# 불필요한 객체는 만들지 말라
기능적으로 객체를 새로 만드는 것보다 재사용하는 편이 훨씬 빠르다.
```java
String s = new String("Hello, World");
```
이런식으로 객체를 만들면 호출 될 때마다 항상 새로운 객체가 만들어지기 때문에 이러한 방법은 피하는 편이 좋고 아래와 같은 방법을 쓰는것이 좋다.
```java
String s = "Hello, World";
```

실제로 다음과 같은 코드를 실행시키면 다음과 같은 결과를 얻는다.
```java
public class ImmutableObject {
    public static void main(String[] args) {
        String a = new String("Hello, World");
        String b = new String("Hello, World");

        String c = "Hello, World";
        String d = "Hello, World";

        System.out.println(a == b);
        System.out.println(c == d);
    }
}
```

![](/_img/effectivejava/string_address.png)

변경 불가능한 객체인 경우 정적 팩토리 메소드를 사용하면 새로운 객체를 만들지 않는다. 변경 가능한 객체도 재사용할 수 있다. 단, 변경하지 않는다면 말이다. 다음의 예를 보자

```java
public class Person {
    private final Date birthDate;

    public boolean isBabyBoomer() {
        Calendar gmtCal = Calendar.getInstance(TimeZone.getTimeZone("GMT"));

        gmtCal.set(1946, Calendar.JANUARY, 1, 0, 0, 0);
        Date boomStart = gmtCal.getTime();
        gmtCal.set(1965, Calendar.JANUARY, 1, 0, 0, 0);
        Date boomEnd = gmtCal.getTime();
        return birthDate.compareTo(boomStart) >= 0 && birthDate.compareTo(boomEnd) < 0;
    }
}
```

이렇게 만들면 해당 메소드를 호출될 때마다 Calendar 객체 하나, TimeZone 객체 하나, 그리고 Date 객체 두 개를 쓸데없이 만들어 낸다. 이렇게 비효율적인 코드는 정적 초기화 블록(static initializer)을 통해 개선하는 것이 좋다.

```java
public class Person {
    private final Date birthDate;

    private static final Date BOOM_START;
    private static final Date BOOM_END;
    
    static {
        Calendar gmtCal = Calendar.getInstance(TimeZone.getTimeZone("GMT"));
        gmtCal.set(1946, Calendar.JANUARY, 1, 0, 0, 0);
        BOOM_START = gmtCal.getTime();
        gmtCal.set(1965, Calendar.JANUARY, 1, 0, 0, 0);
        BOOM_END = gmtCal.getTime();
    }
    
    public boolean isBabyBoomer() {
        return birthDate.compareTo(BOOM_START) >= 0 && birthDate.compareTo(BOOM_END) < 0;
    }
}
```

이 클래스는 클래스가 초기화 될 때 한 번만 만든다. 그리고 BOOM_START와 BOOM_END가 한번만 초기화 되는 상수라는 사실도 알수있다. 또 객체 자료형보다 기본 자료형을 사용하는 것이 좋다.

```java
Integer i = 3;
// Integer i = new Integer(3);
```
auto-boxing이 되면서 불필요한 객체를 만들기 때문이다.
