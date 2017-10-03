---
title: "[Effective Java] 유효기간이 지난 객체는 폐기 하라"
date: 2017-10-03 01:23:45
categories:
- Effective Java
---

# 유효기간이 지난 객체는 폐기 하라
메모리를 손수 관리 해주는 C와 C++을 쓰다가 쓰레기 수집기가 있는 언어를 사용하게 되면 정말 편해진다. 편해짐에 따라 메모리 관리를 하지 않아도 쓰레기 수집기가 알아서 해준다는 생각은 하면 안된다. Stack 클래스를 예로 들어보자.

```java
public class MyStack {
    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITAL_CAPACITY = 16;
    
    public MyStack() {
        elements = new Object[DEFAULT_INITAL_CAPACITY];
    }
    
    public void push(Object e) {
        ensureCapacity();
        elements[size++] = e;
    }
    
    public Object pop() {
        if (size == 0) {
            throw new EmptyStackException();
        }
        return elements[--size];
    }

    /**
     * 적어도 하나 이상의 원소를 담을 공간을 보장한다.
     * 배열의 길이를 늘려야 할 때마다 대략 두 배씩 늘인다.
     */
    private void ensureCapacity() {
        if (elements.length == size) {
            elements = Arrays.copyOf(elements, 2 * size + 1);
        }
    }
}
```

뚜렷하게 보이는 오류는 없지만 메모리 누수(memory leak)문제가 있다. 그래서 쓰레기 수집기가 해야할 일이 늘어나 성능이 저하되거나, 메모리 요구향이 증가할 것이다. 극단적인 경우에는 디스크 페이징(disk paging)이 발생하고 심지어는 OutOfMemoryError가 던져지면서 프로그램이 중단될 것이다. 그렇다면 저기서 메모리 누수는 어디서 생길까? pop하는 부분에서 스택이 줄어들면서 제거한 객체들을 쓰레기 수집기가 처리하지 못해서 생긴다. 스택이 그런 객체에 대한 만기 참조(obsolete reference)를 제거하지 않기 때문이다. 이런 문제는 간단히 고칠 수 있다. 쓸 일 없는 객체 참조는 무조건 null로 만드는 것이다. Stack클래스의 경우, 스택에서 pop된 객체에 대한 참조는 그 즉시 null로 만들면 된다.

```java
public Object pop() {
    if (size == 0) {
        throw new EmptyStackException();
    }
    Object results = elements[--size];
    elements[size] = null;
    return results;
}
```

만기 참조를 null로 만들면 나중에 실수로 그 참조를 사용하더라도 NullPointerException이 발생하기 때문에 프로그램은 오동작하는 대신 바로 종료된다는 장점이 있다. 프로그램의 오류는 최대한 빨리 알아내는 것이 좋다. 이런 오류를 몇 번 접하고 나면 객체 사용이 끝나면 즉시 그 참조를 null로 만들어야 한다는 강박관념에 사로잡히는 경우가 있다. 하지만 그럴필요도 없고 바람직하지도 않다. 객체 참조를 null 처리하는 것은 규범(norm)이라기보단 예외적인 조치가 되어야 한다. 만기 참조를 제거하는 가장 좋은 방법은 해당 잠조가 보관된 변수가 유효범위(scope)를 벗어나게 두는 것이다. 변수를 정의할 때 그 유효범위를 최대한 좁게 만들면 자연 스럽게 해결된다.

메모리 누수가 흔히 발생하는 곳은 캐시(chche)나 리스너(listener)등 이다. 메모리 누수가 보통 뚜렷한 오류로 이어지지 않고 눈에도 잘 안띄기 때문에 코딩할 때 각별한 주의가 필요하다.
