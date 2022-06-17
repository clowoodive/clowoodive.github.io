---
title: "[Spring] Mockito - when().thenReturn과 doReturn().when의 차이"
excerpt: ""

categories:
  - Spring
tags:
  - Mockito
last_modified_at: 2022-06-17T00:00:00
---

{% capture notice-env %}
#### Environment
- Java 11
- Spring Boot 2.6.8
- Mockito Core 3.9.0
{% endcapture %}
<div class="notice--primary">{{ notice-env | markdownify }}</div>


테스트 코드를 작성하기 시작하던 초기에 블로그들을 검색하다 보면 JUnit 버전에 따라 @ExtendWith/@RunWith 애너테이션을 사용하는 것이나 다양한 assert 구문의 사용, 그리고 Mockito의 thenReturn이나 doReturn을 사용하는 등의 것들이 베리에이션이 많아서 혼란스러웠었다.

다른 것들은 어느정도 확인을 했지만 thenReturn/doReturn은 최근에 차이점 한가지 알게되어 정리한다.

```java
/**
     * Sets a return value to be returned when the method is called. E.g:
     * <pre class="code"><code class="java">
     * when(mock.someMethod()).thenReturn(10);
     * </code></pre>
     *
     * See examples in javadoc for {@link Mockito#when}
     *
     * @param value return value
     *
     * @return object that allows stubbing consecutive calls
     */
    OngoingStubbing<T> thenReturn(T value);
```

thenReturn에는 별 다른 설명이 나오지 않는다. 다음은 doReturn을 살펴보자.

```java
/**
     * Use <code>doReturn()</code> in those rare occasions when you cannot use {@link Mockito#when(Object)}.
     * <p>
     * <b>Beware that {@link Mockito#when(Object)} is always recommended for stubbing because it is argument type-safe
     * and more readable</b> (especially when stubbing consecutive calls).
     * <p>
     * Here are those rare occasions when doReturn() comes handy:
     * <p>
     *
     * <ol>
     * <li>When spying real objects and calling real methods on a spy brings side effects
     *
     * <pre class="code"><code class="java">
     *   List list = new LinkedList();
     *   List spy = spy(list);
     *
     *   //Impossible: real method is called so spy.get(0) throws IndexOutOfBoundsException (the list is yet empty)
     *   when(spy.get(0)).thenReturn("foo");
     *
     *   //You have to use doReturn() for stubbing:
     *   doReturn("foo").when(spy).get(0);
     * </code></pre>
     * </li>
     *
     * <li>Overriding a previous exception-stubbing:
     * <pre class="code"><code class="java">
     *   when(mock.foo()).thenThrow(new RuntimeException());
     *
     *   //Impossible: the exception-stubbed foo() method is called so RuntimeException is thrown.
     *   when(mock.foo()).thenReturn("bar");
     *
     *   //You have to use doReturn() for stubbing:
     *   doReturn("bar").when(mock).foo();
     * </code></pre>
     * </li>
     * </ol>
     *
     * Above scenarios shows a tradeoff of Mockito's elegant syntax. Note that the scenarios are very rare, though.
     * Spying should be sporadic and overriding exception-stubbing is very rare. Not to mention that in general
     * overridding stubbing is a potential code smell that points out too much stubbing.
     * <p>
     * See examples in javadoc for {@link Mockito} class
     *
     * @param toBeReturned to be returned when the stubbed method is called
     * @return stubber - to select a method for stubbing
     */
    @CheckReturnValue
    public static Stubber doReturn(Object toBeReturned) {
        return MOCKITO_CORE.stubber().doReturn(toBeReturned);
    }
```

doReturn은 설명이 장황하다.

spy로 stubbing하면서 when().thenReturn()을 사용할 경우 실제로 stubbing 할 메소드를 실행하여 확인하는데, 이 때 구현을 제대로 안한 상태라면 문제가 발생하는 것이다. 

이런 경우에 실제로 실행해서 확인하지 않는 doReturn().when()을 사용하라는 것이고, 그외에는 when().thenReturn()을 사용하는 것이 전달인자 타입에 있어서 안전하고 읽기도 쉽다고 한다.

잘 모르던 때에 이 케이스에 걸려서 stubbing 한 메소드가 계속 실행되어(내부에서 DB에 쿼리하는데 귀찮아서 table을 안만들었음) 에러가 발생해서 한참을 헤맸던 기억이 있다.

제대로 찾아보지 않았다면 “앞으로 doReturn만 써야지!” 했을텐데, 최대한 stubbing 메소드까지 제대로 확인 할 수 있도록 thenReturn을 우선적으로 사용하고 부득이한 경우에만 doReturn을 사용하는 방식으로 하는게 좋을 것 같다.

두번째 케이스는 매우 희귀한 경우이니 조만간 예제로 확인하면 추가로 정리 하는 걸로.