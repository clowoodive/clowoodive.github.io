---
title: "[Design Pattern-행위] 커맨드(Command) 패턴"
excerpt: ""

categories:
  - Design Pattern
tags:
  - 행위 패턴
last_modified_at: 2023-04-20T00:00:00
---


## 정의

다른 객체로의 직접적인 요청을 요청에 대한 모든 정보(i.e. param)를 포함하는 독립 실행 개체(Command)로 변환해서 결합도를 낮추는 패턴.

주로 “관심사 분리”를 통해 이루어지며 아래 예제로 보면 버튼(Invoker)은 어떤 객체(Editor)가 요청을 받고 처리할지 알 필요 없게되면서 인터페이스 레이어와 비즈니스 로직 레이어가 분리됨.

- 중재자 패턴 - 객체들간의 직접 연결을 제거하고 중재자가 간접적으로 전달
- 커맨드 패턴 - 발신자와 수신자 간의 단방향 연결
- 옵저버 패턴 - 수신자들이 동적으로 구독 및 구독 취소
- 책임연쇄 패턴 - 수신자 중 하나에 의해 요청이 처리 될 때까지 요청을 순차적으로 전달

![design-pattern-command1]({{ '/assets/images/design-pattern-command1.png' | relative_url }}){: .align-center}

## 예제

```java
interface Command {
    void execute();
}

class CopyButtonInvoker {

    private Command command;

    public void setCommand(Command command) {
        this.command = command;
    }

    public void executeCommand() {
        command.execute();
    }
}

class PasteButtonInvoker {

    private Command command;

    public void setCommand(Command command) {
        this.command = command;
    }

    public void executeCommand() {
        command.execute();
    }
}

class EditorReceiver {

    private String text1 = "text1";

    private String text2 = "text2";

    private String clipboard = "";

    public String getText1() {
        return text1;
    }

    public void setText1(String text1) {
        this.text1 = text1;
    }

    public String getText2() {
        return text2;
    }

    public void setText2(String text2) {
        this.text2 = text2;
    }

    public String getClipboard() {
        return clipboard;
    }

    public void setClipboard(String clipboard) {
        this.clipboard = clipboard;
    }
}

class CopyCommand implements Command {

    EditorReceiver editor;

    String param;

    public CopyCommand(EditorReceiver editor, String param) {
        this.editor = editor;
        this.param = param;
    }

    @Override
    public void execute() {
        this.editor.setClipboard(this.editor.getText1() + param);
        System.out.println("copyCommand executed, editor.text1 copy to clipboard");
    }
}

class PasteCommand implements Command {

    EditorReceiver editor;

    String param;

    public PasteCommand(EditorReceiver editor, String param) {
        this.editor = editor;
        this.param = param;
    }

    @Override
    public void execute() {
        this.editor.setText2(this.editor.getClipboard() + param);
        System.out.println("pasteCommand executed, clipboard copy to editor.text2");
    }
}
```

테스트 코드로 확인.

```java
@Test
void command() {
    EditorReceiver editor = new EditorReceiver();
    Command copyCommand = new CopyCommand(editor, "(copied to clipboard)");
    Command pastCommand = new PasteCommand(editor, "(and copied to text2)");
    CopyButtonInvoker copyButton = new CopyButtonInvoker();
    PasteButtonInvoker pasteButton = new PasteButtonInvoker();
    copyButton.setCommand(copyCommand);
    pasteButton.setCommand(pastCommand);

    System.out.println("----- before -----");
    System.out.println("editor.text1 : " + editor.getText1());
    System.out.println("editor.text2 : " + editor.getText2());
    System.out.println("editor.clipboard : " + editor.getClipboard());

    System.out.println("----- action -----");
    copyButton.executeCommand();
    pasteButton.executeCommand();

    System.out.println("----- after -----");
    System.out.println("editor.text1 : " + editor.getText1());
    System.out.println("editor.text2 : " + editor.getText2());
    System.out.println("editor.clipboard : " + editor.getClipboard());
}
```

출력 결과.

```powershell
----- before -----
editor.text1 : text1
editor.text2 : text2
editor.clipboard : 
----- action -----
copyCommand executed, editor.text1 copy to clipboard
pasteCommand executed, clipboard copy to editor.text2
----- after -----
editor.text1 : text1
editor.text2 : text1(copied to clipboard)(and copied to text2)
editor.clipboard : text1(copied to clipboard)
```

