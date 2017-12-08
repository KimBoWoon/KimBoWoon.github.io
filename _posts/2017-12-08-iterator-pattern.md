---
title: "[pattern] Iterator Pattern"
date: 2017-12-08 01:23:45
categories:
- Pattern
---

# Iterator Pattern
for나 while를 대신하여 사용할 수 있는 도구

# 코드
```java
public class IteratorPattern {
    private static ArrayList<Integer> integers = new ArrayList<Integer>();

    public static void main(String[] args) {
        integers.add(1);
        integers.add(2);
        integers.add(3);
        integers.add(4);
        integers.add(5);

        Iterator iterator = integers.iterator();

        for (int i = 0; i < integers.size(); i++) {
            System.out.println(integers.get(i));
        }

        while (iterator.hasNext()) {
            System.out.println(iterator.next());
        }
    }
}
```
* 여기서 for나 while를 사용할 수 있지만 iterator를 사용하는 이유는 유지보수가 편하고, 만약 컬렉션이나 다른 클래스를 사용한다 하여도 반복문은 바뀌지 않기 때문입니다.