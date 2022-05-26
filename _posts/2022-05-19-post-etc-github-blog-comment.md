---
title:  "[Etc] GitHub 블로그(Jekyll)에 Utterances 댓글 적용하기"
excerpt: ""

categories:
  - Etc
tags:
  - Jekyll
  - GitHub 블로그
last_modified_at: 2022-05-19T00:00:00
---

{% capture notice-env %}
#### Environment
- minimal-mistakes-jekyll 4.24.0
{% endcapture %}

<div class="notice--primary">{{ notice-env | markdownify }}</div>

Disqus의 경우 광고가 너무 많이 달린다는 것을 알고 있었기에 댓글 기능 추가를 미루고 있었는데 Utterances가 적용된 블로그를 보니 깔끔하고 광고도 없다고 해서 도입 결정!

## Utterances?

- 광고가 없다.
- 가볍고 Github 이슈 기반으로 이슈 생성도 해준다.
- 댓글에 마트다운 적용이 가능하다.

이슈를 찾아서 댓글로 보여주기 위해 GitHub issue search API도 사용하는 것으로 보이고, 이슈 생성은 [utterances-bot](https://github.com/utterances-bot) 이라는 것이 해주는 것 같다. 댓글을 달면 이슈가 생성되는 방식이어서 그런지 작성 시 Utterances가 GitHub [OAuth flow](https://developer.github.com/v3/oauth/#web-application-flow)를 사용하여 대신 게시하는 것을 동의하는 절차도 있었다.

## 적용하기

### 설치

[GitHub Apps - utterances](https://github.com/apps/utterances) 접속.

![github_blog_utterances1]({{ '/assets/images/github_blog_utterances1.png' | relative_url }}){: .align-center}

Install을 클릭하면 아래 설치 설정 페이지가 나온다. 모든 repository에 적용할 필요는 없어서 블로그 repository만 지정했다.

![github_blog_utterances2]({{ '/assets/images/github_blog_utterances2.png' | relative_url }}){: .align-center}

### 스크립트 생성

설치후 [https://utteranc.es/](https://utteranc.es/) 페이지로 이동하는데 repo경로는 방금 설치한 utterances app이 설치된 경로여야하고 이슈가 게시되는 경로이다. 
post와 issue가 매핑되는 방식은 잘 읽어보고 적절히 결정하고, issue에 달릴 label과 thema를 선택한다.

![github_blog_utterances3]({{ '/assets/images/github_blog_utterances3.png' | relative_url }}){: .align-center}

![github_blog_utterances4]({{ '/assets/images/github_blog_utterances4.png' | relative_url }}){: .align-center}

세팅을 다하면 하단에 세팅한 script가 작성되고 이걸 복사해서 post에 적용되는 layout에 넣으면 된다.

![github_blog_utterances5]({{ '/assets/images/github_blog_utterances5.png' | relative_url }}){: .align-center}

### 스크립트 적용

보통은 Jekyll프로젝트에서 _layouts/post.html 파일이 포스트 layout인것 같은데 필자의 블로그는 single.html 파일이었다.
최하단 이외에 적용시 위치가 제멋대로여서 어쩔 수 없이 최하단에 아래와 같이 적용했다.

```html
... 생략
    </div>
</div>

<div>
    <script src="https://utteranc.es/client.js"
        repo="clowoodive/clowoodive.github.io"
        issue-term="pathname"
        label="Comment"
        theme="github-light"
        crossorigin="anonymous"
        async>
    </script>
</div>
```

## 확인

![github_blog_utterances6]({{ '/assets/images/github_blog_utterances6.png' | relative_url }}){: .align-center}

위와 같이 post에 댓글을 달고 해당 repository에 가서 봤더니 issues 탭에 올라와 있었다.

![github_blog_utterances7]({{ '/assets/images/github_blog_utterances7.png' | relative_url }}){: .align-center}