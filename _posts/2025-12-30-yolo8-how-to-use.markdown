---
layout: post
title:  vision AI 기술 적용을 위한 ultralytics yolov 활용법
date:   2024-12-30
author: Bjy
excerpt_separator: "<!--more-->"
tags:
    - YOLOv11
    - vision ai
    - how to use
---

딥러닝을 활용한 객체 감지, 세그멘테이션, 분류, 포즈(키포인트) 추출, 오리엔티드 바운딩 박스(방향을 고려한 감지) 등의 연구가 활발히 이루어지고 있습니다. 그중 YOLOv8, 11은 YOLO(You Only Look Once) 모델을 개발 및 유지보수하는 AI 연구 기업인 Ultralytics가 2023, 2024년에 개발 및 배포한 모델입니다. Ultralytics는 2020년에 PyTorch 기반 YOLOv5를 배포했으며, YOLOv8은 그 후속 버전으로서 높은 사용성을 인정받아 다양한 분야에서 활용되고 있고, 2024년 발표한 YOLOv11은 더욱 계선된 정확도와 효율성을 보여줍니다. . 본 포스팅에서는 YOLOv11을 신속하게 사용할 수 있는 방법을 설명합니다.
<!--more-->

# 환경 구축
YOLOv11 작업을 위한 아나코다 가상 환경 등을 만든 후 터미널에서 아래 명령어를 통해 ultralytics 패키지를 설치 합니다.

```bash
pip install ultralytics
```

# 데이터 준비
## 디렉션 구성
`datasets` 디렉션을 생성한 후, 하위 디렉션으로 images, labels을 각각 생성합니다.
```bash
$tree
datasets
 └──images
     └──example_01.jpg
     └──example_02.jpg
 └──labels
     └──example_01.txt
     └──example_02.txt
```

[주의] 학습데이터의 이미지명과 라벨명은 반드시 동일해야 합니다. (확장자 제외)

## train/val 분할
train/val 데이터 분할 방법은 `txt파일 분할`, `폴더분할`  2가 있으나, 주로 txt 파일 분할 방식을 사용합니다. 
```bash
$tree
datasets
 └──images
     └──example_01.jpg
     └──example_02.jpg
     └──example_03.jpg
     └──example_04.jpg
     └──example_05.jpg
     └──example_06.jpg
 └──labels
     └──example_01.txt
     └──example_02.txt
     └──example_03.txt
     └──example_04.txt
     └──example_05.txt
     └──example_06.txt
 └──train.txt
 └──val.txt
```
`txt파일 분할` 방식은 위 코드와 같이 `train.txt`와 `val.txt` 파일을 생성하여 분할해줄 수 있으며, `train.txt`와 `val.txt` 파일에는 아래와 같이 해당 이미지의 경로를 작성합니다. 단, 경로는 절대 경로를 입력합니다. 

[참고] `train.txt`와 `val.txt`파일은 활용의 편의를 위해 `datasets`아래 위치하였을 뿐 반드시 위치해야 하는 것은 아닙니다.

```bash
$ cat dataset/train.txt
/home/{user}/{project_name}/datasets/images/example_01.jpg
/home/{user}/{project_name}/datasets/images/example_02.jpg
/home/{user}/{project_name}/datasets/images/example_03.jpg
/home/{user}/{project_name}datasets/images/example_04.jpg
```

```bash
$ cat dataset/val.txt 
/home/{user}/{project_name}datasets/images/example_05.jpg
/home/{user}/{project_name}datasets/images/example_06.jpg
```

## 학습 데이터 환경설정(data.yaml)
데이터셋 정보는 `data.yaml` 파일을 생성하고, 데이터셋의 경로, 클래스 수, 클래스 이름 등으로 정의하며, 아래 예시를 참고하여 데이터 경로와 class 정보를 입력합니다. 아래 예시는 이미지 감지 모델 학습 예시 입니다. 감지, 세그멘테이션, 포즈(키포인트) 추출의 예시는 아래와 같습니다.

- 세그멘테이션

```bash
$cat data.yaml
train: /home/{user}/{project_name}datasets/train.txt
val: /home/{user}/{project_name}datasets/val.txt
test: #optional

nc: 2
names: ['class1', 'class2']
```

- 포즈(키포인트) 추출 [참조문서](https://github.com/ultralytics/ultralytics/blob/main/ultralytics/cfg/datasets/coco8-pose.yaml)

```bash
$cat data.yaml
train: /home/{user}/{project_name}datasets/train.txt
val: /home/{user}/{project_name}datasets/val.txt
test: #optional

kpt_shape: [10, 3]  # number of keypoints, number of dims (2 for x,y or 3 for x,y,visible)
flip_idx: [0,1,2,3,4,5,6,7,8,9]
names:
    0:st_leaf
```

## 사용법
ultralytics의 YOLO를 사용하기 위해서 CLI(Command-Line Interface), 파이썬 혹은 tralytics웹사이트를 통해 활용할 수 있습니다. 본 문서에서는 CLI 활용법을 다루며, `yolo` 명령어 뒤 에 `TASK`, `MODE`, `ARGS` 순으로 입력하여 사용합니다.
```bash
yolo {TASK} {MODE} {ARGS}
```

위 규칙에 맞춰 학습을 위해 사용하는 명령어 예시는 아래와 같으며, 세그멘테이션 작업을 YOLO11 Small segmentation 모델을 사용하여 수행하데 data.yaml의 데이터 정보를 참고하며, 베치는 1로 하는 모델을 학습하는 명령어입니다.
```bash
yolo segment train model=YOLO11s-seg data=data.yaml batch=1
```

위 규칙에 맞춰 추론하는 명령어는 아래와 같습니다.
```bash
yolo detect predict model=/path/to/trained_weights.pt source=/path/to/images
```

각 옵션에 대한 설명은 아래 표와 같습니다.
|구분|목록|
|---|---|
|TASK|detect, segment, classify, pose, obb 등|
|MODE|train, val, pedict, export, track, benchmark 등|
|ARGS|batch, imgsz, epochs 등 |

MODE 옵션에 대한 yolo 모델에 대한 상세 종류는 [ultralytics 깃허브 레파지토리](https://github.com/ultralytics/ultralytics/tree/main?tab=readme-ov-file#models) 에서 확인 이가능하며, ARGS에 대한 상세 종류 또한 [ultralytics 문서](https://github.com/ultralytics/ultralytics/blob/main/ultralytics/cfg/default.yaml)를 참고 하여 확인할 수 있습니다.

[참고] model 옆에 yolo 모델 명이 아닌 사전 학습된 모델의 위치를 입력하면 재학습이 가능합니다.
