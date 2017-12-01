---
title: "[Effective Java] equals를 재정의할 때는 반드시 hashCode도 재정의하라"
date: 2017-12-01 01:23:45
categories:
- Effective Java
---

# equals를 재정의할 때는 반드시 hashCode도 재정의하라
equals 메소드가 같은 객체라는 것을 확인하더라고 hashCode가 다르면 hash로 만들어진 다른 자료구조에서 null이라는 값을 보낸다. 이처럼 hashCode를 재정의 하지 않음으로서 많은 오류가 생길 수 있다.

# 문제점
```java
public class HashCodeOverride {
    public static void main(String[] args) {
        Map<PhoneNumber, String> m = new HashMap<PhoneNumber, String>();
        m.put(new PhoneNumber(707, 867, 5309), "Jenny");

        System.out.println(m.get(new PhoneNumber(707, 867, 5309)));
    }
}

final class PhoneNumber {
    private final short areaCode;
    private final short prefix;
    private final short lineNumber;

    public PhoneNumber(int areaCode, int prefix, int lineNumber) {
        rangeCheck(areaCode, 999, "area code");
        rangeCheck(prefix, 999, "prefix");
        rangeCheck(lineNumber, 9999, "lineNumber");

        this.areaCode = (short) areaCode;
        this.prefix = (short) prefix;
        this.lineNumber = (short) lineNumber;
    }

    private static void rangeCheck(int arg, int max, String name) {
        if (arg < 0 || arg > max) {
            throw new IllegalArgumentException(name + ": " + arg);
        }
    }

    @Override
    public boolean equals(Object o) {
        if (o == this) {
            return true;
        }
        if (!(o instanceof PhoneNumber)) {
            return false;
        }
        PhoneNumber pn = (PhoneNumber) o;
        return pn.lineNumber == lineNumber && pn.prefix == prefix && pn.areaCode == areaCode;
    }

    @Override
    public int hashCode() {
        int result = 17;
        result = 31 * result + areaCode;
        result = 31 * result + prefix;
        result = 31 * result + lineNumber;
        return result;
    }
}
```

여기서 사용한 PhoneNumber 객체는 두가지 이다. 하나는 put 메소드에 사용하고 다른 하나는 get 함수에 사용했다. hashCode가 재정의가 안되어 있으면 논리적으로 같은 객체라 하더라도 null값을 반환할것이다. 그렇기 때문에 hashCode를 꼭 재정의 해야한다.

# 어떻게 오버라이딩을 해야할까?
hashCode를 완벽하게 재정의 하는 법은 없다. 없다기 보다는 아직까지 확실한 연구결과가 나오지 않았다. 그렇기 때문에 제곱법이나 제산법, 폴딩법등이 있다.