## 더 복잡한 구조의 예제는 아래 펼쳐보기.

<details>

  <summary> 참고문서의 예제처럼 복잡한 기능을 사용한 예제 </summary>
  <div markdown="1">

```java
//  - 인터페이스 대신 추상 클래스를 상속한 커맨드 사용
//  - 버튼의 리스너 등록방식을 흉내내기 위한 콜백 사용
//  - 히스토리를 사용하기 위해 버튼이 커맨드를 직접 호출하지 않고 application의 executeCommand를 통해서 호출

interface ExecuteCallback {
    void callExecuteCommand();
}

// invoker
class CopyButton {

    ExecuteCallback callback;

    public CopyButton(ExecuteCallback callback) {
        this.callback = callback;
    }

    public void action() {
        this.callback.callExecuteCommand();
    }
}

class PasteButton {

    ExecuteCallback callback;

    public PasteButton(ExecuteCallback callback) {
        this.callback = callback;
    }

    public void action() {
        this.callback.callExecuteCommand();
    }
}

class EditorReceiver {

    private String text1 = "editor's text1";

    private String text2 = "editor's text2";

    private String clipboard = "";

    public String getText1() {
        return text1;
    }

    public void setText1(String text1) {
        this.text1 = text1;
    }

    public String getText2() {
        return text2;
    }

    public void setText2(String text2) {
        this.text2 = text2;
    }

    public String getClipboard() {
        return clipboard;
    }

    public void setClipboard(String clipboard) {
        this.clipboard = clipboard;
    }
}

public class CommandApplication {

    public CopyButton copyButton;

    public PasteButton pasteButton;

    public EditorReceiver editorReceiver;

    public void init() {
        editorReceiver = new EditorReceiver();

        this.copyButton = new CopyButton(new ExecuteCallback() {
            @Override
            public void callExecuteCommand() {
                executeCommand(new CopyCommand(editorReceiver));
            }
        });

        this.pasteButton = new PasteButton(new ExecuteCallback() {
            @Override
            public void callExecuteCommand() {
                executeCommand(new PasteCommand(editorReceiver));
            }
        });
    }

    public void executeCommand(Command command) {
        command.execute();
    }
}

abstract class Command {

    EditorReceiver editorReceiver;

    public Command(EditorReceiver editorReceiver) {
        this.editorReceiver = editorReceiver;
    }

    // 공통 기능
    public void undo() {
    }

    // command 제공 인터페이스
    public abstract void execute();
}

class CopyCommand extends Command {

    public CopyCommand(EditorReceiver editorReceiver) {
        super(editorReceiver);
    }

    @Override
    public void execute() {
        this.editorReceiver.setClipboard(this.editorReceiver.getText1());
        System.out.println("copyCommand executed, editor.text1 saved in clipboard");
    }
}

class PasteCommand extends Command {

    public PasteCommand(EditorReceiver editorReceiver) {
        super(editorReceiver);
    }

    @Override
    public void execute() {
        this.editorReceiver.setText2(this.editorReceiver.getClipboard());
        System.out.println("pasteCommand executed, clipboard saved in editor.text2");
    }
}
```
테스트 코드로 확인.
```java
@Test
void command() {
    CommandApplication application = new CommandApplication();
    application.init();

    System.out.println("editor.text1 : " + application.editorReceiver.getText1());
    System.out.println("editor.text2 : " + application.editorReceiver.getText2());
    System.out.println("editor.clipboard : " + application.editorReceiver.getClipboard());

    application.copyButton.action();
    application.pasteButton.action();

    System.out.println("editor.text1 : " + application.editorReceiver.getText1());
    System.out.println("editor.text2 : " + application.editorReceiver.getText2());
    System.out.println("editor.clipboard : " + application.editorReceiver.getClipboard());
}
```
  </div>
</details>

<br>

## 적용 케이스

- 런타임에 작업(커맨드)를 변경하고 싶을 때
- 작업들을 독립 실행형 객체로 변환하는 것이므로, 작업들의 실행 예약, 원격실행을 하고싶을 때
- undo/redo 기능을 구현하고 싶을 때

## 고려사항

- 코드가 복잡함.

### References

- [https://refactoring.guru/ko/design-patterns/command](https://refactoring.guru/ko/design-patterns/command)