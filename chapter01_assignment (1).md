# Chapter 01 제출 답안. AI와 함께하는 데이터 분석의 시작

> 이 파일은 Chapter 01 실습 결과를 정리하여 제출하기 위한 학생용 템플릿입니다.  
> 강사 저장소의 원본 템플릿을 직접 수정하지 말고, 자신의 PC에 복사한 뒤 작성합니다.

---

## 0. 제출 정보

- 이름: 조유라
- GitHub ID: joyula3
- 개인 저장소명: `llm-data-analysis-study`
- 작성일: 9월 5일
- 사용한 LLM: 클로드

### 최종 제출 URL

```text
여기에 개인 GitHub 저장소의 chapter01/chapter01.md 파일 URL을 입력하세요.
```

---

## 1. 원래 업무 질문

### 내가 선택한 막연한 질문

```text
: 어떤 고객이 제품을 많이 구매하나요?
```

### 왜 이 질문이 모호하다고 생각했는가?
: 어떤 제품인지 명확하지 않고 많이 구매했다는 의미가 자주 구매했다는 의미인지, 총액이 많다는 의미인지, 한번 구매했을 때 많은 금액을 쓴 것인지에 대한 기준 모호함 

- 대상: 주문이 완료된 기준
- 기간: 2025-01-01 ~ 2025-12-31
- 기준: 총액 기준
- 비교 방법: payment_method별 비교
- 분석 목적: payment_method별로 completed된 금액 집계 

### 분석 가능한 질문으로 다시 작성

```text
2025-01-01 ~ 2025-12-31, 기간을 구분하여 payment_method별로 completed가 된 기준에서
각 제품별로, 고객별로 얼만큼 금액으로 구매했는지를 집계/비교 
```

### 결과 관찰

굉장히 모호했던 질문들을, 답변 가능한 질문으로 구체화되는 것을 확인하였음 

### 나의 해석과 판단

LLM 입장에서도 실제로 답변이 가능한 상태로 만들어 주었습니다. 

### 업무·분석적 의미

기간별, payment_method별 completed 필터하여 
각 제품별, 고객별 구매금액을 통해서 
어떤 payment_method에 프로모션을 주어서 고객들을 추가적으로 유치할지 
그리고 어떤 payment_method에서 취소자가 많은지를 
payment별로 고객 성향을 유추할 수 있음

### 한계와 추가 확인 사항

2025 1년간 

### Evidence

![STEP 1 질문 구체화 결과](images/step01_question.png)
<img width="739" height="653" alt="image" src="https://github.com/user-attachments/assets/f2b32628-4867-4cc0-9597-40e9f1c1f1d8" />

---

## 2. 질문과 필요한 데이터 연결

### 필요한 데이터 파일

- [  ] `customers.csv`
- [ ] `products.csv`
- [ ㅇ ] `orders.csv`
- [ ㅇ ] `order_items.csv`

### 필요한 컬럼 후보

| 파일 | 필요한 컬럼 | 필요한 이유 |
| --- | --- | --- |
1. order_items -> order_id로 합산 후 orders에 결합
2. order_status == 'completed' and order_date == '2025'
----- customers, products는 이 질문에서 불필요함 



### 데이터 연결 관계

```text
<img width="748" height="238" alt="image" src="https://github.com/user-attachments/assets/4b80dadd-39b8-4508-b6d3-7beb258cfea9" />

```

### 결과 관찰

네 order_items, order_status 정보가 필요합니다. 

### 나의 해석과 판단

데이터간 조인, 필터가 필요합니다. 

### 업무·분석적 의미

하나의 데이터만으로 답을 할 수 없고 데이터간 연결성을 파악해야지만 답을 할 수 있다는 사실을 확인했습니다. 

### 한계와 추가 확인 사항

전부 확인하였습니다. 결측치 없음 

### Evidence

필요한 경우 관계도 또는 데이터 파일 확인 화면을 첨부하세요.

![STEP 2 데이터 구조 확인](images/step02_data_structure.png)

<img width="467" height="811" alt="image" src="https://github.com/user-attachments/assets/63f086fd-25db-4c14-8541-4d80ebc8955f" />

---

## 3. LLM에게 분석 질문 후보 요청

### 사용 목적

```text
평소에서 클로드 코드를 통해서 python 코드를 짜기도 하고, 본인에게 익숙한 ui를 보유하고 있고
익숙하기 때문에 해당 LLM을 선택하였습니다. 

```

### 사용한 Prompt

