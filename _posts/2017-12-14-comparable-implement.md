---
title: "[Effective Java] Comparable 구현을 고려하라"
date: 2017-12-14 01:23:45
categories:
- Effective Java
---

# Comparable
이 인터페이스는 다른 인터페이스와 달리 하나의 메소드를 가지고 있다. compareTo 메소들을 가지고 있는데 이 메소드는 Object 클래스에는 포함되어 있지 않으며 오직 Comparable만이 가지고 있다. Comparable을 구현하는 클래스의 객체들은 모두 자연적 순서를 가지고 있다. 또한 최대/최소치를 계산하기도 간단하며, 그 컬렉션을 정렬된 상태로 유지하도 쉽다. 그리고 다양한 제네릭 알고리즘 및 Comparable 인터페이스를 이용하도록 작성된 컬렉션 구현체와도 전부 연동할 수 있다.

# compareTo 메소드의 일반 규약
* 자신과 주어진 객체를 비교하는데 자신이 주어진 객체보다 작으면 음수를, 같으면 0, 크다면 양수를 반환한다. 만약 비교 불가능한 객체가 전달된다면 ClassCastException 예외를 던진다.
* 모든 x와 y에 대해 x.compareTo(y) == y.compareTo(x)는 같아야 한다. 만약 예외를 던진다면 역도 반드시 그래야한다.
* compareTo를 구현할 때는 추이성이 만족되도록 해야한다. (x.compareTo(y) > 0 && y.compareTo(z) > 0 이면 x.compareTo(z) > 이어야한다.)
* x.compareTo(y) == 0 이면 x.compareTo(z) == y.compareTo(z)의 관계가 모든 z에 대해 성립해야한다.
* (x.compareTo(y) == 0) == x.equals(y) 이 규약은 강력히 추천하지만 절대적으로 요구되는 것이 아니다. 일반적으로, Compareable 인터페이스를 구현하면서 이 조건을 만족하지 않는 클래스는 반드시 그 사실을 명시해야한다.

이 규약들은 모두 한 클래스 객체 사이에 자연적 순서가 존재하기만 하면 어떤 것이든 이 규약을 만족할 깃이다.

# 코드
```java
public class ComparablePractice {
    public static void main(String[] args) {
        Fruit[] fruits = new Fruit[5];

        fruits[0] = new Fruit("사과", 10);
        fruits[1] = new Fruit("오렌지", 34);
        fruits[2] = new Fruit("포도", 24);
        fruits[3] = new Fruit("배", 19);
        fruits[4] = new Fruit("키위", 37);

        Arrays.sort(fruits);

        for (int i = 0; i < 5; i++) {
            System.out.println(fruits[i].getName());
        }
        System.out.println("----------------------------");

        Arrays.sort(fruits, new Comparator<Fruit>() {
            @Override
            public int compare(Fruit o1, Fruit o2) {
                return o2.getName().compareTo(o1.getName());
            }
        });

        for (int i = 0; i < 5; i++) {
            System.out.println(fruits[i].getName());
        }
    }
}

class Fruit implements Comparable<Fruit> {
    private String name;
    private int price;

    public Fruit(String name, int price) {
        this.name = name;
        this.price = price;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getPrice() {
        return price;
    }

    public void setPrice(int price) {
        this.price = price;
    }

    @Override
    public int compareTo(Fruit o) {
        return name.compareTo(o.getName());
    }
}
```
* compareTo 메소드에서 사용자가 원하는 값을 기준으로 정렬해야한다.

# Comparable와 Comparator 차이
서로 같은일을 하는 인터페이스지만 왜 따로 존재하냐면 Comparable는 자기 자신과 다른 객체 하나를 비교하고 Comparator은 다른 두개의 객체를 비교한다. Comparator 인터페이스를 구현한 객체를 기준으로 다른 두개의 객체를 비교한다는 말이다.