---
title:  "[Spring] 단위 테스트 & 통합 테스트"
excerpt: "단위 테스트를 적용하면서 종합한 내용을 간략히 정리 해보려고 한다"

categories:
  - Spring
tags:
  - Spring
last_modified_at: 2022-03-25T00:00:00
---


진행 중인 프로젝트가 단위 테스트에 적합하지 않은 구조로 설계되어 있어서 그동안 통합 테스트 방식으로만 테스트를 진행 해 왔었다. 그래서 단위 테스트를 적용하면서 종합한 내용을 간략히 정리 해보려고 한다. 단계별로 자세한 내용들은 추후 시간이 나면 한번 정리해 보기로.
JUnit버전에 따라 적용해야 할 애너테이션이 다른데 여기서는 JUnit 5(Jupiter)기준으로 다룬다.

<br>

## 단위 테스트

어플리케이션을 작은 단위로 쪼개어 각 단위가 제대로 동작하는지 테스트 하는 것을 말한다. 다른 모듈이나 계층(ex controller, service, repository)과의 결합도가 낮을수록 애너테이션을 통해 필요한 구성만 로드하여 빠르게 테스트가 가능하며, 이는 다른 계층 구현이 덜 되었더라도 테스트가 가능하다는 뜻도 된다.

- @DataJpaTest - JPA관련 설정을 로드하는 Repository 계층 테스트.
    - Mybatis의 경우는 @DataJpaTest 대신 @MyBatisTest를 사용.
- @WebMvcTest - MVC관련 설정을 로드하는 Controller 계층 테스트.
- @ExtendWith(MockitoExtension.class) - Spring Container를 사용하지 않고 Mockito 프레임워크를 사용한 기본적인 단위 테스트. Service 계층 테스트에 적합.
- @ExtendWith(SpringExtension.class) - Spring Container(test container)를 로드하여  @Autowired, @MockBean 등이 사용 가능한 테스트. @DataJpaTest와 @WebMvcTest등에는 이미 포함되어 있음.
    - @Autowired 사용하려면 Bean등록은 @ContextConfiguration를 사용.

- [annotation 참고 링크](https://docs.spring.io/spring-boot/docs/2.4.13/reference/html/appendix-test-auto-configuration.html#test-auto-configuration)

<br>

## 통합 테스트

단위 테스트 된 모듈들 간에 제대로 동작하는지 통합적으로 테스트 하는 것이며,  모든 구성과 Bean을 로드하기에 단위 테스트에 비해 무겁고 느리지만, 실제 서비스와 가장 가까운 환경의 테스트이며 따라서 전체적인 흐름을 테스트 할 수 있다.  다만 다양한 input에 대해 모든 테스트가 어렵기에 단위 테스트에서 각 모듈의 테스트가 빈틈없이 수행되어야 할것 같다.

- @SpringBootTest 애너테이션 사용.
- @SpringBootTest(properties = "classpath:application-dev.properties")와 같은 방식으로 환경 구성 가능.
- @TestMethodOrder, @Order(1) 을 이용해서 테스트 순서를 정할 수 있음.

<br>

## Mockito 프레임워크

빠른 테스트를 위해 미구현 되었거나 의존성을 분리해야 할 객체를 대신할  Mock 객체를 지원하는 프레임워크.

- @ Mock -  모의 객체를 만들수 있게 해주는 애너테이션으로 해당 객체의 메소드 리턴값을 stub 할 수 있음.
- @ InjectMocks - @Mock으로 지정된 객체를 주입받는 객체.
- @Spy - @Mock과 유사하지만 이 애너테이션은 객체의 일부 메소드는 원래것을 사용하고 일부는 stub할수 있음.
- @MockBean / @SpyBean - stub개념 자체는 @Mock / @Spy와 같으나 Spring Container(context) 에 등록되어 @Autowired 로 주입 받는것이 가능함.

> stub : 테스트를 받는 모듈이 의존하는 소프트웨어 구성 요소(또는 모듈)의 동작을 시뮬레이션하는 프로그램.
간단하게 의존하는 객체의 메소드가 미구현이거나 메소드의 리턴 값을 원하는 값으로 하고 싶을 경우 이를 대체해서 가능하게 함.
> 

<br>

단위/통합 이라는 명칭으로 나누어 부르고 사용하는 애너테이션이나 프레임워크도 다르지만, 어디에 포커스를 두고 테스트를 진행 한다는 차이가 있을 뿐 문제 없는 어플리케이션을 만들기 위해서는 둘다 필수로 수행해야 한다.

<!---

mock mockbean spy spybean 차이점

[https://cobbybb.tistory.com/16](https://cobbybb.tistory.com/16)

SpringExtension만 사용 시 bean 등록

[https://perfectacle.github.io/2019/06/23/auto-scanning-annotation-based-bean/](https://perfectacle.github.io/2019/06/23/auto-scanning-annotation-based-bean/)

[https://velog.io/@neity16/SpringBoot-입문5-DB-접근-기술-단위테스트-vs-통합테스트](https://velog.io/@neity16/SpringBoot-%EC%9E%85%EB%AC%B85-DB-%EC%A0%91%EA%B7%BC-%EA%B8%B0%EC%88%A0-%EB%8B%A8%EC%9C%84%ED%85%8C%EC%8A%A4%ED%8A%B8-vs-%ED%86%B5%ED%95%A9%ED%85%8C%EC%8A%A4%ED%8A%B8)

[https://jiminidaddy.github.io/dev/2021/05/18/dev-spring-단위테스트-API/](https://jiminidaddy.github.io/dev/2021/05/18/dev-spring-%EB%8B%A8%EC%9C%84%ED%85%8C%EC%8A%A4%ED%8A%B8-API/)

[https://jiminidaddy.github.io/dev/2021/05/20/dev-spring-단위테스트-Repository/](https://jiminidaddy.github.io/dev/2021/05/20/dev-spring-%EB%8B%A8%EC%9C%84%ED%85%8C%EC%8A%A4%ED%8A%B8-Repository/)

MockBean은 컨테이너에 등록해주므로, 컨테이너를 

*@ExtendWith*(MockitoExtension.*class*)적용한 테스트 시에 해당 객체 참조하지 못함

@ExtendWith(SpringExtension.class)는 컨테이너를 로드하므로 통합테스트에 사용

MockBean을 사용 할 수 있음

[https://brunch.co.kr/@springboot/418](https://brunch.co.kr/@springboot/418)

-->