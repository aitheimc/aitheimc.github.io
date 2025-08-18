---
layout: post
title:  [교육용] AI 모델 배포를 위한 fastAPI 기본 작성 규정
date:   2025-07-28 수정
author: Jin
excerpt_separator: "<!--more-->"
tags:
    - python
    - fastapi
    - model inference
---

완성된 사내 AI 모듈을 테스트 용도로 배포할 때, 현재는 코드 전달 등 다양한 방식과 여러 프레임워크가 무질서하게 사용되고 있는 실정입니다. 이에 따라 별도 요청이 없는 한, Python FastAPI 프레임워크를 기본으로 사용하기로 하였습니다. 이에 따라, `http 프로토콜 메서드`, `요청형태`, `응답방식`, `도큐멘팅` 4가지 요소에서 대한 최소한의 표준을 아래와 같이 권장합니다. 특히 `요청형태`는 별도의 요청이 없는 경우 반드시 준수해 주시기 바랍니다.

<!--more-->

## http 프로토콜 메서드
http 프로토콜 메서드는 웹 브라우저와 웹서버 간의 통신을 위해 표준화된 규약입니다. 대표적인 규약인 `GET`은 URL에 데이터가 노출되고, `POST`는 노출되지 않는 것이 주요 특징이며, 요청문이 길어지면 `POST`를 쓰는 경향이 있습니다. 

fastapi를 통해 완성된 API는 주로 파이썬으로 테스트 되기도 하고, 굳이 URL에 데이터를 노출시킬 필요가 없으며, 텍스트 입력 시 최소 한 문장에서 최대 여러 문장의 입력값이 넘어가 긴 편에 속하기 때문에 별다른 요청이 없다면 `POST` 방식으로 코드를 구성합니다.

## 요청 형태

터미널에서 API를 요청하는 방식인 curl을 활용할 때, 매개변수를 작성하는 대표적인 포멧은 x-www-form-urlencoded(이하 폼)와 json이 있고, 그 예시는 아래와 같습니다.

- 폼

```bash
curl -X POST http://0.0.0.0.:8080/item_post -H 'Content-Type: application/x-www-form-urlencoded' -d "name=string&description=example&price=0"
```

- json

```bash
curl -X POST http://0.0.0.0.:8080/item_post -H 'Content-Type: application/json' -d '{"name":"string", "description":"example", "price":0}'
```

JSON 형식은 API 통신에서 널리 사용되는 표준 포맷으로, 다양한 언어 및 플랫폼과의 호환성이 뛰어나고 구조화된 데이터 전달에 유리합니다. 또한, curl 기능을 수행하는 Python의 requests 패키지에서도 JSON 형식은 dict 타입으로 간편하게 처리됩니다. 이에 따라, 별도의 요청이 없는 한 JSON 형식으로 데이터를 수신하도록 하고, FastAPI의 Request Body를 활용하여 코드를 작성합니다. JSON 형식을 처리하는 코드 예시는 아래와 같습니다.


```python
from typing import Optional
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()

# 요청용 모델
class Item(BaseModel):
    name: str
    description: str
    price: Optional[float] = 0.7

# 응답용 모델 (필요에 따라 요청 모델과 동일하게 사용 가능)
class ItemResponse(BaseModel):
    name: str
    description: str
    price: float

@app.post("/items_post", response_model=ItemResponse)
async def items_post(item: Item):
    # 여기서 필요한 추가 로직 처리 가능
    return ItemResponse(
        name=item.name,
        description=item.description,
        price=item.price
    )
```

[주의] 최근 JSON 방식에 대한 요청도 빈번히 일어나고 있으며 이에 따라,  pydantic패키지에 BaseModel 메서드를 이용하여 json 포멧으로 요청을 할 수 있게 코드를 구성할 경우, request 메서드를 통해 요청을 보낸다면 json.dumps를 통해 dict 타입을 str타입으로 변경하여 요청해야 합니다.

## 응답방식
FastAPI를 이용한 AI 모델 API는 기본적으로 RESTful 응답을 사용합니다. 즉, 클라이언트가 요청을 보내면 서버가 처리를 완료한 후 JSON 형식으로 결과를 반환합니다. 요청형태 예시 코드를 참조해주세요.

단, LLM(대형 언어 모델) API의 경우, 사용자 입력에 대한 응답이 길고 실시간 생성되는 특징이 있어 RESTful 방식만으로는 효율적이지 않을 수 있습니다. 이때는 SSE(Server-Sent Events) 또는 WebSocket과 같은 스트리밍 응답 방식을 고려할 수 있습니다. 서버 스팩 등 특별한 요청이 없으면 SSE 방식을 고려하고 해당 내용은 아래 링크를 통해 확인할 수 있습니다. 

## 도큐멘팅
fastAPI는 아래 그림과 같이 “/redoc” url을 활용하면 내가 만든 API를  설명할 수 있는 웹페이지 생성 기능을 제공하고 있으며, 저희 팀은 이를 활용하고자 합니다. 본 기능은 fastapi에서 제공하는 객체선언기능과 함수별 도큐멘팅 기능을 활용하여 완성할 수 있으며, 마크다운 문법을 따릅니다.
![fastapi_redoc_sample](/images/jin/fastapi_redoc_sample.jpg)

### 객체선언기능
FastAPI 매서드를 활용하여 app 객체를 선언할 때 `title`, `version`, `description` 옵션을 활용하면 API 전체를 설명할 수 있습니다. 활용 예시 코드는 아래와 같습니다.

```python
from fastapi import FastAPI

app = FastAPI(
        title="hyungjin api",
        version="0.1.0",
        description= """마크다운 문법을 이용항여 필요시 내용을 추가합니다.
        """
        )
```

위 코드를 적용하였을 때 나타나는 웹페이지 일부는 아래 그림과 같습니다.
![fastapi_title](/images/jin/fastapi_title.png){: width="200px"}

### 함수별 도큐멘팅 기능
python 코드 내에서 내가 작성한 함수를 설명하 듯 함수 선언문 아래 멀티라인 문자열(“”“ “”“)을 이용하고 안에 마크다운 문법을 사용하면 redoc 웹사이트 내 정리가 자동으로 완성 됩니다. 예시는 아래와 같습니다.

```python
@app.post("/items_post")
async def items_post(
    name: str = Form(...), 
    description: str = Form(...), 
    price: float = Form(...)
    ):
    """
    ### header2
    - (필요시) 설명해주세요

    ### python code
    ```python
    import ast 
    import requests
    from pprint import pprint

    url = 'http://0.0.0.0:0000/items_post/'
    data = {
        "name": "name", 
        "description" : "example", # 설명
        "price": 0 
        }

    response = requests.post(url, data=data)
    response_dict = ast.literal_eval(response.text) # str to dict
    pprint(response_dict)
    ```

    ### output
    ```
    {'description': 'example', 'name': 'string', 'price': 0.0}
    ```
    """
```

위 코드를 적용하였을 때 나타나는 웹페이지 일부는 아래 그림과 같습니다.
![fastapi_def_explain](/images/jin/fastapi_def_explain.png)
