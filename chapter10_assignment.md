# Chapter 10 답안 양식. 분류 분석으로 주문 취소 여부 예측하기

> 이 내용을 `chapter10.ipynb`의 Markdown 셀로 작성합니다. 실제 실행 결과를 근거로 작성하고, 예측 패턴을 원인처럼 단정하지 않습니다.

## 제출 정보
- 이름: 조유라
- GitHub ID: joyula3-jpg
- 작성일: 11월 15일(일)
- 최종 Notebook URL: https://github.com/joyula3-jpg/llm-data-analysis-study_rev2/edit/main/chapter10_assignment.md
 
## 1. 분류 문제 정의
- 타깃: is_cancelled (이진)
- 0의 의미: completed — 정상 완료된 주문 (184건)
- 1의 의미: cancelled — 취소된 주문 (64건)
- 제외한 상태: refunded 52건 — 배송·결제가 완료된 뒤 환불된 건이므로 '취소'와 발생 시점·의미가 다름
- 예측 단위: 주문 1건 (order_id, 총 248행)
- 예측 시점: 주문이 접수·결제 확정된 직후, 취소/완료 결과가 확정되기 전
- 예측 시점에 사용할 수 있는 정보: 주문 구성(item_count, total_quantity, order_amount, average_unit_price, max_unit_price), 주문일 파생(order_month_num, order_day_of_week, order_day_of_month, is_weekend), 결제수단(payment_method), 고객 비식별 속성(customer_age, customer_gender, customer_city) — 총 13개
  
- 예측 이후에만 알 수 있어 제외한 정보: order_status(타깃 원본), is_cancelled(타깃 자신), 그리고 식별자·개인정보(order_id, customer_id, product_id, name)

### 나의 해석과 판단
왜 이 타깃 범위와 예측 시점을 사용했는지 작성하세요.
refunded 52건을 제외한 이유는 '취소'와 '환불'이 서로 다른 사건이기 때문입니다. 취소는 주문 접수 후 배송 전에 발생하고, 환불은 배송·수령이 끝난 뒤에 발생합니다. 두 상태를 하나의 1로 묶으면 모델이 무엇을 학습했는지 설명할 수 없고, 실제 대응 시점도 달라 활용할 수 없습니다. 그래서 completed=0, cancelled=1로 범위를 좁혔습니다.

예측 시점을 '주문 접수·결제 확정 직후'로 고정한 이유는 이 모델의 용도가 선제 대응(결제 확인 연락, 재고 홀딩 보류)에 있기 때문입니다. 시점을 뒤로 미루면 order_status 같은 사후 정보가 자연스럽게 feature에 들어와 성능 지표는 올라가지만, 운영에서는 이미 결과를 아는 시점이라 쓸 수 없는 모델이 됩니다.

같은 기준으로 가입 경과일 파생변수는 쓰지 않았습니다. 248건 중 45건(18.1%)이 order_date < signup_date여서 가입일 기준 계산을 신뢰할 수 없다고 판단했습니다.

---

## 2. Feature Contract와 Leakage Audit

### 사용한 숫자형 Feature
item_count            주문에 포함된 상세 품목 수
total_quantity        주문 총 수량
order_amount          주문 금액 합계
average_unit_price    품목 평균 단가
max_unit_price        품목 최고 단가
customer_age          고객 나이
order_month_num       주문 월
order_day_of_week     주문 요일(0=월)
order_day_of_month    주문 일자

### 사용한 범주형 Feature
payment_method        card / naver_pay / kakao_pay / bank_transfer
customer_gender       F / M
customer_city         10개 지역
is_weekend            weekday / weekend

### 금지 Feature
order_status          타깃 원본 (완전 누수)
is_cancelled          타깃 자신
order_id              식별자
customer_id           식별자
product_id            식별자
name                  개인 식별 정보
가입 경과일 계열        order_date < signup_date 45건(18.1%)으로 계산 불가
취소 이후 정보          환불일, 취소사유, CS 접수 이력 등

