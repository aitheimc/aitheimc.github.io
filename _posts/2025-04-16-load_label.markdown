---
layout: post
title: 도로 라벨링 구축
date: 2025-04-16
author: Whale
excerpt_separator: "<!--more-->"
tags:
    - 도로데이터
    - 라벨링
    - 데이터구축
---

창원시의 도로 데이터를 정밀하게 활용하기 위해 도로 라벨링 데이터를 구축하였습니다.<br> QGIS 기반의 공간정보 분석 환경을 활용해 도로에 대한 라벨링을 진행하고, 도로별 속성 및 구간 정보를 표준 노드링크 데이터에 맞게 가공하였습니다. 이후 국토교통부의 표준 노드링크 체계와 연계하여 도로 간의 일대일 매핑을 수행했습니다. 이 매핑 과정에서는 표준 노드링크가 보유한 최고제한속도 정보를 라벨링된 도로에 연결함으로써, 주요 교통 속성 데이터를 반영할 수 있도록 구성하였습니다.<br>
이번 포스팅에서는 도로 라벨링 진행 과정부터 데이터 구축, 속성 보정, 표준화에 이르기까지 전체 작업 흐름을 순차적으로 소개하고자 합니다.
<!--more-->

## 1. 도로 라벨링 데이터 구축

### 공간분석 툴
- QGIS : 오픈소스 기반의 공간정보(GIS) 분석 소프트웨어로, 도형 데이터를 시각화하고 편집할 수 있는 도구입니다. 다양한 플러그인을 통해 공간 데이터 처리와 시각화를 직관적으로 수행할 수 있습니다.

### 도로 라벨링 기준

- 도로 라벨링 작업은 일관된 기준에 따라 수행되어야 정확한 데이터 품질을 확보할 수 있습니다.  
- 창원시 라벨링 작업에서는 도로의 **우선순위, 분기점 처리, 방향성 구분, 간격 유지, 속도 기준 우선 처리** 등을 종합적으로 고려하였습니다.

#### 1. 도로 우선 순위에 따른 라벨링

복수의 도로가 중첩되거나 병행할 경우, **도로의 등급에 따라 우선순위를 부여**하여 상위 도로를 기준으로 라벨링합니다.
<div style="text-align: center">
    <img src="../images/whale/ppt1.png" style="width: auto; height: auto;" alt="도로라벨링우선순위">
</div>
- 1순위: 고속도로, 자동차 전용도로 등 주요 간선도로<br>
- 2순위: 국도, 지방도 등 중간급 간선도로<br>
- 3순위: 일반 시내도로 및 지선도로<br>

> 📌 중복 구간에서는 상위 도로만 라벨링되며, 하위 도로는 생략합니다.

---

#### 2. 분기점(갈림길) 처리 기준

도로가 분기되거나 교차되는 지점에서는 **갈림길이 시작되는 시점을 기준으로 라벨링**합니다.  
특히 분기된 각 도로는 개별 객체로 구분하여 각각의 흐름을 반영해야 합니다.
<div style="text-align: center">
    <img src="../images/whale/ppt1_2.png" style="width: auto; height: auto;" alt="도로라벨링분기점처리">
</div>
- 분기 발생 시 갈라지는 각 도로에 대해 별도로 라벨링<br>
<div style="text-align: center">
    <img src="../images/whale/ppt2.png" style="width: auto; height: auto;" alt="도로라벨링우선순위가다른분기점처리">
</div>

---

#### 3. 방향(상·하행) 구분 기준

하나의 도로도 **진출입 방향(상행·하행)**에 따라 각각 별도로 라벨링합니다.  
단일 객체로 보일 수 있는 도로라도, 실제로 차량 흐름이 구분되는 경우에는 **각 방향마다 개별 객체로 처리**합니다.
<div style="text-align: center">
    <img src="../images/whale/ppt3.png" style="width: auto; height: auto;" alt="상행하행구분">
</div>

> 📌 상하행이 지리적으로 나뉜 경우 → 두 개의 객체로 라벨링<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 동일 도로 위에 상·하행 모두 있을 경우 → 한 번에 라벨링 가능

---

#### 4. 도로 간격 및 겹침 방지

폴리라인 혹은 폴리곤으로 구축된 도로 라벨링 객체는 **서로 겹치지 않도록 충분한 간격을 유지**해야 합니다.  
확대 시에도 객체 간 경계가 명확히 보이도록 하는 것이 중요합니다.
<div style="text-align: center">
    <img src="../images/whale/ppt4.png" style="width: auto; height: auto;" alt="겹침방지예시">
</div>

> 📌 겹침 없이 시각적 분리감이 있어야 후속 매핑시 오류가 발생하지 않습니다.

---

#### 5. 상·하행 구분 및 중간 갈림길 처리 기준

도로에 **상행/하행 방향이 명확하게 표시된 경우**, 해당 표시를 기준으로 라벨링을 구분하여 수행합니다.
<div style="text-align: center">
    <img src="../images/whale/ppt5.png" style="width: auto; height: auto;" alt="강,갈림길라벨링기준">
</div>
- 하천 등으로 인해 상·하행이 물리적으로 나뉜 경우, 상행과 하행 도로를 한 번에 라벨링합니다.<br>
- 위아래 도로 모두에 상행/하행이 존재하는 경우, 각각의 도로를 별도로 라벨링합니다.<br>


> 📌 상·하행 구분이 애매하거나 지도에 표기되지 않은 경우에는 네이버 지도 등 외부 지도를 참고하여 판단합니다.

---

#### 6. 고가도로 및 터널 우선순위

