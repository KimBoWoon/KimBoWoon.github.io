---
title: "[Effective Java] 변경 가능성을 최소화하라"
date: 2017-12-19 01:23:45
categories:
- Effective Java
---

# 변경 불가능 클래스
변경 불가능 클래스는 말 그대로 내부의 정보를 수정할 수 없는 클래스이다. 객체 내부의 정보는 객체가 생성될 때 주어진 것이며 객체가 만들어 지고 없어질 때까지 변경되지 않는다. 이러한 클래스를 만드는 이유는 다양하다. 변경 가능 클래스보다 설계 및 구현 하기가 쉽고, 오류가 적으며, 사용하기도 쉽다.

# 변경 불가능 클래스를 만들기 위한 규칙
1. 객체 상태를 변경하는 메소드를 제공하지 않는다.
2. 상속하지 못하게 final로 클래스를 만든다. 상속을 받아서 클래스를 정의하게 되면 객체 상태가 변경된 것처럼 동작할 수 있기 때문이다.
3. 모든 필드를 final로 선언한다. 시스템이 강제하는 형태대로 프로그래머의 의도가 분명히 드러나고 새로 생성된 객체에 대한 참조가 동기화 없이 다른 스레드로 전달 되어도 안전하기 때문이다.
4. 모든 필드를 private로 선언한다. 이 객체를 사용하는 다른 사용자가 필드를 변경할 수 없기 때문이다.
5. 변경 가능 컴포넌트에 대한 독점적 접근권을 보장한다. 클래스에 포함된 변경 가능 객체에 대한 참조를 클라이언트는 획득할 수 없어야 한다.

# 변경 불가능 클래스 예시
```java
public final class Complex {
    // 많이 사용하는 것들은 미리 만들어서 사용하는 것이 좋다
    public static final Complex ZERO = new Complex(0, 0);
    public static final Complex ONE = new Complex(1, 0);
    public static final Complex I = new Complex(0, 1);

    private final double re;
    private final double im;

    public Complex(double re, double im) {
        this.re = re;
        this.im = im;
    }

    // 대응되는 수정자가 없는 접근자들
    public double realPart() {
        return re;
    }

    public double imginaryPart() {
        return im;
    }

    public Complex add(Complex c) {
        return new Complex(re + c.re, im + c.im);
    }

    public Complex subtract(Complex c) {
        return new Complex(re - c.re, im - c.im);
    }

    public Complex multiply(Complex c) {
        return new Complex(re * c.re - im * c.im, re * c.im + im * c.re);
    }

    public Complex divide(Complex c) {
        double tmp = c.re * c.re + c.im * c.im;
        return new Complex((re * c.re + im * c.im) / tmp, (im * c.re - re * c.im) / tmp);
    }

    @Override
    public boolean equals(Object o) {
        if (o == this) {
            return true;
        }
        if (!(o instanceof Complex)) {
            return false;
        }

        Complex c = (Complex) o;

        return Double.compare(re, c.re) == 0 && Double.compare(im, c.im) == 0;
    }

    @Override
    public int hashCode() {
        int result = 17 + hashDouble(re);
        result = 31 * result + hashDouble(im);
        return result;
    }

    private static int hashDouble(double val) {
        long longBits = Double.doubleToLongBits(val);
        return (int) (longBits ^ (longBits >>> 32));
    }

    @Override
    public String toString() {
        return "(" + re + " + " + im + "i)";
    }
}
```
* 간단하게 작성된 것이며 실제로 사용하기에는 부적합할 것이다.

# 단점
유일한 단점은 만약 반환해야 한다면 새로운 객체를 만들어서 반환해햐 한다. 이러면 많은 메모리가 낭비 될수도 있다.