---
title: "[Effective Java] 객체 생성을 막을 때는 private 생성자를 사용하라"
date: 2017-09-24 01:23:45
categories:
- Effective Java
---

# 객체 생성을 막을 때는 private 생성자를 사용하라
코딩을 하다 보면 상수나 배열 등을 모아둔 유틸리티 클래스를 만들 때도 있다. 이러한 클래스들은 객체가 생성되면 안된다. 생성자를 만들지 않으면 컴파일러가 public 생성자를 넣기 때문에 안되고 abstract로 만들면 이 클래스를 상속받는 하위 클래스들을 사용해서 객체를 만들 수 있기 때문에 안된다. 그래서 private 생성자를 두어 객체 생성을 막는다.

```java
public class PrivateConstructor {
    private PrivateConstructor() {
        throw new AssertionError();
    }
}
```
