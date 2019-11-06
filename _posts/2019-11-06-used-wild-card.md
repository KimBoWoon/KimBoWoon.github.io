---
title: "[Effective Java] 한정적 와일드카드를 써서 API 유연성을 높여라"
date: 2019-11-06 21:19:42
categories:
- Effective Java
---

# 한정적 와일드카드
형인자 자료형은 불변 자료형이다. `List<Object>`랑 `List<String>`은 같지 않다는 말이다. 직관적으로 이해하기 어렵겠지만 이치에는 맞는 말이다. 때로는 이보다 높은 유연성이 필요할 때가 있다. 전에 만들었던 스택 클래스는 아래와 같은 public API를 갖는다.

```java
public class Stack<E> {
    public Stack();
    public void push(E e);
    public E pop();
    public boolean isEmpty();
}
```

여기에, 일령의 원소들을 인자로 받아 차례로 스택에 집어넣는 메서드를 추가 해보자. 아마 이런 코드를 만들게 될것이다.

```java
// 와일드카드 자료형을 사용하지 않는 pushAll 메서드 - 문제가 있다.
public void pushAll(Iterable<E> src) {
    for (E e : src) {
        push(e);
    }
}
```

이 메서드는 깔끔하게 컴파일되고 제대로 동작할것같다. 스택이 `Stack<Number>`이라고 생각해보자 그러면 Integer 이나 Double이 제대로 들어가고 정상작동할 것이라고 생각한다. 다음 코드 처럼 말이다.

```java
Stack<Number> numberStack = new Stack<Number>();
Iterable<Integer> integers = ...;
numberStack.pushAll(integers);
```

하지만 실제로 해 보면 아래와 같은 오류 메시지가 출력된다. 

```java
StackTest.java: pushAll(Iterable<Number>) in Stack<Number>
Cannot be applied to (Iterable<Integer>)
numberStack.pushAll(integers);
           ＾
```

형인자 자료형은 불변이기 때문이다. 다행히도 이문제를 해결할 수 있다. 한정적 와일드 카드를 사용하면된다. pushAll의 인자 자료형을 "E의 Iterable"이 아니라 "E의 하위 자료형의 Iterable"이라고 명시하기 위해 `Iterable<? extends E>`를 사용한다.

```java
// E 객체 생산자 역할을 하는 인자에 대한 와일드카드 자료형
public void pushAll(Iterable<? extends E> src) {
    for (E e : src) {
        push(e);
    }
}
```

이렇게 만들면 전에 나오던 경고문이 사라질 뿐더러 형안전성이 확보된다. 이와 같이 popAll 메서드도 만들어 보자.

```java
// 와일드카드 자료형 없이 구현한 popAll 메서드 - 문제가 있다.
public void popAll(Collection<E> dst) {
    while (!isEmptu()) {
        dst.add(pop());
    }
}
```

이 메서드는 깔끔하게 컴파일될 뿐 아니라 인자로 주어진 컬렉션의 원소 자료형이 스택의 원소 자료형과 일치할 때는 완벽히 동작한다. 하지만 스택이 `Stack<Number>`이고 Object 형의 변수가 하나 있다고 하자. 스택에서 원소 하나를 꺼내서 해당 변수에 대입하는 코드는 오류 없이 컴파일하고 실행할 수 있다. 그렇다면 아래와 같은 코드도 만들 수 있어야 하지 않을까?

```java
Stack<Number> numberStack = new Stack<Number>();
Collection<Object> objects = ...;
numberStack.popAll(objects);
```

이 코드를 컴파일 해보면 pushAll 메서드에서 발생한 오류와 비슷한 오류가 발생한다. `Collection<Object>`가 `Collection<Number>`의 하위 자료형이 아니라는 오류가 난다는 것이다. 이 문제도 와일드카드 자료형을 쓰면 극복할 수 있다. popAll의 인자 자료형을 "E의 컬렉션"이 아니라 "E의 상위 자료형의 컬렉션"이라고 명시하는 것이다. 그러면 다음과 같이 코드를 수정해보자.

```java
// E의 소비자 구실을 하는 인자에 대한 와일드카드 자료형
public void popAll(Collection<? super E> dst) {
    while (!isEmpty()) {
        dst.add(pop());
    }
}
```

이렇게 바꾸면 Stack과 클라이언트 코드는 깔끔하게 컴파일된다. 유연성을 최대화 하려면 객체 생산자나 소비자 구실을 하는 메서드 인자의 자료형은 와일드카드 자료형으로 하라는 것이다. 이 때, 어떤 와일드카드 자료형을 쓸지 모르겠다면 다음을 참고하자.

PECS (Produce - Extends, Consumer - Super)

그러니까, 인자가 T 생산자라면 <? extends T>라고 하고 T 소비자라면 <? super T>라고 하라는 것이다. 연습으로 전에 만들었던 메서드들에 적용 시켜보자.

```java
static <E> reduce(List<E> list, FUnction<E> f, E initVal)

/*
list 인자를 E 생산자로만 사용하므로 와일드카드 자료형 "? extends E"로 선언
f는 생산자와 소비자가 동일하기 때문에 와일드카드 자료형을 사용하면 안된다.
*/
// E 생산자 구실을 하는 인자에 와일드카드 자료형 적용
static <E> E reduce(List<? extends E> list, Function<E> f, E initVal)
```

