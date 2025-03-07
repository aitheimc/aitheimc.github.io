---
layout: post
title: 하늘상태와 음영기복도를 활용한 일조시간 보간도 구축
date: 2025-03-06
author: Jhg
excerpt_separator: "<!--more-->"
tags:
    - 일조시간
    - 음영기복도
    - 공간보간
---


`일조시간`은 농업, 에너지, 관광 등 다양한 분야에서 중요한 기상 요소이지만, 현재 기상 예보 데이터에서는 직접적으로 제공되지 않는 한계가 있다. 특히, 지형적 특성과 구름량에 따라 공간적 변동성이 크지만, 관측소 간격이 넓어 정확한 분포 파악이 어렵다. 본 연구에서는 하늘상태 정보와 지형의 음영기복도를 결합하여 보다 정밀한 `일조시간 보간도`를 구축하고자 한다.

<br>
* 일조시간 : 태양의 직사광이 지표면에 비친 시간
<br>
* 음영기복도 : 지형의 표고에 따른 음영효과를 시각적으로 표현함으로써 2차원 표면의 높낮이를 3차원으로 보이도록 만든 영상 또는 지도
<br>
* DEM(Digital Elevation Model) : 수치표고모델의 약어이며, 수치지면자료를 이용해 제작한 지표 모형
<br>

<!--more-->

## 1. 활용 데이터

| 데이터 종류 | 설명 |
|-------------|------|
| DEM 데이터 | 수치표고모델(Digital Elevation Model) |
| 상주시 행정구역 경계 | Shapefile 형식 |
| 상주시 30m 단위 격자 데이터 | 지형 데이터 |
| 기상청 단기예보 데이터 | 5km 단위 격자 예보 (하늘상태 예보: 1=맑음, 3=구름조금, 4=흐림) |

## 2. 구축 프로세스

<div style="text-align: center">
    <img src="../images/jhg/0304_process.png" style="max-width: 1000px; height: auto;" alt="프로세스">
</div>

<center> 그림1 - 일조시간 보간도 구축 프로세스 </center>

### (1) 일조시간/하늘상태 데이터 공간보간 처리
- 기상청 단기예보 데이터는 일조시간 데이터가 존재하지 않으므로, 하늘상태 데이터를 활용하여 일조시간 보간
- 하늘 상태에 따른 운량 보정계수를 DEM 해상도에 맞춰 공간 보간 처리
- 시간대별 운량 보정이 적용된 DEM 데이터 구축


<div style="text-align: center">
    <img src="../images/jhg/0304_process2.png" style="max-width: 1500px; height: auto;" alt="프로세스">
</div>

<center> 그림2 - 시간대별 하늘상태 데이터 구축 프로세스 </center>


### (2) 시간대별 태양 방위각 및 고도 산출
- 한국 천문연구원 출몰시각 정보 API 활용하여 해당 일자의 일출몰 시각을 산출
- Python `pysolar` 패키지를 이용하여 해당 날짜의 일출몰 시각 사이의 시간대별 태양 방위각 및 고도 산출

```python
from pysolar.solar import get_altitude, get_azimuth
from datetime import datetime
import pytz

# 위치와 시간 지정
latitude = 37.5665   # 위도 (예: 서울)
longitude = 126.9780  # 경도 (예: 서울)
utc = pytz.utc  # UTC 시간대

time = utc.localize(datetime(2025, 1, 1, 12, 0, 0))  # 시간 (UTC 기준, 시간대 추가)

# 태양의 고도와 방위각 계산
altitude = get_altitude(latitude, longitude, time)
azimuth = get_azimuth(latitude, longitude, time)
```

### (3) 시간대별 일조시간 산출
- `DEM` 데이터는 2차원 배열의 형태로 각 픽셀이 해당 위치의 고도값을 가지고 있으며, 해당 값을 활용하여 경사도(Slope) 및 향(Aspect) 계산 후 음영기복도 생성
- 음영기복도는 `0~255` 범위의 값을 가지며, `255`에 가까울 수록 휘도가 높음을 의미
- 태양의 방위각 및 고도를 고려하여 시간대별 음영기복도 생성

