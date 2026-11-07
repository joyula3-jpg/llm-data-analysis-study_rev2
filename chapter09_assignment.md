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
행이 늘어나도 에러가 나지 않는다. products 나 customers 쪽 키에 중복이 있으면 한 주문이 여러 행으로 복제된다. 그 상태로 Target을 합산하면 금액이 부풀고, 같은 주문이 Train과 Test에 동시에 들어가 성능이 실제보다 좋게 나온다. 누수의 한 형태다.
행이 줄어도 에러가 나지 않는다. how="inner" 로 붙였는데 매칭되지 않는 키가 있으면 그 주문이 조용히 사라진다. 모델링 대상이 줄었는데 알아차릴 방법이 없다.
validate 옵션은 이 조용한 실패를 즉시 멈추는 예외로 바꾼다. 비용은 인자 하나이고, 얻는 것은 실패를 알아차릴 기회다.
line_total 재계산도 같은 이유다. 원본에 들어 있는 값을 그대로 믿으면, 그 값이 다른 기준으로 계산됐을 때(할인 반영 등) Target 정의가 달라진 것을 모른다. Target은 분석의 기준점이라 한 번 틀리면 이후 모든 지표가 틀린다.

---

## 4. Train / Final Test 분할

- Train 시작일: 2025-07-09
- Train 종료일: 2026-04-08
- Train 행 수: 242건 (80.7%)
- Final Test 시작일: 2026-04-10
- Final Test 종료일: 2026-07-08
- Final Test 행 수: 58건 (19.3%)
- 같은 달력 날짜 중복 여부: 없음 (겹치는 날짜 0건)

### 분할 판단
왜 이번 문제에서 시간 순서 분할을 사용하는지 작성하세요.
미래를 예측하는 문제이기 때문이다. 무작위 분할을 하면 2026년 4월 주문이 Train에, 2025년 8월 주문이 Test에 들어간다. 미래 데이터로 학습해 과거를 맞히는 셈이고, 실제 운영에서는 불가능한 조건이다. 성능이 실제보다 좋게 나온다.
날짜 단위로 자른 이유는 같은 날 주문들이 서로 비슷할 수 있어서다. 프로모션이나 이벤트가 있던 날이면 그날 주문이 모두 영향을 받는다. 한 날짜가 양쪽에 걸치면 Train의 그 정보가 Test로 새어 들어간다.
Final Test를 모델 선택이 끝날 때까지 쓰지 않은 것도 같은 맥락이다. Test를 보고 후보를 고르면 Test가 사실상 검증셋이 되어, 보고된 성능이 과대평가된다. 그래서 모델 선택은 Train 구간 안에서 TimeSeriesSplit으로만 했다.

![시간 분할](images/step04_split.png)
<img width="756" height="329" alt="image" src="https://github.com/user-attachments/assets/9d48b82a-b395-4129-a5bb-f74a5729d592" />

---

## 5. Train TimeSeriesSplit 후보 비교

모델	CV MAE 평균	CV MAE 표준편차	CV RMSE 평균	비고
Dummy Baseline (median)	419,065	41,758	511,072	baseline
Linear Regression	457,660	71,649	574,652	변동 폭 최대
Random Forest	440,511	31,907	545,836	비베이스라인 중 최저 MAE

### Train CV로 선택한 비베이스라인 모델
- Selected Model: Random Forest
- 선택 기준: Baseline을 제외한 후보 중 Train CV 평균 MAE 최저 (440,511 < 457,660)
- Final Test 결과를 보기 전에 선택했는가: 예 

### 나의 해석과 판단
평균은 순위를 정하고, 표준편차는 그 순위를 믿어도 되는지 말해준다.
Random Forest(440,511)와 Linear Regression(457,660)의 평균 차이는 17,149다. 그런데 Linear Regression의 표준편차가 71,649로 그 차이의 4배가 넘는다. fold를 어떻게 자르냐에 따라 순위가 뒤집힐 수 있다는 뜻이다. 평균만 보고 “Random Forest가 더 좋다”고 단정하면 안 된다.
표준편차가 크다는 것은 구간에 따라 성능이 흔들린다는 의미이기도 하다. TimeSeriesSplit은 fold가 시간 순서대로 나뉘므로, 변동이 크면 특정 시기에만 잘 맞고 다른 시기에는 못 맞힌다. 운영에 올렸을 때 성능이 들쭉날쭉해진다. Random Forest의 31,907이 Linear Regression의 절반 이하라, 평균이 비슷했더라도 안정성 면에서는 Random Forest를 고를 근거가 된다.
Train 242건을 5-fold로 나누면 fold당 40건 수준이라 변동이 클 수밖에 없다. 표본이 작을수록 표준편차를 함께 봐야 한다.

