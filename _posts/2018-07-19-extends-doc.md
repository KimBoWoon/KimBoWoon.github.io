---
title: "[Effective Java] 계승을 위한 설계와 문서를 갖추거나, 그럴 수 없다면 계승을 금지하라"
date: 2018-07-19 01:23:45
categories:
- Effective Java
---

# 상속을 위해서는 어떤 문서를 만들어야 할까?
오버라이딩이 되는 메소드에 대해서는 내부적으로 어떻게 사용하지 문서에 남겨야한다. 좀 더 세부적으로 어떤 메소드를 호출하고 어떤 결과를 반환하는지를 명시하면 좋다.

# Java Doc Example
```text
public boolean remove(Object o)
Removes a single instance of the specified element from this collection, if it is present (optional operation). More formally, removes an element e such that (o==null ? e==null : o.equals(e)), if this collection contains one or more such elements. Returns true if this collection contained the specified element (or equivalently, if this collection changed as a result of the call).
This implementation iterates over the collection looking for the specified element. If it finds the element, it removes the element from the collection using the iterator's remove method.

Note that this implementation throws an UnsupportedOperationException if the iterator returned by this collection's iterator method does not implement the remove method and this collection contains the specified object.

Specified by:
remove in interface Collection<E>
Parameters:
o - element to be removed from this collection, if present
Returns:
true if an element was removed as a result of this call
Throws:
UnsupportedOperationException - if the remove operation is not supported by this collection
ClassCastException - if the type of the specified element is incompatible with this collection (optional)
NullPointerException - if the specified element is null and this collection does not permit null elements (optional)
```

java8 버전에 있는 java.util.AbstractCollection에 remove 메소드이다. 메소드에 설명을 보면 어떤 예외가 발생할 수 있는지에 대한 명시가 제대로 적혀있다.