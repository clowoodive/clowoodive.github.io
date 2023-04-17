---
title: "[Design Pattern-행위] 상태(State) 패턴"
excerpt: ""

categories:
  - Design Pattern
tags:
  - 행위 패턴
  - 합성
last_modified_at: 2023-04-16T00:00:00
---


## 정의

객체(Context)의 상태에 따라 메소드들의 내부 구현이 달라지는 경우 그 상태와 그 상태에 종속된 메소드 구현을 클래스로 분리해서 Context가 상태 객체를 참조하게 함으로써, Context로의 요청을 그 상태 객체에게 위임하는 패턴.

- State도 Context를 참조하고 있으므로 필요에 따라 Context 메소드/필드를 호출 가능
- 전략 패턴과 유사하지만 상태 패턴은 상태끼리 서로 알고 상태 천이가 가능한 반면 전략 패턴의 전략들은 서로에 대해 알지 못함

![design-pattern-state1]({{ '/assets/images/design-pattern-state1.png' | relative_url }}){: .align-center}
## 예제

```java
public class StateContext {
    private State state;

    public StateContext() {
        this.state = new ReadyState(this);
    }

    public void changeState(State state) {
        this.state = state;
    }

    // state에 따라 달라지는 행동
    public void stopAction() {
        this.state.stop();
    }

    public void playAction() {
        this.state.play();
    }

    // state에서 호출도 가능
    private void stopMusic() {
        System.out.println("music stop");
    }

    private void playMusic() {
        System.out.println("music play");
    }

    // 중첩 클래스
    abstract class State {
        protected StateContext stateContext;

        public State(StateContext stateContext) {
            this.stateContext = stateContext;
        }

        abstract void stop();

        abstract void play();
    }

    // 중첩 클래스
    class ReadyState extends State {
        public ReadyState(StateContext stateContext) {
            super(stateContext);
        }

        @Override
        void stop() {
            System.out.println("ready state - stop() called, already stopped");
        }

        @Override
        void play() {
            this.stateContext.changeState(new PlayState(this.stateContext));
            System.out.println("ready state - play() called, change to play state");
            this.stateContext.playMusic();
        }
    }

    // 중첩 클래스
    class PlayState extends State {
        public PlayState(StateContext stateContext) {
            super(stateContext);
        }

        @Override
        void stop() {
            this.stateContext.changeState(new ReadyState(this.stateContext));
            System.out.println("play state - stop() called, change to stp state");
            this.stateContext.stopMusic();
        }

        @Override
        void play() {
            System.out.println("play state - play() called, already playing");
        }
    }
}
```

테스트 코드로 확인.

```java
@Test
void state() {
    StateContext stateContext = new StateContext();
    stateContext.stopAction();
    stateContext.playAction();
    stateContext.playAction();
    stateContext.stopAction();
}
```

출력 결과.

```powershell
ready state - stop() called, already stopped
ready state - play() called, change to play state
music play
play state - play() called, already playing
play state - stop() called, change to stp state
music stop
```

## 적용 케이스

- 현재 상태에 따라 다른게 행동하는 객체일 때
- 클래스의 필드값에 따라 알고리즘이 분기되는 거대한 조건문이 있는 경우

## 고려사항

- 상태의 코드가 Context의 비공개 필드나 메서드에 의존한다면 우선 public으로 선언
- 중첩 클래스를 지원한다면 상태 클래스들을 Context에 넣으면 비공개 필드/메서드에 접근 가능

### References

- [https://refactoring.guru/ko/design-patterns/state](https://refactoring.guru/ko/design-patterns/state)