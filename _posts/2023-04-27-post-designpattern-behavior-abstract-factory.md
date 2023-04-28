---
title: "[Design Pattern-생성] 추상 팩토리(Abstract Factory) 패턴"
excerpt: ""

categories:
  - Design Pattern
tags:
  - 생성 패턴
last_modified_at: 2023-04-27T00:00:00
---


## 정의

패밀리를 구성하는 객체들을 전담하는 팩토리를 통해서 생성하는 패턴. 

팩토리들도 같은 인터페이스를 따르고 구성하는 객체들도 같은 타입은 같은 인터페이스를 따름.

따라서 클라이언트에서 구성 객체 사용 시 인터페이스를 통해 호환되며, 객체 생성 로직들만 따로 분리해서 단일 책임 원칙을 지킬수 있고, 클라이언트 코드 수정 없이 새로운 패밀리 구성 객체를 추가 할 수 있으므로 개방/폐쇄 원칙도 지켜짐.

![design-pattern-abstractfactory1]({{ '/assets/images/design-pattern-abstractfactory1.png' | relative_url }}){: .align-center}

## 예제

```java
interface GUIFactory {

    Button createButton();

    Checkbox createCheckbox();
}

class WindowsFactory implements GUIFactory {

    @Override
    public Button createButton() {
        System.out.println("create WindowsButton");
        return new WindowsButton();
    }

    @Override
    public Checkbox createCheckbox() {
        System.out.println("create WindowsChecbox");
        return new WindowsCheckbox();
    }
}

class MacFactory implements GUIFactory {

    @Override
    public Button createButton() {
        System.out.println("create MacButton");
        return new MacButton();
    }

    @Override
    public Checkbox createCheckbox() {
        System.out.println("create MacCheckbox");
        return new MacCheckbox();
    }
}

interface Button {

    void push();
}

class WindowsButton implements Button {

    @Override
    public void push() {
        System.out.println("WindowsButton pushed.");
    }
}

class MacButton implements Button {

    @Override
    public void push() {
        System.out.println("MacButton pushed.");
    }
}

interface Checkbox {

    void checkToggle();
}

class WindowsCheckbox implements Checkbox {

    @Override
    public void checkToggle() {
        System.out.println("WindowsCheckbox checked.");
    }
}

class MacCheckbox implements Checkbox {

    @Override
    public void checkToggle() {
        System.out.println("MacCheckbox checked.");
    }
}

class OsApplication {

    private Button button;
    private Checkbox checkbox;

    public OsApplication(GUIFactory factory) {
        this.button = factory.createButton();
        this.checkbox = factory.createCheckbox();
    }

    public void clickButton() {
        this.button.push();
    }

    public void clickChecbox() {
        this.checkbox.checkToggle();
    }
}
```

테스트 코드로 확인.

```java
@Test
void abstractFactory() {
    GUIFactory factory = new MacFactory();
    OsApplication app = new OsApplication(factory);

    app.clickButton();
    app.clickChecbox();
}
```

출력 결과.

```powershell
create MacButton
create MacCheckbox
MacButton pushed.
MacCheckbox checked.
```

## 적용 케이스

- 클라이언트가 다양한 패밀리 제품군들과 동작해야 하지만 제품군들의 구상클래스와 결합도를 낮추고(인터페이스) 싶을 때
- 한 클래스의 Factory Method들이 여러 패밀리 제품을 생성 해서 여러 책임을 가질 때
- 사용 객체가 늘어날 때 마다 생성 로직 분기를 추가하기 싫을 때
- javax.xml.parsers.DocumentBuilderFactory.newInstance()

## 고려사항

- 코드가 복잡해질 수 있음.

### References

- [https://refactoring.guru/ko/design-patterns/abstract-factory](https://refactoring.guru/ko/design-patterns/abstract-factory)