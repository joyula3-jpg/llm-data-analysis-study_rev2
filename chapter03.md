# Chapter 03 제출 답안 양식. 데이터의 첫인상 읽기

> 주 제출물은 실행 완료 Notebook `chapter03/chapter03.ipynb`입니다. 이 양식의 항목을 Notebook의 Markdown 셀로 추가해 작성합니다.

## 0. 제출 정보
- 이름: 조유라
- GitHub ID: joyula3-jpg
- 작성일:9월 20일(일)
- 최종 제출 URL: https://github.com/joyula3-jpg/llm-data-analysis-study_rev2/edit/main/chapter03_assignment.md

## 1. 데이터 로딩과 구조 확인
### 실행/결과
- 4개 CSV 로딩 여부: 완료되었습니다. 
- 각 데이터 shape:
  : customers - 150행, 6열
  : order - 300행, 6열
  : order_items - 764행, 5열
  : products - 100행, 4열

- 주요 컬럼:
  : customers - customer_id, signup_date, name, gender 
  : order - order_id, customer_id, order_date, payment_method, order_status
  : order_items - order_item_id, order_id, product_id, quantity, unit_price
  : products - product_id, product_name, category, price
  
- dtypes에서 주목한 컬럼:
  : customers - int64, str 혼재 어떤 것을 보고 주목을 해야 할지 판단 불가 
  : order - int64, str 혼재 어떤 것을 보고 주목을 해야 할지 판단 불가 
  : order_items - int64, str 혼재 어떤 것을 보고 주목을 해야 할지 판단 불가 
  : products - int64, str 혼재 어떤 것을 보고 주목을 해야 할지 판단 불가 

### Evidence
![데이터 구조 확인](images/step01_structure.png)

<img width="524" height="351" alt="order_items" src="https://github.com/user-attachments/assets/a70a37f4-0257-4e9b-8061-f4690f1d4055" />
<img width="372" height="338" alt="data load_orders" src="https://github.com/user-attachments/assets/95446d4a-8bce-4660-bab2-56d11c727dc1" />
<img width="441" height="326" alt="data load_customers" src="https://github.com/user-attachments/assets/68fd6638-9dec-4441-bc03-4896ee52f2c2" />
<img width="470" height="366" alt="data load_producdts" src="https://github.com/user-attachments/assets/fc40f1b7-31c9-433e-bb18-34beef7acf84" />
<img width="457" height="457" alt="스크린샷 2026-09-20 오후 2 11 13" src="https://github.com/user-attachments/assets/e2933b4e-4e53-4555-9ab1-2e57170e4a49" />
<img width="457" height="457" alt="스크린샷 2026-09-20 오후 2 11 13" src="https://github.com/user-attachments/assets/76783684-fb48-48f5-b453-21cf360d557c" />

### 결과 관찰
각 데이터들의 연결관계를 확인할 수 있음 
- customers 는 orders 데이터와 연결
- orders는 order_items 데이터와 연결
- products는 order_items 데이터와 연결 

### 나의 해석과 판단
order_items에서 고객정보로 가려면 반드시 orders를 거쳐야 함 
order_items에는 customer_id가 없기 때문에 
order_items --order_id--> orders --customer_id--> customers
order_items --product_id--> products

### 업무·분석적 의미
데이터 간의 연결성을 파악하지 않고 분석을 한다면 
분석 결과를 제대로 해석할 수 있고, 데이터 안에 담겨있는 인사이트를 확인할 수 없다. 
반드시 데이터 분석 전에 데이터간의 연결관계에 대해서 확인하고 데이터 분석을 시작해야 한다. 

### 한계와 추가 확인 사항
데이터 간의 연결 관계는 확인하였지만, 각 데이터 내 컬럼 간의 관련성에 대해서는 파악하기 어려움 

## 2. 결측·중복·키 품질
- 주요 ID 결측: 없음
- 주요 ID 중복: 없음
- 전체 행 중복: 없음 

![결측 중복 점검](images/step02_quality.png)

<img width="615" height="568" alt="스크린샷 2026-09-20 오후 2 14 51" src="https://github.com/user-attachments/assets/dc5c63e3-c38c-4990-9fab-1f915157a841" />
<img width="625" height="610" alt="스크린샷 2026-09-20 오후 2 15 21" src="https://github.com/user-attachments/assets/1676978c-68bd-421b-8174-cc066025a730" />


### 결과 관찰

