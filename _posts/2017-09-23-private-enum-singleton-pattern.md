---
title: "[Effective Java] private 생성자나 enum 자료형은 싱글턴 패턴을 따르도록 설계하라"
date: 2017-09-23 01:23:45
categories:
- Effective Java
---

# private 생성자나 enum 자료형은 싱글턴 패턴을 따르도록 설계하라
싱글턴 패턴은 객체를 하나만 만들 수 있는 클래스이다. 창 관리자나 파일 시스템같은 것들이 그 예이다. 하지만 AccessibleObject.setAccessible 메소드의 도움을 받아 권한을 획득한 클라이언트는 리플렉션 기능을 통해 private 생성자를 다시 호출할 수 있다. 이러한 공격을 방어하려면 두 번째 객체를 생성하라는 요철을 받으면 예욀르 던지도록 생성자를 고쳐야 한다. 또 다른 방법은 public으로 선언된 정적 팩토리 메소드를 이용한다. 정적 팩토리 메소드를 이용해서 첫 번째 방법으로 공격하면 막지 못한다.
```java
public class Book {
    private static final Book INSTANCE = new Book();

    private Book() {
		...
    }

    public static Book getInstance() {
        return INSTANCE;
    }
}
```

싱글턴 클래스를 직렬화 가능 클래스로 만들려면 클래스 선언에 implements Serializable을 추가하는 것으로는 부족하다. 모든 필드를 transient로 선언하고 readResolve 메소드를 추가해야한다. 그렇지 않으면 serialize된 객체가 역직화 될 때마다 새로운 객체가 생기게 된다.
```java
// 싱글턴 상태를 유지하기 위한 readResolve 구현
private Object readResolve() {
	// 동일한 Book 객체가 반환되도록 하는 동시에, 가짜 Book 객체는
    // 쓰레기 수집기가 처리하도록 만든다.
	return INSTANCE;
}
```

jdk 1.5부터는 싱글턴을 구현할 때 새로운 방법을 사용할 수 있다. 원소가 하나뿐인 enum자료형을 정의하는 것이다.
```java
public enum Book {
	INSTANCE;
    
    public void readBook() {...}
}
```

이렇게 만들면 직렬화가 자동으로 처리된다는 것이다. 여러 객체가 생길일이 없고 리플렉션을 통한 공격에도 안전하다. 원소가 하나뿐인 enum 자료형이야말로 싱글턴을 구현하는 가장 좋은 방법이다. 클래스를 싱글턴으로 만들면 클라이언트를 테스트하기가 어려워질 수가 있다. 싱글턴이 어떤 인터페이스를 구현하는 것이 아니면 가짜 구현으로 대체할 수 없기 때문이다.
