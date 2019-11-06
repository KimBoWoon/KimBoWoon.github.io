---
title: "[Effective Java] 가능하면 제네릭 메서드로 만들 것"
date: 2019-11-04 21:19:42
categories:
- Effective Java
---

# 메서드 제네릭화
제네릭화 혜택을 받는 것는 클래스 뿐만이 아니다 메서드도 혜택을 받는다. 부가적 기능을 제공하는 static 유틸리티 메서드는 특히 제네릭화하기 좋은 후보다. 두 집합의 합집합을 반환하는 메서드를 만들어 보자.

```java
// 무인자 자료형 사용 - 권할 수 없는 방법
public static Set union(Set s1, Set s2) {
    Set result = new HashSet(s1);
    result.addAll(s2);
    return result;
}
```

컴파일은 되지만 경고 메시지가 나온다.

```java
Union.java: warning: [unchecked] unchecked call to
HashSet(Collection<? extends E>) as a member of raw type HashSet
Set result = new HashSet(s1);
            ＾
Union.java: warning: [unchecked] unchecked call to
addAll(Collection<? extends E>) as a member of raw type Set
result.addAll(s2);
             ＾
```

경고를 없애고 형 안전성이 보장된 메서드를 구현해보자.

```java
// 무인자 자료형 사용 - 권할 수 없는 방법
public static <E> Set<E> union(Set<E> s1, Set<E> s2) {
    Set<E> result = new HashSet<E>(s1);
    result.addAll(s2);
    return result;
}
```

컴파일러는 전달받은 인자를 보고 E의 자료형을 유추한다. 지금은 `Set<String>`이므로 E가 String인것을 알 수 있다. 이 과정을 자료형 유추라고 한다. 이 과정을 응용하면 더욱 간결한 코드를 만들 수 있다. 아래를 보자.

```java
// 생성자를 통한 형인자 자료형 객체 생성
Map<String, List<String>> anagrams = new HashMap<String, List<String>>();
```

왼쪽과 오른쪽에 똑같이 적어줘야 한다는 불편함이 있다. 제네릭 정적 팩터리 메서드를 만들면 한결 깔끔해진다.

```java
// 제네릭 정적 팩터리 메서드 - 아무 인자도 받지 않는 HashMap 생성자
public static <K, V> HashMap<K, V> newHashMap() {
    return new HashMap<K, V>();
}
```

이 생성자를 사용하면 중복되는 형인자를 제거하여 간결한 코드를 만들 수 있다.

```java
// 정적 팩터리 메서드를 통한 형인자 자료형 객체 생성
Map<String, List<String>> anagrams = newHashMap();
```

Java 1.6 버전까지는 이렇게 사용했지만 지금은 생성자에도 자료형 추론이 지원된다.

# 제네릭 싱글턴 패턴
가령 T 형의 값을 받고 반환하는 함수를 나타내는 인터페이스가 있다고 하자.

```java
public interface UnaryFunction<T> {
    T apply(T arg);
}
```

항등함수를 구현하면 아래와 같이 될것이다. 항등함수는 무상태 함수이므로, 필요할 때마다 새 함수를 만드는 것은 낭비이다. 제네릭이 실체화 되는 자료형이었다면 자료형마다 별도의 항등함수가 필요하겠지만, 자료형 정보는 컴파일이 끝나면 삭제된다는 점을 이용하면 제네릭 싱글턴 하나만 있으면 된다.

```java
private static UnaryFunction<Object> IDENTITY_FUNCTION = 
    new UnaryFunction<Object>() {
        public Object apply(Object arg) { 
            return arg; 
        }
    };

// IDENTITY_FUNCTION은 무상태 객체고 형인자는 비한정 인자이므로
// 모든 자료형이 같은 객체를 공유해도 안전하다.
@SuppressWarnings("unchecked")
public static <T> UnaryFunction<T> identityFunction() {
    return (UnaryFunction<T>) IDENTITY_FUNCTION;
}
```

인자를 수정 없이 반환하므로, T가 무엇이건 간에 `UnaryFunction<T>`인 것처럼 써도 형 안전성이 보장된다. 아래의 몇가지 예제 프로그램을 적어두겠다.

```java
// 제네릭 싱글턴 사용 예제
public static void main(String[] args) {
    String[] strings = { "jute", "hemp", "nylon" };
    UnaryFunction<String> sameString = identityFunction();
    for (String s : strings) {
        System.out.println(sameString.apply(s));
    }

    Number[] numbers = { 1, 2.0, 3L };
    UnaryFunction<Number> sameNumber = identityFunction();
    for (Number n : numbers) {
        System.out.println(sameNumber.apply(n));
    }
}
```

# 재귀적 자료형 한정
상대적으로 사용 빈도가 낮긴 하나, 형인자가 포함된 표현식으로 형인자를 한정하는 것도 가능하다. 재귀적 자료형 한정은 Comparable 인터페이스와 함께 가장 흔히 쓰인다.

```java
public interface Comparable<T> {
    int compareTo(T o);
}
```

이 인터페이스의 형인자 T는 `Comparable<T>`를 구현하는 자료형의 객체와 비교 가능한 객체의 자료형이다. 실제로 거의 모든 자료형은 같은 자료형 객체하고만 비교 할 수 있다. 이 인터페이스를 구현하는 원소들의 리스트를 인자로 받는 메서드는 많다. 정렬, 탐색, 최댓값 및 최솟값을 계산하기도 한다. 그런데 그런 작업이 가능하려면 리스트 내의 원소들이 서로 비교 가능해한다. 이런 조건을 어떻게 표현하는지 알아보자.

```java
// 리스트의 최대 값 반환 - 재귀적 자료형 한정 사용
public static <T extends Comparable<T>> T max(List<T> list) {
    Iterator<T> i = list.iterator();
    T result - i.next();
    while (i.hasNext()) {
        T t = i.next();
        ir (t.compareTo(result) > 0) {
            result = t;
        }
    }
    return result;
}
```

`<T extends Comparable<T>>` 이 부분은 "자기 자신과 비교 가능한 모든 자료형 T"라는 뜻으로 읽을 수 있다. 상호 비교 가능성이라는 개념을 어느 정도 정확하게 표현하는 문장이다.

재귀적 자료형 한정은 이보다 복잡하게 쓰일 수도 있으나, 다행히도 흔한 일은 아니다. 위에 모두를 이해하고 와일드카드 용법을 이해하게 되면 실무에서 만나는 재귀적 자료형 한정 용법 상당수를 다룰 수 있게 될 것이다.