---
title:  "[Spring] @Async와 ThreadPoolTaskExecutor"
excerpt: "@Async, @EnableAsync 애너테이션을 이용한 구현에 대해 간단히 알아보고 ThreadPoolTaskExecutor의 값들로 확인"

categories:
  - Spring
tags:
  - Spring
  - Annotation
  - 비동기
last_modified_at: 2022-05-17T00:00:00
---


Spring에는 비동기 처리를 위한 클래스들을 제공하고 있는데 @Async, @EnableAsync 애너테이션을 이용한 구현에 대해 간단히 알아보고 ThreadPoolTaskExecutor의 pool, queue가 어떻게 동작하는지 확인한 것을 정리한다.

- @Async
    - 클래스에 선언하면 클래스 내의 모든 메소드가 지정된 executor를 사용.
    - 메소드에 선언하면 해당 메소드만 비동기 처리, 클래스에서 지정한 excecutor를 재정의 가능.
    - 같은 클래스 내의 메소드에 @Async를 선언하고 호출하면 안됨.
    - private 메소드에는 사용 불가.
    - 비동기 메소드의 리턴타입이 없다면 void, 있다면 Future\<V> 인터페이스를 통해 반환.


- @EnableAsync
    - @Configuration가 선언된 executor 설정 클래스 위에 선언.


- ThreadPoolTaskExecutor
    - Java의 ThreadPoolExecutor를 bean 스타일로 세팅하고 감싸서 Spring에서 TaskExecutor로 노출 시켜주는 클래스.
    - poolSize, queueSize, activeCount등을 확인하기 위해 생성자 주입으로 얻어옴.
    - @Async를 사용하면 기본적으로 SimpleAsyncTaskExecutor를 사용한다고 나와 있으나 주입을 통해  default 상태의 ThreadPoolTaskExecutor를 생성한 것으로 추정.

설정을 통해 Executor bean을 등록하는 방법을 다룬 글들은 흔하게 찾아 볼 수 있으니, 이 글에서는 따로 executor를 설정하는 코드 없이  생성자 주입을 통해 ThreadPoolTaskExecutor를 가져오는 방식으로 한다. 따라서 설정 클래스가 없기에 service에 @EnableAsync를 붙여주었고 pool 관련 속성들은 application.properties 파일에 정의했다.

```
# application.properties
spring.task.execution.pool.core-size=4
spring.task.execution.pool.max-size=19
spring.task.execution.pool.queue-capacity=7
```

```java
@Slf4j
@EnableAsync
@Service
public class AsyncService {
    @Async
    public void runAction(int i) throws InterruptedException {
        Thread.sleep(500);
        log.info(Thread.currentThread().getName() + " count: " + i);
    }
}
```

비동기로 처리될 메소드 runAction은 확인을 쉽게 하기위해 sleep을 주고, 파라미터로 받은 값(호출순서) 만 출력한다.

```java
@RestController
public class AsyncController {
    private final AsyncService asyncService;
    private final ThreadPoolTaskExecutor executor;

    public AsyncController(AsyncService asyncService, ThreadPoolTaskExecutor executor) {
        this.asyncService = asyncService;
        this.executor = executor;
    }

    @GetMapping("/call-async")
    public String callAsync() throws InterruptedException {
        System.out.println("- init state");
        printExecutorState();
        System.out.println("- running state");
        for (int i = 0; i < 26; i++) {
            this.asyncService.runAction(i);
            printExecutorState();
        }

        return "success";
    }

    private void printExecutorState() {
        System.out.print("corePoolSize: " + this.executor.getCorePoolSize());
        System.out.print(", maxPoolSize: " + this.executor.getMaxPoolSize());
        System.out.print(", poolSize: " + this.executor.getPoolSize());
        System.out.print(", queueSize: " + this.executor.getThreadPoolExecutor().getQueue().size());
        System.out.println(", activeCount: " + this.executor.getActiveCount());
    }
}
```

호출하는 controller에서는 주입받은 ThreadPoolTaskExecutor의 속성값들을 출력한다.

