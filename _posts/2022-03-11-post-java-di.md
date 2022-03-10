---
title:  "의존성 주입(DI)"
excerpt: "Dependency Injection의 장점과 방법"

categories:
  - Java
  - Spring
tags:
  - Java
  - Spring
last_modified_at: 2022-03-11T15:06:00-05:00
---

# 의존성 주입(DI)

진행 중이던 프로젝트를 이어 받으면서 Spring Framework를 접하게 되었는데

정말 문제 되는 구조를 수정하거나 기능 구현하는 것만 해도 시간이 부족해서

의존성 주입 구조에 대해 개선하거나 깊게 생각하지 못했었다.

테스트시 어려움이나 IntelliJ에서 참조 추적이 잘 안되어서 검색하다 보니

**필드 주입(Field Injection)** 은 최근들어 권장하지 않는 방법이었다.

지금부터라도 조금씩 **생성자 주입(Constructor Injection)** 방식으로 변경해 나가야겠다.

<br>

### 의존성 주입이란?

의존하는 객체를 외부에서 선언 한 뒤 주입받아 사용하는 것을 말한다.

인터페이스로 추상화 해서 구현하면 객체간 의존성이 줄어들게 되어 주입된 객체가 변경되어도 주입받은 객체를 수정하게될 가능성이 줄어들고, 각 객체를 따로 테스트 할 수 있는 장점이 있다.

<!--

테스트시 용이하다. IoC컨테이너에 의존하지 않기에 단위 테스트 시 의존성을 가지는 필요한 객체만 생성해서 넘겨주면 된다.

-->

<br>

---

### 주입 방법1 - 필드 주입(Field Injection)

```java
@Controller 
public class AdminController {
	@Autowired
	private AdminService adminService; 
}
```

간결하고 이해하기 쉬운 장점이 있으나, 테스트를 위한 객체를 주입이 불가능해서 권장되지 않는 주입 방법이다.

  <br>

---

### 주입 방법2 - 수정자 주입(Setter Injection)

```java
@Controller 
public class AdminController {
	private AdminService adminService; 

	@Autowired 
	public void setAdminService(AdminService adminService) { 
		this.adminService= adminService; 
	} 
}
```

주입받는 객체가 변경될 필요가 있는 경우 setter를 통해서 주입하는 방법이다.

<br>

---

### 주입 방법3 - 생성자 주입(Constructor Injection)

```java
@Controller 
public class AdminController {
	private final AdminService adminService; 

	@Autowired 
	public AdminController(AdminService adminService) { 
		this.adminService= adminService; 
	} 
}
```

최초 객체가 생성될 때 주입되므로 한번 주입받은 객체가 변하지 않아야 하는 경우에 사용되고, 최근 가장 권장되는 방법이다.  생성자가 하나인 경우 @Autowired 어노테이션을 생략할 수 있지만, 나의 경우 스프링에 미숙할 때 어노테이션이 없어서 분석하기 어려웠다.

<br>

---

## 왜 생성자 주입이 좋을까?

순환 참조를 방지 할 수 있다. 두 객체 내에서 서로간에 순환 참조하는 로직이 있을 경우 필드 주입과 수정자 주입은 객체 생성 이후 해당 비즈니스 로직을 호출하면서 순환참조 문제가 드러나는데, 생성자 주입의 경우는 객체 생성시점에 순환 참조가 발견된다.