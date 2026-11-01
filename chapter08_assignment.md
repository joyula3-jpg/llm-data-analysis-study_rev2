# Chapter 08 답안 양식. 작은 데이터 분석 프로젝트 완성하기

> 이 내용을 `chapter08.ipynb`의 Markdown 셀로 작성합니다.

## 제출 정보
- 이름: 조유라 
- GitHub ID: joyula3
- 작성일:11월 1일
- 최종 Notebook URL: https://github.com/joyula3-jpg/llm-data-analysis-study_rev2/edit/main/chapter08_assignment.md

## 1. 프로젝트 질문
### 분석 질문
- completed 주문 금액은 카테고리별로 얼마나 다르며, 금액 순위와 판매 수량 순위가 일치하는가?
- 월별 completed 주문 금액은 어떻게 변해 왔으며, 변동이 주문 건수 때문인가 객단가 때문인가?
- completed 주문 금액은 상위 고객에 얼마나 집중되어 있는가?

### 분석 범위
<img width="758" height="259" alt="image" src="https://github.com/user-attachments/assets/68153051-c80f-406f-8595-0de686aa9a57" />


### 사용할 데이터와 지표
<img width="758" height="259" alt="image" src="https://github.com/user-attachments/assets/32ba4b67-a7b0-4a5f-ba50-b9fa89bc7eaf" />
<img width="748" height="374" alt="image" src="https://github.com/user-attachments/assets/454a6a87-6095-4f6c-8f95-eb70befee638" />



### 계산 기준
- `order_status == "completed"` 적용 여부: 적용함
금액성 집계 3개(카테고리·월별·고객별)에 모두 적용한다.
적용 시점은 조인 이후, 집계 이전이다. 조인 전에 거르면 FK 검증 결과가 달라진다.
주문 상태 분포 그래프는 예외로 전체 주문을 쓰며, 그래프 제목에 "전체 주문 기준"을 명시한다.
completed 148,990,000원 / 전체 255,610,000원. 필터 하나로 106,620,000원(41.7%)이 제외되므로, 모든 결과 표기에 범위를 함께 적는다.

- `line_total = quantity × unit_price` 확인 여부: 확인함
order_items에 금액 컬럼이 없어 파생변수로 생성한다.
생성 후 line_total == quantity × unit_price로 재계산해 불일치 건수를 센다. 기대값 0건.
line_total > 0 도 함께 확인한다. 기대값 위반 0건.
unit_price가 products.price와 764건 전부 일치한다. 할인·쿠폰이 반영되지 않은 정가 기준이므로, line_total은 내부 비교 지표로만 쓰고 회계상 순매출과 동일시하지 않는다.

### 완료 기준
<img width="756" height="410" alt="image" src="https://github.com/user-attachments/assets/f5282996-55a2-4964-a30f-a400697a4e2b" />


## 2. 입력 데이터와 전처리 검증
- 사용 원본 파일:
   <img width="755" height="208" alt="image" src="https://github.com/user-attachments/assets/0ed6ce3a-0323-45c8-bdbe-6c46ada9d6dd" />

- shape:
  <img width="744" height="231" alt="image" src="https://github.com/user-attachments/assets/c56ddef8-d882-44e0-a19e-6cadea1edf1e" />

- 핵심 결측/중복 결과:
  <img width="749" height="207" alt="image" src="https://github.com/user-attachments/assets/30cbec74-7306-46f6-9b58-daf54402d9ad" />

- 전처리 전후 변화:
  <img width="750" height="211" alt="image" src="https://github.com/user-attachments/assets/9eb6db67-7c18-4213-8466-4936123053ec" />


![입력 데이터 검증](images/step02_data_validation.png)
<img width="861" height="854" alt="image" src="https://github.com/user-attachments/assets/3be5d42b-d00a-4f00-aae9-901b3d1e8084" />
<img width="811" height="112" alt="image" src="https://github.com/user-attachments/assets/da3d118d-5faa-4736-92de-ea821d2284eb" />

