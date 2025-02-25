---
layout: post
title:  스트림릿(Streamlit)에서 제공하는 기본 챗봇 디자인 변경하기
date:   2025-02-19
author: Jin
excerpt_separator: "<!--more-->"
tags:
    - streamlit
    - huggingface space
    - chatbot
---

스트림릿은 파이썬 코드만으로 손쉽게 대화형 웹 애플리케이션을 개발할 수 있도록 도와주는 오픈소스 프레임워크입니다. 모델러가 자신의 AI 모델 애플리케이션을 배포하는 플랫폼인 허깅페이스 스페이스에서도 기본 구조로 제공할 만큼 모델러들 사이에서 많이 사용되는 도구입니다. 하지만 기본적으로 제공되는 프레임워크의 디자인이 정형화되어 있어 사용자에게 다소 지루하게 느껴질 수 있습니다. 이에 따라, 본 포스팅에서는 스트림릿(Streamlit) 패키지를 활용하여 기본 챗봇 웹페이지를 커스터마이징하는 방법을 다루고자 합니다.

<!--more-->

# 코드 구성
스트림릿에서 제공하는 기본 챗봇 디자인을 커스텀하기 위해 작성한 아래 코드는 `패키지`, `글로벌`, css 스타일과 html 틀을 스트림릿에 적용시키기 위한 `Front` 그리고 과거 챗봇 내용을 기억하고 인증과정과 챗봇 내용을 생성하는 `Back` 크게 4가지로 구성됩니다. 

# 패키지
챗봇 구동을 위한 패키지는 아래 코드와 갔습니다.

```python
# 패키지 불러오기
import re
import streamlit as st
from transformers import pipeline
```

# 글로벌 변수
챗봇을 구동하기 위해 필요한 언어 모델, 챗봇에 프로필 이미지 경로, 챗봇 정보 저장과 인증 정보를 기억하기 위한 있는 세션 객체를 선언하는 영역으로 코드는 아래와 같습니다.

```python
# 글로벌 변수 선언
## 본 코드에서 사용된 신문기사 요약용 모델(https://huggingface.co/gangyeolkim/kobart-korean-summarizer-v2)은 테스트용 입니다.
pipe = pipeline("{purpose}", model="{model/path}")

## assitant 아바타 이미지 경로
assistant_icon_path = "icon/to/path"

## 세션 상태에서 approval_state 및 messages 초기화
if "approval_state" not in st.session_state:
    st.session_state.approval_state = "require"
if "messages" not in st.session_state:
    st.session_state.messages = []
```

- [참고] 세션이란?
```
사용자의 인터랙션을 관리하는 기능을 말합니다. Streamlit은 기본적으로 사용자가 앱과 상호작용할 때마다 전체 코드가 다시 실행되는 구조인데, 세션 상태(Session State)를 활용하면 데이터를 유지하면서도 효율적인 앱을 만들 수 있습니다.
```

# Front
스트림릿 챗봇 커스텀을 위해 html과 css 코드를 작성한 부분입니다.

## CSS
챗봇 디자인을 커스텀하는 코드 영역입니다. 사용자의 질문 표시 위치, 폰트, 사이즈 등과 챗봇의 프로필, 응답 표시 위치, 폰트, 사이즈, 응답 말풍선등이 주로 수정되었습니다. 스트림릿 객체 st.markdown는 텍스트를 마크다운 형식으로 랜더링 하는 함수이지만, unsafe_allow_html=True 옵션을 사용하면 HTML/CSS를 직접 적용할 수 있습니다. css 코드는 아래와 같으며, 요소 별 기능은 주석으로 설명해 두었습니다.