```java
public static <E> Set<E> union(Set<E> s1, Set<E> s2)

// 인자 모두가 전부 생산자 이므로 다음과 같이 고쳐진다.
public static <E> Set<E> union(Set<? extends E> s1, Set<? extends E> s2)
```

```java
public static <T extends Comparable<T>> T max(LIst<T> list)

// list는 T 객체의 생산자이므로 자료형을 List<? extends T>로 변경
// Comparable는 항상 소비자이므로 <? super T>로 변경
public static <T extends Comparable<? super T>> T max(List<? extends T> list)

// 이렇게 고치면 더 광범위하게 사용할 수 있다
List<ScheduledFuture<?>> scheduledFutures = ...;
```

원래 메서드가 위의 리스트에 적요될 수 없는 것은 ScheduledFuture가 `Comparable<ScheduledFuture>`를 구현하지 않기 때문이다. ScheduledFuture는 `Comparable<Delayed>`를 확장하는 Delayed의 하위 인터페이스이다. 다시 말해서, ScheduledFuture 객체는 단순히 다른 ScheduledFuture 객체들하고만 비교할 수 있는 것이 아니다. 어느 Delayed 객체와도 비교가 가능하다. 그래서 수정되기 전의 선언부로는 처리할 수 없는 것이다.

선언부를 수정하고 컴파일하게 되면 아래와 같은 오류가 뜬다.

```java
Max.java: incompatible types
found: Iterator<capture#591 of ? extends T>
required: Iterator<T>
Iterator<T> i = list.iterator();
                             ＾
```

이 오류는 list가 `List<T>`가 아니므로 iterator 메서드가 `Iterator<T>`를 반환하지 않는다는 뜻이다. 따라서 아래와 같이 내부를 수정하면 된다.

```java
public static <T extends Comparable<? super T>> T max(List<? extends T> list) {
    Iterator<? extends T> i = list.iterator();
    T result = i.next();
    while (i.hasNext()) {
        T t = i.next();
        if (t.compateTo(result) > 0) {
            result = t;
        }
    }
    return result;
}
```

반환값에는 와일드카드 자료형을 사용하면 안된다. 좀 더 유연한 코드를 만들 수 있도록 도와주기는커녕, 클라이언트 코드 안에도 와일드카드 자료형을 명시해야 하기 때문이다. 또한, 클래스 사용자가 와일드카드 자료형에 대해 고민하게 된다면, 그것은 아마도 클래스 API가 잘못 설계된 탓일 것이다.

# 형인자와 와일드카드 사이에 존재하는 이원성
상당수의 메서드를 선언하는 방법에는 두 가지 방법 중 어떤 것으로도 선언될 수 있다는 것이다. swap 메서드를 예제로 들어보자.

```java
public static <E> void swap(List<E> list, int i, int j);
public static void swap(List<?> list, int i, int j);
```

두 가지 방법 모두 옳은 방법이지만 public API를 만들고 있다면 두 번째 방법이 더 바람직하다. 형인자를 신경 쓸 필요가 없어져서 더 간단하기 때문이다. 원칙은, 형인자가 메서드 선언에 단 한군데 나타난다면 해당 인자를 와일드카드로 바꾸라는 것이다. 비한정적 형인자이면 비한정적 와일드카드로 바꾸고, 한정적 형인자이면 한정적 와일드카드로 바꾸라.

그런데 형인자 대신 와일드카드를 사용한 swap의 두 번째 선언에는 한 가지 문제가 있다. 당연해 보이는 코드가 컴파일되지 않는 것이다.

```java
public static void swap(List<?> list, int i, int j) {
    list.set(i, list.set(j, list.get(i)));
}
```

다음과 같은 오류가 발생한다.

```java
Swap.java: set(int, capture#282 of ?) in List<capture#282 of ?>
cannot be aplied to (int,Object)
list.set(i, list.set(j, list.get(i)));
                ＾
```

리스트에서 방금 꺼낸 원소를 그 리스트에 다시 넣을 수 없다는 것은 옳지 않아 보인다. 문제는 list의 자료형이 `List<?>`라는 것이다. `List<?>`에는 null 이외의 어떤 값도 넣을 수 없다. 다행히도 형 안전성이 보장되지 않는 형변환이나 무인자 자료형을 쓰지 않고도 이 문제를 해겨할 수 있다. private 도움 메서드를 이용해 와일드카드 자료형을 포착하는 것이 기본적인 아이디어이다. 이 도움 메서드는 제네릭 메서드로 정의해야 한다. 그래야 자료형을 포착할 수 있다.

```java
public static void swap(List<?> list, int i, int j)

// 와일드카드 자료형을 포착하기 위한 private 도움 메서드
private static <E> void swapHelper(List<E> list, int i, int j) {
    list.set(i, list.set(j, list.get(i)));
}
```

swapHelper 메서드는 list가 `List<E>`라는 것을 안다. 따라서 해당 리스트에서 꺼낸 값의 자료형은 E다. 그리고 E 형의 값은 리스트에 넣어도 안전하다. 조금 복잡해 보이지만 이 코드는 깔끔하게 컴파일된다. 