```text
대상 — 어떤 행만 볼 것인가 (예: order_status == 'completed')
기간 — 언제부터 언제까지 (실제 데이터 범위와 맞는지 확인)
측정값 — 무엇을 셀 것인가 (총액인가 건수인가 비율인가, 어떻게 계산하는가)
비교축 — 무엇끼리 비교할 것인가 (결제수단별, 카테고리별, 월별)
판단 기준 — 어떤 결과가 나오면 무엇을 할 것인가

이제 2025년 기준으로 order_status가 된 completed된 주문을 기준으로
payment별, 제품별, 구매금액에 대하여 집계해 주세요 

```

### LLM 답변 요약

LLM의 전체 답변을 그대로 복사하지 말고 핵심 제안 3~5개를 요약하세요.

1. 결제수단별 주문 금액 및 비중 
2. 카테고리별 주문 금액 및 비중
3. 상위 상품 top10
4. 결제수단 x 카테고리 교차(총액)
5.

### 결과 관찰

다양한 방향으로 분석을 제안해 주었습니다. 
제가 생각하지 못한 부분까지도 
결제수단, 카테고리별, 상위 상품을 추천해 주었습니다. 
특히 금액 뿐만 아니라 그 비중까지도 확인해 주었습니다. 

### 나의 해석과 판단

다양한 분석 방향을 제시한 점은 좋았습니다. 
그대로 사용하기 어려운 점은 아직까지는 확인하기 어렵습니다. 
이렇게 결과가 나왔지만, 이게 제대로 나온 결과인지는 
csv 원본데이터를 보고 다시 한번 확인이 필요해 보입니다. 

### 업무·분석적 의미

내 생각에만 갇혀, 아 이 분석을 해야지 라고만 생각했는데
생각을 확장해서 a부분 b 부분을 확장해서 고민할 수 있다고 생각하였습니다. 

### 한계와 추가 확인 사항

사실 이렇게 나온 부분을 다시 한번 다 검증해야 한다고 생각합니다. 
llm은 분석의 방향성과 컨셉을 제안해 주지만 그 결과는 사람이 상당부분 
확인을 해야한다고 생각합니다. 

단, 모든 분석별로 총합이 같다고 한다면, 원본 데이터에서도 총합이 같다고 한다면 
그 분석은 높은 확률로 맞는 결과이지 않을까 생각해 봅니다. 

### Evidence

![STEP 3 LLM Prompt와 응답](images/step03_llm_response.png)
<img width="760" height="711" alt="image" src="https://github.com/user-attachments/assets/b1ba9fc6-38c7-48a8-a4df-d3311a38fb33" />

---

## 4. LLM 제안 검증

LLM 제안 중 하나 이상을 선택해 검토합니다.

| 검증 항목 | 확인 내용 |
| --- | --- |
| 선택한 LLM 제안 | 결제수단별 주문 건수 및 총액 |
| 필요한 파일 | order |
| 필요한 컬럼 | order_date, payment_method, order_status |
| 계산 범위 | 2025-01-01 ~ 2025-12-31, completed |
| 실제 데이터 확인 필요 여부 | 필요함 |
| 원인 단정 여부 | - |
| 최종 판단 | 사용 가능 |

### 내가 수정한 내용

```text
결과 확인하여 llm이 제안한 사실을 사용해도 된다는 것을 파악하였습니다. 
```

### 결과 관찰

네 확인이 완료되었습니다. llm에서 제안한 내용과 결과를 사용 가능합니다. 
python 결과도 동일하게 나오는 것을 확인하였습니다. 

### 나의 해석과 판단

왜 `사용 / 수정 후 사용 / 보류` 중 해당 결정을 내렸는지 작성하세요.

### 업무·분석적 의미

LLM 제안을 검증하지 않고 바로 사용하는 경우 어떤 문제가 생길 수 있는지 작성하세요.

### 한계와 추가 확인 사항

아직 실제 데이터로 검증하지 못한 부분을 명확히 작성하세요.

### Evidence

![STEP 4 LLM 제안 검증](images/step04_validation.png)

---

## 5. Prompt Log

- 사용 목적:
- 입력 Prompt 요약:
- LLM 답변 요약:
- 실제 반영 여부:
- 사람이 검증한 항목:
- 사람이 수정한 내용:
- 남은 확인 사항:

### 결과 관찰

LLM 사용 기록에서 어떤 의사결정 과정을 확인할 수 있는지 작성하세요.

### 나의 해석과 판단

Prompt Log를 남기는 것이 왜 필요한지 자신의 말로 작성하세요.

