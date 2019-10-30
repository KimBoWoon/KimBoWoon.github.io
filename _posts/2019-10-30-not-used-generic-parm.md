---
title: "새 코드에는 무인자 제네릭 자료형을 사용하지 마라"
date: 2019-10-29 23:19:42
categories:
- Effective Java
---

# 과거 Java의 컬렉션
```java
// 무인자 컬렉션 자료형. 이렇게 만들면 안된다.
/**
* 내 우표 컬렉션. Stamp 객체만 보관한다.
*/
private final Collection stamps = ...;
```

엉뚱한 자료형을 넣어도 아무런 오류없이 컴파일이 된다.

```java
// 우표 객체만 담을 수 있는 컬렉션에 동전을 넣었다.
stamps.add(new Coin(...));
```

사용하려고 하면 당연히 오류가 발생된다.

```java
for (Iterator i = stamps.iterator(); i.hasNext(); ) {
    Stamp s = (Stamp) i.next(); // ClassCastException 예외 발생
    ... // 우표 객체로 뭔가 작업을 한다.
}
```

오류는 가능한 빨리 발견해야 하며, 컴파일할 때 발견할 수 있으면 가장 이상적이다. 하지만 위의 코드는 컴파일이 되고 한참 뒤에나 오류를 발견할 수 있다. 그리고 문제가 뭔지 한참을 해매게 될것이다.
컬렉션을 사용하면 위와 같은 에러를 컴파일 타임에 발견할 수 있다.

# 제네릭의 사용

```java
// 형인자 컬렉션 자료형 - 형 안전성 확보
private final Collection<Stamp> stamps = ...;
```

이 선언을 보고 컴파일러는 stamps 컬렉션에 Stamp만 들어가야 한다는 것을 이해하며, 실제로 Stamp 객체만 삽입 된다. 또한 형변환을 하지 않아도 컴파일러가 형변환을 진행하며 이 형변환은 절대 실패할 일이없다. (제네릭을 이해하는 컴파일러로 모든 코드를 컴파일하고, 컴파일 할 때 발생하거나 경고가 없어야 한다.)

# 제네릭과 무인자 자료형
과거의 Java는 무인자 자료형을 사용해서 형 안전성이 없었고, 제네릭의 장점 중 하나인 표현력 측면에서 손해를 보았다. 무인자 자료형은 사용하면 안되지만 ```List<Object>```와 같은 자료형은 써도 괜찮다. 무인자 자료형과의 차이는 형 검사 절차를 완전히 생략 하냐 안하냐 인것이다.

```java
// 실행 도중에 오류를 일으키는 무인자 자료형 사용 예
public static void main(String[] args) {
    List<String> strings = new ArrayList<String>();
    unsafeAdd(strings, new Integer(42));
    String s = strings.get(0); // 컴파일러가 자동으로 형변환
}
private static void unsateAdd(List list, Object o) {
    list.add(o);
}
```

이 프로그램은 컴파일은 성공하겠지만 무인자 자료형의 사용으로 경고가 뜬다. 그리고 실제로 실행하면 ```strings.get(0)```의 실행 결과를 String으로 변환하려 하기 때문에 ClassCastException 예외가 발생한다. 위의 unsateAdd의 List를 ```List<Object>```로 수정하면 오류가 발생하면서 컴파일이 실패할 것이다.

컬렉션에 들어갈 원소들이 자료형을 모르고 상관할 필요도 없다면 무인자 자료형을 써보고 싶을 것이다. 제네릭에 익숙하지 않다면 아래와 같은 코드를 만들것이다.

```java
static int numElementsInCommon(Set s1, Set s2) {
    int result = 0;
    for (Object o1 : s1) {
        if (s2.contains(o1)) {
            result++;
        }
    }
    return result;
}
```

위의 코드는 정상 작동하지만 무인자 자료형을 사용하므로 위험하다.
Java는 비안정적 와일드카드 자료형이라는 좀 더 안전한 대안을 제공한다. 제네릭 자료형을 쓰고 싶으나 실제 형 인자가 무엇인지는 모르거나 신경 쓰고 싶지 않을 때는 형 인자로 '?'를 쓰면 된다. 이 자료형은 어떤 Set이건 참조가 가능하다. 위의 코드에 비한정적 와일드카드 자료형을 사용하면 다음과 같은 코드가 된다.

```java
static int numElementsInCommon(Set<?> s1, Set<?> s2) {
    int result = 0;
    for (Object o1 : s1) {
        if (s2.contains(o1)) {
            result++;
        }
    }
    return result;
}
```

?만 넣는다고 정말 안전 하냐고 생각할 수 있지만 정말로 안전하다. 무인자 자료형은 아무 객체나 넣을 수 있어서, 컬렉션의 자료형 불변식이 쉽게 깨진다.

# 예외
1. 클래스 리터럴에는 반드시 무인자 자료형을 사용해야 한다.
    * Java 표준에 따르면, 클래스 리터럴에는 형인자 자료형을 쓸 수 없다. List.class, String[].class, int.class는 가능하나 List<String>.class, List<?>.class는 불가능하다는 말이다.
2. instanceof 연산자 사용 규칙
    * 제네릭 자료형 정보는 프로그램이 실행될 때는 지워지기 때문에, instanceof 연산자는 비한정적 와일드카드 자료형 이외의 형인자 자료형에 적용할 수 없다.
    * 제네릭 자료형에 instanceof 연산자를 적용할 때는 다음과 같이 하는 것이 좋다.
    ```java
    // instanceof 연산자에는 무인자 자료형을 써도 OK
    if (o instanceof Set) { // 무인자 자료형
        Set<?> m = (Set<?>) o; // 와일드카드 자료형
    }
    ```
