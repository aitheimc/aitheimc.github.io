---
layout: post
title:  "LLM을 활용한 비정형 데이터 전처리"
date:   2024-12-10 00:00:00 +0900
author: Jhk
---
<br>

# <br> <center>LLM을 활용한 비정형 데이터 전처리</center> <br>
####  <center>keyword : LLM, GPT, 비정형데이터</center>

---
####  <center>[배경]</center>
- 현재 산업안전보험공단과 국가에서 관리 중인 화학물질 DB 구축 사업을 진행하고 있음

- 비정형 데이터 정형화에 대한 코딩 방식과 LLM 활용의 차이점을 비교

- 향후 유사 사업 제안에서 LLM을 활용한 기술 제안 방식을 공유하기 위해 주제 선정

<br>

---
### 1. 산업 배경 및 관련 동향

<center>

<br>


![12321](./241210_jh_01.png)

<center>- 그림 1 : 데이터 공유 플랫폼 구축 계획 -</center>

<br>

</center>

▷ 행정안전부의 “2024년 데이터기반행정 활성화 시행계획”에 따르면 범정부 데이터공유플랫폼을 목표로, 24년에만 약 5500억원의 예산 규모로 기관별 공유데이터관리시스템을 구축하고 26년까지 지속할 계획으로 다양한 기관에서 데이터관리시스템 구축 사업이 진행 중 <br>

▷ 산업안전보험공단과 진행 중인 본 사업도 화학물질 관련 정보 수집과 자체적인 DB 구축 후, 추가적인 법률이나 다양한 산업 분야에 대한 정보 수집하여 데이터관리시스템을 확장하려고 함 <br>

▷ 타 회사에서 제안하는 파일 데이터 정제 솔루션은 이미 구조화된 문서에서 데이터 추출(예 : 이름 칸에서 이름 정보 추출)하는 경우나 인력을 바탕으로 자료 정리가 대부분인 상황

---

### 2. 코딩 방식 한계

<center>

|    **고유번호**   | **화학 물질의 명칭** |
| :-----------: | :-----------: |
  97-1-119 | 비소[Arsenic; 7440-38-2]와 그 화합물 및 이를 0.1% 이상 함유한 혼합물 |

</center>

<center>- 표 1 : 데이터 예시 -</center>

<center>▼</center>

<center>

|    **고유번호**   | **한글명** | **영문명** | **CAS** | **혼합물 기준** |
| :-----------: | :-----------: | :-----------: | :-----------: | :-----------: |
|97-1-119|비소|Arsenic|7440-38-2|0.1%|

</center>

<center>- 표 2 : 데이터 전처리 예시 -</center>
<br>

▷ 기존의 자료는 하나의 셀 안에 여러 개의 정보를 포함<br>

