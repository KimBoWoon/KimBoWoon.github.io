---
title: "[Effective Java] 추상 클래스 대신 인터페이스를 사용하라"
date: 2018-12-11 14:54:36
categories:
- Effective Java
---

# 추상 클래스와 인터페이스
추상 클래스는 메소드의 구현부가 포함 될 수 있지만 인터페이스는 그렇지 않다. 좀 더 큰 차이는 추상 클래스는 구현하기 위해 상속을 필요로 하지만 인터페이스는 그렇지 않기 때문에 다중상속을 만들어 낼 수 있다.

# 믹스인(Mixin)
인터페이스는 믹스인을 정의하는 데 이상적이다. 믹스인이란 이미 사용중이던 클래스에 쉽게 붙여서 기능을 확장하는 것을 말한다.
예를 들어 가수를 표현하는 인터페이스와 작곡가를 표현하는 인터페이스가 있다고 하자.

```java
public interface Singer {
    AudioClip sing(Song s);
}

public interface Songwriter {
    Song compose(boolean hit);
}
```

그런데 가수 가운데는 작곡가인 사람도 있다. 추상 클래스 대신 인터페이스를 사용해 자료형을 만들었으므로, 아무런 문법적 문제없이 Singer와 Songwriter를 동시에 구현하는 클래스를 만들 수 있다.

```java
public interface SingerSongwriter extends Singer, Songwriter {
    AudioClip strum();
    void actSensitive();
}
```

# 추상 클래스와 인터페이스의 결합
추상 클래스와 인터페이스를 같이 사용하면 두개의 장점 모두 사용할 수 있다. 인터페이스는 자료혀을 정의하고 구현하는 일은 골격 구현 클래스에게 맡기면 된다.

```java
// 골격 구현 위에서 만들어진 완전한 List 구현
static List<Integer> intArrayAsList(final int[] a) {
    if (a == null) {
        throw new NullPointerException();
    }
    
    return new AbstractList<Integer>() {
        public Integer get(int i) {
            return a[i];
        }
        
        @Override
        public Integer set(int i, Integer val) {
            int oldVal = a[i];
            a[i] = val; // auto unboxing
            return oldVal; // auto boxing
        }
        
        public int size() {
            return a.length;
        }
    }
}
```

이미 존재하는 List 구현체가 프로그래머를 위해 어떤 일을 할 수 있ㅇㄹ지 알고 싶은 사람에게 아주 유용할 것이다. (위 코드는 성능면에서는 좋지 않다.)
골격 구현 클래스를 만드는 작업은 상대적으로 멍청해 보일지도 모르겠지만 단순한 작업이다. 우선 인터페이스를 살펴보고 다른 메서드를 구현할 때 쓸 기본 메서드들을 추려 낸다. 그리고 이 기본 메서드들을은 골격 구현 클래스에 추상 메서드로 선언하고, 나머지 메서드들은 실제로 구현해서 넣는다.

```java
// 골격 구현
public abstract class AbstractMapEntry<K, V> implements Map.Entry<K, V> {
    // 기본 메서드
    public abstract K getKey();
    public abstract V getValue();
    
    // 변경 가능 맵에 들어갈 Entry는 이 메서드를 재정의해야 한다.
    public V setValue(V value) {
        throw new UnsupportedOperationException();
    }
    
    // Map.Entry.equals의 일반 규약 구현
    @Override
    public boolean equals(Object o) {
        if (o == this) {
            return true;
        }
        
        if (!(o instanceof Map.Entry)) {
            return false;
        }
        
        Map.Entry<?, ?> arg = (Map.Entry) o;
        
        return equals(getKey(), arg.getKey()) && equals(getValue(), arg.getValue());
    }
    
    private static boolean equals(Object o1, Object o2) {
        return (o1 == null) ? o2 == null : o1.equals(o2);
    }
    
    // Map.Entry.hashCode의 일반 규약 구현
    @Override
    public int hashCode() {
        return hashCode(getKey()) ^ hashCode(getValue());
    }
    
    private static int hashCode(Object obj) {
        return (obj == null) ? 0 : obj.hashCode();
    }
}
```