```css
st.markdown("""
    <style>
    .chat-container { /* 채팅 메시지를 감싸는 컨테이너를 설명합니다. */
        display: flex; /* 채팅 요소(아바타, 말풍선 등)을 가로 방향으로 정렬합니다. */
        margin-bottom: 10px; /*메세지간 간격을 10px로 합니다. */
    }
    .user-container {
        justify-content: flex-end; /* 사용자의 메시지를 우측정렬합니다.*/
    }
    .assistant-container {
        justify-content: flex-start; /* 어시스턴스 메시지를 좌측 정렬합니다.*/
        flex-direction: row; /* Avatar와 Bubble을 수평으로 정렬합니다. */
    }
    .chat-bubble { /* 말풍선 스타일을 정의합니다. */
        padding: 10px; /* 말풍선 안에 메시지간 간격을 10px로 합니다. */
        border-radius: 10px; /*말풍선의 모서리를 동글게 만듭니다.*/
        max-width: 70%; /* 한줄이 너무 길지 않게 제한합니다. */
        word-wrap: break-word; /* 텍스트를 단어 단위로 줄내림합니다. */
        white-space: pre-wrap; /* 텍스트를 단어 단위로 줄내림합니다. */
    }
    .user-bubble { /* 사용자 메시지 스타일을 정의합니다. */
        font-size: 14px; /* 폰트 사이즈를 지정합니다. */
        color: #333;  /* 폰트 색상을 지정합니다. */
        background-color: #e5e5e5; /* 말풍선 배경색을 지정합니다. */
    }
    .assistant-bubble {
        font-size: 14px; /* 폰트 사이즈를 지정합니다. */
        color: #333;  /* 폰트 색상을 지정합니다. */
        text-align: left; /* 메세지를 왼쪽 정렬 합니다. */
        background-color: #feeddf; /* 말풍선 배경색을 지정합니다. */
    }
    .avatar { /* 프로필 이미지(아바타) 스타일을 지정합니다. */
        width: 30px; /* 아바타 폭을 지정합니다. */
        height: 30px; /* 아바타 너을 지정합니다. */
        border-radius: 50%; /* 아바타를 원경으로 만듭니다. */
        margin-right: 10px; /*메세지간 간격을 10px로 합니다. */
    }
    </style>
""", unsafe_allow_html=True)
```

## 메세지 wrapper
html 테그를 이용하여 채팅 매세지마다 css 스타일을 적용하기 위해 작성된 함수입니다.

```html
def user_message_style(question):
    return f"""<div class="chat-container user-container">
        <div class="chat-bubble user-bubble">{question}</div>
    </div>"""

def assistant_message_style(assistant_icon_path, answer):
    return f"""<div class="chat-container assistant-container">
        <img src="{assistant_icon_path}" class="avatar">
        <div class="chat-bubble assistant-bubble">{answer}</div>
    </div>"""
```
# BACK
## 과거 채팅 내용 출력
스트림릿의 st.chat_input() 메서드는 사용자가 입력을 주면 세션 내용은 기억하면서 코드를 다시 실행하는 특성이 있습니다. 따라서 세션에 저장된 과거 대화 내용을 화면에 출력하기 위해서 새로 실행된 코드에 과거 내용을 미리 뿌려둘 수 있게 아래와 같은 코드를 작성해 두어야 합니다. 

```python
for message in st.session_state.messages:
    if message["role"] == "user":
        st.markdown(user_message_style(message["content"]), unsafe_allow_html=True)
    else:
        st.markdown(assistant_message_style(assistant_icon_path, message["content"]), unsafe_allow_html=True)
```

## 인증 메세지
본 코드는 다른 챗봇 코드와 달리 인증을 위해 챗봇이 먼저 사용자에게 말을 거는 구조 입니다. 세션내 과거 기록이 없다면 첫 대화로 인지하고 인증을 위한 메세지를 먼저 말하게 구성하기 위해 아래와 같은 코드를 작성합니다.

```python
if not st.session_state.messages:
    greeting = "{인사말말}"
    st.session_state.messages.append({"role": "assistant", "content": greeting})
    st.markdown(assistant_message_style(assistant_icon_path, greeting), unsafe_allow_html=True)
``` 
## 인증 및 응답 생성
아래 코드에서 사용된 `prompt :=`는 "월러스 연산자" (:=, walrus operator)로, 입력값이 있으면 이를 prompt 변수에 할당하면서 동시에 조건문을 실행하는 기능을 합니다. 아래 코드와 같이 윌러스 연산자를 이용하여 챗봇이 구성되었습니다. 각 코드 별 설명은 아래 코드 내 주석을 참고하세요.