executor의 속성값들에 따라 queue와 pool동작이 다르고, 서비스 환경과 맞지 않으면 exception이 발생 할 수도 있는데 속성값 이름으로 의미를 판단해서 잘못 동작하는 사례가 많은것 같았다. 
실 서비스에 사용 시에는 무턱대고 블로그를 따라하기 보다는 공식 [document](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/ThreadPoolExecutor.html)를 보는게 좋을 것 같다.

```
- init state
corePoolSize: 4, maxPoolSize: 19, poolSize: 0, queueSize: 0, activeCount: 0
- increase poolSize
corePoolSize: 4, maxPoolSize: 19, poolSize: 1, queueSize: 0, activeCount: 1
corePoolSize: 4, maxPoolSize: 19, poolSize: 2, queueSize: 0, activeCount: 2
corePoolSize: 4, maxPoolSize: 19, poolSize: 3, queueSize: 0, activeCount: 3
corePoolSize: 4, maxPoolSize: 19, poolSize: 4, queueSize: 0, activeCount: 4
- using queue
corePoolSize: 4, maxPoolSize: 19, poolSize: 4, queueSize: 1, activeCount: 4
corePoolSize: 4, maxPoolSize: 19, poolSize: 4, queueSize: 2, activeCount: 4
corePoolSize: 4, maxPoolSize: 19, poolSize: 4, queueSize: 3, activeCount: 4
corePoolSize: 4, maxPoolSize: 19, poolSize: 4, queueSize: 4, activeCount: 4
corePoolSize: 4, maxPoolSize: 19, poolSize: 4, queueSize: 5, activeCount: 4
corePoolSize: 4, maxPoolSize: 19, poolSize: 4, queueSize: 6, activeCount: 4
corePoolSize: 4, maxPoolSize: 19, poolSize: 4, queueSize: 7, activeCount: 4
- expand poolSize
corePoolSize: 4, maxPoolSize: 19, poolSize: 5, queueSize: 7, activeCount: 5
corePoolSize: 4, maxPoolSize: 19, poolSize: 6, queueSize: 7, activeCount: 6
corePoolSize: 4, maxPoolSize: 19, poolSize: 7, queueSize: 7, activeCount: 7
corePoolSize: 4, maxPoolSize: 19, poolSize: 8, queueSize: 7, activeCount: 8
corePoolSize: 4, maxPoolSize: 19, poolSize: 9, queueSize: 7, activeCount: 9
corePoolSize: 4, maxPoolSize: 19, poolSize: 10, queueSize: 7, activeCount: 10
corePoolSize: 4, maxPoolSize: 19, poolSize: 11, queueSize: 7, activeCount: 11
corePoolSize: 4, maxPoolSize: 19, poolSize: 12, queueSize: 7, activeCount: 12
corePoolSize: 4, maxPoolSize: 19, poolSize: 13, queueSize: 7, activeCount: 13
corePoolSize: 4, maxPoolSize: 19, poolSize: 14, queueSize: 7, activeCount: 14
corePoolSize: 4, maxPoolSize: 19, poolSize: 15, queueSize: 7, activeCount: 15
corePoolSize: 4, maxPoolSize: 19, poolSize: 16, queueSize: 7, activeCount: 16
corePoolSize: 4, maxPoolSize: 19, poolSize: 17, queueSize: 7, activeCount: 17
corePoolSize: 4, maxPoolSize: 19, poolSize: 18, queueSize: 7, activeCount: 18
corePoolSize: 4, maxPoolSize: 19, poolSize: 19, queueSize: 7, activeCount: 19

2022-05-17 16:29:13.478  INFO 24736 --- [ task-1] AsyncService  : task-1 count: 0
2022-05-17 16:29:13.478  INFO 24736 --- [ task-2] AsyncService  : task-2 count: 1
2022-05-17 16:29:13.478  INFO 24736 --- [task-12] AsyncService  : task-12 count: 18
2022-05-17 16:29:13.478  INFO 24736 --- [task-11] AsyncService  : task-11 count: 17
2022-05-17 16:29:13.478  INFO 24736 --- [ task-4] AsyncService  : task-4 count: 3
2022-05-17 16:29:13.478  INFO 24736 --- [ task-8] AsyncService  : task-8 count: 14
2022-05-17 16:29:13.478  INFO 24736 --- [task-15] AsyncService  : task-15 count: 21
2022-05-17 16:29:13.478  INFO 24736 --- [ task-9] AsyncService  : task-9 count: 15
2022-05-17 16:29:13.478  INFO 24736 --- [task-17] AsyncService  : task-17 count: 23
2022-05-17 16:29:13.478  INFO 24736 --- [task-16] AsyncService  : task-16 count: 22
2022-05-17 16:29:13.478  INFO 24736 --- [ task-6] AsyncService  : task-6 count: 12
2022-05-17 16:29:13.478  INFO 24736 --- [task-18] AsyncService  : task-18 count: 24
2022-05-17 16:29:13.478  INFO 24736 --- [ task-7] AsyncService  : task-7 count: 13
2022-05-17 16:29:13.478  INFO 24736 --- [ task-5] AsyncService  : task-5 count: 11
2022-05-17 16:29:13.478  INFO 24736 --- [task-13] AsyncService  : task-13 count: 19
2022-05-17 16:29:13.478  INFO 24736 --- [ task-3] AsyncService  : task-3 count: 2
2022-05-17 16:29:13.478  INFO 24736 --- [task-10] AsyncService  : task-10 count: 16
2022-05-17 16:29:13.478  INFO 24736 --- [task-14] AsyncService  : task-14 count: 20
2022-05-17 16:29:13.478  INFO 24736 --- [task-19] AsyncService  : task-19 count: 25

2022-05-17 16:29:13.980  INFO 24736 --- [task-12] AsyncService  : task-12 count: 5
2022-05-17 16:29:13.980  INFO 24736 --- [ task-1] AsyncService  : task-1 count: 8
2022-05-17 16:29:13.980  INFO 24736 --- [task-15] AsyncService  : task-15 count: 10
2022-05-17 16:29:13.980  INFO 24736 --- [task-11] AsyncService  : task-11 count: 6
2022-05-17 16:29:13.980  INFO 24736 --- [ task-2] AsyncService  : task-2 count: 4
2022-05-17 16:29:13.980  INFO 24736 --- [ task-8] AsyncService  : task-8 count: 9
2022-05-17 16:29:13.980  INFO 24736 --- [ task-4] AsyncService  : task-4 count: 7
```

