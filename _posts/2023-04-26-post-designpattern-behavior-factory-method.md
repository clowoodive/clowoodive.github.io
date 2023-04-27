---
title: "[Design Pattern-생성] 팩토리 메서드(Factory Method) 패턴"
excerpt: ""

categories:
  - Design Pattern
tags:
  - 생성 패턴
last_modified_at: 2023-04-26T00:00:00
---


## 정의

클래스 내에 존재하는 다양한 객체 생성 로직을 이를 전담하는 팩토리 메서드로 분리함으로써, 해당 클래스의 주요 로직과 생성 로직의 결합도를 낮추는 패턴. 

생성할 사용 객체(Product)가 추가되어도 원래의 클래스를 수정하지 않아도 됨.

전담하는 Factory 클래스를 따로 두는 법과, 상속을 통해 팩토리 메서드를 오버라이딩 하는 방법이 있음.

![design-pattern-design-pattern-factorymethod1]({{ '/assets/images/design-pattern-factorymethod1.png' | relative_url }}){: .align-center}

## 예제

```java
abstract class ProductMaker {

    // 주요 관심사 로직 따로 존재
    public Product makeProduct(int modelNumber, int price) {
        Product product = createProduct();
        product.setModelNumber(modelNumber);
        product.setPrice(price);
        return product;
    }

    // 내부에서 필요한 객체 생성 로직 분리
    abstract Product createProduct();
}

class FullPackageProductMaker extends ProductMaker {

    @Override
    protected Product createProduct() {
        return new FullPkgProduct();
    }
}

class BulkPackageProductMaker extends ProductMaker {

    @Override
    protected Product createProduct() {
        return new BulkPkgProduct();
    }
}

interface Product {

    void setModelNumber(int modelNumber);

    void setPrice(int price);

    void delivery();
}

class FullPkgProduct implements Product {
    private int modelNumber;

    private int price;

    @Override
    public void setModelNumber(int modelNumber) {
        this.modelNumber = modelNumber;
    }

    @Override
    public void setPrice(int price) {
        this.price = price;
    }

    @Override
    public void delivery() {
        System.out.println("FullPkgProduct delivery");
    }
}

class BulkPkgProduct implements Product {
    private int modelNumber;

    private int price;

    @Override
    public void setModelNumber(int modelNumber) {
        this.modelNumber = modelNumber;
    }

    @Override
    public void setPrice(int price) {
        this.price = price;
    }

    @Override
    public void delivery() {
        System.out.println("BulkPkgProduct delivery");
    }
}
```

테스트 코드로 확인.

```java
@Test
void factoryMethod() {
    ProductMaker fullPkgProductMaker = new FullPackageProductMaker();
    ProductMaker bulkPkgProductMaker = new BulkPackageProductMaker();
    Product fullProduct = fullPkgProductMaker.makeProduct(1234, 5000);
    Product bulkProduct = bulkPkgProductMaker.makeProduct(5678, 12000);

    fullProduct.delivery();
    bulkProduct.delivery();
}
```

출력 결과.

```powershell
FullPkgProduct delivery
BulkPkgProduct delivery
```

## 적용 케이스

- 주체 클래스가 사용할 객체의 유형과 의존관계를 모를 때
- 주체 클래스를 사용하는 사용자들에게 내부 사용 객체 확장 방법을 제공하고 싶을 때
- 사용 객체가 늘어날 때 마다 생성 로직 분기를 추가하기 싫을 때
- java.util.Calendar.getInstance()

## 고려사항

- 코드가 복잡해질 수 있으므로, 기존 클래스 구조 내에 패턴을 적용하면 좋음(i.e. java.util.Calendar.getInstance())

### References

- [https://refactoring.guru/ko/design-patterns/factory-method](https://refactoring.guru/ko/design-patterns/factory-method)