---
title: "[Effective Java] 무점검 경고를 제거하라"
date: 2019-10-29 23:19:42
categories:
- Effective Java
---

# 제네릭의 경고를 무시하지 마라!
제네릭을 사용하다 보면 무수히 많은 경고를 보게 된다. 무점검 형변환 경고, 무점검 메서드 호출 경고, 무점검 제네릭 배열 생겅 경고, 무점검 변환 경고 등이 있다.
아무리 고수라도 제네릭을 사용하여 만들어낸 코드가 깔끔하게 컴파일이 되기는 어렵다. 그래서 우리는 신경을 쓰면서 제네릭을 만들어야 하고 나중에 경고를 가능하다면 모두 지워주는게 좋다.
몇 가지는 컴파일러가 친절히 알려준다. 아래에 코드를 보자.

```java
Set<Lark> exaltation = new HashSete();
```

컴파일러는 에러 메시지를 보여준다.

```java
Practice.java warning: [unchecked] unchecked conversion
found : HashSet, required: Set<Lark>
Set<Lark> exaltation = new HashSet();
                    ＾
```

컴파일러가 가르쳐준대로 수정하면 경고는 사라진다.

```java
Set<Lark> exaltation = new HashSete<Lark>();
```

이렇게 가능하다면 모든 경고 메시지는 제거해주는 것이 좋다. 코드의 형 안전성이 보장되기 때문이다.

# @SupressWarnings("unchecked")
이 어노테이션의 사용은 최대한 자제해야한다. 형 안전성 증명도 없이 경고를 억제하면, 프로그램 안전성에 대해 그릇된 생각을 갖게 될 뿐이다. 컴파일은 되겠지만 프로그램 실행 도중에 ClassCastException을 만나게 된다면 어디서 어떻게 수정해야하는지 감이 안잡히게 될것이다.

만약 @SupressWarnings("unchecked") 어노테이션을 사용하게 된다면 가능한 한 작은 범위에 적용해야한다. 보통은 변수 선언이나, 아주 짧은 메서드 또는 생성자에 붙인다. 절대로 클래스 전체에 @SupressWarnings("unchecked")를 적용하지 말아라. 중요한 경고 메시지를 놓치는 원인이다.

길이가 한 줄 이상인 메서드나 생성자에 사용된 @SupressWarnings("unchecked") 어노테이션을 지역 변수 선언문 앞으로 이동시킬 수 있을 때가 있다. 그러려면 새로운 지역 변수 선언문을 작성해야 할 수도 있으나, 그럴만한 가치는 있다. 다음은 ArrayList 클래스에 toArray 메서드이다.

```java
public <T> T[] toArray(T[] a) {
    if (a.length < size) {
        return (T[]) Arrays.copyOf(elements, size, a.getClass());
    }
    System.arraycopy(elements, 0, a, 0, size);
    if (a.length > size) {
        a[size] = null;
    }
    return a;
}
```

ArrayList 클래스를 컴파일해 보면 아래와 같은 경고 메시지가 출력된다.

```java
Practice.java warning: [unchecked] unchecked cast
found : Object[], required: T[]
return (T[]) Arrays.copyOf(elements, size, a.getClass());
                          ＾
```

@SupressWarnings("unchecked") 어노테이션은 return문에 붙일 수 없다. 선언문이 아니기 때문이다. 메서드 전체에 붙이고 싶겠지만 절대로 그러면 안된다. 대신, 반환값을 담을 지역 변수를 선언한 다음에 해당 선언문 앞에 어노테이션을 붙여라.

```java
public <T> T[] toArray(T[] a) {
    if (a.length < size) {
        // 아래의 형변환은 배열의 자료형이 인자로 전달된 자료형인 T[]와
        // 같으므로 정확하다.
        @SupressWarnings("unchecked") T[] result = 
            (T[]) Arrays.copyOf(elements, size, a.getClass());

        return result;
    }
    System.arraycopy(elements, 0, a, 0, size);
    if (a.length > size) {
        a[size] = null;
    }
    return a;
}
```

이 메서드는 깔끔하게 컴파일 될 뿐 아니라 무점검 코드에 대한 경고 메시지의 억제 범위를 최소한으로 줄이고 있다.

그리고 @SupressWarnings("unchecked") 어노테이션을 사용할 때마다 왜 형 안전성을 위반하지 않는지 밝히는 주석을 반드시 붙여라. 다른 사람이 이해하기 좋고 형 안전성을 깨뜨릴 가능성이 줄어들게 된다.