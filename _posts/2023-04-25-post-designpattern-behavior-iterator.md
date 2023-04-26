---
title: "[Design Pattern-행위] 반복자(Iterator) 패턴"
excerpt: ""

categories:
  - Design Pattern
tags:
  - 행위 패턴
last_modified_at: 2023-04-25T00:00:00
---


## 정의

컬렉션의 타입 및 세부 정보를 노출하지 않고 다양한 순회 방식을 가진 Iterator들을 통해 순회 할수 있게 하는 패턴.  

순회 알고리즘들만 따로 분류해서 관리 할 수 있고, 새로운 컬렉션 구현체와 반복자 구현체를 통해 대상 객체의 변경 없이 순회 방식 추가 가능.

아래 예제 중 `OddNumberCollection` 는 ArrayList의 iterator와 유사한 구조로서, 반복자 패턴의 장점인 확장에 어울리는 예제는 아니니 참고.

![design-pattern-iterator1]({{ '/assets/images/design-pattern-iterator1.png' | relative_url }}){: .align-center}

## 예제

```java
interface NumberIterator {

    int getNext();

    boolean hasMore();
}

class AscendingIterator implements NumberIterator {

    private List<Integer> numbers = new ArrayList<>();

    private int curPos = 0;

    public AscendingIterator(EvenNumberCollection collection) {
        this.numbers = collection.numList.stream().sorted().collect(Collectors.toList());
    }

    @Override
    public int getNext() {
        if (!hasMore()) {
            return -1;
        }

        return this.numbers.get(curPos++);
    }

    @Override
    public boolean hasMore() {
        return curPos < numbers.size();
    }
}

class DescendingIterator implements NumberIterator {

    private List<Integer> numbers = new ArrayList<>();

    private int curPos = 0;

    public DescendingIterator(EvenNumberCollection collection) {
        this.numbers = collection.numList.stream().sorted(Comparator.reverseOrder()).collect(Collectors.toList());
    }

    @Override
    public int getNext() {
        if (!hasMore()) {
            return -1;
        }

        return this.numbers.get(curPos++);
    }

    @Override
    public boolean hasMore() {
        return curPos < numbers.size();
    }
}

interface NumberCollection {

    NumberIterator createAscendingNumberIterator();

    NumberIterator createDescendingNumberIterator();
}

class EvenNumberCollection implements NumberCollection {

    List<Integer> numList;

    public EvenNumberCollection(List<Integer> intList) {
        this.numList = intList;
    }

    // 해당 컬렉션 구현 사항들
    public void addNum(int addNum) {
    }

    @Override
    public NumberIterator createAscendingNumberIterator() {
        return new AscendingIterator(this);
    }

    @Override
    public NumberIterator createDescendingNumberIterator() {
        return new DescendingIterator(this);
    }
}

// class 내부에 iterator 선언 방식
class OddNumberCollection {

    List<Integer> numList;

    public OddNumberCollection(List<Integer> intList) {
        this.numList = intList;
    }

    // 해당 컬렉션 구현 사항들
    public void addNum(int addNum) {
    }

    // 기본 오름차순 반복자
    NumberIterator getIterator() {
        return new DefaultIterator();
    }

    class DefaultIterator implements NumberIterator {

        private int curPos = 0;

        @Override
        public int getNext() {
            if(!hasMore()) {
                return -1;
            }

            List<Integer> cache = OddNumberCollection.this.numList.stream().sorted().collect(Collectors.toList());
            return cache.get(curPos++);
        }

        @Override
        public boolean hasMore() {
            return curPos < OddNumberCollection.this.numList.size();
        }
    }
}
```

테스트 코드로 확인.

```java
@Test
void chainOfResponsibility() {
    EvenNumberCollection evenNumberCollection = new EvenNumberCollection(Arrays.asList(4, 8, 6, 2, 10));
    NumberIterator evenAscIterator = evenNumberCollection.createAscendingNumberIterator();
    NumberIterator evenDescIterator = evenNumberCollection.createDescendingNumberIterator();

    System.out.println("-------  EvenCollection ascending -------");
    while (evenAscIterator.hasMore()) {
        System.out.println(evenAscIterator.getNext());
    }

    System.out.println("-------  EvenCollection descending -------");
    while (evenDescIterator.hasMore()) {
        System.out.println(evenDescIterator.getNext());
    }

    OddNumberCollection oddNumberCollection = new OddNumberCollection(Arrays.asList(7, 1, 3, 9, 5));
    NumberIterator oddAscIterator = oddNumberCollection.getIterator();

    System.out.println("-------  OddCollection ascending -------");
    while (oddAscIterator.hasMore()) {
        System.out.println(oddAscIterator.getNext());
    }
}
```

출력 결과.

```powershell
-------  EvenCollection ascending -------
2
4
6
8
10
-------  EvenCollection descending -------
10
8
6
4
2
-------  OddCollection ascending -------
1
3
5
7
9
```

## 적용 케이스

- 컬렉션 내부의 로직을 클라이언트에게 숨기고 싶을 때
- 순회 코드의 중복을 줄이고 싶을 때
- java.util.Iterator
- java.util.Scanner
- java.util.Enumeration

## 고려사항

- 컬렉션의 직접 순회보다 느릴 수 있음

### References

- [https://refactoring.guru/ko/design-patterns/iterator](https://refactoring.guru/ko/design-patterns/iterator)