### 나의 해석과 판단
파일에 컬럼이 존재하는 것과 예측 시점에 사용할 수 있는 것이 왜 다른지 작성하세요.
CSV에 컬럼이 있다는 것은 "데이터가 수집되어 저장되었다"는 사실일 뿐이고, 그 컬럼이 언제 채워졌는지는 말해주지 않습니다. orders.csv의 한 행은 주문이 접수될 때 만들어지지만 order_status는 며칠 뒤 결과가 확정된 다음에 갱신된 값입니다. 파일을 읽는 순간에는 두 컬럼이 나란히 보여 구분이 되지 않습니다.

이 구분을 놓치면 학습 때는 정답이 입력에 섞여 들어와 성능이 거의 완벽하게 나오지만, 실제로 주문이 들어온 순간에는 그 값이 비어 있어 모델이 작동하지 않습니다. 그래서 feature 선정 기준은 "파일에 있는가"가 아니라 **”예측해야 하는 시점에 이미 확정된 값인가"**여야 합니다.

order_amount나 item_count처럼 주문 상세에서 집계한 값은 주문 접수 시점에 이미 확정되므로 허용했고, 반대로 order_status는 결과 그 자체이므로 제외했습니다. 판단 기준은 컬럼의 위치가 아니라 값이 기록되는 시간 순서입니다.

---

## 3. 주문 단위 특징과 병합 검증

### 계산 관계 확인

```text
line_total = quantity × unit_price
```

- 계산 불일치 건수: 0건 (764행 전체, 절대오차 합계 0)
- 주문 특징 `order_id` 중복 건수: 0건 (248개 주문 → 248행 집계)

### 병합 검증

병합	관계	병합 전 행 수	병합 후 행 수	미매칭	판정
주문 ↔ 주문 특징	one_to_one	248	248	0	PASS
주문 ↔ 고객	many_to_one	248	248	0	PASS

### 결과 관찰
두 병합 모두 행 수가 248로 유지되었고 indicator=True 결과가 전부 both(좌측만 0, 우측만 0)였습니다.
주문 상세가 없는 주문은 0건이므로 집계 결측이 발생하지 않았습니다.
validate 인자가 통과했다는 것은 집계 테이블에 order_id 중복이 없고, 고객 테이블에 customer_id 중복이 없다는 뜻입니다.
고객 테이블에서는 customer_id, gender, age, city만 가져왔고 name은 병합 대상에서 제외했습니다.

### 나의 해석과 판단
미매칭 행을 0으로 채우면 "그 값이 없다"는 사실이 "그 값이 0이다"라는 정보로 바뀝니다. 둘은 전혀 다른 의미입니다. 예를 들어 주문 상세가 없는 주문을 order_amount=0으로 채우면 모델은 "금액 0원 주문은 취소가 많다"는 패턴을 학습하지만, 실제 원인은 데이터 적재 누락일 수 있습니다. 그러면 모델은 데이터 파이프라인의 결함을 고객 행동으로 오해한 채 학습합니다.

더 큰 문제는 미매칭 자체가 상위 단계의 오류 신호라는 점입니다. 키 컬럼의 타입 불일치, 공백·대소문자 차이, 전처리 과정에서의 행 삭제, 실제 FK 무결성 깨짐 — 원인에 따라 조치가 전혀 달라집니다. 0으로 채우면 이 신호가 사라져 이후 모든 분석이 조용히 잘못된 전제 위에서 돌아갑니다.

그래서 순서는 ① 미매칭 건수와 방향(좌측만/우측만) 확인 → ② 원인 규명 → ③ 그다음에 결측 처리 방식 결정입니다. 이번 데이터는 미매칭이 0건이어서 채울 값이 없었지만, 만약 발생했다면 0 대입이 아니라 Pipeline 내부 SimpleImputer(median)로 처리하고 그 사실을 점검표에 남기는 방식을 택했을 것입니다.

---

## 4. 클래스 비율

클래스	의미	개수	비율
0	completed	184	74.2%
1	cancelled	64	25.8%