<div style="border: 1px solid #ddd; padding: 15px; text-align: center; margin: 20px 0;">

#### 경사도(Slope) 계산

$$slope = \arctan(\sqrt{dx^2 + dy^2})$$

#### 향(Aspect) 계산

$$aspect = \arctan2(-dx, dy)$$

#### 음영기복도 계산

$$shaded = \cos(zenith) \times \cos(slope) + \sin(zenith) \times \sin(slope) \times \cos(az\_rad - aspect)$$

</div>

| 변수 | 설명 |
|------|------|
| zenith | 태양 천정각 (90° - 고도각) |
| az_rad | 태양 방위각 (라디안) |
| slope | 경사도 (라디안) |
| aspect | 향 (라디안) |

- 음영기복도 값을 독립변수(`0~255`), 일조시간을 종속변수(`0~60분`)로 설정하여 일조시간 도출


<div style="text-align: center">
    <img src="../images/jhg/0304_map.png" style="max-width: 1000px; height: auto;" alt="지도">
</div>

<center> 그림3 - 상주시 음영기복도 </center>


### (4) 하늘상태 보정
- 하늘상태 보정계수를 적용하여 최종 일조시간 산출


<div style="border: 1px solid #ddd; padding: 15px; text-align: center; margin: 20px 0;">
<strong> ■ 하늘 상태 보정계수 적용한 일조시간 산출 예시</strong><br>
- 산출된 일조시간 : 45분<br>
- 하늘 상태 : 구름 조금<br>
- 최종 일조시간 = 45분 x 0.683(하늘 보정계수) = 30.7분
</div>

### (5) 시간대별 처리 및 누적
- 일출/일몰 시간이 정각이 아닐 경우, 비일조시간을 제외하여 보정

<div style="border: 1px solid #ddd; padding: 15px; text-align: center; margin: 20px 0;">
<strong> ■ 일출/일몰 시간대 처리 예시 </strong><br>
✔ 일몰시각이 18:20 인 경우 <br>
- 산출된 18시의 일조시간 : 45분 <br>
- 하늘 상태 : 구름 조금<br>
- 전체 60분 중 20분만 일조 가능하므로, 45분 X (20/60) = 15분
</div>


### (6) 최종 일조시간 산출
- 시간대별 일조시간을 누적하여 일 단위 일조시간 보간도를 구축
- 산출된 값은 30m 단위 격자와 매칭하여 최종 보간도 생성

## 3. 결과 및 비교

### 맑은 날

| 구축 보간도 | 조기경보 시스템 보간도 |
|-------------|----------------------|
| <img src="../images/jhg/theimc_1.png" style="max-width: 500px; height: auto;">  | <img src="../images/jhg/ori_1.png" style="max-width: 500px; height: auto;">  |

### 구름이 낀 날 1
| 구축 보간도 | 조기경보 시스템 보간도 |
|-------------|----------------------|
| <img src="../images/jhg/theimc_2.png" style="max-width: 500px; height: auto;">  | <img src="../images/jhg/ori_2.png" style="max-width: 500px; height: auto;">  |

### 구름이 낀 날 2
| 구축 보간도 | 조기경보 시스템 보간도 |
|-------------|----------------------|
| <img src="../images/jhg/theimc_3.png" style="max-width: 500px; height: auto;">  | <img src="../images/jhg/ori_3.png" style="max-width: 500px; height: auto;">  |

## 4. 결론 및 제언
- 본 연구에서 구축한 일조시간 보간도는 조기경보 시스템의 보간도와 유사한 분포를 보였으며, 실질적인 활용 가능성을 확인함
- 하늘상태와 음영기복도를 결합한 방법론은 특히 산악지형이 많은 한국의 지형 특성에 적합한 것으로 판단됨
- 하지만 기상청 제공 일조시간 관측 지점이 부족하여 정확성 검증에는 한계가 존재
- 다양한 기상 조건에서의 추가 테스트를 통해 모델의 신뢰성을 향상시킬 필요가 있음

## 참고 문헌
김승호, 윤진일, 2016: 하늘상태와 음영기복도에 근거한 복잡지형의 일조시간 분포 상세화. 한국농림기상학회지 18(4), 233-241.

