---
title: "[Spring Boot] 스프링 부트 애플리이션 초기화 후 코드를 실행 하는 몇가지 방법"
excerpt: ""

categories:
  - Spring Boot
tags:
  - Runner
last_modified_at: 2023-01-01T00:00:00
---

{% capture notice-env %}
#### Environment
 - Java 11
 - Spring Boot 2.5.13
 - Gradle 7.5.1 
{% endcapture %}
<div class="notice--primary">{{ notice-env | markdownify }}</div>


스프링 부트 초기 구동 후 코드를 실행하는 방법 몇가지를 알아본다.

# CommandLineRunner

이 인터페이스 구현체를 빈으로 등록하면 초기화 구동 후 run 메소드를 실행 해 준다.

아래와 같이 @Component 를 사용해서 빈으로 등록해도 되고

```java
@Component
public class DemoCommandLineRunner implements CommandLineRunner {
    @Override
    public void run(String... args) throws Exception {
        System.out.println("========== CommandLineRunner ==========");
        for (var arg : args) {
            System.out.println("arg : " + arg);
        }
        System.out.println("=======================================");
    }
}
```

아래와 같이 @Configuration 클래스에 빈으로 등록해도 된다.

```java
@SpringBootApplication
public class ExampleApplication {

    public static void main(String[] args) {
        SpringApplication.run(ExampleApplication.class, args);
    }

    @Bean
    public CommandLineRunner demoCommandLineRunner() {
        return new DemoCommandLineRunner();
    }

//    @Component
    public class DemoCommandLineRunner implements CommandLineRunner {
        @Override
        public void run(String... args) throws Exception {
            System.out.println("========== CommandLineRunner ==========");
            for (var arg : args) {
                System.out.println("arg : " + arg);
            }
            System.out.println("=======================================");
        }
    }
}
```

그리고 이를 좀더 간단하게 아래와 같이 익명 클래스로 등록 할 수도 있다.

```java
@Bean
public CommandLineRunner demoCommandLineRunner() {
    return new CommandLineRunner() {
        @Override
        public void run(String... args) throws Exception {
            System.out.println("========== CommandLineRunner ==========");
            for (var arg : args) {
                System.out.println("arg : " + arg);
            }
            System.out.println("=======================================");
        }
    };
}
```

CommandLineRunner는 함수형 인터페이스 이기에 람다식으로 더 간결하게 등록 할 수 있다.

```java
@Bean
public CommandLineRunner demoCommandLineRunner() {
    return args -> {
        System.out.println("========== CommandLineRunner ==========");
        for (var arg : args) {
            System.out.println("arg : " + arg);
        }
        System.out.println("=======================================");
    };
}
```

커맨드로 여러 인수(argument)와 함께 구동시키면 아래와 같은 결과를 얻을 수 있다.

커맨드 : > java -jar gitblog-example-0.0.1-SNAPSHOT.jar --fruit1=apple --fruit2=orange robot dog 

```java
========== CommandLineRunner ==========
arg : --fruit1=apple
arg : --fruit2=orange
arg : robot
arg : dog
=======================================
```

<br><br>

# ApplicationRunner

CommandLineRunner 보다 나중 공개된 인터페이스로 인수들을 String으로만 받던 것이 개선되어 옵션 인수의 이름/값 처리와 논옵션 인수를 처리 할 수 있게 되었다.

인수를 받는 타입 외에 빈으로 등록하는 방식이나 람다로 표현하는 것은 CommandLineRunner 와 같으니 인수에 대해서만 살펴본다.

```java
@Component
public class DemoApplicationRunner implements ApplicationRunner {
    @Override
    public void run(ApplicationArguments args) throws Exception {
        System.out.println("========== ApplicationRunner ==========");
        System.out.println("SourceArgs : " + Arrays.toString(args.getSourceArgs()));
        System.out.println("OptionNames : " + args.getOptionNames());
        for (var name : args.getOptionNames()) {
            System.out.println(name + "OptionValue : " + args.getOptionValues(name));
        }
        System.out.println("NonOptionArgs : " + args.getNonOptionArgs());
        System.out.println("=======================================");
    }
}
```

String 대신 `ApplicationArguments` 타입으로 인수를 받고, 아래의 선언된 메소드 들을 호출 할 수 있다. 