```python

if prompt := st.chat_input():
    # 사용자 질문 세션에 저장
    st.session_state.messages.append({"role": "user", "content": prompt})
    
    # 사용자 질문 화면에 표시
    st.markdown(user_message_style(prompt), unsafe_allow_html=True)
    
    # 인증 
    if st.session_state.approval_state == "require":
        if prompt.lower() == {인증 코드}:
            st.session_state.approval_state = "approved"
            response = "{인증 완료 메세지}"
        else:
            response = "{인증 실패 메세지}"
    # 응답 생성
    else:
        result = re.sub(r'\s+', ' ', prompt)
        summarized = pipe(result)
        response = summarized[0]["summary_text"]
    
    # 어시스턴스의 메세지를 세션에 저장
    st.session_state.messages.append({"role": "assistant", "content": response})
    # 어시스턴스의 메세지를 화면에 표시
    st.markdown(assistant_message_style(assistant_icon_path, response), unsafe_allow_html=True)

```

# 커스텀 디자인 화면
위코드를 적용하여 커스텀한 디자인은 아래와 같습니다.
![fastapi_title](/images/jin/streamlit_custom.png)
[참고] 코드 내 내용은 단순예시 입니다.

# [참고] 전체코드
자사 레퍼런스용 챗봇 개발 당시 위 내용을 기반하여 초기 테스트로 사용한 전체 코드는 아래와 같습니다. 

