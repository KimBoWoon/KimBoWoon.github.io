---
title: "[Effective Java] 전략을 표현하고 싶을 때는 함수 객체를 사용하라"
date: 2019-06-04 오후 9:57
categories:
- Effective Java
---

# 함수 객체
다른 언어에서는 함수 포인터, 람다, 델리게이트 처럼 특정 함수를 호출할 수 있는 능력을 저장하고 전달할 수 있도록 하는 것들이 있다.
이러한 것들은 C언어의 qsort의 마지막 인자인 comparator에 들어갈 수 있다.
자바에서는 함수 포인터를 지원하지 않는다. 하지만 비슷하게 만들어서 효과를 볼 수 있다. 이러한 것을 자바에서는 함수 객체라고 부른다.

```java
class StringLengthComparator {
    public int compare(String s1, String s2) {
        return s1.length() - s2.length();
    }
}
```
위의 메소드는 문자열을 알파벳순이 아닌 길이 순으로 정렬하는 메소드이다. 위의 메소드를 사용하려면 객체 생성이 필요하지만 다음과 같이 객체 생성을 하지 않고 사용할 수 있다.

```java
class StringLengthComparator {
    private StringLengthComparator() {}
    public static final StringLengthComparator INSTANCE = new StringLengthComparator();
    public int compare(String s1, String s2) {
        return s1.length() - s2.length();
    }
}
```
compare 메소드는 오직 문자열만 매개변수로 받고있다. 하지만 제네릭을 사용하면 다양한 인자를 받을 수 있게 된다.

```java
public interface Comparator<T> {
    public in compare(T t1, T t2);
}

class StringLengthComparator implements Comparator<String> {
    // 위와 동일한 메소드
}

// 익명 클래스로도 정의 가능
Arrays.sort(stringArray, new Comparator<String>) {
    public int compare(String s1, String s2) {
        return s1.length() - s2.length();
    }
}
```