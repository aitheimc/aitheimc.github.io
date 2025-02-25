---
layout: post
title:  파이썬 logger 제작
date:   2024-08-02
author: Jin
excerpt_separator: "<!--more-->"
tags:
    - python
    - logger
    - model inference
---

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
