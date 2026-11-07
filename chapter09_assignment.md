# Chapter 09 답안 양식. 회귀 분석으로 숫자 예측하기

> 이 내용을 `chapter09.ipynb`의 Markdown 셀로 작성합니다.

## 제출 정보
- 이름:조유라  
- GitHub ID: joyula3  
- 작성일: 11월 8일
- 최종 Notebook URL: https://github.com/joyula3-jpg/llm-data-analysis-study_rev2/edit/main/chapter09_assignment.md

## 1. 예측 문제 정의
- Target:
  주문 1건의 주문 상세 금액 합계.
  order_items 를 order_id 로 묶어 sum(quantity × unit_price) 로 만든 값이다. 생성 과정에서 line_total 재계산 불일치 0건을 확인했고, 주문과 Target은 validate="one_to_one" 으로 1:1 관계임을 강제했다.
- 예측 단위:
  주문 1건(order_id 1개)이 한 행이다. 총 300행.
  품목 단위(764행)가 아니라 주문 단위로 올린 이유는 Target이 주문 금액이기 때문이다. 품목 단위로 두면 같은 주문이 여러 행으로 나뉘어 같은 주문의 정보가 Train과 Test에 동시에 들어갈 수 있다.
- 예측 시점:
  주문이 생성되는 순간, 주문 상세가 확정되기 전.
  구체적으로는 "이 고객이 이 결제수단으로 오늘 주문을 넣었다"까지는 알지만, 무엇을 몇 개 담았는지는 모르는 시점이다. 그 시점에 주문 금액이 얼마가 될지 맞히는 문제로 정의했다.
- 예측 시점에 알 수 있는 정보:
  <img width="759" height="405" alt="image" src="https://github.com/user-attachments/assets/c7dca3b9-db03-4335-9392-614117ffe1cb" />

  
- 예측 시점에 알 수 없는 정보:
  목표 계산 재료 — 이걸 넣으면 Target을 그대로 복원한다.
  quantity, unit_price, line_total, order_total, item_count, total_quantity
  사후 정보 — 주문 시점 이후에 결정된다.
  order_status
  completed / cancelled / refunded는 주문이 들어온 뒤에 정해진다. 예측 시점에는 이 주문이 완료될지 취소될지 알 수 없다. 컬럼이 존재하고 넣으면 성능도 올라가지만, 현실에서는 쓸 수 없는 정보다. 이번 설계에서 가장 판단이 필요했던 항목이다.
  식별자 — 예측에 쓰면 특정 주문·고객을 외우는 것이 된다.
  order_id, customer_id, product_id, order_item_id
  개인 식별 정보 — 모델 입력에 들어가면 안 된다.
  name, customer_name, email, phone, address

### 나의 해석과 판단
왜 이 예측 시점을 선택했는지 작성하세요.
주문 상세가 확정된 뒤에는 quantity × unit_price 를 더하면 금액이 나온다. 예측이 쓸모 있는 시점은 아직 금액이 결정되지 않은 주문 생성 직후뿐이다.
시점을 느슨하게 잡을수록 쓸 수 있는 정보가 늘고 점수도 오르지만, 그건 답을 알려주고 답을 맞히게 하는 것이다. 점수를 기준으로 시점을 고르면 누수로 끌려간다. 그래서 시점을 먼저 고정하고 feature는 거기서 자동으로 정해지게 했다.
order_status 를 금지한 것이 이 판단의 핵심이다. 컬럼은 존재하고 넣으면 성능이 오를 수 있지만, 주문이 들어오는 순간에는 완료될지 취소될지 알 수 없다. 컬럼이 있다는 것과 예측 시점에 쓸 수 있다는 것은 다르다.
결과적으로 Final Test에서 R² -0.119, Baseline 대비 MAE -6.7%로 개선이 없었다. 주문 금액을 결정하는 정보를 전부 뺐으니 설명력이 낮은 것이 구조적으로 당연하다. 오히려 성능이 좋았다면 누수를 의심했어야 한다.

---

## 2. Feature Leakage Audit

<img width="746" height="628" alt="image" src="https://github.com/user-attachments/assets/04cf650f-0cc3-4be8-b371-3bd9a2c433b4" />



- 금지 feature overlap 개수: 0개
- 가장 위험하다고 판단한 feature: order_status
- 이유:데이터에 존재하고 성능도 적당히 올라 의심하기 어렵지만, 주문 시점에는 완료·취소 여부를 알 수 없는 사후 정보다. 운영에 올리면 비어 있는 컬럼이다.

![Leakage 점검](images/step02_leakage.png)
<img width="1367" height="378" alt="image" src="https://github.com/user-attachments/assets/6345497d-2385-41a1-bc14-4ac6191bc5fd" />

---

## 3. Target 생성과 관계 검증

### 계산 기준

```text
line_total  = quantity × unit_price
order_total = order_items.groupby("order_id")["line_total"].sum()
```

- line_total 검증 결과:
  재계산 불일치 0건. build_regression_dataset() 안에서 quantity × unit_price 로 다시 계산해 기존 line_total 과 대조했고, 1건이라도 다르면 ValueError 로 중단하도록 했다.
- orders ↔ target 관계 검증 결과:
  validate="one_to_one" 통과. 주문 300건과 Target 300건이 1:1로 매칭됐다. how="inner" 를 썼고 모델링 행 수가 300으로 유지됐다.
- orders → customers 관계 검증 결과:
  validate="many_to_one" 통과. 병합 전후 행 수가 300으로 동일함을 별도로 확인했고, 달라지면 ValueError 로 중단한다.