### 나의 해석과 판단
행 수가 변하지 않은 것을 의도한 결과로 판단했다. 근거는 전처리 코드의 삭제 지점을 사전에 대조한 것이다. 행을 줄일 수 있는 곳은 각 전처리 함수 끝의 drop_duplicates() 뿐인데, 전체 행 중복이 애초에 0건이므로 지워질 행이 없다. 예측과 결과가 일치했다.
만약 행이 줄었다면 몇 건이 어느 조건에 걸렸는지 특정할 수 있어야 한다. 설명할 수 없는 감소는 버그로 취급해야 한다. 반대로 행이 늘었다면 무조건 버그다. 전처리는 행을 늘리는 작업이 아니기 때문이다.
이번 전처리의 실질적 성과는 타입 변환이다. 결측도 중복도 손댈 것이 없었고, 날짜 두 컬럼을 datetime64 로 바꾼 것이 유일하게 의미 있는 변화다. 비교표의 네 항목(행·열·결측)으로는 이 변화가 포착되지 않으므로, 보고서에 별도로 적어야 한다.
"결측 0, 중복 0"을 데이터 품질이 좋다는 근거로 쓰지 않기로 했다. 형식 검증은 전부 통과했지만, 날짜를 변환한 뒤 두 컬럼을 비교하면 주문일이 가입일보다 앞선 행이 56건(전체 주문의 18.7%) 나온다. 논리적으로 불가능한 레코드인데 결측도 중복도 아니라서 이 표에 전혀 나타나지 않는다.
이 건들을 삭제하지 않기로 판단했다. 전체의 19%라 지우면 표본이 크게 줄고, 어느 날짜가 틀렸는지 판단할 근거도 없다. 다만 이번 프로젝트의 금액성 분석은 order_date 만 사용하고 signup_date 를 쓰지 않으므로, 결과에 영향을 주지 않는다. 가입일을 사용하는 분석(코호트, 생애주기)으로 확장할 때만 문제가 된다는 점을 한계로 명시한다.
FK 역방향의 24건은 오류가 아니다. 주문 이력이 없는 고객이며, 가입만 하고 구매하지 않은 정상 상태다. 다만 고객당 평균 구매액을 낼 때 분모를 150으로 할지 126으로 할지, 또는 completed 구매 고객 100명으로 할지 먼저 정해야 한다.

### 한계와 추가 확인 사항
- 이 데이터 자체의 한계
결측이 0건인 것이 오히려 비현실적이다. 실제 운영 데이터라면 선택 입력 항목에 결측이 있는 것이 정상이다. 이미 정제됐거나 생성 데이터일 가능성이 높다.
IQR 기준 이상값이 네 수치형 컬럼 모두 0건이다. 가격 5,000~200,000원, 수량 1~5로 범위가 딱 떨어진다. 균등분포로 생성된 흔적이며, 여기서 얻은 결론을 실제 현장에 적용할 수 없다.
unit_price 가 products.price 와 764건 전부 일치한다. 할인이 반영되지 않아 금액 해석에 제약이 있다.
100개 상품이 전부 최소 1회 팔렸다. 실제 쇼핑몰에서는 드문 일이다.

- 검증 자체의 한계
삭제 로직이 실제로 작동하는지 확인되지 않았다. 걸러진 행이 0건이라 코드가 의도대로 동작하는지 알 수 없다. 일부러 이상값을 넣은 테스트 데이터로 한 번 돌려봐야 한다.
문자열 정리가 앞뒤 공백만 다룬다. 중간 공백, 대소문자, 전각/반각은 확인하지 않았다.
값 수준의 일관성을 보지 않았다. gender, city 의 고유값은 확인했으나 표기 흔들림의 가능성은 데이터가 갱신되면 다시 생긴다.
전처리 순서에 따른 차이를 확인하지 않았다. drop_duplicates() 를 타입 변환 전에 하느냐 후에 하느냐에 따라 결과가 달라질 수 있다. 문자열 '1' 과 숫자 1 은 변환 전에는 다르지만 변환 후에는 같다.


