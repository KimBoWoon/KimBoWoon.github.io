---
title: "[Effective Java] public 클래스 안에는 public 필드를 두지 말고 접근자 메소드를 사용하라"
date: 2017-12-19 01:23:45
categories:
- Effective Java
---

# 접근자 메소드
어느 클래스는 필드만 가지고 있는 클래스가 있다. 이 클래스에는 접근자 메소드가 있어야 여러모로 좋다. 캡슐화의 이점을 누릴 수 없고 API를 변경하지 않고서는 내부 표현을 변경할 수도 없다. 또한 불변식도 강제할 수 없고, 필드를 사용하는 순간에 어떤 종작이 실행되록 만들 수도 없다. 그래서 다음과 같이 접근자 메소드를 가져야한다.
```java
public class Point {
	private double x;
    private double y;
    
    public Point(double x, double y) {
    	this.x = x;
        this.y = y;
    }
    
    public getX() {
    	return x;
    }
    
    public getY() {
    	return y;
    }
    
    public setX(double x) {
    	this.x = x;
    }
    
    public setY(double y) {
    	this.y = y;
    }
}
```

이렇게 만들어야 클래스 내부표현을 자유로이 수정할 수 있게 된다.