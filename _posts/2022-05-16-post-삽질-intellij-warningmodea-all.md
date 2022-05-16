---
title:  "[삽질] IntelliJ 프로젝트 --warning-mode all"
excerpt: ""

categories:
  - 삽질
tags:
  - 삽질
  - IntelliJ
last_modified_at: 2022-05-16T00:00:00
---


빈 껍데기 프로젝트를 IntelliJ에서 로드 한 뒤 프로젝트 명을 변경하고 실행을 했는데 아래 메시지가 계속 출력된다.

```
Deprecated Gradle features were used in this build, making it incompatible with Gradle 8.0.
```

검색한 방법대로 아래처럼 옵션에 추가를 이리저리 해봤지만 정보가 더이상 나오지는 않았다.

![intellij_warningmodeall]({{ '/assets/images/intellij_warningmodeall.png' | relative_url }}){: .align-center}

결국 gradle.properties 파일을 생성한 뒤 아래 옵션을 넣어주고

```
org.gradle.warning.mode=all
```

실행을 했더니 자세한 정보가 아래와 같이 나온다.

```
The JavaExec.main property has been deprecated.
...생략
```

해당 warning의 원인은 검색으로 찾지 못했고 삽질 끝에 알아낸 것은 프로젝트 명을 변경하면서 기존 프로젝트 명으로 세팅된 IntelliJ 세팅들이 남아있었기 때문이었다.

- 오늘의 교훈
    - 귀찮아도 새 프로젝트 세팅은 새로 하자.
    - CLI 옵션이 잘 적용되지 않으면 세팅 파일로 적용하자.