하단은 비동기 메소드 내의 로그인데 count값이 뒤죽박죽이고 thread가 1개가 아닌 것으로 보아 잘 동작하고 있다.
중요한 것은 상단 로그인데, 최초 값들의 상태는 application.properties에 정의한 것과 같다. “increase poolSize” 부터는 task가 thread pool에 하나씩 할당되면서 poolSize가 corePoolSize 에 도달 할 때 까지 poolSize와 active개수가 늘어난다. 그리고 corePoolSize에 도달하고나면 queue에 적재하면서 queueSize가 늘어나는 것을 “using queue” 에서 알수 있다. queue도 최대 용량까지 다 사용하고 나면 그때서야 maxPoolSize만큼까지 poolSize를 늘려가며 task를 할당하게 된다. 

예제에서는 26개의 메소드를 호출했고 poolSize가 maxPoolSize에 딱맞게 도달할 만큼 사용했는데 여기서 27개의 메소드를 호출하면 어떻게 될까?

```
java.util.concurrent.RejectedExecutionException: Task java.util.concurrent.FutureTask@7e69bb2c[Not completed, task = org.springframework.aop.interceptor.AsyncExecutionInterceptor$$Lambda$681/0x000000080051b840@5acd4831] rejected from java.util.concurrent.ThreadPoolExecutor@35b094ed[Running, pool size = 19, active threads = 19, queued tasks = 7, completed tasks = 0]

```

이렇게 상태값과 함께 exception이 발생해 버린다.

알아낸 사실은 동시에 허용 가능한 비동기 처리의 개수는 queue-capacity + max-size 라는 것이다.

그리고 메소드에서 남긴 로그 중 상단 19개와 하단 7개의 시간차가 있는것으로 보아 pool에 적재된 19개를 먼저 처리하고, 비교적 빨리 호출되었지만 queue에 적재된 7개는 나중 처리되었음을 알 수 있다.

확인 하면서 놓친것이 있을 수도 있으니 실 서비스에는 꼼꼼하게 더 알아보고 예측을 잘해서 속성값을 세팅해야 겠다.