---

## 6. Final Test: Baseline vs Frozen Model

모델	selection role	Train MAE	Test MAE	Test RMSE	Test R²	Baseline 대비 MAE 개선율
Dummy Baseline (median)	baseline	419,065	445,862	524,336	-0.009	0.0%
Random Forest (Frozen)	selected_by_train_cv	440,511	475,761	552,135	-0.119	-6.7%

![최종 회귀 평가](images/step06_final_metrics.png)
<img width="816" height="411" alt="image" src="https://github.com/user-attachments/assets/31771687-5d4c-4775-8e08-8aaa99a99a96" />

### 결과 관찰
  Frozen Model이 baseline보다 실제로 개선되었는가?
  Baseline MAE 445,862 / Frozen Model MAE 475,761. 고정 모델이 29,899원 더 틀린다.
  MAE 개선율 -6.7%.
  R²는 Baseline -0.009, Frozen Model -0.119로 둘 다 음수다.
  RMSE도 Baseline 524,336 < Frozen 552,135로 같은 순서다.
  Train CV에서도 Baseline(419,065)이 두 후보보다 MAE가 낮았다. Final Test에서 순서가 뒤집히지 않았다.

### 나의 해석과 판단
- Frozen Model이 baseline보다 실제로 개선되었는가
  개선되지 않았다. MAE·RMSE·R² 세 지표 모두 Baseline이 낫다. Train CV에서 이미 Baseline이 앞섰고 Final Test에서도 동일한 결과가 나왔으므로, 우연한 결과가 아니라 일관된 패턴이다.

- MAE와 RMSE 차이에서 무엇을 볼 수 있는가?
  Frozen Model의 RMSE(552,135)가 MAE(475,761)보다 76,374 크다. RMSE는 오차를 제곱해 평균 내므로 큰 오차에 더 민감하다. 둘의 격차가 크다는 것은 전반적으로 조금씩 틀리는 게 아니라 크게 빗나가는 주문이 일부 있다는 뜻이다.
  Baseline도 격차가 78,474로 비슷하다. 특정 모델의 문제가 아니라 order_total 자체의 분산이 크기 때문으로 본다. 주문 금액이 1개 품목부터 4개 품목까지 섞여 있어 범위가 넓다.
  
- R²는 어떻게 해석할 수 있는가?
  R² -0.119는 테스트 구간의 실제 평균으로 예측하는 것보다 못하다는 뜻이다. 0이면 평균 예측과 같고, 음수면 그보다 나쁘다.
  모델이 feature에서 뭔가 패턴을 찾았다고 믿고 예측했는데, 그 패턴이 Test 구간에서는 통하지 않았다. 결제수단·요일·고객 속성이 주문 금액과 거의 무관하다는 신호다.
  Baseline의 -0.009도 음수인데, Train 중앙값을 Test에 적용했기 때문이다. Test 구간 자체의 평균과 미세하게 달라 0에서 조금 내려갔다.

### 업무·분석적 의미
현재 오차 수준이 실제 사용에서 어떤 의미인지 작성하세요.
평균 주문 금액이 85만 원 수준인데 평균 오차가 47만 원이다. 상대 오차로 보면 55% 수준이다.
실제로 이 모델을 쓴다면 85만 원짜리 주문을 38만 원이나 132만 원으로 예측하는 일이 흔하다는 뜻이다. 재고 준비, 물류 용량 산정, 매출 전망 어디에도 쓸 수 없다. Baseline보다 못하므로 "그냥 중앙값을 쓰는 것"이 더 나은 선택이다.
RMSE가 55만 원이라는 점도 중요하다. 큰 오차가 섞여 있어, 평균적으로는 그럭저럭 맞는 것처럼 보여도 개별 주문에서는 크게 빗나가는 경우가 생긴다. 주문 단위 의사결정에는 더 쓰기 어렵다.
결론은 모델 사용 보류다. 다만 이것은 모델링이 실패했다는 뜻이 아니라, 예측 시점에 허용된 정보가 부족하다는 진단이다. 주문 금액은 무엇을 몇 개 샀는지로 정해지는데 그 정보를 전부 뺐으니 당연한 결과다.
다음에 할 일은 알고리즘을 바꾸는 것이 아니라 예측 시점에 쓸 수 있으면서 아직 안 쓴 정보를 찾는 것이다. 고객의 직전 주문 금액이나 누적 주문 횟수 같은 과거 이력이 후보인데, 주문 시점 이전 데이터만 써야 하므로 시간 기준 집계를 별도로 설계해야 한다.

