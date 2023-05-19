---
title: "[Design Pattern-구조] 퍼사드(Facade) 패턴"
excerpt: ""

categories:
  - Design Pattern
tags:
  - 구조 패턴
last_modified_at: 2023-05-18T00:00:00
---


## 정의

라이브러리, 프레임워크 또는 복잡한 클래스 집합을 사용하기 위한 단순한 인터페이스를 제공하는 패턴.

![design-pattern-facade1]({{ '/assets/images/design-pattern-facade1.png' | relative_url }}){: .align-center}

## 예제

```java
class ComplexLibrary1 {

    private String data;

    public ComplexLibrary1(String data) {
        this.data = data;
    }

    public String getData() {
        return data + " from lib1\n";
    }
}

class ComplexLibrary2 {

    ComplexLibrary1 complexLibrary1;

    public ComplexLibrary2(ComplexLibrary1 lib1) {
        this.complexLibrary1 = lib1;
    }

    public String calcData() {
        return complexLibrary1.getData() + "is calculated by lib2\n";
    }
}

class ComplexLibrary3 {

    public void printData(ComplexLibrary2 lib2) {
        System.out.println(lib2.calcData() + "and printed by lib3");
    }
}

class Facade {

    public void processComplexData(String input) {
        ComplexLibrary1 lib1 = new ComplexLibrary1(input);
        ComplexLibrary2 lib2 = new ComplexLibrary2(lib1);
        ComplexLibrary3 lib3 = new ComplexLibrary3();
        lib3.printData(lib2);
    }
}
```

테스트 코드로 확인.

```java
@Test
void facade() {
    Facade facade = new Facade();
    facade.processComplexData("input data");
}
```

출력 결과.

```powershell
input data from lib1
is calculated by lib2
and printed by lib3
```

## 적용 케이스

- 복잡한 하위 시스템 중 일부만을 사용 할 때
- 복잡한 하위 시스템과의 결합도를 줄이고 싶을 때
- javax.faces.context.FacesContext 가 내부적으로 LifeCycle, NavigationHandler 등 사용
- javax.faces.context.ExternalContext가 내부적으로 ServletContext, HttpSession, HttpServletRequest, HttpServletResponse 등 사용

## 고려사항

- 여러개의 책임을 가짐으로써 God Object가 되는 안티 패턴으로 전개 될 수 있음

### References

- [https://refactoring.guru/ko/design-patterns/facade](https://refactoring.guru/ko/design-patterns/facade)