---
layout: post
title:  python 커스텀 로거 예시
date:   2024-08-02
author: Jin
excerpt_separator: "<!--more-->"
tags:
    - python
    - logger
    - model inference
---
Python에서 로그를 관리하는 것은 시스템을 모니터링하고 오류를 추적하는 데 필수적입니다. 특히, 프로젝트가 커지거나 여러 환경에서 실행될 때, 로그를 어떻게 설정하느냐에 따라 디버깅과 유지 보수가 훨씬 더 쉬워질 수 있습니다. 이번 글에서는 Python 로깅 기능을 활용하여 로그 파일을 효과적으로 관리하고, 콘솔 출력과 시간별 로그 분리 기능을 설정하는 방법을 소개하려고 합니다.
<!--more-->

아래 코드에서는 logging 모듈과 TimedRotatingFileHandler를 활용하여 날짜별로 로그 파일을 분리하고, 콘솔에도 실시간으로 로그를 출력할 수 있는 로거를 생성하는 방법을 설명합니다. 이를 통해, 시스템이 생성하는 로그가 잘 분리되고 관리될 수 있으며, 개발 중 실시간으로 로그를 모니터링할 수 있습니다.

```python
import os
import shutil
from tqdm import tqdm
import logging
from logging import handlers

def get_logger(get: str, root_path: str, log_file_name: str, time_handler: bool = False, console_display: bool = False, logging_level: str = 'info') -> logging.Logger:
    """
    로거 생성 함수
    
    Args:
        get (str): 로거의 이름.
        root_path (str): 로그 파일이 저장될 디렉터리 경로.
        log_file_name (str): 로그 파일 이름.
        time_handler (bool, optional): 날짜별 로그 파일 분리 여부. 기본값은 False.
        console_display (bool, optional): 로그를 콘솔에도 출력할지 여부. 기본값은 False.
        logging_level (str, optional): 로깅 수준 설정 ('notset', 'debug', 'info', 'warning', 'error', 'critical'). 기본값은 'info'.
    
    Returns:
        logging.Logger: 설정된 로거 객체.
    """
    log_path = os.path.join(root_path, 'logs')  # 로그 저장 경로 설정
    os.makedirs(log_path, exist_ok=True)  # 디렉터리 생성
    
    logger = logging.getLogger(get)  # 로거 생성
    logger.setLevel(getattr(logging, logging_level.upper(), logging.INFO))  # 로깅 레벨 설정
    
    formatter = logging.Formatter('%(asctime)s level:%(levelname)s %(filename)s line %(lineno)d %(message)s')
    
    if console_display:  # 콘솔 출력 여부 확인
        stream_handler = logging.StreamHandler()
        stream_handler.setFormatter(formatter)
        logger.addHandler(stream_handler)
    
    log_file_path = os.path.join(log_path, log_file_name)
    if time_handler:
        file_handler = handlers.TimedRotatingFileHandler(
            filename=log_file_path, when="midnight", interval=1, backupCount=30, encoding="utf-8"
        )
        file_handler.suffix = '%Y%m%d'
    else:
        file_handler = logging.FileHandler(log_file_path)
    
    file_handler.setFormatter(formatter)
    logger.addHandler(file_handler)
    
    return logger
```