---

## 7. 대표 오차 사례

### 사례 1 - 최대 과소예측
- 실제값:1,877,000원
- 예측값:542,918원
- residual:+1,334,082원
- absolute error:1,334,082원 (58건 중 최대)
- 관찰:
  주문일 2026-06-15, kakao_pay, 32세 여성, 서울, 평일. feature상 특별할 것이 없는 조합인데 실제 금액은 Test 구간 상위권이다. 모델은 평범한 조건이라 판단해 54만 원대를 예측했다.
- 가능한 이유 후보:
  허용 feature 어디에도 "이 주문이 크다"는 신호가 없다. 결제수단·요일·고객 속성이 주문 규모와 무관하다면 모델은 평균 근처를 예측할 수밖에 없다. 금액을 결정하는 정보가 입력에 없는 상태로 본다.
- 추가 확인:
  내부적으로 확인하니 이 주문은 품목 3개, 총수량 13개였다. 품목 수별 평균 금액은 1개 368,032원 / 2개 695,965원 / 3개 976,714원 / 4개 1,307,467원으로, 품목 수가 금액을 거의 선형으로 결정한다. 다만 품목 수는 주문 상세가 확정된 뒤에야 알 수 있어 현재 예측 시점에서는 쓸 수 없다. 장바구니 담기 시점에 품목 수를 관측할 수 있는지가 다음 확인 대상이다.

### 사례 2 - 최소 오차
- 실제값:558,000원
- 예측값:585,616원
- residual:-27,616원
- absolute error:27,616원 (58건 중 최소)
- 관찰:주문일 2026-05-05. 예측값이 Test 구간 평균(856,172원)보다 낮은 58만 원대이고 실제도 그 근처였다.
- 가능한 이유 후보:
  모델이 이 주문의 특성을 포착했다고 보기 어렵다. 58건 중 하나는 우연히 맞을 수 있다. 예측값 585,616원은 품목 2개 주문의 평균 695,965원과 1개 주문 368,032원 사이이고, 실제 이 주문은 품목 2개였다. 우연히 가까웠을 가능성이 크다.
- 추가 확인:
  작은 오차가 모델의 능력인지 확인하려면 비슷한 feature 조합의 다른 주문들도 함께 잘 맞는지 봐야 한다. 단일 사례로는 판단할 수 없다. 예측값의 분산을 보는 것도 방법이다. 모델이 대부분 비슷한 값을 뱉고 있다면, 실제가 그 근처인 주문만 맞는 셈이다.

> 원인을 데이터가 직접 보여 주지 않는다면 사실처럼 단정하지 않습니다.

---

## 8. Validation Evidence

`reports/ch09_regression_validation.csv` 기준으로 작성하세요.
<img width="1287" height="521" alt="image" src="https://github.com/user-attachments/assets/87481fa6-5891-4ec3-afd1-d825c5629040" />


### FAIL 항목
- 없음
- 있다면 원인과 수정 내용: 9개 게이트 전부 PASS

---

## 9. Evidence와 공개 범위

### 생성된 Evidence
- [o] `ch09_regression_split_summary.csv`
- [o] `ch09_regression_feature_audit.csv`
- [o] `ch09_regression_cv_summary.csv`
- [o] `ch09_regression_model_comparison.csv`
- [o] `ch09_regression_validation.csv`
- [o] `ch09_regression_checklist.csv`

### Internal 파일
- [o] `ch09_regression_model_data_internal.csv`
- [o] `ch09_regression_predictions_internal.csv`

### 공개 결과
- [o] `ch09_regression_report.md`
- [o] `ch09_actual_vs_predicted.png`
- [o] `ch09_residual_histogram.png`

### 공개 범위 판단
왜 `order_id`가 포함된 예측 결과를 Internal로 구분해야 하는지 작성하세요.
order_id 만 있으면 orders 에서 customer_id 를, 거기서 고객 정보를 찾을 수 있다. 금액만 공개한 줄 알았는데 누가 얼마를 썼는지가 복원된다.
예측 결과는 한 행이 한 주문이라 개별 금액이 그대로 노출된다. 집계표와 달리 여러 건에 섞여 가려지지 않는다.
그런데 잔차 진단에는 order_id 가 필요하다. 크게 빗나간 주문을 원본에서 확인해야 원인을 좁힐 수 있기 때문이다. 그래서 필요한 곳에서는 쓰되 공개 경로에서만 제거하도록 reports/internal/ 로 분리했다. 공개용 CSV는 order_id 대신 오차 순위만 남겨, 분포는 보이되 개별 주문은 특정할 수 없다.