![클래스 비율](images/step02_class_ratio.png)
<img width="1321" height="448" alt="image" src="https://github.com/user-attachments/assets/58775f01-3eba-4982-a9fb-07ee0d4756e1" />


### 결과 관찰
전체 248건 중 다수 클래스(completed)가 74.2%, 소수 클래스(cancelled)가 25.8%로 약 2.9 : 1 불균형입니다.
train / validation / test 분할 후에도 양성 비율이 25.7% / 26.0% / 26.0%로 유지되었습니다(계층 분할).
실제 Dummy 모델(다수 클래스만 예측)의 validation accuracy는 0.74, 같은 모델의 f1은 0.000이었습니다.
선택된 Logistic Regression의 validation accuracy는 0.60으로 Dummy보다 낮았지만, f1은 0.000 → 0.524로 올라갔습니다.

### 나의 해석과 판단
accuracy는 "맞힌 비율"일 뿐이어서 다수 클래스가 많으면 아무것도 학습하지 않아도 높게 나옵니다. 이번 데이터에서는 모든 주문을 completed로 찍기만 해도 accuracy 0.74가 나오고, 이것이 실제 Dummy 모델에서 그대로 확인됐습니다. 즉 accuracy 0.74는 "성능"이 아니라 클래스 비율 그 자체입니다.

그래서 accuracy 단독 해석은 위험합니다. Dummy의 accuracy 0.74는 취소 주문 64건 중 한 건도 잡지 못한 상태이고(recall 0, precision 0, f1 0), 취소를 미리 감지하려는 이 문제의 목적에서는 완전히 쓸모없는 모델입니다. 반대로 Logistic Regression은 accuracy가 0.60으로 더 낮지만 validation recall 0.846으로 취소를 대부분 잡아냈습니다. accuracy만 보면 더 나쁜 모델을 고르게 됩니다.

따라서 판단 기준을 accuracy가 아니라 소수 클래스에 대한 precision·recall·f1으로 두고, accuracy는 “Dummy의 0.742를 넘겼는지”가 아니라 "소수 클래스를 포기하지 않았는지"를 확인하는 보조 지표로만 읽었습니다. 모델 선택도 validation f1 기준으로 했습니다.
---

## 5. Dummy Baseline과 Validation 모델 비교

| 모델 | Accuracy | Precision | Recall | F1 |
| --- | ---: | ---: | ---: | ---: |
| Dummy Most Frequent | | | | |
| Logistic Regression | | | | |
| Random Forest | | | | |

![Baseline 비교](images/step03_baseline.png)

- Validation에서 선택한 모델:

### 결과 관찰

### 나의 해석과 판단
선택한 모델이 Dummy baseline보다 어떤 점에서 나아졌는지 작성하세요.

### 한계와 추가 확인 사항

---

## 6. Train / Validation / Test 역할

| Split | 역할 | 클래스 0 건수 | 클래스 1 건수 |
| --- | --- | ---: | ---: |
| Train | 모델 학습 | | |
| Validation | 모델·Threshold 선택 | | |
| Test | 선택 이후 최종 평가 | | |

### 나의 해석과 판단
왜 Test를 모델 또는 Threshold 선택에 사용하면 안 되는지 작성하세요.

### 교육용 random split의 한계
실제 미래 주문 예측 운영 전에 추가해야 할 평가를 작성하세요.

---

## 7. Validation 성능과 우선 지표

- Accuracy:
- Precision:
- Recall:
- F1:

### 내가 가장 중요하게 본 지표

### 그 이유

> 실제 FP/FN 비용 정보가 없다면 가정임을 명시하세요.

---

## 8. Threshold 비교

| Threshold | Precision | Recall | F1 | 예상 특징 |
| ---: | ---: | ---: | ---: | --- |
| | | | | |
| | | | | |
| | | | | |

![Threshold 비교](images/step05_threshold.png)

### 내가 선택한 Threshold

### 선택 이유

### 나의 해석과 판단
Threshold를 Final Test가 아니라 Validation에서 정해야 하는 이유를 작성하세요.

---

## 9. FP/FN 해석

