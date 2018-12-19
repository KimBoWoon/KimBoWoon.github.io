---
title: "[Effective Java] 인터페이스는 자료형을 정의할 때만 사용하라"
date: 2018-12-19 20:27:30
categories:
- Effective Java
---

# 인터페이스 사용의 부적절한 예
인터페이스는 인스턴스를 만들지는 못하지만 인터페이스 자료형으로 해당하는 인터페이스를 구현하고 있는 클래스는 모두 참조가 가능하다.
이 외의 다른 목적으로 사용은 바람직하지 않다. 그 예로 상수 인터페이스가 있다.
```java
// 상수 인터페이스 안티패턴 - 사용하지 말 것!
public interface PhysicalConstants {
    // 아보가드로 수(1 / mod)
    static final double AVOGADROS_NUMBER = 6.02214199e23;
    
    // 볼쯔만 상수 (j / k)
    static final double BOLTZMANN_CONSTANT = 1.3806503e-23;
    
    // 전자 질량 (kg)
    static final double ELECTRON_MASS = 9.10938188e-31;
}
```

클래스가 어떤 상수를 사용한다는 것은 구현의 세부 사항이고 클라이언트가 이를 알아둘 필요가 없다. 클라이언트를 혼란 시킬것이다.

이를 해결할 방법이 있다. 상수가 해당 클래스에 강하게 연결 되어 있을 경우 그 상수들을 클래스에 추가 시킨다.
Integer 클래스에 MAX_VALUE, MIN_VALUE 처럼 말이다.

```java
public class PhysicalConstants {
    private PhysicalConstants() {} // 객체의 생성을 막음
    
    public static final double AVOGADROS_NUMBER = 6.02214199e23;
    public static final double BOLTZMANN_CONSTANT = 1.3806503e-23;
    public static final double ELECTRON_MASS = 9.10938188e-31; 
}
```