---
title: "[Effective Java] 계승하는 대신 구성하라"
date: 2018-07-18 01:23:45
categories:
- Effective Java
---

# 계승
상속을 하게 되면 코드를 재활용하고 다형성을 사용할 수 있다는 점은 좋은 이점이지만 단점이 더 많다.

# 어떤 단점?
```java
public class InstrumentedHashSet<E> extends HashSet<E> {
    private int addCount = 0;

    public InstrumentedHashSet() {}

    public InstrumentedHashSet(int initCap, float loadFactor) {
        super(initCap, loadFactor);
    }

    @Override
    public boolean add(E e) {
        addCount++;
        return super.add(e);
    }

    @Override
    public boolean addAll(Collection<? extends E> c) {
        addCount += c.size();
        return super.addAll(c);
    }

    public int getAddCount() {
        return addCount;
    }
}
```

다음과 같이 HashSet<E> 클래스를 상속받아 데이터가 몇번 삽입 되는지 확인하는 코드가 있다. add를 사용하여 데이터를 삽입하면 정상적으로 작동하지만 다음과 같은 코드는 문제가 발생 하게 된다.

```java
InstrumentedHashSet<Integer> i = new InstrumentedHashSet<>();
i.addAll(Arrays.asList(1, 2, 3));
```

실행하고 결과를 보면 6이라는 결과 값이 나오게 된다. 왜 그러냐면 HashSet<E>의 addAll은 add를 통해서 데이터를 삽입된다. 처음 addAll을 호출하고 addCount += c.size();를 통해 3이 증가하고 상위 클래스의 addAll를 호출해 데이터를 삽입하면 다시 add를 호출하게 되어서 addCount++ 때문에 또 증가되기 때문이다. 이처럼 상위 클래스가 어떻게 작성되었는지 정확하게 모르면 어이없는 실수를 범하게 될것이다.
또 다른 문제는 상위 클래스가 변경되면 그에 따라 하위 클래스가 제대로 작동할지 보장되지 않는다.

# 그럼 어떻게?
새롭게 작성될 클래스에 기존 클래스를 private 멤버로 두는 것이다. 이런 설계 기법을 구성이라고 부르는데 기존 클래스가 새 클래스의 일부가 되기 때문이다. 

```java
// 계승 대신 구성하는 클래스
public class InstrumentedSet<E> extends ForwardingSet<E> {
    private int addCount = 0;

    public InstrumentedSet(Set<E> s) {
        super(s);
    }

    @Override
    public boolean add(E e) {
        addCount++;
        return super.add(e);
    }

    @Override
    public boolean addAll(Collection<? extends E> c) {
        addCount += c.size();
        return super.addAll(c);
    }

    public int getAddCount() {
        return addCount;
    }
}

// 재사용 가능한 전달 클래스
public class ForwardingSet<E> implements Set<E> {
    private final Set<E> s;

    public ForwardingSet(Set<E> s) {
        this.s = s;
    }

    @Override
    public int size() {
        return s.size();
    }

    @Override
    public boolean isEmpty() {
        return s.isEmpty();
    }

    @Override
    public boolean contains(Object o) {
        return s.contains(o);
    }

    @Override
    public Iterator<E> iterator() {
        return s.iterator();
    }

    @Override
    public Object[] toArray() {
        return s.toArray();
    }

    @Override
    public <T> T[] toArray(T[] a) {
        return s.toArray(a);
    }

    @Override
    public boolean add(E e) {
        return s.add(e);
    }

    @Override
    public boolean remove(Object o) {
        return s.remove(o);
    }

    @Override
    public boolean containsAll(Collection<?> c) {
        return s.containsAll(c);
    }

    @Override
    public boolean addAll(Collection<? extends E> c) {
        return s.addAll(c);
    }

    @Override
    public boolean retainAll(Collection<?> c) {
        return s.retainAll(c);
    }

    @Override
    public boolean removeAll(Collection<?> c) {
        return s.removeAll(c);
    }

    @Override
    public void clear() {
        s.clear();
    }
}
```

계승을 사용하게 되면 어떤 한 클래스에서만 적용이 가능하고, 상위 클래스 생성자 마다 별도의 생성자를 만들어야 한다. 하지만 포장 클래스 기법을 사용하면 어떠한 Set 클래스에 대해 구현할 수 있고 이미 있는 생성자도 그대로 사용할 수 있다. 

# 그럼 상속은 언제?
옛날 부터 수 없이 들어 왔던 말이 있다 상속을 하기 위한 최선의 조건은 "IS-A"조건이 이다. 이 조건을 다시 한번 생각하면서 상속을 해야할지 포장을 해야할지 생각해야한다.