- FP 의미:
- FN 의미:
- 현재 목적에서 더 부담스러울 수 있는 오류:
- 그 판단에 추가로 필요한 업무 정보:

![Confusion Matrix](images/step06_confusion_matrix.png)

### 업무·분석적 의미

### 한계와 추가 확인 사항

---

## 10. Final Test

- 고정한 모델:
- 고정한 Threshold:
- Accuracy:
- Precision:
- Recall:
- F1:
- Validation 대비 변화:

### 결과 관찰

### 나의 해석과 판단
Final Test 결과를 보고 모델의 일반화 가능성을 어떻게 판단했는지 작성하세요.

### 선택 보호 확인
- [ ] Test 결과를 보기 전에 모델을 고정했습니다.
- [ ] Test 결과를 보기 전에 Threshold를 고정했습니다.
- [ ] Test 결과를 본 뒤 같은 Test를 이용해 선택을 바꾸지 않았습니다.

---

## 11. Internal / Public 결과 구분

### 공개 Prediction 컬럼

```text
record_id
actual_is_cancelled
predicted_is_cancelled
cancel_probability
model
threshold
```

### 공개 결과에 없어야 할 컬럼

```text
source_index
order_id
customer_id
product_id
```

### 결과 관찰

### 나의 해석과 판단
모델 오류 분석용 내부 자료와 공개 결과를 왜 분리해야 하는지 작성하세요.

---

## 12. Validation Evidence

다음 Evidence 중 확인한 항목에 체크합니다.

- [ ] `ch10_target_distribution.csv`
- [ ] `ch10_feature_audit.csv`
- [ ] `ch10_merge_checks.csv`
- [ ] `ch10_data_quality_checks.csv`
- [ ] `ch10_split_summary.csv`
- [ ] `ch10_validation_model_comparison.csv`
- [ ] `ch10_validation_threshold_metrics.csv`
- [ ] `ch10_test_metrics.csv`
- [ ] `ch10_confusion_matrix.csv`
- [ ] `ch10_classification_validation.csv`

### 자동 Validation 결과

### 자동 PASS가 의미하지 않는 것
운영 적합성, 공정성, 시간에 따른 성능 변화 등 자동 검증만으로 판단할 수 없는 내용을 작성하세요.

---

## 13. 최종 사용 판단

- [ ] 참고용 사용 가능
- [ ] 추가 검증 후 사용 가능
- [ ] 현재 사용 보류

### 판단 근거
1.
2.
3.

### 현재 모델의 가장 큰 위험

### 다음 개선 우선순위
1.
2.
3.

### 예측과 인과 구분
현재 모델에서 확인한 것은 어떤 **예측 패턴**이며, 어떤 **원인 주장**은 아직 할 수 없는지 작성하세요.

---

## 최종 체크

- [ ] `completed=0`, `cancelled=1` 타깃 범위를 정의했습니다.
- [ ] `refunded`와 기타 상태를 제외했습니다.
- [ ] 예측 시점과 Feature Contract를 정의했습니다.
- [ ] target, ID, 사후 정보를 feature에서 제외했습니다.
- [ ] `line_total = quantity × unit_price`를 검증했습니다.
- [ ] 주문 단위 특징과 병합 관계를 검증했습니다.
- [ ] 클래스 비율을 확인했습니다.
- [ ] Dummy baseline과 비교했습니다.
- [ ] Train / Validation / Test 역할을 구분했습니다.
- [ ] 모델을 Validation에서 선택했습니다.
- [ ] Threshold를 Validation에서 선택했습니다.
- [ ] Final Test 전에 선택을 고정했습니다.
- [ ] Precision / Recall / F1을 해석했습니다.
- [ ] FP/FN의 의미를 설명했습니다.
- [ ] Public prediction에서 식별자를 제거했습니다.
- [ ] Validation Evidence를 확인했습니다.
- [ ] random split의 한계를 기록했습니다.
- [ ] 예측 패턴을 원인처럼 단정하지 않았습니다.
- [ ] 최종 Notebook URL을 제출합니다.