## 3. PK/FK·병합 검증
### PK 결과
- 결측:
- 중복:
- PASS/FAIL:

### FK 결과
- 미매칭:
- PASS/FAIL:

### 병합 결과
- validate 관계:
- 병합 전 행 수:
- 병합 후 행 수:
- 미매칭:
- PASS/FAIL:

### 나의 해석과 판단

## 4. 핵심 EDA 결과
### 결과 1
- 수치/표:
- 결과 관찰:
- 나의 해석과 판단:
- 업무·분석적 의미:
- 한계:

### 결과 2
- 수치/표:
- 결과 관찰:
- 나의 해석과 판단:
- 업무·분석적 의미:
- 한계:

### 결과 3
- 수치/표:
- 결과 관찰:
- 나의 해석과 판단:
- 업무·분석적 의미:
- 한계:

## 5. Total consistency와 날짜 검증
- completed source total:
- category total:
- monthly total:
- customer total:
- 총합 일치 여부:
- completed 주문 날짜 오류 건수:
- 날짜 오류 영향 금액:

### 불일치가 있었다면 원인

## 6. 대표 시각화
![대표 그래프 1](images/graph01.png)
![대표 그래프 2](images/graph02.png)

### 그래프 선택 이유

### 그래프와 원본 집계값 일치 여부

### 그래프에서 직접 관찰한 사실

### 그래프만으로 말할 수 없는 것

## 7. 개인정보 검증
- 공개 고객 CSV 컬럼:
- 원본 `customer_id` 포함 여부:
- 이름/이메일/전화번호/주소 포함 여부:
- 익명 라벨 방식:
- PASS/FAIL:

### 나의 판단

## 8. 최종 Validation
`reports/ch08_project_validation.csv`의 결과를 요약합니다.

| 검증 항목 | 결과 | 내가 확인한 근거 |
| --- | --- | --- |
| pk_integrity |  |  |
| fk_integrity |  |  |
| merge_checks_pass |  |  |
| line_total_consistency |  |  |
| completed_total_consistency |  |  |
| category_sales_ratio_pct_sum |  |  |
| completed_rows_with_invalid_order_date |  |  |
| public_customer_columns_safe |  |  |

### FAIL이 있었다면 수정 내용

## 9. LLM 활용 기록
- 사용 여부:
- 사용 목적:
- Safe Context:
- Prompt 요약:
- 제안 요약:
- 반영/수정/보류:
- 사람이 검증한 근거:

## 10. 프로젝트 재현 확인
- `python scripts/run_midterm_project.py` 실행 여부:
- 재실행 결과:
- Notebook과 핵심 수치 일치 여부:
- 생성된 핵심 Evidence 파일 확인 여부:

![재실행 결과](images/step06_reproduce.png)

### 나의 해석과 판단
재실행 가능한 프로젝트가 왜 중요한지 작성하세요.

## 11. 최종 프로젝트 요약
### 핵심 인사이트 3개
1.
2.
3.

### 가장 중요한 업무·분석적 의미

### 현재 분석의 한계

### 다음 분석 제안
1.
2.
3.

## 최종 체크
- [ ] 질문과 지표가 연결됩니다.
- [ ] 원본 데이터에서 다시 시작했습니다.
- [ ] PK/FK·병합 검증 근거가 있습니다.
- [ ] `line_total` 계산 관계를 확인했습니다.
- [ ] 금액성 분석은 completed 범위를 사용했습니다.
- [ ] category/month/customer 총합이 source total과 일치합니다.
- [ ] 날짜 오류를 확인했습니다.
- [ ] 공개 고객 결과에서 원본 ID와 직접 식별정보를 제거했습니다.
- [ ] 대표 그래프와 원본 집계값을 비교했습니다.
- [ ] 원인 과대 해석이 없습니다.
- [ ] 최종 Validation이 모두 PASS입니다.
- [ ] 전체 프로젝트를 재실행했습니다.
- [ ] 한계와 다음 분석을 작성했습니다.
- [ ] 최종 Notebook URL을 제출합니다.