### 나의 해석과 판단
관계 검증 없이 병합만 수행하면 어떤 문제가 생길 수 있는지 작성하세요.

---

## 4. Train / Final Test 분할

- Train 시작일:
- Train 종료일:
- Train 행 수:
- Final Test 시작일:
- Final Test 종료일:
- Final Test 행 수:
- 같은 달력 날짜 중복 여부:

### 분할 판단
왜 이번 문제에서 시간 순서 분할을 사용하는지 작성하세요.

![시간 분할](images/step04_split.png)

---

## 5. Train TimeSeriesSplit 후보 비교

| 모델 | CV MAE 평균 | CV MAE 표준편차 | CV R² 평균 | 비고 |
| --- | ---: | ---: | ---: | --- |
| Baseline Mean | | | | |
| Linear Regression | | | | |
| Random Forest | | | | |

### Train CV로 선택한 비베이스라인 모델
- Selected Model:
- 선택 기준:
- Final Test 결과를 보기 전에 선택했는가: 예 / 아니오

### 나의 해석과 판단
CV 평균뿐 아니라 표준편차도 함께 봐야 하는 이유를 작성하세요.

---

## 6. Final Test: Baseline vs Frozen Model

| 모델 | selection role | Train MAE | Test MAE | Test RMSE | Test R² | Baseline 대비 MAE 개선율 |
| --- | --- | ---: | ---: | ---: | ---: | ---: |
| Baseline Mean | baseline | | | | | |
| Frozen Selected Model | selected_by_train_cv | | | | | |

![최종 회귀 평가](images/step06_final_metrics.png)

### 결과 관찰

### 나의 해석과 판단
- Frozen Model이 baseline보다 실제로 개선되었는가?
- MAE와 RMSE 차이에서 무엇을 볼 수 있는가?
- R²는 어떻게 해석할 수 있는가?

### 업무·분석적 의미
현재 오차 수준이 실제 사용에서 어떤 의미인지 작성하세요.

---

## 7. 대표 오차 사례

### 사례 1
- 실제값:
- 예측값:
- residual:
- absolute error:
- 관찰:
- 가능한 이유 후보:
- 추가 확인:

### 사례 2
- 실제값:
- 예측값:
- residual:
- absolute error:
- 관찰:
- 가능한 이유 후보:
- 추가 확인:

> 원인을 데이터가 직접 보여 주지 않는다면 사실처럼 단정하지 않습니다.

---

## 8. Validation Evidence

`reports/ch09_regression_validation.csv` 기준으로 작성하세요.

| check | value | status |
| --- | --- | --- |
| forbidden_feature_overlap | | |
| strict_train_before_test | | |
| selected_model_exists_in_train_cv | | |
| final_test_contains_baseline | | |
| final_test_contains_frozen_selected_model | | |
| test_rows_for_r2 | | |

### FAIL 항목
- 없음 / 있음:
- 있다면 원인과 수정 내용:

---

## 9. Evidence와 공개 범위

### 생성된 Evidence
- [ ] `ch09_regression_split_summary.csv`
- [ ] `ch09_regression_feature_audit.csv`
- [ ] `ch09_regression_cv_summary.csv`
- [ ] `ch09_regression_model_comparison.csv`
- [ ] `ch09_regression_validation.csv`
- [ ] `ch09_regression_checklist.csv`

### Internal 파일
- [ ] `ch09_regression_model_data_internal.csv`
- [ ] `ch09_regression_predictions_internal.csv`

### 공개 결과
- [ ] `ch09_regression_report.md`
- [ ] `ch09_actual_vs_predicted.png`
- [ ] `ch09_residual_histogram.png`

### 공개 범위 판단
왜 `order_id`가 포함된 예측 결과를 Internal로 구분해야 하는지 작성하세요.

---

## 10. 최종 사용 판단

- [ ] 현재 수준에서 참고용 사용 가능
- [ ] 추가 검증 후 사용 가능
- [ ] 현재 데이터로는 사용 보류

### 판단 근거
1.
2.
3.

### 업무적 위험

### 현재 데이터의 한계

### 다음 개선 우선순위
1.
2.
3.

---

## 11. 전체 재실행 확인

실행 명령:

```powershell
python scripts/run_regression_analysis.py
```

- 실행 성공 여부:
- Train CV Selected Model:
- Final Validation 전체 PASS 여부:
- Notebook 결과와 스크립트 결과의 일관성:

---

## 최종 체크

- [ ] Target과 예측 시점을 정의했습니다.
- [ ] Leakage Audit을 수행했습니다.
- [ ] Target 계산과 관계 검증을 확인했습니다.
- [ ] Train과 Final Test가 시간 순서로 엄격히 분리되었습니다.
- [ ] 전처리가 Pipeline 내부에서 학습됩니다.
- [ ] Train TimeSeriesSplit으로 후보를 비교했습니다.
- [ ] Final Test 전에 Selected Model을 고정했습니다.
- [ ] Final Test에서는 Baseline과 Frozen Model만 비교했습니다.
- [ ] MAE, RMSE, R²를 함께 해석했습니다.
- [ ] 대표 오차 사례를 관찰과 가설로 구분했습니다.
- [ ] Validation Evidence가 모두 PASS입니다.
- [ ] Internal 결과와 공개 결과를 구분했습니다.
- [ ] 낮은 성능도 숨기지 않았습니다.
- [ ] 전체 스크립트를 재실행했습니다.
- [ ] 최종 Notebook URL을 제출합니다.
