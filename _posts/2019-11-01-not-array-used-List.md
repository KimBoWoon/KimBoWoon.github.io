---
title: "[Effective Java] 배열 대신 리스트를 써라"
date: 2019-11-01 23:19:42
categories:
- Effective Java
---

# 배열과 제네릭의 차이
첫번째로는 배열은 공변 자료형이다. Sub가 Super의 하위 자료형 이라면 Sub[]도 Super[]의 하위 자료형이라는 것이다. 반면에 제네릭은 불변 자료형이다. Type1과 Type2가 있을 때 `List<Type1>`은 `List<Type2>`의 상위 자료형이나 하위 자료형이 될 수 없다. 아래의 코드를 보자

```java
// 실행중에 문제를 일으킴
Object[] objectArray = new Long[1];
objectArray[0] = "I don't fit in";
```

하지만 아래의 코드는 컴파일 조차 안된다.

```java
List<Object> ol = new ArrayList<Long>; // 자료형 불일치
ol.add("I don't fit in");
```

둘 중 어떤 방법을 써도 Long 객체 컨터이너 안에 String을 넣을 수 없다. 그러나 배열을 쓰면 실수를 저지른 사실을 프로그램 실행 중에나 알 수 있다. 

두번째로는 배열은 실체화 되는 자료형이라는 것이다. 즉, 배열의 각 원소의 자료형은 실행시간에 결정된다는 것이다. 반면 제네릭은 삭제 과정을 통해 구현된다. 즉, 자료형에 관계된 조건들은 컴파일 시점에만 적용되고, 그 각 원소의 자료형 정보는 프로그램이 실행될 때는 삭제된다는 것이다. 

이러한 차이점 때문에 배열과 제네릭은 섞어 쓰기 어렵다. `List<String>[]`, `new E[]` 와 같은 것을 만들려고 하면 제네릭 배열 생성이라는 오류가 발생한다. 제네릭 배열이 생성되지 않는 이유는 형 안전성이 보장되지 않기 때문이다. 아래의 코드를 보자.

```java
// 제네릭 배열 생성이 허용되지 않는 이유 - 아래의 코드는 컴파일되지 않는다!
List<String>[] stringLists = new List<String>[1];   // (1)
List<Integer> intList = Arrays.asList(42);          // (2)
Object[] objects = stringLists;                     // (3)
objects[0] = intList;                               // (4)
String s = stringLists[0].get(0);                   // (5)
```

(1)이 문제없이 컴파일이 된다면, 제네릭 배열이 만들어질 것이다. (2)는 하나의 원소를 갖는 배열 `List<Integer>`를 초기화한다. (3)은 `List<String>` 배열을 Object 배열 변수에 대입한다. 배열은 공변 자료형이므로, 가능하다. (4) 에서는 `List<Integer>`를 Object 배열에 있는 유일한 원소에 대입한다. 제네릭이 형 삭제를 통해 구현되므로 여기에도 하자는 없다. 하지만 `List<String>`만 들어간다고 만들어둔 배열에 `List<Integer>`을 저장한것이다. (5)에서는 저장된 원소를 꺼내는 작업을 하는데 사실을 Integer값을 꺼내서 String으로 변환하려는 것이다. 그래서 ClassCastException이 발생한다. 그래서 컴파일러는 (1)에서 오류를 발생시키는 것이다. 

작성중...