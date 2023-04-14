---
title: "[Design Pattern-행위] 템플릿 메서드(Template Method) 패턴"
excerpt: ""

categories:
  - Design Pattern
tags:
  - 행위 패턴
  - 상속
last_modified_at: 2023-04-12T00:00:00
---


## 정의

기본 알고리즘을 일련의 메서드들로 나누어 슈퍼 클래스의 템플릿 메서드 내부에서 나누어진 일련의 메서드들을 호출함. 그리고 서브 클래스에서 그 일련의 메서드들 중 일부를 재정의 함으로써 확장하는 패턴.

템플릿 메서드는 재정의가 불가능(final) 하며 변하지 않는 알고리즘을 가진 공통 메소드, 필수로 구현해야 하는 추상 메소드, 비어있어서 선택에 따라 구현하는 훅(Hook) 메소드가 있음.

템플릿 메서드 패턴을 특화 시킨 팩토리 메서드(생성) 패턴이 있음.

![design-pattern-template-method1]({{ '/assets/images/design-pattern-template-method1.png' | relative_url }}){: .align-center}

## 예제

```java
public abstract class TemplateMethodClass {
    public final void templateMethod() {
        step1();  // 공통 로직
        step2();  // 공통 로직
        hookMethod();  // 선택 오버라이드
        step3();  // 필수 오버라이드
        step4();  // 공통 로직
    }

    protected void step1() {
        System.out.println("template step1");
    }

    protected void step2() {
        System.out.println("template step2");
    }

    protected void hookMethod() {
        System.out.println("template hookMethod");
    }

    protected abstract void step3();

    protected void step4() {
        System.out.println("template step4");
    }
}

class ConcreteClass1 extends TemplateMethodClass {

    // hookMethod가 아니어도 필요에 따라 선택 오버라이드
    @Override
    public void step2() {
        System.out.println("concrete1 step2");
    }

    // 필수 오버라이드
    @Override
    public void step3() {
        System.out.println("concrete1 step3");
    }
}

class ConcreteClass2 extends TemplateMethodClass {

    // 선택 오버라이드
    @Override
    protected void hookMethod() {
        System.out.println("concrete2 hookMethod");
    }

    // 필수 오버라이드
    @Override
    public void step3() {
        System.out.println("concrete2 step3");
    }
}
```

테스트 코드로 확인.

```java
@Test
void templateMethod() {
    System.out.println("----- concrete1 -----");
    TemplateMethodClass concrete1 = new ConcreteClass1();
    concrete1.templateMethod();

    System.out.println("----- concrete2 -----");
    TemplateMethodClass concrete2 = new ConcreteClass2();
    concrete2.templateMethod();
}
```

출력 결과.

```powershell
----- concrete1 -----
template step1
concrete1 step2
template hookMethod
concrete1 step3
template step4
----- concrete2 -----
template step1
template step2
concrete2 hookMethod
concrete2 step3
template step4
```

## 적용 케이스

- 클라이언트에서 알고리즘의 특정 단계만 확장하게 하고 싶을때
- 거대한 모놀리식 알고리즘의 공통 부분을 재사용하고 개별 단계로 나누고 싶을때
- 거의 같은 알고리즘을 사용하는 여러 클래스가 있을때

## 고려사항

- 메소드 단계가 너무 많으면 유지가 어려움
- 템플릿 메서드는 상속을 기반으로 하는 클래스 수준의 정적인 패턴이기에, 합성을 기반으로 하는  런타임 수준의 동적인 전략(Strategy) 패턴이 더 유리 할 수 있음

### References

- [https://refactoring.guru/ko/design-patterns/template-method](https://refactoring.guru/ko/design-patterns/template-method)