```python
# 패키지 불러오기
import re
import streamlit as st
from transformers import pipeline

# 글로벌 변수 선언
## 모델 불러오기(https://huggingface.co/gangyeolkim/kobart-korean-summarizer-v2) 는 테스트용 모델 입니다.
pipe = pipeline("summarization", model="gangyeolkim/kobart-korean-summarizer-v2")

## assitant 아바타 이미지 경로
assistant_icon_path = "https://huggingface.co/spaces/randmimc/aitom/resolve/main/chat_icon/assistant.ico"

## 세션 상태에서 approval_state 및 messages 초기화
##  ※ 세션(session) : 
##          사용자의 인터랙션을 관리하는 기능입니다. 
##          Streamlit은 기본적으로 사용자가 앱과 상호작용할 때마다 전체 코드가 다시 실행되는 구조인데, 
##          세션 상태(Session State)를 활용하면 데이터를 유지하면서도 효율적인 앱을 만들 수 있습니다.
if "approval_state" not in st.session_state:
    st.session_state.approval_state = "require"
if "messages" not in st.session_state:
    st.session_state.messages = []

# CSS 작성 
## st.markdown는 텍스트를 마크다운 형식으로 랜더링 하는 함수이지만, unsafe_allow_html=True 옵션을 사용하면 HTML/CSS를 직접 적용할 수 있습니다.
## 스트림릿에서 디폴트로 적용하는 디자인이 맘에 들지 않는다면 CSS를 작성하여 변경할 수 있습니다.
st.markdown("""
    <style>
    .chat-container { /* 채팅 메시지를 감싸는 컨테이너를 설명합니다. */
        display: flex; /* 채팅 요소(아바타, 말풍선 등)을 가로 방향으로 정렬합니다. */
        margin-bottom: 10px; /*메세지간 간격을 10px로 합니다. */
    }
    .user-container {
        justify-content: flex-end; /* 사용자의 메시지를 우측정렬합니다.*/
    }
    .assistant-container {
        justify-content: flex-start; /* 어시스턴스 메시지를 좌측 정렬합니다.*/
        flex-direction: row; /* Avatar와 Bubble을 수평으로 정렬합니다. */
    }
    .chat-bubble { /* 말풍선 스타일을 정의합니다. */
        padding: 10px; /* 말풍선 안에 메시지간 간격을 10px로 합니다. */
        border-radius: 10px; /*말풍선의 모서리를 동글게 만듭니다.*/
        max-width: 70%; /* 한줄이 너무 길지 않게 제한합니다. */
        word-wrap: break-word; /* 텍스트를 단어 단위로 줄내림합니다. */
        white-space: pre-wrap; /* 텍스트를 단어 단위로 줄내림합니다. */
    }
    .user-bubble { /* 사용자 메시지 스타일을 정의합니다. */
        font-size: 14px; /* 폰트 사이즈를 지정합니다. */
        color: #333;  /* 폰트 색상을 지정합니다. */
        background-color: #e5e5e5; /* 말풍선 배경색을 지정합니다. */
    }
    .assistant-bubble {
        font-size: 14px; /* 폰트 사이즈를 지정합니다. */
        color: #333;  /* 폰트 색상을 지정합니다. */
        text-align: left; /* 메세지를 왼쪽 정렬 합니다. */
        background-color: #feeddf; /* 말풍선 배경색을 지정합니다. */
    }
    .avatar { /* 프로필 이미지(아바타) 스타일을 지정합니다. */
        width: 30px; /* 아바타 폭을 지정합니다. */
        height: 30px; /* 아바타 너을 지정합니다. */
        border-radius: 50%; /* 아바타를 원경으로 만듭니다. */
        margin-right: 10px; /*메세지간 간격을 10px로 합니다. */
    }
    </style>
""", unsafe_allow_html=True)

# 메세지 wrapper
# html 테그를 이용하여 챗팅 매세지마다 css 스타일을 적용하기 위해 작성된 함수입니다.
def user_message_style(question):
    return f"""<div class="chat-container user-container">
        <div class="chat-bubble user-bubble">{question}</div>
    </div>"""

def assistant_message_style(assistant_icon_path, answer)):
    return f"""<div class="chat-container assistant-container">
        <img src="{assistant_icon_path}" class="avatar">
        <div class="chat-bubble assistant-bubble">{answer}</div>
    </div>"""

# BACK
## 과거 채팅 내용 출력
## 스트림릿의 st.chat_input() 메서드는 사용자가 입력을 주면 세션내용을 기억하면서 코드를 다시 실행하는 특성이 있습니다. 
## 따라서 세션에 저장된 과거 대화 내용을 화면에 출력하기 위해서 새로 실행된 코드에 과거 내용을 미리 뿌려둘 수 잇게 아래와 같은 코드를 작성해 두어야 합니다. 
for message in st.session_state.messages:
    if message["role"] == "user":
        st.markdown(user_message_style(message["content"]), unsafe_allow_html=True)
    else:
        st.markdown(assistant_message_style(assistant_icon_path, message["content"]), unsafe_allow_html=True)

# 인증 메세지
## 본 코드는 다른 챗봇과 달리 인증을 위해 챗봇이 먼저 사용자에게 말을 거는 주고를 가집니다.
## 세션이 아무 메세지도 가지고 있지 않다면 인증을 위한 말을 먼저 걸게 구성합니다.

## 인증 메세지
if not st.session_state.messages:
    greeting = "더아이엠씨에 aitom은 현재 공사중에 있습니다.\n간단한 신문기사를 입력하시면 요약을 제공합니다.\n계속 사용하려면 \"yes\"를 입력해주세요."
    st.session_state.messages.append({"role": "assistant", "content": greeting})
    st.markdown(assistant_message_style(assistant_icon_path, greeting), unsafe_allow_html=True)

## 인증 및 응답 생성
## prompt :=`는 "월러스 연산자" (:=, walrus operator)로, 입력값이 있으면 이를 prompt 변수에 할당하면서 동시에 조건문을 실행하는 기능을 합니다. 
if prompt := st.chat_input():
    # 사용자 질문 세션에 저장
    st.session_state.messages.append({"role": "user", "content": prompt})
    # 사용자 질문 화면에 표시
    st.markdown(user_message_style(prompt), unsafe_allow_html=True)

    # 인증 
    if st.session_state.approval_state == "require":
        if prompt.lower() == "yes":
            st.session_state.approval_state = "approved"
            response = "이제 요약 모델을 사용할 수 있습니다. 신문기사를 입력해주세요."
        else:
            response = "승인되지 않았습니다. 계속하려면 \"yes\"를 입력해주세요."
    # 응답 생성
    else:
        result = re.sub(r'\s+', ' ', prompt)
        summarized = pipe(result)
        response = summarized[0]["summary_text"]
    # 어시스턴스의 메세지를 세션에 저장
    st.session_state.messages.append({"role": "assistant", "content": response})
    # 어시스턴스의 메세지를 화면에 표시
    st.markdown(assistant_message_style(assistant_icon_path, response), unsafe_allow_html=True)

```