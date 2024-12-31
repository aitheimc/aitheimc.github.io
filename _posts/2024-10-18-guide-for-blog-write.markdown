---
layout: post
title:  더아이엠씨 AI 블로그 작성법 및 작성 규정정
date:   2024-10-18
author: Jin
excerpt_separator: "<!--more-->"
tags:
    - guide
    - markdown
    - rule
---

본 포스팅은 더아이엠씨 AI 블로그 작성을 위해 필요한 정보를 서술하였습니다. `작성 위치`, `활용 문법 및 파일 이름 구성`, `문체`, `layout 작성법`, `디자인 참조`에 대한 내용을 포함하고 있습니다. 참고로 처음 블로그 글을 한다면 `author`내용을 추가해야하며, `layout`은 블로그 글 첫줄 부터 반드시 작성하고 나머지 글을 작성해야 합니다.

<!--more-->

> ⚠️ **Warning:** IP, 비밀번호 등 중요한 정보가 어느 브랜치든 실수로 업로드 되면 레포지토리를 폭파시키고 다시 만들어야 하니 반드시 공유해주세요. 리포지토리를 폭파시키고 다시 만들면 `collaborator` 초대 작업만 다시 하면 됩니다.

## 작성 위치
`/_posts` 디렉션 아래 파일 블로그 글을 위치시킵니다. 만약, 이미지가 있을 경우 `/images/{아이디}`아래에 위치시키고 사용합니다.

## 활용 문법 및 파일 이름 구성
블로그 글은 마크다운 문법을 따르며 확장자는 `.markdown` 으로 지정합니다. 블로그 글 파일 예시는 `2024-10-18-guide-for-blog-write.markdown` 과 같습니다.

### 마크 다운 문법 사용 예시 
olvimama 블로그(한글)
  
[https://olvimama.github.io/post/](https://olvimama.github.io/post/) # 웹사이트<br>
[https://github.com/olvimama/olvimama.github.io/tree/master/_posts](https://github.com/olvimama/olvimama.github.io/tree/master/_posts) # 마크다운 코드

so-simple-theme 블로그(영문) 

[https://mmistakes.github.io/so-simple-theme/posts/](https://mmistakes.github.io/so-simple-theme/posts/) # 웹사이트<br>
[https://github.com/mmistakes/so-simple-theme/tree/master/docs/_posts](https://github.com/mmistakes/so-simple-theme/tree/master/docs/_posts) # 마크다운 코드

## 문체
본 블로그는 팀원간 업무 공유가 우선이기 때문에 문체를 대중이 아닌 어색하게 느껴져도 팀원에게 이야기하는 어투를 사용하길 권장합니다. 

## layout 작성법
### 개요 및 작성 예시
작성한 그대로 보여지진 않지만 타이틀, 날짜, 저자 등 전면 정보를 테마에 적용된 룰을 준수하여 웹사이트에 표시 됩니다. 반드시 첫줄부터 작성해야 하며, 본 포스팅에 사용된 예시는 아래와 같습니다.

```md
---
layout: post
title:  더아이엠씨 AI 블로그 작성 간단 설명
date:   2024-10-18
author: Jin
excerpt_separator: "<!--more-->"
tags:
    - guide
    - markdown
    - rule
---
```

### 필수 레이아웃 기능 설명
#### layout, title, date
`layout`은 `post` 로 고정하여 작성된 문서가 `post` 룰을 따른다는 것을 반드시 알려주고, 문서에 대한 제목과 작성 날짜를 `title`과 `date` 항목에 아래와 같이 작성합니다. 

```md
---
layout: post
title: 더아이엠씨 AI 블로그 작성 간단 설명
date: 2024-10-18
----
```

#### author
본 테마는`_data/authors.yml` 에 작성된 작성자 정보를 불러와서 작성된 문서에 표시하는 기능을 제공합니다. 이를 위해 아래와 같이 작성합니다.

```md
---
author: Jin
---
```

만약, 신입사원이 입사하거나 기존 직원의 상태가 변경된다면 `_data/authors.yml` 에 신입사원 정보를 추가하거나, 수정 합니다. 예시 및 항목별 설명은 아래와 같습니다.
```
Jin: # key value
  name: Jin # 이름 및 별명
  picture: /images/jin/jin_profile.jpg # 프로필 사진
  position: R&M teamleader # 직무, 직책, 업무 등 자신이 원하는 설명 내용 작성
  description: 다양한 산업에 AI 모델을 이식하기 위한 테스트 기획 및 수행을 담당하고 있습니다. # 담당업무에 대한 간략한 설명 (optional)

```

#### excerpt_separator
목차에 설명글로 표시되는 부분을 임으로 설정하는 기능을 제공하며, `"<!--more-->"` 등 구분자를 사용하면 구분자가 있는 위치까지만 목차에 표출 됩니다. 사용예시는 아래와 같습니다. 목차에서 설명하는 글은 3줄 넘게 작성해서 목차의 가시성을 높이는데 도움을 주었으면 합니다.

```md
---
excerpt_separator: "<!--more-->"
---
```

#### tag
글의 키워드는 tag를 이용하여 작성합니다. layout에 tag를 이용하여 작성하는 예시는 아래와 같습니다.

```md
tags:
    - guide
    - markdown
    - rule
```

## 디자인 참조
### 디자인 참조 링크
아래 링크는 아마존 기술 블로그이며 아래 블로그와 같은 디자인을 지향합니다.

> [AWD 기술 블로그](https://aws.amazon.com/ko/blogs/tech/)

### 주안점
한글 및 ppt를 이용하여 제안서 및 보고서를 쓰는 것에 익숙해서 디자인 혼선이 오는 부분을 잡고자 작성하였으니 아래 내용을 참고해 주세요.
- 각 글마다 반드시 구분자가 있어야 하는 것은 아닙니다.
- 표, 그림을 넣을 때도 반드시 캡션이 들어가야 하는 것은 아닙니다. 만약, 캡션을 넣는다면 표는 왼쪽 상단, 그림은 아래 가운데로 통일해 주세요.
- 표도 ppt, 한글, 워드 등에서 작성하고 캡처하여 이미지로 넣는 것도 가능합니다.
- 목차에 노출되는 설명글은 GPT를 이용하여 요약문을 작성하는 것을 추천하며, 생성 후 내용 확인만 진행해 주세요.
