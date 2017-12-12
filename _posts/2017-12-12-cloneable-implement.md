---
title: "[Effective Java] clone을 재정의 할 때는 신중하라"
date: 2017-12-12 01:23:45
categories:
- Effective Java
---

# Cloneable Interface
Cloneable은 어떤 객체가 복제를 허용한다는 사실을 알리는 데 쓰려고 만들어진 믹스인(mixin) 인터페이스다. 이 인터페이스는 아무런 추상 메소드들을 가지고 있지 않고 오직 Object 클래스의 clone을 사용할것인지 여부를 결정하기만 한다. clone 메소드를 호출하면 해당하는 객체의 복사본을 만들어낸다. 또한 원본객체와 같은 필드값을 가지며 필드의 값도 복사된다. 이 인터페이스를 implement를 하지않고 clone메소드를 호출하면 CloneNotSupportedException을 뱉어낸다.

# clone 메소드의 일반 규약
* x.clone() != x
* x.clone().getClass() == x.getClass()
* x.clone.equals(x)

이 규약들은 참이여야 하지만 꼭 그렇지만도 않다.

# clone override
final이 아닌 슈퍼 클래스를 오버라이딩할 경우 반드시 서브 클래스에서 super.clone()을 호출하여 얻은 객체를 반환하야 한다. 그리고 해당하는 객체를 복사할 때 레퍼런스 타입을 가지고 있는 경우에는 레퍼런스 타입에도 clone 메소드를 호출해야한다.

```java
class Stack implements Cloneable {
    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public Stack() {
        this.elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }

    public void push(Object e) {
        ensureCapacity();
        elements[size++] = e;
    }

    public Object pop() {
        if (size == 0) {
            throw new EmptyStackException();
        }
        Object result = elements[--size];
        elements[size] = null;
        return result;
    }

    private void ensureCapacity() {
        if (elements.length == size) {
            elements = Arrays.copyOf(elements, 2 * size + 1);
        }
    }

    @Override
    public Stack clone() {
        try {
            Stack result = (Stack) super.clone();
            result.elements = elements.clone();
            return  result;
        } catch (CloneNotSupportedException e) {
            throw new AssertionError();
        }
    }
}
```
* clone 메소드에서 super를 사용해서 얻어온 객체는 형변환을 해서 넣지만 elements는 clone 메소드을 호출한 결과 그대로 넣고있다. 릴리즈 1.5부터는 배열에 clone을 호출하면 반환되는 배열의 컴파일 시점 자료형은 복제 대상 배열의 자료형과 같기 때문이다.
* clone 메소드는 동기화를 지원하지 않는다. 원한다면 따로 구현야해할 것이다.
* clone 을 사용해 꼭 모두 복사할 필요가 없다. 다른 값을 가지고 싶은 필드가 있다면 clone 메소드에서 따로 설정하는게 좋다.
* clone의 아키텍처는 변경 가능한 객체를 참조하는 final 필드의 일반적 용법과 호환되지 않는다. 따라서 final 선언을 지워야 할 수도 있다.
* 깊은 복사를 진행할 때 재귀보다는 loop를 사용하는게 좋다. Stack Overflow가 발생할 수 있기 때문이다.
* Cloneable 인터페이스를 구현하는 클래스는 제대로 동작하는 public clone 메소드를 제공해야한다.

사실 clone 메소드를 많이 사용하지 않는다. 굳이 사용해야 한다면 제대로 구현해야할 것이다.