### 나의 해석과 판단
- 데이터 분석 전에 데이터 내 결측치를 먼저 처리하거나, 중복을 배제하지 않으면 데이터 분석 결과가 올바르게 나오지 않을 수 있다.
  그렇기 때문에 데이터를 확인하고 반드시 unique 값을 확인하고 제대로 데이터가 기입되어 있는지를 확인하는 절차가 필요하다. 

### 업무·분석적 의미
- 데이터의 중복 및 결측치를 제대로 처리하지 못할 경우, 잘못된 인사이트가 도출될 가능성이 있고 이를 실제 현업 업무에 반영하게 된다면
  잘못된 의사결정을 초래할 수 있기 때문에 반드시 사전에 데이터 전처리를 수행해야 한다. 

### 한계와 추가 확인 사항

## 3. 숫자형·범주형·날짜 점검
- 숫자형 범위에서 주목한 값: products에서 price값
- 범주형 빈도에서 주목한 값: products에서 category 값
- 날짜 변환 실패 건수: 없음
- 날짜 범위: 2025년 7월 9일 ~ 2026년 7월 8일

![기본 분포와 날짜 확인](images/step03_distribution.png)

<img width="454" height="871" alt="스크린샷 2026-09-20 오후 2 19 55" src="https://github.com/user-attachments/assets/035df105-3c47-43f9-a7bc-49d590de4015" />

<img width="669" height="290" alt="스크린샷 2026-09-20 오후 2 22 37" src="https://github.com/user-attachments/assets/0fce9068-7e4c-4171-8cec-b1186b36bcf3" />


### 결과 관찰
- 네 특이사항 없이 확인하였습니다. 

### 나의 해석과 판단
- 특이사항 없습니다. 

### 업무·분석적 의미
- 

### 한계와 추가 확인 사항
- 

## 4. CSV 간 키 관계 검증
: 없는 customer_id: 0건 — orders의 모든 고객이 customers에 존재
: 없는 order_id: 0건 — order_items의 모든 주문이 orders에 존재
: 없는 product_id: 0건 — order_items의 모든 상품이 products에 존재

![PK FK 관계 검증](images/step04_relationship.png)

<img width="830" height="303" alt="스크린샷 2026-09-20 오후 2 22 42" src="https://github.com/user-attachments/assets/189058fe-f577-4f8f-a5ea-96c94ea06dcd" />


### 결과 관찰
- 특이사항 없음 

### 나의 해석과 판단
- 무엇이 원인인지 원인 파악을 위함 

### 업무·분석적 의미
- 

### 한계와 추가 확인 사항
- 

## 5. LLM 구조 설명 검증
- LLM에 제공한 Safe Context: 4개 CSV의 컬럼명·행수·dtype, order_status 값 종류, 분석 목적 등
- LLM이 제안한 추가 점검: 고객 연령에 대한 부분, 결측치 규모 등 
- 실제 데이터에서 확인한 항목: false, age 결측 0, 정합성 이상 없음, quantity & unit price 매출 계산 가능
- 채택/수정/보류한 내용:
  채택: 3)4) 분석 전제로 삼음
  수정: 생년월일 대신 age 컬럼 사용  

![LLM 구조 검토](images/step05_llm.png)

<img width="754" height="748" alt="스크린샷 2026-09-20 오후 2 29 43" src="https://github.com/user-attachments/assets/89063d21-fa1c-45d6-828c-6319530cba90" />


### 나의 해석과 판단
유용: 계산 가능성 사전 확인, 결측치 확인
조심: 생년월일을 물은 것(이건 사실 본인이 자체 변경)

### 한계와 추가 확인 사항
- 

## 6. Chapter 03 최종 판단
### 데이터의 첫인상 3가지
1. 내가 가진 4개의 csv 파일을 프로그램 상에 업로드 하고 형태를 확인할 수 있구나 
2. 분석 전에결측치를 확인해야 하는 구나. 
3. llm을 기반으로 데이터 분석을 1차적으로 할 수 있구나

### 다음 Chapter 전에 반드시 확인/처리해야 할 항목
1. 데이터 형태 확인 
2. 데이터 결측치 확인
3. 데이터 간 연결관계 확인 

### 현재 데이터만으로 단정할 수 없는 것

## 최종 제출 체크
- [ O ] Notebook을 처음부터 끝까지 실행했습니다.
- [ O ] 오류 셀이 남아 있지 않습니다.
- [ 0 ] 핵심 Evidence를 첨부했습니다.
- [ 0 ] 관찰과 해석을 구분했습니다.
- [ 0 ] 개인정보/Secret이 없습니다.
- [ 0 ] `chapter03/chapter03.ipynb`가 GitHub에서 정상 표시됩니다.
- [ 0 ] 최종 Notebook 파일 URL을 제출합니다.