고가도로, 지하차도, 터널 등 **도로가 입체적으로 교차되는 구간**에서는  
**표준노드링크 데이터의 MAX_SPD(최고제한속도)** 값을 참고하여, **속도가 높은 도로를 기준으로 라벨링**합니다.
<div style="text-align: center">
    <img src="../images/whale/ppt6.png" style="width: auto; height: auto;" alt="터널,고가도로예시">
</div>
<div style="text-align: center">
    <img src="../images/whale/ppt7.png" style="width: auto; height: auto;" alt="애매한우선순위예시">
</div>

> 📌겹치는 경우, 더 빠른 도로를 기준으로 라벨링하고, 라벨링 기준 도로 외 나머지는 생략합니다.

---

#### 7. 전체 예시

실제 복잡한 교차로나 IC(인터체인지) 구간에서는  
위 기준을 종합적으로 적용하여 **도로 방향, 우선순위, 중첩구간 등**을 분리 라벨링합니다.
<div style="text-align: center">
    <img src="../images/whale/ppt8.png" style="width: auto; height: auto;" alt="전체예시">
</div>

> 📌 교차점, 램프, 고가도로 등 다양한 요소가 혼합된 구간에서는 세부 판단 기준을 종합적으로 고려하여 라벨링을 수행합니다.

---

## 2. 데이터 매핑
도로 라벨링의 완성도를 높이기 위해, 라벨링된 도로 데이터에 표준노드링크 및 경찰청 데이터의 주요 속성값을 매핑하는 작업을 수행합니다. <br>
이를 통해 단순한 도형 정보에 그치지 않고, **도로의 구조적 특성 및 교통 관련 정보**를 포함한 정형화된 공간 데이터로 확장할 수 있습니다.

### 표준노드링크 데이터
- 표준노드링크 데이터는 국토교통부에서 제공하는 지능형 교통체계(ITS) 기반의 국가 표준 도로망 데이터입니다. 이 데이터는 차량 주행 시 속도 변화가 발생하는 지점을 나타내는 **노드(Node)**와, 노드 간의 도로 구간을 연결한 **링크(Link)**로 구성되어 있습니다.
- 링크는 실제 도로구간에 대한 정보를 담고 있으며 링크ID, 차로수, 도로등급, 도로유형(도로, 교량, 고가도로, 지하차도, 터널 등), 도로번호, 최고제한속도 등이 링크로 표현되어 있습니다.
<div style="text-align: center">
    <img src="../images/whale/loadlink.png" style="width: auto; height: auto;" alt="표준노드링크예시">
</div>

### 경찰청 교통안전 데이터
- 경찰청 도시교통정보센터(UTIC)는 교통사고, 돌발 상황, 교통안전 시설 등 다양한 교통 정보를 SHP(Shapefile) 형식으로 제공합니다. 이 데이터는 도로별 사고 이력 및 안전 요소를 공간적으로 분석하는 데 활용되며, 사고다발지역(스쿨존, 노인보호구역, 무단횡단 등), 차종별, 법규위반별, 보행자별, 도로환경별 위험요소 등으로 구성되어 있습니다.
<div style="text-align: center">
    <img src="../images/whale/UTIC.png" style="width: auto; height: auto;" alt="표준노드링크예시">
</div>

### 도로 라벨링 작업 가이드 (참고용)
- 아래 가이드는 이전 사업에서 QGIS를 활용해 도로 라벨링 작업을 수행하는 작업자(라벨러)를 위해 제작된 안내서입니다.<br>
👉 [QGIS 도로 라벨링 작업 가이드](../images/whale/labelG/QGIS가이드_창원버스.html)<br>
※ 해당 가이드는 기존 프로젝트 기준에 기반하고 있으며 라벨링 툴 활용가이드로 참고하기 바랍니다.
---

### 매핑(mapping) 방식

  매핑은 **지리적 위치 기반(Geometry Matching)** 또는 **속성 기반(Key Matching)**으로 수행할 수 있으며,  
이번 프로젝트에서는 라벨링된 도로와 표준노드링크 간 **공간 위치의 유사성**을 기준으로 다음 절차를 거칩니다:

1. **좌표계 통일 및 도로 정렬**  
   - 라벨링 도로(SHP)와 표준노드링크 데이터를 동일 좌표계(EPSG:5186 등)로 정렬  
   - 공간 겹침이 가능한 위치 기준을 확보

2. **공간 매칭 (Spatial Join)**  
   - QGIS에서 `공간조인(Spatial Join)` 도구를 사용  
   - 라벨링 도로와 가장 겹치는 표준노드링크 객체의 속성을 가져옴

3. **속성값 병합**  
   - 매칭된 링크로부터 `차로수`, `최고제한속도`, `도로유형` 등의 값을 라벨링 도로에 삽입  
   - 이후 도로 객체별로 시각화 또는 데이터 분석이 가능한 상태로 정비

<div style="text-align: center">
    <img src="../images/whale/mapping.png" style="width: auto; height: auto;" alt="표준노드링크예시">
</div>

> 📌 동일 위치에 여러 링크가 겹치는 경우에는 제한속도(`MAX_SPD`) 등 주요 속성을 기준으로 **우선순위를 부여**하여 적용합니다.

---

### 결과 활용

속성이 매핑된 도로 데이터는 단순한 공간 객체를 넘어 **정량적 분석이 가능한 공간 정보**로 다양한 분야에서 활용이 가능합니다.

- 도로별 속도제한 시각화 및 교통 안전지도 제작  
- 특정 유형 도로에 대한 정비 우선순위 분석  
- 스마트시티 기반의 도로 최적 경로 분석 및 시뮬레이션

아래 링크에서는 라벨링된 도로 데이터를 기반으로, 실제 버스의 운행 데이터를 매핑하여 시각화한 결과를 확인할 수 있습니다.<br>
👉 [도로 라벨링 활용 시각화 예시 보기](../images/whale/1238bus_visu.html)<br>