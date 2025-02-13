---
layout: post
title:  AI 모델 배포를 위한 fastAPI 기본 작성 규정
date:   2024-08-01
author: Jin
excerpt_separator: "<!--more-->"
tags:
    - python
    - fastapi
    - model inference
---

완성된 사내 AI 모듈을 테스트 용으로 배포할 때 여러 방식(코드 전달 등) 및 여러 프레임워크 등 무질서하게 사용되고 있는 실정에서, 별도 요청이 없다면 파이썬 fastapi 프레임워크를 사용하는 것을 고정하였습니다. 이에 따라, `http 프로토콜 메서드`, `요청형태`, `미들웨어`, `도큐멘팅` 4가지 요소에서 대한 최소한의 사내 표준을 아래와 같이 설정합니다.

<!--more-->

## http 프로토콜 메서드
http 프로토콜 메서드는 웹 브라우저와 웹서버 간의 통신을 위해 표준화된 규약입니다. 대표적인 규약인 `GET`은 URL에 데이터가 노출되고, `POST`는 노출되지 않는 것이 주요 특징이며, 요청문이 길어지면 `POST`를 쓰는 경향이 있습니다. 

fastapi를 통해 완성된 API는 주로 파이썬으로 테스트 되기도 하고, 굳이 URL에 데이터를 노출시킬 필요가 없으며, 텍스트 입력 시 최소 한 문장에서 최대 여러 문장의 입력값이 넘어가 긴 편에 속하기 때문에 별다른 요청이 없다면 `POST` 방식으로 코드를 구성합니다.

## 요청 형태

터미널에서 API를 요청하는 방식인 curl을 활용할 때, 매개변수를 작성하는 대표적인 포멧은 x-www-form-urlencoded(이하 폼)와 json이 있고, 그 예시는 아래와 같습니다.

- 폼

```bash
curl -X POST http://0.0.0.0.:8080/item_post =H 'Content-Type: application/x-www-form-urlencoded' -d "name=string&description=example&price=0"
```

- json

```bash
curl -X POST http://0.0.0.0.:8080/item_post =H 'Content-Type: application/json' -d '{"name":"string", "description":"example", "price":0}'
```

폼 형식은 제안된 지 오래되고 익숙하게 사용되는 포멧이며, curl 기능을 수행하는 파이썬 request 패키지에서 폼 형식을 파이썬 dict 타입으로 받는 특징을 가집니다. 이에 따라, 별다른 요청이 없다면 폼 형식을 받을 수 있게 fastapi의 Form 메서드를 활용하여 코드를 작성합니다. Form 메서드를 활용항 코드 예시는 아래와 같습니다.

```python
from fastapi import FastAPI, Form

@app.post("/items_post")
async def items_post(
    name: str = Form(...), 
    description: str = Form(...), 
    price: float = Form(...)
    ):

    return {
        "name": name,
        "description": description,
        "price": price
        }
```

[주의] 최근 JSON 방식에 대한 요청도 빈번히 일어나고 있으며 이에 따라,  pydantic패키지에 BaseModel 메서드를 이용하여 json 포멧으로 요청을 할 수 있게 코드를 구성할 경우, request 메서드를 통해 요청을 보낸다면 json.dumps를 통해 dict 타입을 str타입으로 변경하여 요청해야 합니다.

