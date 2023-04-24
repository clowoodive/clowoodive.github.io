---
title: "[Design Pattern-행위] 책임연쇄(Chain of Responsibility) 패턴"
excerpt: ""

categories:
  - Design Pattern
tags:
  - 행위 패턴
last_modified_at: 2023-04-23T00:00:00
---


## 정의

특정 행동들을 handler라는 독립 실행형 개체로 변환해서 체인을 구성한 뒤, 그 체인을 따라 실행하면서 처리가 가능할때 까지, 혹은 불가능할때 까지 등의 방식으로 연쇄적으로 수행하는 패턴.

모든 개체는 같은 인터페이스를 구현하고 동일한 메소드를 간접적으로 호출하게 됨.

핸들러 수행은 체인의 어느 위치에서도 시작 가능.

- 중재자 패턴 - 객체들간의 직접 연결을 제거하고 중재자가 간접적으로 전달
- 커맨드 패턴 - 발신자와 수신자 간의 단방향 연결
- 옵저버 패턴 - 수신자들이 동적으로 구독 및 구독 취소
- 책임연쇄 패턴 - 수신자 중 하나에 의해 요청이 처리 될 때까지 요청을 순차적으로 전달

![design-pattern-chain1]({{ '/assets/images/design-pattern-chain1.png' | relative_url }}){: .align-center}

## 예제

```java
interface Handler {
    void setNext(Handler handler);

    boolean handle(String request);
}

abstract class BaseHandler implements Handler {

    Handler next;

    @Override
    public void setNext(Handler handler) {
        this.next = handler;
    }

    @Override
    public boolean handle(String request) {
        return false;
    }
}

class LogHandler extends BaseHandler {

    @Override
    public void setNext(Handler handler) {
        super.setNext(handler);
    }

    @Override
    public boolean handle(String request) {
        System.out.println("LogHandler - logging");
        if (this.next != null)
            return this.next.handle(request);

        return true;
    }
}

class PathCheckHandler extends BaseHandler {

    private String checkStr;

    public void setCheckString(String checkStr) {
        this.checkStr = checkStr;
    }

    private boolean isValidPath(String request) {
        return request.contains(checkStr);
    }

    @Override
    public void setNext(Handler handler) {
        super.setNext(handler);
    }

    @Override
    public boolean handle(String request) {
        if (isValidPath(request)) {
            System.out.println("PathCheckHandler - valid");
        } else {
            System.out.println("PathCheckHandler - invalid");
            return false;
        }

        if (this.next != null)
            return this.next.handle(request);
        else {
            return true;
        }
    }
}

class LengthCheckHandler extends BaseHandler {

    private int checkLengthMax;

    public void setCheckLengthMax(int lengthMax) {
        this.checkLengthMax = lengthMax;
    }

    private boolean isValidLength(String request) {
        return request.length() <= this.checkLengthMax;
    }

    @Override
    public void setNext(Handler handler) {
        super.setNext(handler);
    }

    @Override
    public boolean handle(String request) {
        if (isValidLength(request)) {
            System.out.println("LengthCheckHandler - valid");
        } else {
            System.out.println("LengthCheckHandler - invalid");
            return false;
        }

        if (this.next != null)
            return this.next.handle(request);
        else {
            return true;
        }
    }
}

public class ChainOfResponsibilityServer {

    private Handler handler;

    public void setHandler(Handler handler) {
        this.handler = handler;
    }

    public boolean processRequest(String request) {
        System.out.println("------- request(" + request + ") process start -------");
        return this.handler.handle(request);
    }
}
```

테스트 코드로 확인.

```java
@Test
void chainOfResponsibility() {
    ChainOfResponsibilityServer server = new ChainOfResponsibilityServer();
    LogHandler logHandler = new LogHandler();
    PathCheckHandler pathCheckHandler = new PathCheckHandler();
    pathCheckHandler.setCheckString("admin");
    LengthCheckHandler lengthCheckHandler = new LengthCheckHandler();
    lengthCheckHandler.setCheckLengthMax(15);

    server.setHandler(logHandler);
    logHandler.setNext(pathCheckHandler);
    pathCheckHandler.setNext(lengthCheckHandler);

    // all pass
    boolean case1 = server.processRequest("/case1/admin");
    System.out.println("case1 result : " + case1);

    // second handler fail
    pathCheckHandler.setCheckString("user");
    boolean case2 = server.processRequest("/case2/admin");
    System.out.println("case2 result : " + case2);

    // third handler fail
    lengthCheckHandler.setCheckLengthMax(10);
    boolean case3 = server.processRequest("/case3/user/login");
    System.out.println("case3 result : " + case3);
}
```

출력 결과.

```powershell
------- request(/case1/admin) process start -------
LogHandler - logging
PathCheckHandler - valid
LengthCheckHandler - valid
case1 result : true
------- request(/case2/admin) process start -------
LogHandler - logging
PathCheckHandler - invalid
case2 result : false
------- request(/case3/user/login) process start -------
LogHandler - logging
PathCheckHandler - valid
LengthCheckHandler - invalid
case3 result : false
```

## 적용 케이스

- 다양한 방식의 다양한 종류의 처리를 할 것으로 예상되지만 정확한 순서를 미리 정할수 없을 때
- 핸들러의 집합 구성과 순서를 런타임에 변경해야 할 때
- java.servlet.Filter.doFilter()
- java.util.logging.Logger.log()

## 고려사항

- 처리할 핸들러를 찾기 위한 것인지, 실패 하기 전 핸들러까지 모두 수행할 것인지 등의 결정 필요

### References

- [https://refactoring.guru/ko/design-patterns/chain-of-responsibility](https://refactoring.guru/ko/design-patterns/chain-of-responsibility)