```java
public interface ApplicationArguments {
    String[] getSourceArgs();

    Set<String> getOptionNames();

    boolean containsOption(String name);

    List<String> getOptionValues(String name);

    List<String> getNonOptionArgs();
}
```

메소드만 봐도 동작이 예상되기에 따로 설명은 하지 않아도 될 것 같고,

아까와 동일한 커맨드로 실행 해 보면 아래와 같은 결과를 얻을 수 있다.

```java
========== ApplicationRunner ==========
SourceArgs : [--fruit1=apple, --fruit2=orange, robot, dog]
OptionNames : [fruit2, fruit1]
fruit2OptionValue : [orange]
fruit1OptionValue : [apple]
NonOptionArgs : [robot, dog]
=======================================
```

<br><br>

# ApplicationReadyEvent

@EventListener 애노테이션을 사용해서 실행 할 메소드를 ApplicationReadyEvent의 리스너로 등록하는 방법이다.

동일하게 @Component 또는 @Configuration 클래스 내에 정의하면 된다.

```java
@Component
public class DemoApplicationReadyEvent {
    @EventListener(ApplicationReadyEvent.class)
    public void init() {
        System.out.println("DemoApplicationReadyEvent listener");
    }
}
```

<br><br>

세가지 방법 모두 동시에 사용 가능하며, 이 경우 실행 순서는 아래와 같다.

```java
========== ApplicationRunner ==========
SourceArgs : [--fruit1=apple, --fruit2=orange, robot, dog]
OptionNames : [fruit2, fruit1]
fruit2OptionValue : [orange]
fruit1OptionValue : [apple]
NonOptionArgs : [robot, dog]
=======================================

========== CommandLineRunner ==========
arg : --fruit1=apple
arg : --fruit2=orange
arg : robot
arg : dog
=======================================

DemoApplicationReadyEvent listener
```

ApplicationRunner와 CommandLineRunner의 순서는 @Order 애노테이션으로 조정 가능하지만 @EventListener 는 마지막에 실행 될 수 밖에 없으며, 이는 아래 Spring 코드(SpringApplication.java)를 통해 확인 할 수 있다.

```java
public ConfigurableApplicationContext run(String... args) {
		StopWatch stopWatch = new StopWatch();
		stopWatch.start();
		DefaultBootstrapContext bootstrapContext = createBootstrapContext();
		ConfigurableApplicationContext context = null;
		configureHeadlessProperty();
		SpringApplicationRunListeners listeners = getRunListeners(args);
		listeners.starting(bootstrapContext, this.mainApplicationClass);
		try {
			ApplicationArguments applicationArguments = new DefaultApplicationArguments(args);
			ConfigurableEnvironment environment = prepareEnvironment(listeners, bootstrapContext, applicationArguments);
			configureIgnoreBeanInfo(environment);
			Banner printedBanner = printBanner(environment);
			context = createApplicationContext();
			context.setApplicationStartup(this.applicationStartup);
			prepareContext(bootstrapContext, context, environment, listeners, applicationArguments, printedBanner);
			refreshContext(context);
			afterRefresh(context, applicationArguments);
			stopWatch.stop();
			if (this.logStartupInfo) {
				new StartupInfoLogger(this.mainApplicationClass).logStarted(getApplicationLog(), stopWatch);
			}
			listeners.started(context);
			callRunners(context, applicationArguments); // 내부에서 Runner 세팅 및 call
		}
		catch (Throwable ex) {
			handleRunFailure(context, ex, listeners);
			throw new IllegalStateException(ex);
		}

		try {
			listeners.running(context); // 내부에서 ApplicationReadyEvent publish
		}
		catch (Throwable ex) {
			handleRunFailure(context, ex, null);
			throw new IllegalStateException(ex);
		}
		return context;
	}
```

코드는 [여기](https://github.com/clowoodive/gitblog.example).

위 프로젝트의 runner 패키지의 소스코드를 참고.

### References

- [https://madplay.github.io/post/run-code-on-application-startup-in-springboot](https://madplay.github.io/post/run-code-on-application-startup-in-springboot)
- [https://jeong-pro.tistory.com/206](https://jeong-pro.tistory.com/206)