## 미들웨어
### 1. 로깅
utils 디렉션 [부록](#부록)과 같이 logutils.py 파일을 작성한 후 로거를 불러와 사용합니다.

### 2. CORS처리 (Cross-Origin Resource Sharing)
접근 가능한 도메인, 프로토콜 메서드, 헤더를 정의하고 쿠키 및 인증 접근 허용하는 것을 설정하는 기능을 말합니다. 다른 요청이 없다면, 주로, 모든 도메인 허용, GET, POST 메서드 허용, 모든 헤더 허용, 자격증명 허용으로 설정하지만, 고객사와 커뮤니케이션 할때는 반드시 위 내용을 확인해 주세요. fastapi에서 CORS 처리하는 코드 예시는 아래와 같습니다.

```python
from fastapi.middleware.cors import CORSMiddleware

app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],          # 모든 도메인에서 이 API로의 액세스 허용
    allow_credentials=True,       # 자격 증명(예: 쿠키, HTTP 인증)을 허용
    allow_methods=["GET", "POST"],# 모든 HTTP 메서드 (GET, POST, PUT, DELETE 등)를 허용
    allow_headers=["authorization"],          # 모든 헤더를 허용
)

```

### 3. 인증
인증은 여러 방식이 있지만, fastAPI를 활용한 IP 허용만 설명합니다. 다른 방식으로 요청이 오면 개발팀과 논의하고 가능한 수준에서 작성합니다. ip 허용을 위한 미들웨어는 BaseHTTPMiddleware를 상속받아 아래와 같이 객체를 만들어 사용할 수 있습니다.
```python
from starlette.middleware.base import BaseHTTPMiddleware

ALLOWED_IPS = {"0.0.0.0", "0.0.0.1"}  # 허용할 IP 주소를 여기에 추가
# IP 인증을위한 모듈
class IPFilterMiddleware(BaseHTTPMiddleware):
    async def dispatch(self, request: Request, call_next):
        client_ip = request.client.host
        if client_ip not in ALLOWED_IPS:
            raise HTTPException(status_code=403, detail="Access forbidden")

        response = await call_next(request)
        return response

app.add_middleware(IPFilterMiddleware)
```

### 4. 도큐멘팅
fastAPI는 아래 그림과 같이 “/redoc” url을 활용하면 내가 만든 API를  설명할 수 있는 웹페이지 생성 기능을 제공하고 있으며, 저희 팀은 이를 활용하고자 합니다. 본 기능은 fastapi에서 제공하는 객체선언기능과 함수별 도큐멘팅 기능을 활용하여 완성할 수 있으며, 마크다운 문법을 따릅니다.
![fastapi_redoc_sample](/images/jin/fastapi_redoc_sample.jpg)

#### 4.1 객체선언기능
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

#### 함수별 도큐멘팅 기능
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

## 부록

logutils.py

```python
import os
from tqdm import tqdm
import shutil
import warnings
warnings.filterwarnings('ignore')


def get_logger(get, root_path, log_file_name, time_handler=False, console_display=False, logging_level='info', schedule=None):
    '''
    로거 함수

    parameters
    ----------
    get: str
        log 생성용 이름.

    log_file_name: str
        logger 파일을 생성할 때 적용할 파일 이름 + path.

    time_handler: bool (default: False)
        자정(00:00) 을 넘긴 경우 그때까지 쌓인 기록을 이전 날짜 기록으로 뺄지 여부
    
    console_display: bool (default: False)
        로그 기록값을 콘솔에 표시할것인지 여부
    
    logging_level: str
        logger 를 표시할 수준. (notset < debug < info < warning < error < critical)
    
    returns
    -------
    logger: logger
        로거를 적용할 수 있는 로거 변수
    '''
    import logging
    from logging import handlers
    if schedule:
        log_path = f'{root_path}/log_schedule'
    else:
        log_path = f'{root_path}/log_direct'
    os.makedirs(log_path, exist_ok=True)

    logger = logging.getLogger(get)
    if logging_level == 'critical':
        logger.setLevel(logging.CRITICAL)
    if logging_level == 'error':
        logger.setLevel(logging.ERROR)
    if logging_level == 'warning':
        logger.setLevel(logging.WARNING)
    if logging_level == 'info':
        logger.setLevel(logging.INFO)
    if logging_level == 'debug':
        logger.setLevel(logging.DEBUG)
    if logging_level == 'notset':
        logger.setLevel(logging.NOTSET)
    
    # formatter = logging.Formatter('%(asctime)s - %(name)s - %(levelname)s - %(message)s')
    formatter = logging.Formatter('%(asctime)s level:%(levelname)s %(filename)s line %(lineno)d %(message)s')
    if console_display:
        stream_handler = logging.StreamHandler()
        stream_handler.setFormatter(formatter)
        logger.addHandler(stream_handler)

    if time_handler:
        file_handler = handlers.TimedRotatingFileHandler(
            filename=f'{log_path}/{log_file_name}',
            when="midnight",
            interval=1,
            backupCount=30,
            encoding="utf-8")
        file_handler.suffix = '%Y%m%d'
    else:
        file_handler = logging.FileHandler(f'{log_path}/{log_file_name}')

    file_handler.setFormatter(formatter)
    logger.addHandler(file_handler)
    return logger


def log_sort(root_path, schedule):
    if schedule:
        log_path = f'{root_path}/log_schedule'
    else:
        log_path = f'{root_path}/log_direct'
    os.makedirs(f'{log_path}_history', exist_ok=True)
    log_file_list = os.listdir(log_path)
    log_file_list.sort()

    log_dict = {}
    for log_file in log_file_list:
        log_name = log_file.split('.')[0]
        date = log_file.split('.')[-1]
        if date == 'log':
            continue
        try:
            log_dict[log_name].append(log_file)
        except KeyError:
            log_dict[log_name] = [log_file]

    
    for log_name, log_list in log_dict.items():
        for log_file in tqdm(log_list):
            date = log_file.split('.')[-1]
            yyyy = date[:4]
            mm = date[4:6]
            # dd = date[6:]
            move_path = f'{log_path}_history/{yyyy}/{mm}/{log_name}'
            os.makedirs(move_path, exist_ok=True)
            shutil.move(
                f'{log_path}/{log_file}',
                f'{move_path}/{log_file}'
            )


if __name__ == "__main__":
    root_path = ''
    log_sort(root_path)

```
