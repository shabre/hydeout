---
layout: post
title: Github pull request 디폴트 리뷰어 설정
categories:
    - Development
excerpt_separator: "<!--more-->"
---


Github 에서 매번 PR(pull request)을 올릴 때마다 저는 리뷰어를 한땀한땀 검색 후 지정했었는데 그러지 않아도 되는 방법을 알게되어 포스팅 합니다.

**1. 프로젝트 root 폴더에 `CODEOWNERS` 파일 생성
2. 적용할 범위와 사용자 깃허브 아이디(또는 이메일) 를 작성**


```
# This is a comment.
# Each line is a file pattern followed by one or more owners.

# These owners will be the default owners for everything in
# the repo. Unless a later match takes precedence,
# @global-owner1 and @global-owner2 will be requested for
# review when someone opens a pull request.
*       @global-owner1 @global-owner2
```

**3. 리뷰어를 별도 지정하지 않고 pull reqeust 생성**

이제 아래 그림처럼 자동으로 리뷰어가 설정됩니다.

![image.png](({{ site.baseurl }}/images/reviewer.png)

원문: https://docs.github.com/en/github/creating-cloning-and-archiving-repositories/about-code-owners