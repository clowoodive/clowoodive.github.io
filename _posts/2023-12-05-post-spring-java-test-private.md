---
title: "[Spring/Java - Test] private 필드/메서드 테스트"
excerpt: ""

categories:
  - Spring
tags:
  - Java
  - Spring
  - Test
last_modified_at: 2023-12-05T00:00:00
---

{% capture notice-env %}
#### Environment
 - Java 11
 - Spring 5.3.18
 - Spring Boot 2.5.12
 - Gradle 6.9.2 
{% endcapture %}
<div class="notice--primary">{{ notice-env | markdownify }}</div>


# 문제 발생

구조가 잘 짜여 있으면 private 메서드를 테스트할 상황은 많지 않을 수 있겠지만, 테스트 코드를 작성하다 보면 한 번쯤은 맞닥뜨리게 되는 것 같다. 

어쩔 수 없이 테스트 해야 할 경우 방법을 알아본다.

<br>

<br>

# 1. `public`으로 접근 제한자 변경

파트원 수가 한두명이고 프로젝트 전체적으로 캡슐화 같은걸 신경쓰지 않는다면 상관 없겠지만,
되도록이면 권장하지 않는 방법이다. 

추후에 굳이 테스트를 할 필요가 없는 메서드라면 초기에 단발성으로 테스트 하고 다시 `private`으로 돌려놓는 식으로 써볼만 하다.

<br>

<br>

# 2. *`@VisibleForTesting` 애너테이션 사용*

메서드 접근 제한자를 `default`(혹은 비움)로 지정해서 패키지 내에서만 접근 할 수 있게 하고, 
애너테이션을 붙여서 이유를 표기하는 방법이다.

우선 `public` 보다는 안전하지만, 테스트 코드 외에 같은 패키지 내에서 호출 될 경우에 경고나 표시가 없기 때문에 주석을 다는 것과 크게 다르지는 않다.

이 애너테이션을 사용하기 위해서는 `guava` 의존성을 추가해야 한다.

```groovy
// build.gradle
dependencies {
	implementation 'com.google.guava:guava:32.0.1-jre'
}
```

```java
// test
*@VisibleForTesting
void method() {
	...
}*
```

<br>

<br>

# 3. Java Reflection 사용

자바의 리플렉션을 이용해서 `private`필드나 메서드의 접근을 가능하게 하는 방법이다.

```java
public class TestPrivate {
    private int privateField;

    private int privateMethod(int paramVal) {
        return privateField * paramVal;
    }
}
```

위 `TestPrivate` 클래스의 `privateField`와 `privateMethod`를 리플렉션을 이용해서 아래와 같이 테스트 할 수 있다.

```java
@Test
void testPrivateFieldAndMethod() throws NoSuchFieldException, IllegalAccessException, NoSuchMethodException, InvocationTargetException {
    // given
    int paramVal = 3;
    int fieldVal = 5;
    TestPrivate testPrivateInstance = new TestPrivate();

    Field privateField = TestPrivate.class.getDeclaredField("privateField");
    privateField.setAccessible(true);
    privateField.set(testPrivateInstance, fieldVal);

    Method privateMethod = TestPrivate.class.getDeclaredMethod("privateMethod", int.class);
    privateMethod.setAccessible(true);

    // when
    int retVal = (int) privateMethod.invoke(testPrivateInstance, paramVal);

    // then
    BDDAssertions.then(retVal).isEqualTo(paramVal * fieldVal);
}
```

<br>

<br>

# 4. ReflectionTestUtils 사용

마지막으로 Spring Framework에서 제공하는 `ReflectionTestUtils`를 사용하는 방법이다.

```java
@Test
void testPrivateFieldAndMethod() {
    // given
    int paramVal = 3;
    int fieldVal = 5;
    CouponController testPrivateInstance = new CouponController();
    ReflectionTestUtils.setField(testPrivateInstance, "privateField", fieldVal);

    // when
    int retVal = ReflectionTestUtils.invokeMethod(testPrivateInstance, "privateMethod", paramVal);

    // then
    int privateFieldVal = (int) ReflectionTestUtils.getField(testPrivateInstance, "privateField");
    BDDAssertions.then(privateFieldVal).isEqualTo(fieldVal);
    BDDAssertions.then(retVal).isEqualTo(paramVal * fieldVal);
}
```

아무래도 Spring Test에서 제공하는 유틸이다 보니 직관적이고 간결하게 작성 할 수 있다.

따로 의존성을 추가해야 하는 것도 아니니 `ReflectionTestUtils`를 우선적으로 사용하면 좋을것 같다.

<br>

<br>

### References

- [https://www.baeldung.com/spring-reflection-test-utils](https://www.baeldung.com/spring-reflection-test-utils)