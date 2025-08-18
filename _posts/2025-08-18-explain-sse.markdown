---
layout: post
title:  LLM API에서 한 글자씩 출력되는 실시간 응답 구현을 위한 SSE(Server-Sent Events) 가이드
date:   2025-08-18
author: Jin
excerpt_separator: "<!--more-->"
tags:
    - python
    - fastapi
    - sse
    - llm
---
LLM 기반 API를 개발 할때, 응답을 실시간으로 한 글자씩 출력하는 기능을 적용할 경우 사용자 경험이 크게 개선됩니다. 사용자는 긴 응답을 기다릴 필요없이 즉시 결과를 확인할 수 있습니다. 이를 위해, 유용하게 사용할 수 있는 기술이 SSE입니다. 이번 글에서는 ~~~~
<!--more-->

## SSE란 무엇인가?
SSE는 서버에서 클라이언트로 단방향 실시간 데이터 스트리밍을 지원하는 웹 표준 기술입니다. 서버 개발 자가 API를 개발 할때 SSE 표준을 준수해서 개발해두면, 브라우저와 클라이언트에서는 내장된 EventSource객체로 손쉽게 SSE데이터를 수신할 수 있습니다.

## SSE 표준 형식
SSE 메시지는 텍스트 블록으로 구성되며, 각 이벤트는 **두 줄 개행(\n\n)**으로 구분하는 것이 필수 입니다. 주로 `event`와 `data` 타입으로 구성되며, `data`는 필수 구성 요소 입니다. 스트리밍 결과가 아래 예시 처럼 표출 될 수 있게 코드를 구성합니다.

```bash
event: message
data: {"text": "H"}

event: message
data: {"text": "e"}

event: message
data: {"text": "l"}

event: message
data: {"text": "l"}

event: message
data: {"text": "o"}

event: info
data: {"progress": 20}

event: warning
data: {"msg": "GPU 90%"}
```

## 브라우저/ 클라이언트 SSE 활용 예시
앞에서 설명한 SSE 표준에 준수하여 코드를 작성하였을 경우, 클라이언트는 `javascript`코드를 아래와 같이 작성함으로써 서비스에 스트리밍 결과를 표시 합니다.
```javascript
const source = new EventSource("/v1/llm/stream");

source.addEventListener("message", (e) => {
    const data = JSON.parse(e.data);
    process.stdout.write(data.text); // 한 글자씩 출력
});

source.addEventListener("info", (e) => {
    console.log("진행률:", e.data);
});

source.addEventListener("warning", (e) => {
    console.warn("경고:", e.data);
});
```
## SSE와 WebScoket비교
LLM API나 실시간 애플리케이션에서 비동기 방식으로 스트리밍 응답을 구현할 때는 대표적으로 SSE(Server-Sent Events)와 WebSocket 두 가지 방식이 사용됩니다. SSE는 서버에서 클라이언트로 단방향 데이터를 전송하는 데 최적화되어 있어 실시간 알림이나 LLM 응답을 한 글자씩 출력하는 스트리밍에 적합하며, HTTP 기반으로 방화벽이나 프록시 환경에서도 안정적으로 동작합니다. 그러나 클라이언트가 중간에 응답을 멈추거나 다른 내용을 요청하는 등 실시간 피드백이 필요한 경우에는 한계가 있습니다. 반면 WebSocket은 양방향 통신을 지원해 클라이언트가 LLM 응답을 중단하거나 재생성 요청을 보내고, 출력 속도를 조절하는 등 실시간 제어와 상호작용이 가능하므로 더 유연하고 유용합니다. 따라서 비동기/스트리밍 응답을 구현할 때, SSE는 단방향 스트리밍에, WebSocket은 실시간 제어와 양방향 상호작용이 필요한 상황에 적합하다고 할 수 있습니다.

|항목|SSE|WebSocket|
|---|---|---|
|통신 방향|단방향(Server → Client)|양방향|
|프로토콜|HTTP|WS/WSS|
|브라우저 지원|EventSource 내장|WebSocket 내장|

## FastAPI에서 한글자씩 출력되는 SSE 구현하기
FastAPI의 StreamingResponse를 활용하면 LLM 한 글자씩 출력되는 스트리밍을 구현할 수 있으며, **SSE 표준 형식(event:, data:, 두 줄 개행)**을 준수하면 브라우저의 EventSource에서 안정적으로 처리할 수 있습니다. 아래는 SSE 기능이 가능하게 구성한 fastAPI 코드 예시 입니다.
```python
from fastapi import FastAPI
from fastapi.responses import StreamingResponse
import time
import json

app = FastAPI()

# SSE 형식으로 데이터를 한 글자씩 스트리밍
def llm_sse_stream(text: str):
    for idx, char in enumerate(text):
        # SSE 표준: data 필드 + 두 줄 개행
        yield f"id: {idx}\n"
        yield f"event: message\n"
        yield f"data: {json.dumps({'text': char})}\n\n"
        time.sleep(0.1)  # 한 글자씩 전송되는 느낌
    # 완료 이벤트
    yield f"event: done\n"
    yield f"data: {json.dumps({'msg': 'Streaming complete'})}\n\n"

@app.get("/llm-sse")
def stream_llm_sse():
    return StreamingResponse(
        llm_sse_stream("Hello, SSE streaming!"),
        media_type="text/event-stream"  # SSE 표준 MIME 타입
    )

```

이 구조를 따르면 클라이언트에서 다음과 같이 처리할 수 있습니다.
```javascript
const source = new EventSource("/llm-sse");

source.addEventListener("message", (e) => console.log("char:", e.data));
source.addEventListener("done", (e) => console.log("완료:", e.data));
```