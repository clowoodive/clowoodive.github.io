---
title: "[Design Pattern-행위] 중재자(Mediator) 패턴"
excerpt: ""

categories:
  - Design Pattern
tags:
  - 행위 패턴
last_modified_at: 2023-04-17T00:00:00
---


## 정의

클래스들이 다른 클래스들과 의존도(결합도)가 높고 복잡할 때 직접 통신하지 않고 중재자(Mediator)를 통해 협력하는 패턴. 아래 패턴들과 유사하지만 차이점이 있음.

- 커맨드 패턴 - 발신자와 수신자 간의 단방향 연결
- 옵저버 패턴 - 수신자들이 동적으로 구독 및 구독 취소
- 책임연쇄 패턴 - 수신자 중 하나에 의해 요청이 처리 될 때까지 요청을 순차적으로 전달

![design-pattern-mediator1]({{ '/assets/images/design-pattern-mediator1.png' | relative_url }}){: .align-center}

## 예제

```java
interface Mediator {
    void notify(Component component, String event);

    void setComponent1(Component component);

    void setComponent2(Component component);

    void setComponent3(Component component);
}

public class MediatorClass implements Mediator {

    // 컴포넌트들의 리스트가 될 수도 있음
    private Component component1;

    private Component component2;

    private Component component3;

    @Override
    public void setComponent1(Component component) {
        this.component1 = component;
    }

    @Override
    public void setComponent2(Component component) {
        this.component2 = component;
    }

    @Override
    public void setComponent3(Component component) {
        this.component3 = component;
    }

    @Override
    public void notify(Component component, String event) {
        if (component instanceof ConcreteComponent1) {
            if (event.equals("click")) {
                reactOnClickComponent1();
            } else if (event.equals("move")) {
                reactOnMoveComponent1();
            }
        } else if (component instanceof ConcreteComponent2) {
            if (event.equals("click")) {
                reactOnClickComponent2();
            } else if (event.equals("move")) {
                reactOnMoveComponent2();
            }
        } else if (component instanceof ConcreteComponent3) {
            if (event.equals("click")) {
                reactOnClickComponent3();
            } else if (event.equals("move")) {
                reactOnMoveComponent3();
            }
        }
    }

    private void reactOnClickComponent1() {
        component2.receiveClick(" from component1");
        component3.receiveClick(" from component1");
    }

    private void reactOnMoveComponent1() {
        component2.receiveMove(" from component1");
        component3.receiveMove(" from component1");
    }

    private void reactOnClickComponent2() {
        component1.receiveClick(" from component2");
        component3.receiveClick(" from component2");
    }

    private void reactOnMoveComponent2() {
        component1.receiveMove(" from component2");
        component3.receiveMove(" from component2");
    }

    private void reactOnClickComponent3() {
        component1.receiveClick(" from component3");
        component2.receiveClick(" from component3");
    }

    private void reactOnMoveComponent3() {
        component1.receiveMove(" from component3");
        component2.receiveMove(" from component3");
    }
}

abstract class Component {
    protected Mediator mediator;

    public Component(Mediator mediator) {
        this.mediator = mediator;
    }

    abstract void sendClick();

    abstract void sendMove();

    abstract void receiveClick(String event);

    abstract void receiveMove(String event);
}

class ConcreteComponent1 extends Component {

    public ConcreteComponent1(Mediator mediator) {
        super(mediator);
    }

    @Override
    public void sendClick() {
        System.out.println("Component1 send click");
        super.mediator.notify(this, "click");
    }

    @Override
    public void sendMove() {
        System.out.println("Component1 send move");
        super.mediator.notify(this, "move");
    }

    @Override
    public void receiveClick(String event) {
        System.out.println("Component1 receive click" + event);
    }

    @Override
    public void receiveMove(String event) {
        System.out.println("Component1 receive move" + event);
    }
}

class ConcreteComponent2 extends Component {

    public ConcreteComponent2(Mediator mediator) {
        super(mediator);
    }

    @Override
    public void sendClick() {
        System.out.println("Component2 send click");
        super.mediator.notify(this, "click");
    }

    @Override
    public void sendMove() {
        System.out.println("Component2 send move");
        super.mediator.notify(this, "move");
    }

    @Override
    public void receiveClick(String event) {
        System.out.println("Component2 receive click" + event);
    }

    @Override
    public void receiveMove(String event) {
        System.out.println("Component2 receive move" + event);
    }
}

class ConcreteComponent3 extends Component {

    public ConcreteComponent3(Mediator mediator) {
        super(mediator);
    }

    @Override
    public void sendClick() {
        System.out.println("Component3 send click");
        super.mediator.notify(this, "click");
    }

    @Override
    public void sendMove() {
        System.out.println("Component3 send move");
        super.mediator.notify(this, "move");
    }

    @Override
    public void receiveClick(String event) {
        System.out.println("Component3 receive click" + event);
    }

    @Override
    public void receiveMove(String event) {
        System.out.println("Component3 receive move" + event);
    }
}
```

테스트 코드로 확인.

```java
@Test
void mediator() {
    Mediator mediator = new MediatorClass();
    Component component1 = new ConcreteComponent1(mediator);
    Component component2 = new ConcreteComponent2(mediator);
    Component component3 = new ConcreteComponent3(mediator);
    mediator.setComponent1(component1);
    mediator.setComponent2(component2);
    mediator.setComponent3(component3);
    component1.sendClick();
    component3.sendMove();
}
```

출력 결과.

```powershell
Component1 send click
Component2 receive click from component1
Component3 receive click from component1
Component3 send move
Component1 receive move from component3
Component2 receive move from component3
```

## 적용 케이스

- 강한 결합으로 묶인 클래스들을 분리 할 때
- 재사용하기 어려운 의존성으로 엮여 있을 때
- 컴포넌트 변경 없이 컴포넌트 간 상호작용을 새로운 방법으로 정의하고 싶을 때

## 고려사항

- 중재자가 컴포넌트 객체의 생성을 담당하면 팩토리/퍼사드 패턴과 유사해짐
- 중재자 클래스가 너무 많은 권한을 가지지 않도록 조심

### References

- [https://refactoring.guru/ko/design-patterns/mediator](https://refactoring.guru/ko/design-patterns/mediator)