▷ 이를 정규화하기 위해 “[”를 기준으로 앞에는 한글명, “;” 앞에는 영문명 등 기준으로 코드를 작성해서 전처리함

▷ 하지만 화학물질 자체에 '[' 문자가 포함된다거나 순서가 반대인 것과 같은 예외들로 약 2000건의 데이터에서 200건 정도 오류가 발생함

▷ 환경부에서만 관리 중인 전체 5만개 이상의 화학물질 데이터를 대상으로 코드를 작성하면 코드가 엄청나게 복잡해지거나 사람이 데이터를 하나씩 수정해야 함

---

### 3. LLM

<center>

|    **구분**   |    **예시**   |
| :-----------: | :-----------: |
|비고 정보 포함|무기시안 화합물[Inorganic cyanide compounds] 및 이를 1% 이상 함유한 혼합물. <span style='background-color:#F7DD'>다만, 베를린청(Ferric ferrocyanide), ...</span>|
|명칭에 대괄호 기호 포함|이염화1-<span style='background-color:#F7DD'>[2-[에틸[4-[4-[4-[</span>에틸(2-피리디노에틸)아미노]-2-메틸페닐 아조]벤|
|혼합물 기준이 없는 경우|인독사캅[Indoxacarb; 173584-44-6]. 다만, 이를 함유한 혼합물은 제외|
|이명 정보 포함|브로노폴[<span style='background-color:#F7DD'>Bronopol; 2-Bromo-2-nitro-1,3-propanediol</span>; 52-51-7] 및 이를 2.5% 이상 함유한 혼합물|
|영문명 및 CAS가 복수|에피클로로히드린[<span style='background-color:#F7DD'>Epichlorohydrin; 1-Chloro-2,3-epoxypropane; 2-(Chloromethyl)oxirane; 106-89-8, 51594-55-9, 67843-74-7</span>] 및 이를 0.1% 이상 함유한 혼합물|
|한 행의 다수에 물질 정보 포함|<span style='background-color:#F7DD'>수산화 세트리모늄</span>[Cetrimonium hydroxide; 505-86-2]과 <span style='background-color:#F7DD'>세트리모늄 염류</span>[Cetrimonium, salts; 112-02-7, 57-09-0, 124-23-2 등] 및 이를 2.5% 이상 함유한 혼합물|
|혼합물(CAS 미포함)|산화 트리헥실포스핀, 산화 트리-n-옥틸포스핀, 산화 디옥틸 모노옥틸 디헥실포스핀의 <span style='background-color:#F7DD'>혼합물[Mixture</span> of trihexylphosphine oxide, tri-n-octylphoshine oxide, dioctyl monooctyl dihexylphosphine oxide] 및 이를 25% 이상 함유한 혼합물|
|혼합물(CAS 포함)|2-[[(3-에테닐페닐)메톡시]메틸]옥시란과 2-[[(4-에테닐페닐)메톡시]메틸]옥시란의 혼합물[Mixture of 2-[[(3-ethenylphenyl)methoxy]methyl]oxirane(<span style='background-color:#F7DD'>113538-79-7</span>) and 2-[[(4-ethenylphenyl)methoxy]methyl]oxirane<span style='background-color:#F7DD'>(113538-80-0); 부여안됨</span>] 및 이를 1% 이상 함유한 혼합물|
|기타|산화니켈 및 황화니켈류[<span style='background-color:#F7DD'>Nickel oxide; 1313-99-1, 11099-02-8, 12035-36-8, 1314-06-3 / Nickel sulfide; 16812-54-7, 11113-75-0, 12035-72-2</span>]와 그 중 하나를 0.1% 이상 함유한 혼합물|

<center> - 표 3 : 예외 예시 - </center>

<br>

</center>

▷ 기존 코딩 방식의 한계를 회피하기 위해서 Chat-GPT 를 활용해 데이터 전처리를 진행함

▷ 데이터 전처리는 Chat-GPT에게 규칙을 작성한 뒤, 코딩 방식에서 전처리 하지 못한 데이터들 중 대표적인 오류들을 Input 으로 Output 값을 확인하는 방식으로 진행함 


<center>

|    **고유번호**   | **화학 물질의 명칭** |
| :-----------: | :-----------: |
 Output Formatting Prompting | 출력 결과물에 대한 형태 명시 |
 Directional Stimulus Prompting | 문제 해결을 위한 힌트를 제시 |
 One-Shot Prompting | 입력물과 결과물에 대한 예시 1개 제공 <br> (* 없으면 Zero, 2개 이상이면 Few) |

<center> - 표 4 : 프롬프팅 기술 - </center>
 </center>

---
```
Input Text Parsing Rules:
Extract and organize chemical information into structured JSON following the provided structure and rules. Handle multiple substances and mixtures according to these guidelines.

JSON Structure:
{
  "Korean name": "Extracted Korean name from the text",
  "English name": "Primary English name or name in brackets []",
  "Alternative English names": ["Any other English names listed after a semicolon ;"],
  "CAS number": ["List of CAS numbers if present, as strings"],
  "mixture standard": "Percentage requirement for mixture, e.g., '1%'",
  "note": "Additional information in parentheses () or special conditions"
}

Rules:
1. **Korean Name Extraction**:
   - Extract the first mentioned Korean name from the text.

2. **English Name and Alternative English Names**:
   - Extract the first name inside square brackets [] as the primary "English name."
   - Include any names listed after a semicolon (;) as "Alternative English names" in a list.

3. **CAS Number**:
   - Extract all CAS numbers (in the format NNN-NN-N) mentioned in the text.
   - Represent CAS numbers as an array of strings.
   - If no CAS number is mentioned, leave this field as an empty array.

4. **Mixture Standard**:
   - Extract percentage content requirements (e.g., "1%") and assign to this field.
   - If not specified, leave this field empty.

5. **Note**:
   - Include any additional text within parentheses () or specific conditions in Korean.
   - If no relevant information is found, leave this field empty.

6. **Substance-Mixture Distinction**:
   - For mixtures, group all related information into a single JSON object.
   - For multiple distinct substances, create separate JSON objects, ensuring:
     - The same Korean name is assigned to related substances.
     - English names and CAS numbers match appropriately.

7. **Empty or Missing Fields**:
   - If any field is not explicitly mentioned, leave it as an empty string or an empty array.

Example Input:
"에틸알코올 [Ethyl Alcohol]; Alcohol CAS: 64-17-5 (Highly flammable) 1%"

Example Output:
[
  {
    "Korean name": "에틸알코올",
    "English name": "Ethyl Alcohol",
    "Alternative English names": ["Alcohol"],
    "CAS number": ["64-17-5"],
    "mixture standard": "1%",
    "note": "Highly flammable"
  }
]

Remember this rule and if user give you input, answer with output, without Explanation
```
---

<center> - 프롬프트 문 - </center>

<br>

▷ 프롬프팅 기술은 Output Formatting, Dircetional Stimulus, One-Shot 등 간단한 기술을 사용함 <br>
<br>

---
Input : 
무기시안 화합물[Inorganic cyanide compounds] 및 이를 1% 이상 함유한 혼합물. 다만, 베를린청(Ferric ferrocyanide), 페로시안염(Ferrocyanide, salts), 페리시안염(Ferricyanide, salts) 및 그 중 하나를 함유한 혼합물은 제외

Output
```
[
  {
    "Korean name": "무기시안 화합물",
    "English name": "Inorganic cyanide compounds",
    "Alternative English names": [],
    "CAS number": [],
    "mixture standard": "1%",
    "note": "베를린청(Ferric ferrocyanide), 페로시안염(Ferrocyanide, salts), 페리시안염(Ferricyanide, salts) 및 그 중 하나를 함유한 혼합물은 제외"
  }
]
```
---

Input : 
이염화1-[2-[에틸[4-[4-[4-[에틸(2-피리디노에틸)아미노]-2-메틸페닐 아조]벤조일아미노]페닐아조]-3-메틸페닐]아미노]에틸]피리디늄[ 1 - [ 2 - [ Ethyl[4-[4-[4-[ethyl(2-pyridinoethyl)amino]-2-m ethylphenylazo]benzoylamino]phenylazo]-3-methylphenyl]a mino]ethyl]pyridinium dichloride; 163831-67-2] 및 이를 25% 이상 함유한 혼합물

Output
```
[
  {
    "Korean name": "이염화1-[2-[에틸[4-[4-[4-[에틸(2-피리디노에틸)아미노]-2-메틸페닐 아조]벤조일아미노]페닐아조]-3-메틸페닐]아미노]에틸]피리디늄",
    "English name": "1-[2-[Ethyl[4-[4-[4-[ethyl(2-pyridinoethyl)amino]-2-methylphenylazo]benzoylamino]phenylazo]-3-methylphenyl]amino]ethyl]pyridinium dichloride",
    "Alternative English names": [],
    "CAS number": ["163831-67-2"],
    "mixture standard": "25%",
    "note": ""
  }
]
```
---
Input : 
수산화 세트리모늄[Cetrimonium hydroxide; 505-86-2]과 세트리모늄 염류[Cetrimonium, salts; 112-02-7, 57-09-0, 124-23-2 등] 및 이를 2.5% 이상 함유한 혼합물
Output
```
[
  {
    "Korean name": "수산화 세트리모늄",
    "English name": "Cetrimonium hydroxide",
    "Alternative English names": [],
    "CAS number": ["505-86-2"],
    "mixture standard": "2.5%",
    "note": ""
  },
  {
    "Korean name": "세트리모늄 염류",
    "English name": "Cetrimonium, salts",
    "Alternative English names": [],
    "CAS number": ["112-02-7", "57-09-0", "124-23-2"],
    "mixture standard": "2.5%",
    "note": ""
  }
]
```
---

<center> - 전처리 성공 예시 - </center>

<br>

▷ 기존의 코드로 전처리 하기 어려웠었던 예외들 GPT를 통해 전처리했을 때, 대다수의 경우가 비교적 간단한 프롬프트 문으로도 원하는 결과를 반환하는 것을 확인할 수 있었음 

▷ 특히 '한 행의 다수에 물질 정보 포함된 경우'처럼 복잡한 구조의 데이터도 분리하여 전처리하는 것을 확인 가능함

---
Input : 
무기시안 화합물[Inorganic cyanide compounds] 및 이를 1% 이상 함유한 혼합물. 다만, 베를린청(Ferric ferrocyanide), 페로시안염(Ferrocyanide, salts), 페리시안염(Ferricyanide, salts) 및 그 중 하나를 함유한 혼합물은 제외

Output
```
[
  {
    "Korean name": "2-[[(3-에테닐페닐)메톡시]메틸]옥시란과 2-[[(4-에테닐페닐)메톡시]메틸]옥시란의 혼합물",
    "English name": "Mixture of 2-[[(3-ethenylphenyl)methoxy]methyl]oxirane and 2-[[(4-ethenylphenyl)methoxy]methyl]oxirane",
    "Alternative English names": [],
    "CAS number": ["113538-79-7", "113538-80-0"],
    "mixture standard": "1%",
    "note": "부여안됨"
  }
]
```

---
Input : 
무기시안 화합물[Inorganic cyanide compounds] 및 이를 1% 이상 함유한 혼합물. 다만, 베를린청(Ferric ferrocyanide), 페로시안염(Ferrocyanide, salts), 페리시안염(Ferricyanide, salts) 및 그 중 하나를 함유한 혼합물은 제외

Output
```
[
  {
    "Korean name": "2-[[(3-에테닐페닐)메톡시]메틸]옥시란과 2-[[(4-에테닐페닐)메톡시]메틸]옥시란의 혼합물",
    "English name": "Mixture of 2-[[(3-ethenylphenyl)methoxy]methyl]oxirane and 2-[[(4-ethenylphenyl)methoxy]methyl]oxirane",
    "Alternative English names": [],
    "CAS number": ["113538-79-7", "113538-80-0"],
    "mixture standard": "1%",
    "note": "부여안됨"
  }
]
```
---
Input : 
산화니켈 및 황화니켈류[Nickel oxide; 1313-99-1, 11099-02-8, 12035-36-8, 1314-06-3 / Nickel sulfide; 16812-54-7, 11113-75-0, 12035-72-2]와 그 중 하나를 0.1% 이상 함유한 혼합물

Output
```
[
  {
    "Korean name": "산화니켈 및 황화니켈류",
    "English name": "Nickel oxide",
    "Alternative English names": [],
    "CAS number": ["1313-99-1", "11099-02-8", "12035-36-8", "1314-06-3"],
    "mixture standard": "0.1%",
    "note": ""
  },
  {
    "Korean name": "산화니켈 및 황화니켈류",
    "English name": "Nickel sulfide",
    "Alternative English names": [],
    "CAS number": ["16812-54-7", "11113-75-0", "12035-72-2"],
    "mixture standard": "0.1%",
    "note": ""
  }
]
```
---

<center> - 전처리 실패 예시 - </center>

<br>

▷ 하지만 기존의 화학물질과는 더 큰 예외 특성을 가지고 있는 데이터는 전처리가 불완전한 것을 확인함<br>
 (* 혼합물(CAS 포함), 기타)

▷ 이러한 데이터를 전처리하기 위해서는 프롬프트에 규칙이나 예시를 더 추가하는 방법이 있으나 이럴 경우 프롬프트가 복잡해지고 직관성이 떨어짐 

---
####  <center>[결론]</center>
- 공공기관의 데이터 정형화 사업은 시장성이 높은 사업이지만 기존 데이터들의 품질 문제에 단순 코드 방식으로는 사업 수행의 한계가 있음

<br>

- LLM은 코딩 방식에 비해 전처리 작업을 위한 직관성과 성능 모두 높아 효율적임

<br>

- 다만, LLM 또한 완벽하지 않으며 성능을 높이기 위해서 프롬프트 문이 복잡해질 필요성이 있음

<br>

---