---
title: "[Java] Singleton Pattern 잘쓰기"
date: 2017-12-06 01:23:45
categories:
- Java
---

# Eager Initialization
```java
class EagerInitialization {
    // private static 로 선언.
    private static EagerInitialization instance = new EagerInitialization();

    // 생성자
    private EagerInitialization() {
        System.out.println("call EagerInitialization constructor.");
    }

    // 조회 method
    public static EagerInitialization getInstance() {
        return instance;
    }

    public void print() {
        System.out.println("It's print() method in EagerInitialization instance.");
        System.out.println("instance hashCode > " + instance.hashCode());
    }
}
```
* private 접근제어자 때문에 이 인스턴스를 접근해서 사용하려면 getInstance() 메소드를 사용해야한다.
* 다른 클래스들이 이 클래스를 많이 사용하면 아마도 클래스가 로드되는 시점에서 인스턴스를 생성시키는데 무리가 있을 수 있다.
* 이 클래스의 인스턴스를 만들어 주는 과정에서 어떠한 예외 처리도 할 수 없다.

# Static Block Initialization
```java
class StaticBlockInitialization {
    private static StaticBlockInitialization instance;

    private StaticBlockInitialization() {
    }

    static {
        try {
            System.out.println("instance create..");
            instance = new StaticBlockInitialization();
        } catch (Exception e) {
            throw new RuntimeException("Exception creating StaticBlockInitialization instance.");
        }
    }

    public static StaticBlockInitialization getInstance() {
        return instance;
    }

    public void print() {
        System.out.println("It's print() method in StaticBlockInitialization instance.");
        System.out.println("instance hashCode > " + instance.hashCode());
    }
}
```
* static 초기화블럭을 이용하면 클래스가 로딩 될 때 최초 한번 실행하게 된다.
* 초기화블럭을 이용하면 logic을 담을 수 있기 때문에 복잡한 초기변수 셋팅이나 위와 같이 에러처리를 위한 구문을 담을 수 있다.
* 인스턴스가 사용되는 시점에 생성되는 것은 아니다.

# lazy initialization
```java
class LazyInitialization {

    private static LazyInitialization instance;

    private LazyInitialization() {
    }

    public static LazyInitialization getInstance() {
        if (instance == null)
            instance = new LazyInitialization();
        return instance;
    }

    public void print() {
        System.out.println("It's print() method in LazyInitialization instance.");
        System.out.println("instance hashCode > " + instance.hashCode());
    }
}
```
* new LazyInitialization(); 는 getInstance() method 안에서 사용되었다. if문을 이용해 instance가 null 인 경우에만 new를 사용해 객체를 생성한다.
* 최초 사용시점에만 인스턴스화 시키기 때문에 프로그램이 메모리에 적재되는 시점에 부담이 많이 줄게된다.
* 만약 프로그램이 muilti thread 방식이라면 위와 같은 singleton pattern은 안전하지 않다.

# Thread Safe Initalization
```java
class ThreadSafeInitalization {

    private static ThreadSafeInitalization instance;

    private ThreadSafeInitalization() {
    }

    public static synchronized ThreadSafeInitalization getInstance() {
        if (instance == null)
            instance = new ThreadSafeInitalization();
        return instance;
    }

    public void print() {
        System.out.println("It's print() method in ThreadSafeInitalization instance.");
        System.out.println("instance hashCode > " + instance.hashCode());
    }
}
```
* muilit thread문제를 해결하기 위해 synchronized(동기화)를 사용하여 singleton pattern을 구현했다.
* 수 많은 thread 들이 getInstance() method 를 호출하게 되면 높은 cost 비용으로 인해 프로그램 전반에 성능저하가 일어난다.

# Initialization On Demand Holder Idiom
```java
class InitializationOnDemandHolderIdiom {

    private InitializationOnDemandHolderIdiom() {
    }

    private static class Singleton {
        private static final InitializationOnDemandHolderIdiom instance = new InitializationOnDemandHolderIdiom();
    }

    public static InitializationOnDemandHolderIdiom getInstance() {
        System.out.println("create instance");
        return Singleton.instance;
    }

    public void print() {
        System.out.println("InitializationOnDemandHolderIdiom");
    }
}
```
* jvm 의 class loader의 매커니즘과 class의 load 시점을 이용하여 내부 class를 생성시킴으로 thread 간의 동기화 문제를 해결한다.
* initialization on demand holder idiom 역시 lazy initialization이 가능하며 모든 java 버젼과, jvm에서 사용이 가능하다.
* java 에서 singleton 을 생성시킨다고 하면 거의 위의 방법을 사용한다고 보면 된다.

# Enum Initialization
```java
enum EnumInitialization {
    INSTANCE;
    static String test = "";

    public static EnumInitialization getInstance() {
        test = "test";
        return INSTANCE;
    }
}
```
* INSTANCE 가 생성될 때, multi thread 로 부터 안전하다. (추가된 methed 들은 safed 하지 않을 수도 있다.)
* 단 한번의 인스턴스 생성을 보장한다.
* 사용이 간편하다.
* enum value는 자바 프로그램 전역에서 접근이 가능하다.

# Using Reflection To Destroy Singleton
```java
class UsingReflectionToDestroySingleton {

    public static void main(String[] args) {
        EagerInitialization instance = EagerInitialization.getInstance();
        EagerInitialization instance2 = null;

        try {
            Constructor[] constructors = EagerInitialization.class.getDeclaredConstructors();
            for (Constructor constructor : constructors) {
                constructor.setAccessible(true);
                instance2 = (EagerInitialization) constructor.newInstance();
            }
        } catch (Exception e) {

        }

        System.out.println(instance.hashCode());
        System.out.println(instance2.hashCode());

    }
}
```
* 위의 코드를 실행해보면 아래 System.out.println();의 두 라인에서 찍히는 hachCode()값이 다른 것을 확인 할 수 있다.
* java의 reflection은 매우 강력하다. 설령 class 의 생성자가 private 일지라도 강제로 가져와서 새로운 인스턴스 생성이 가능하다.
* 결국 singleton pattern을 깨뜨리는 것이다. 이 외에도 reflection을 여러곳에서 사용할 수 있으니 알아두는 것이 좋다.

# 참고
* https://blog.seotory.com/post/2016/03/java-singleton-pattern