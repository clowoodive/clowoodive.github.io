---
title: "[Book] Clean Code - 2장. 의미 있는 이름(1)"
excerpt: ""

categories:
  - Book
tags:
  - Refactoring
  - Clean Code
last_modified_at: 2022-07-05T00:00:00
---


### 의도를 분명히 밝혀라

변수나 함수 그리고 클래스 이름은 아래 질문에 답할수 있어야 하고 따로 주석이 필요 없어야 한다.

- 존재 이유는? 수행 기능은? 사용 방법은?

```java
int elapsedTimeIndays;
int daysSinceCreation;
int daysSinceModification;
int fileAgeInDays;
```

### 그릇된 정보를 피하라

널리 쓰이는 이름을 변수 이름으로 쓰는 것은 적합하지 않으며, 실제 List로 만들어진 것이 아니라면 xxxList라고 쓰지 말아야 한다.(실제 컨테이너가 List여도 컨테이너 유형을 이름에 쓰지 않는 것이 바람직 함)

### 의미있게 구분하라

- 연속적인 숫자를 덧붙인 이름(a1, a2, …)을 피하라.
- 불용어(noise word)를 피하라.(ex ProductInfo & ProductData, xxxVariable, Customer & CustomerObject)
- 읽는 사람이 차이를 알도록 이름을 지어라.

### 발음하기 위순 이름을 사용하라

커뮤니케이션에서 대화가 가능하도록 지어라.

### 검색하기 쉬운 이름을 사용하라

- 긴 이름이 짧은 이름보다 좋다.
- 검색하기 쉬운 이름이 상수보다 좋다.

### 인코딩을 피하라

이름에 유형이나 범위까지 넣으면 해독하기 어려워진다. 유형이 변경되면 이름까지 변경해야 할 수도 있다. 

인터페이스와 구체 클래스의 이름을 지을 때 인터페이스에 I를 붙이지 않는다. 인터페이스라는 사실을 알리고 싶지 않다.(이부분은 이해가 잘 안됨) 굳이 둘중 하나를 인코딩 해야 한다면 구체 클래스에 xxxImp 라고 붙이는게 낫다.