### Evidence

![STEP 5 Prompt Log](images/step05_prompt_log.png)

---

## 6. 개인정보와 Secret 보호 확인

다음 항목을 확인합니다.

- [ ] 실제 이름·이메일·전화번호 등 고객 개인정보를 Prompt에 사용하지 않았습니다.
- [ ] API Key를 코드나 Notebook에 직접 작성하지 않았습니다.
- [ ] `.env` 실제 내용을 캡처하거나 업로드하지 않았습니다.
- [ ] GitHub Token, 비밀번호, 내부 URL이 캡처에 보이지 않습니다.
- [ ] 제출 전 이미지까지 다시 확인했습니다.

### 나의 판단

이번 실습에서 어떤 정보는 LLM 또는 Public GitHub에 올리면 안 된다고 판단했는지 작성하세요.

---

## 7. Chapter 01 Notebook 확인

Notebook:

```text
notebooks/ch01_ai_data_analysis_intro.ipynb
```

### 내 환경 상태

- [ ] 아직 환경설정 전이라 Notebook 위치만 확인했습니다.
- [ ] 환경설정이 완료되어 Notebook을 직접 실행했습니다.

### 환경설정 완료 학생만 작성

#### 실행한 코드

```python
from pathlib import Path

import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns

DATA_DIR = Path('../data/raw')
sns.set_theme(style='whitegrid')
```

#### 실행 결과

```text
오류 없이 실행되었는지 작성하세요.
```

#### 결과 관찰

실행 결과에서 확인한 사실을 작성하세요.

#### 나의 해석과 판단

현재 Notebook이 본격 분석이 아니라 starter scaffold라는 의미를 자신의 말로 설명하세요.

#### 한계와 추가 확인 사항

Chapter 02 또는 Chapter 03에서 추가로 확인해야 할 내용을 작성하세요.

#### Evidence

![STEP 7 Notebook 실행 결과](images/step07_notebook_result.png)

> 환경설정 전이라면 이 이미지는 생략할 수 있습니다.

---

## 8. Chapter 01 최종 해석

### 이번 장에서 가장 중요하다고 생각한 내용

```text
자신의 말로 3~5문장 작성하세요.
```

### LLM을 데이터 분석에 사용할 때 가장 조심해야 할 점

```text
자신의 판단을 작성하세요.
```

### 사람과 LLM의 역할 차이

| 항목 | LLM이 도울 수 있는 부분 | 사람이 책임져야 하는 부분 |
| --- | --- | --- |
| 질문 정의 |  |  |
| 데이터 확인 |  |  |
| 코드 작성 |  |  |
| 결과 해석 |  |  |
| 최종 판단 |  |  |

### 다음 Chapter에서 확인하고 싶은 것

```text
이번 장에서 남은 의문이나 Chapter 02~03에서 확인하고 싶은 내용을 작성하세요.
```

---

## 9. 최종 제출 체크리스트

- [ ] 원래 업무 질문과 구체화한 분석 질문을 작성했습니다.
- [ ] 질문에 필요한 데이터 파일과 컬럼 후보를 정리했습니다.
- [ ] LLM Prompt와 답변 요약을 작성했습니다.
- [ ] LLM 제안을 실제 데이터 관점에서 검증했습니다.
- [ ] 각 핵심 STEP의 결과 관찰을 작성했습니다.
- [ ] 각 핵심 STEP의 나의 해석과 판단을 작성했습니다.
- [ ] 업무·분석적 의미를 작성했습니다.
- [ ] 한계와 추가 확인 사항을 작성했습니다.
- [ ] 핵심 실행 Evidence 이미지를 첨부했습니다.
- [ ] 이미지가 Markdown에서 정상 표시됩니다.
- [ ] 개인정보가 없습니다.
- [ ] API Key·Secret·Token이 없습니다.
- [ ] 개인 GitHub 저장소에 업로드했습니다.
- [ ] GitHub에서 Markdown과 이미지가 정상 표시됩니다.
- [ ] 아래 최종 파일 URL이 정상적으로 열립니다.

### 최종 파일 URL

```text
https://github.com/<내-GitHub-ID>/llm-data-analysis-study/blob/main/chapter01/chapter01.md
```

---

## 10. 교수자 확인용 요약

### 수행 상태

- [ ] COMPLETE
- [ ] PARTIAL

### 내가 가장 중요하게 내린 판단 1개

```text
여기에 작성하세요.
```

### 아직 확인이 필요한 내용 1개

```text
여기에 작성하세요.
```
