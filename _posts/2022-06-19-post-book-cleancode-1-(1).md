---
title: "[Book] Clean Code - 1장. 깨끗한 코드(1)"
excerpt: ""

categories:
  - Book
tags:
  - Refactoring
  - Clean Code
last_modified_at: 2022-06-19T00:00:00
---


### 코드가 존재하리라

요구사항에 가까운 언어를 만들수도 있고, 요구사항에서 정형 구조를 뽑아내는 도구를 만들수도 있지만 어느 순간에는 정밀한 표현이 필요하기에 코드는 항상 존재할 거라고 얘기한다.

언젠가 조사에서 미래에 인공지능이 대체하게 될 직업의 상위권에 프로그래머가 있는것을 본 기억이 있는데 내 생각도 로버트 마틴의 말처럼 코드가 없어지거나, 프로그래머가 없어지진 않을것 같다는데 동의한다.

하지만 불가능 하다고 생각했던 많은 것들이 현실이 된 지금 앞으로 인공지능과 머신러닝, 연산능력 같은 것들이 더 발달되면 없어지진 않아도 많은 인원이 필요하지 않게 되지는 않을 것 같다는 생각은 든다. 

그런 시간이 와도 개발자로서 일할 수 있도록 실력을 쌓아야지.

### 나쁜 코드

나쁜 코드에 의해 점점 전체 코드가 지저분해져 가고 결국 디버깅/기능 개발이 점점 어려워 진다는 것이다. 쓰레기 코드를 나중 손보겠다는 것은 르블랑의 법칙에 의해 결코 불가능 하다는 것. 

나는 전적으로 동의하지는 않는다. 나중이라는 시점이 오기는 어렵지만 여건이 됐을때 하고자 하는 의지가 더 중요한 것 같다. 시간적인 여건이 부족해서 어쩔 수 없이 미뤄둔 나쁜 코드들도 되도록 신규기능이나 수정을 하면서 근처에 있다면 그때그때 수정해 왔기에 할수 있다고 생각한다. 다만 나중으로 미루는 것이 더 안좋은 방법이라는 것에는 동의한다.

### 태도

몇 줄 고치면 될 것 같았던 일인데 모듈을 수십개 건드리게 된 경험 나도 있었던 것 같다. 설계를 뒤집는 요구사항의 변경? 부족한 일정?  이유야 많지만 실제 코드를 짜는 것은 프로그래머인 우리들이고 기획서대로 착착 코딩만 하지 않는다. 기술적인 어려움과 일정 등으로 기획에 대안을 제시하기도 하고 수정요청을 하기도 한다. 그만큼 관여 하면서 진행 했다면 코드를 설계 하고 구현한 프로그래머의 책임이 더 크지 않을까?

### 깨끗한 코드란?

보기에 즐거운 우아한 코드.

한 가지를 잘하는 코드.

의존성이 낮은 코드.

다른 사람이 고치기 쉬운 코드.

TDD를 지키는 코드.

중복이 적은 코드.