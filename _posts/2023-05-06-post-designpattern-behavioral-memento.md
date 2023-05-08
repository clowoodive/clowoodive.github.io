---
title: "[Design Pattern-행위] 메멘토(Memento) 패턴"
excerpt: ""

categories:
  - Design Pattern
tags:
  - 행위 패턴
last_modified_at: 2023-05-06T00:00:00
---


## 정의

객체의 이전 상태를 캡슐화를 유지하여 저장하고 복원 할수 있게 해주는 패턴.

상태 스냅샷을 생성하고자 하는 그 객체(originator)에게 생성/복원을 위임해서 private 필드들도 복사 할 수 있으며, 생성된 메멘토는 Caretaker라는 객체를 사용해서 보관하고 관리 할 수 있음.

![design-pattern-memento1]({{ '/assets/images/design-pattern-memento1.png' | relative_url }}){: .align-center}

## 예제

```java
class Originator {

    private int data;

    public int getData() {
        return data;
    }

    public void setData(int data) {
        this.data = data;
    }

    public Memento save() {
        return new Memento(data);
    }

    public void restore(Memento memento) {
        this.data = memento.getData();
    }
}

class Memento {

    private int data;

    public Memento(int data) {
        this.data = data;
    }

    public int getData() {
        return data;
    }

    public void setData(int data) {
        this.data = data;
    }
}

class Caretaker {

    private Stack<Memento> stack = new Stack<>();

    public void push(Memento memento) {
        stack.push(memento);
    }

    public Memento pop() {
        if (stack.empty())
            return null;

        return stack.pop();
    }
}
```

테스트 코드로 확인.

```java
@Test
void mementoTest() {
    Originator ori = new Originator();
    Caretaker caretaker = new Caretaker();

    ori.setData(10);
    System.out.println("save memento1 data : " + ori.getData());
    Memento save1 = ori.save();
    caretaker.push(save1);

    ori.setData(ori.getData() + 10);
    System.out.println("save memento2 data : " + ori.getData());
    Memento save2 = ori.save();
    caretaker.push(save2);

    ori.setData(ori.getData() + 10);
    System.out.println("now data : " + ori.getData());

    Memento restore1 = caretaker.pop();
    ori.restore(restore1);
    System.out.println("restore memento1 data : " + ori.getData());

    Memento restore2 = caretaker.pop();
    ori.restore(restore2);
    System.out.println("restore memento2 data : " + ori.getData());
}
```

출력 결과.

```powershell
save memento1 data : 10
save memento2 data : 20
now data : 30
restore memento1 data : 20
restore memento2 data : 10
```

## 적용 케이스

- 객체의 이전 상태를 복원하려는 매커니즘이 필요할 때
- 객체의 스냅샷 생성시 캡슐화를 위반할 때

## 고려사항

- Caretaker에서 메멘토들이 과다하게 적재되지 않도록 관리가 필요함

### References

- [https://refactoring.guru/ko/design-patterns/memento](https://refactoring.guru/ko/design-patterns/memento)