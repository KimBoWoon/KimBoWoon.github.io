---
title: "[Effective Java] 클래스와 멤버의 접근 권한은 최소화하라"
date: 2017-12-15 01:23:45
categories:
- Effective Java
---

# 정보은닉과 캡슐화
잘 설계된 모듈과 그렇지 못한 모듈을 구별하는 방법 중에 하나는 모듈 내부의 데이터를 비롯한 여러 구현 세부사항을 다른 모듈에 잘 감추느냐의 여부다. 잘 설계된 모듈을 모두 API 뒤편으로 감춘다. 그리고 모듈은 API를 통해서만 서로 통신한다. 또한 내부적으로는 무슨 짓을 하는지는 신경 쓰지 않는다. 이 개념이 바로 정보은닉과 캡슐화이다. 정보은닉은 여러모로 중요하다. 대부분이 정보은닉이 시스템을 구성하는 모듈 사이의 의존성을 낮춰서 각자 개별적으로 개발하고 시험하고 최적화하고 이해하고 변경할 수 있도록 한다는 사실에 기초한다. 그렇게 되면 병렬적으로 개발할 수 있기 때문에 개발 속도가 올라간다. 이러한 장점을 지키위해서 행해야 하는것은 정말 간단하다. 각 클래스의 멤버는 가능한 한 접근 불가능하도록 만들면 된다.

# java 접근자
* private : 최상위 레벨 클래스 내부에서만 접근 가능
* package-private : 간은 패키지 내의 아무 클래스나 사용 가능, 기본 접근 권한으로 알려져 있다.
* protected : 해당하는 클래스와 이 클래스를 상속받은 클래스들 내에서 사용이 가능하다.
* public : 언제 어디서는 사용이 가능하다.

public은 항상 조심해야한다. 스레드에도 안전하지 않고 누구나 접근이 가능해서 값이 변경되기 때문이다. 또한 길이가 0이 아닌 배열은 언제나 변경 가능하므로, public static final 배열 필드를 두거나, 배열 필드를 반환하는 접근자를 정의하면 안 된다. 클라이언트가 배열 내용을 변경할 수 있기 때문에 보안에 문제가 생긴다. 따라서 다음과 같이 사용하는게 좋다.

public으로 선언되었던 배열은 private로 바꾸고, 변경이 불가능한 puvlic 리스트를 하나 만드는 것이다.
```java
private static final Integer[] PRIVATE_VALUES = {};
public static final List<Integer> VALUES = Collections.unmodifiableList(Arrays.asList(PRIVATE_VALUES));
```

배열은 private 으로 선언하고, 해당 배열을 복사해서 반환하는 public 메소드를 하나 추가하는 것이다.
```java
private static final Integer[] PRIVATE_VALUES = {};
public static final Integer[] values() {
    return PRIVATE_VALUES.clone();
}
```

두 방법 가운데 하나를 택할 때는 클라이언트가 어떤 작업을 하길 원하는지 따져야 한다.