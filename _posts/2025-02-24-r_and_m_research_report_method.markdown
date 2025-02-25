---
layout: post
title:  리서치 리포팅을 위한 환경 구성 소개
date:   2025-02-25
author: Jin
excerpt_separator: "<!--more-->"
tags:
    - github
    - huggingface
    - fastapi
---

본 포스팅은 회사에서 리서치한 내용을 소개하는 방식과 방법에 대해 정리 해둔 포스팅입니다. 리서치 리포팅은 `기술 블로그`. `API 서비스`, `챗봇 서비스` 3 단계로 구성됩니다. 각 단계를 위해 사용된 서비스는 `Github 블로그`, `fastAPI`, `huggingface`입니다. 본 포스팅은 각 단계에 대한 환경 구성 방법과 컨텐츠 업로드 방법에 대해 설명하고 있습니다.

<!--more-->

## 개요
리서치 리포팅은 3단계로 나누어 진행합니다. 첫번째, 일의 중요도 및 소요시간과 관계없이 동료와 공유하고 싶은 세미나 자료부터 오랜시간 공들인 AI 모델, 학습 및 평가 데이터 소개까지 모두 업로드 할 수 있는 `기슬 블로그` 두번째, 고객사 납품, 내부 서비스 지원, 레퍼런스 확보 등을 위해 개발된 AI 모델을 권한에 따라 사용해 볼 수 있는 `API 서비스`, 세번째, 자체개발된 AI 모델을 누구나 사용가능할 수 있게 하는 `챗봇 서비스` 입니다.

> ⚠️ **Warning:** 리서치 리포팅을 위해 사용하는 서비스는 언제나 장단점이 존재할 뿐만 아니라 장단점이 시점에 따라 달리집니다. 더 좋은 방법 더 나은 방식이 있더라도 우선 정해진 방식을 먼저 준수하고 3번이상 사용해본 후 다른 방법을 팀장님 등에게 제안해 주세요.

## 내용
자세한 설명이 필요한 부분은 아래 표 링크를 통해 확인할 수 있습니다.
|단계|환경구성 방법|컨텐츠 업로드 방식|
|---|---|---|
|기술블로그|[깃허브 블로그](https://github.com/aitheimc/aitheimc.github.io/blob/main/README.md)|[마크다운 ruled  by So Simple](https://aitheimc.github.io/guide-for-blog-write/)|
|API 서비스|파이썬|[fastapi 활용](https://aitheimc.github.io/guide-for-fastapi/)|
|챗봇 서비스|[허깅페이스 스페이스](https://huggingface.co/docs/hub/spaces-overview)<br>[스트림릿커스텀](https://aitheimc.github.io/streamlit-custom-chatbotUI/)|API 서비스와 연동 등등|