---

## 10. 최종 사용 판단

- [-] 현재 수준에서 참고용 사용 가능
- [-] 추가 검증 후 사용 가능
- [o] 현재 데이터로는 사용 보류

### 판단 근거
1.Baseline보다 못하다. Final Test MAE 475,761원 vs Baseline 445,862원. 29,899원 더 틀리고 개선율은 -6.7%다.
2.R²가 음수다. -0.119로 평균 예측보다 설명력이 낮다. Train CV에서도 Baseline이 가장 낮은 MAE(419,065)를 기록해, 우연이 아니라 일관된 결과다.
3.오차가 금액 수준에 비해 크다. 평균 주문 금액 856,172원에 평균 오차 475,761원으로 상대 오차 55%다. RMSE(552,135)가 MAE보다 76,374 커서 큰 금액에서 특히 빗나가며, 최대 오차는 1,334,082원이다.

### 업무적 위험
85만 원 주문을 38만 원이나 132만 원으로 예측한다. 재고나 물류 계획에 쓰면 결품과 과잉 준비가 함께 발생한다.
더 큰 위험은 모델이 존재한다는 사실 자체다. Baseline보다 못한 모델을 배포하면 중앙값을 쓰는 것보다 나쁜 결정을 하면서도 "데이터 기반"이라고 믿게 된다.

### 현재 데이터의 한계
예측 시점에 허용된 정보가 금액과 거의 무관하다. 주문 금액은 무엇을 몇 개 샀는지로 정해지는데 결제수단·요일·고객 속성만 남았다.
Final Test 58건으로 지표 변동 폭이 크다.
order_total 에 할인·배송비·세금이 반영되지 않았다.
1년치라 계절성을 학습할 반복 구간이 없다.
고객 과거 이력, 장바구니 정보, 유입 경로가 데이터에 없다.

### 다음 개선 우선순위
1.고객 과거 주문 이력 feature 추가. 직전 주문 금액, 누적 주문 횟수, 과거 평균. 현재 데이터로 만들 수 있고 예측 시점에도 알 수 있다. 단 주문 시점 이전 데이터만 써야 하므로 시간 기준 집계 설계가 필요하다.
2.장바구니 담기 시점 정보 확보 검토. 품목 수가 금액을 거의 선형으로 결정한다(1개 368,032원 → 4개 1,307,467원). 결제 직전에 관측 가능하면 예측 시점을 조정할 수 있다.
3.표본 확대. 2~3년치로 계절성을 학습하고 Final Test 규모를 키워 지표 변동을 줄인다.

---

## 11. 전체 재실행 확인

실행 명령:

```powershell
python scripts/run_regression_analysis.py
```

- 실행 성공 여부: 성공
- Train CV Selected Model:
  Random Forest (CV 평균 MAE 440,511 < Linear Regression 457,660)
- Final Validation 전체 PASS 여부:9개 게이트 전부 PASS
- Notebook 결과와 스크립트 결과의 일관성:
<img width="697" height="369" alt="image" src="https://github.com/user-attachments/assets/c7909e46-b578-40ba-8456-cdac7969d2be" />
<img width="594" height="288" alt="image" src="https://github.com/user-attachments/assets/d70dd1c6-d1ea-4919-97a8-d7dc4e70a4d7" />

---

## 최종 체크

- [o] Target과 예측 시점을 정의했습니다.
- [o] Leakage Audit을 수행했습니다.
- [o] Target 계산과 관계 검증을 확인했습니다.
- [o] Train과 Final Test가 시간 순서로 엄격히 분리되었습니다.
- [o] 전처리가 Pipeline 내부에서 학습됩니다.
- [o] Train TimeSeriesSplit으로 후보를 비교했습니다.
- [o] Final Test 전에 Selected Model을 고정했습니다.
- [o] Final Test에서는 Baseline과 Frozen Model만 비교했습니다.
- [o] MAE, RMSE, R²를 함께 해석했습니다.
- [o] 대표 오차 사례를 관찰과 가설로 구분했습니다.
- [o] Validation Evidence가 모두 PASS입니다.
- [o] 최종 Notebook URL을 제출합니다.o
- [o] Internal 결과와 공개 결과를 구분했습니다.
- [o] 낮은 성능도 숨기지 않았습니다.
- [o] 전체 스크립트를 재실행했습니다.
- [o] 최종 Notebook URL을 제출합니다.
