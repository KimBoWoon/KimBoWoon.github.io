---
title: "[Effective Java] 가능하면 제네릭 자료형으로 만들 것"
date: 2019-11-03 23:19:42
categories:
- Effective Java
---

# 제네릭 클래스 만들기

컬렉션 객체 선언을 제네릭화 하거나 JDK에서 제공하는 제네릭 자료형과 메서드를 사용하는 것은 어렵지않다. 제네릭 자료형을 직접 만드는 것은 좀 까다로운데 그래도 가치는 있다. 다음 Stack 클래스를 보자.

```java
public class Stack {
    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITAL_CAPACITY = 16;
    
    public Stack() {
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
        Object results = elements[--size];
        elements[size] = null; // 만기 참조 제거
        return results;
    }

    public boolean isEmpty() {
        return size == 0;
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

제네릭으로 변경하기 위한 딱 좋은 예제이다. 호환성을 유지하면서도 제네릭 자료형을 사용하도록 개선할 수 있다. 위의 코드에서는 스택에서 꺼낸 자료를 형변환해서 사용해야 하는데 실패할 가능성이 있다. 

클래스를 제네릭화하는 첫 번째 단계는 선언부에 형인자를 추가하는 것이다. 

```java
// 제네릭을 사용해 작성한 최초의 Stack 클래스 - 컴파일되지 않는다!
public class Stack<E> {
    private E[] elements;
    private int size = 0;
    private static final int DEFAULT_INITAL_CAPACITY = 16;
    
    public Stack() {
        elements = new E[DEFAULT_INITAL_CAPACITY];
    }
    
    public void push(E e) {
        ensureCapacity();
        elements[size++] = e;
    }
    
    public E pop() {
        if (size == 0) {
            throw new EmptyStackException();
        }
        E results = elements[--size];
        elements[size] = null; // 만기 참조 제거
        return results;
    }

    public boolean isEmpty() {
        return size == 0;
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

바로 경고나 오류없이 컴파일이 성공하면 좋겠지만 그럴 확률은 적다. 오히려 경고나 오류가 없다면 더욱 유심히 살펴봐야한다. 그나마 다행인것은 위의 코드에서 한군데에서 경고가 발생한다는 것이다.

```java
Stack.java: generic array creation
elements = new E[DEFAULT_INITIAL_CAPACITY];
           ＾
```

# 경고 및 오류 수정
첫 번째로 제네릭 배열을 만들 수 없다는 조건을 우회하는 것. Object의 배열을 만들어서 제네릭 배열 자료형으로 형변환하는 방법이다. 문법적으로는 문제가 없지만 일반적으로 형 안전성을 보장하는 방법은 아니다.

```java
Stack.java: warning: [unchecked] unchecked cast
found: Object[], required: E[]
elements = (E[]) new Object[DEFAULT_INITIAL_CAPACITY];
             ＾
```

컴파일러는 형 안전성을 입증할 수 없을지 모르지만, 프로그래머는 할 수 있다. elements는 private이며 Stack 클래스내에서만 사용이 되고 push 메서드를 사용하여 데이터의 삽입이 이뤄지며 그 타입은 모두 E 이므로 무점검 형변환을 해도 아무런 문제가 되지 않는다.

무점검 형변환이 안전함을 증명했다면 경고를 억제하되 범위는 최소한으로 해야한다. Stack 클래스에 생성자에서 무점검 배열 생성을 하는 코드가 전부이므로 생성자 전체적으로 경고를 억제해도 무방하다.

```java
// elements 배열에는 push(E)를 통해 전달된 E 형의 객체만 저장된다.
// 이 정도면 형 안전성은 보장할 수 있지만, 배열의 실생시간 자료형은 E[]가
// 아니라 항상 Object[]이다!
@SuppressWarnings("unchecked")
public Stack() {
    elements = new E[DEFAULT_INITAL_CAPACITY];
}
```

두 번째 방법은 elements의 자료형을 E[]에서 Object[]로 바꾸는 것이다. 그러면 아까와는 다른 오류 메시지를 보게 된다.

```java
Stack.java: incompatible types
found: Object, required: E
E result = elements[--size];
                   ＾
```

배열에서 꺼낸 원소의 자료형을 Object에서 E로 변환하도록 코드를 수정하면, 이 오류는 경고로 바뀐다.

```java
Stack.java: warning: [unchecked] unchecked cast
found: Object, required: E
E result = (E) elements[--size];
                       ＾
```

E는 실체화 불가능 자료형이므로 컴파일러는 이 형변환을 실행 중에 검사할 수 없지만 위의 무점검 형변환이 안전하다는 것, 그래서 경고를 억제해도 좋다는 것은 개발자 스스로 쉽게 입증할 수 있다. 다음과 같이 pop 메서드를 수정한다. 단, pop 메서드 전체에 어노테이션을 붙이지 않는다.

```java
// 무점검 경고를 적절히 억제한 사례
public E pop() {
    if (size == 0) {
        throw new EmptyStackException();
    }
    // 자료형이 E인 원소만 push하므로, 아래의 형변환은 안전하다.
    @SuppressWarnings("unchecked")
    E results = (E) elements[--size];
    elements[size] = null; // 만기 참조 제거
    return results;
}
```

위 두가지 방법중 어떤 방법을 사용해도 상관없다. 다만 두 번째 방법은 많은 곳을 수정할 가능성이 있으므로 첫 번째 방법이 많이 사용된다.