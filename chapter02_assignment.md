# Chapter 02 제출 답안. VS Code에서 시작하는 데이터 분석 환경

> 최종 파일은 개인 GitHub 저장소의 `chapter02/chapter02.md`로 저장하는 것을 권장합니다.

## 0. 제출 정보

- 이름: 조유라 
- GitHub ID: joyula3-jpg
- 개인 저장소: `llm-data-analysis-study`
- 작성일:9월 10일
- 운영체제: jupyter notebook & visual studio

### 최종 제출 URL

```text
https://github.com/joyula3-jpg/llm-data-analysis-study_rev2/edit/main/chapter02_assignment.md
```

---

## 1. Python과 Git 환경 확인

### 실행 내용

```text
python --version 또는 py --version
git --version
```

### 실행 결과

```text
3.13.9
```

### Evidence

![Python과 Git 버전](images/step01_versions.png)
<img width="782" height="94" alt="image" src="https://github.com/user-attachments/assets/2be3584f-e6e6-45ce-bc78-a9699e8a14d4" />


### 결과 관찰

jupyter notebook에서 실행 결과 3.13.9로 확인되었습니다. 

### 나의 해석과 판단

적합하다고 생각합니다. 교수님께서 보여주신 강의에서도 확인하였습니다. 

### 업무·분석적 의미

신규 버전은 최신 기술이지만 안정성에서 떨어질 수 있고
버전마다 필요한 library 별 버전이 맞을수도 맞지 않을수도 있다. 
그렇기 때문에 최초에 python 버전을 미리 확인하고 그에 맞는 부분을 선택적 설치해야 한다. 

### 한계와 추가 확인 사항

없습니다. 

---

## 2. 저장소와 `.venv` 준비

### 수행 내용

- [ o ] 공식 Public 저장소 clone
- [ o ] 프로젝트 루트 확인
- [ o ] `.venv` 생성
- [ o ] `.venv` 활성화
- [ o ] `requirements.txt` 설치

### 핵심 실행 결과

```text
현재 프로젝트 경로: LLM-Lecture - .venv
터미널 Python 실행 파일: -
가상환경 활성화 여부: 확인
패키지 설치 결과: 확인
```

### Evidence

![가상환경과 Python 경로](images/step02_venv.png)
<img width="1365" height="925" alt="image" src="https://github.com/user-attachments/assets/4d0f0237-8c9c-42ed-bedc-6e18e55da98f" />


### 결과 관찰

현재 `python`이 어떤 실행 파일을 가리키는지 작성하세요.
: 
<img width="531" height="43" alt="image" src="https://github.com/user-attachments/assets/92aad701-bc22-433c-a731-0a2de11aeb3b" />
: /Users/hyeontan-o/LLM_Lecture/.venv/bin/python


### 나의 해석과 판단

시스템 Python과 프로젝트 `.venv`를 분리하는 것이 왜 필요한지 자신의 말로 작성하세요.
: 시스템 - library 간 버전 충돌을 막기 위해서 
  안정성이 확보되고, 해당 python 버전의 맞는 library를 설치해야 공용 공간의 피해를 막을 수 있기 때문 

### 업무·분석적 의미

다른 사람이 같은 프로젝트를 재실행할 때 가상환경이 주는 이점을 작성하세요.
: 어디서나 재현이 가능하다. 

### 한계와 추가 확인 사항

회사/기관 PC 정책, Python 버전 차이 등 현재 환경의 제약을 작성하세요. 
: 회사pc에서는 낮은 버전의 python을 설치하는 것으로 권장하고 있음 

---

## 3. VS Code 인터프리터와 Jupyter 커널 연결

### 확인 결과

```text
VS Code Python 인터프리터: .venv 
Notebook sys.executable: opt/anaconda3/bin/python
Notebook Path.cwd(): /Users/hyeontan-o/LLM_Lecture/chapter02
```

### Evidence

![VS Code 인터프리터와 Notebook 커널](images/step03_kernel.png)
<img width="737" height="245" alt="image" src="https://github.com/user-attachments/assets/5f11dd0a-5205-4e76-beaa-3ddcbda07a69" />


### 결과 관찰

터미널 Python과 Notebook Python이 같은 `.venv`인지 작성하세요.
네 같은 .venv 입니다. 

### 나의 해석과 판단

둘이 다를 경우 어떤 문제가 발생할 수 있는지 작성하세요.
버전이 다르기 때문에 구현된 코드가 실행이 안되는 케이스가 발생할 수 있습니다. 

### 업무·분석적 의미

`ModuleNotFoundError` 같은 환경 오류를 줄이는 데 어떤 도움이 되는지 작성하세요.
질문의 의미를 모르겠습니다. 

### 한계와 추가 확인 사항

커널 이름만 보고 판단하면 안 되는 이유 등 추가 확인 사항을 작성하세요.
질문의 의미를 모르겠습니다. 
확실하게 가상환경까지 타고 들어가는 것이 필요합니다. 

---

## 4. 샘플 데이터와 Notebook 실행 검증

### 확인 결과

```text
DATA_DIR 존재 여부: 확인하였습니다. 
customers.csv 존재 여부: 확인하였습니다. 
customers.shape: 150행, 6열
주요 컬럼: customer_id, name, age
```

### Evidence

![customers 데이터 정상 로드](images/step04_customers.png)
<img width="592" height="354" alt="image" src="https://github.com/user-attachments/assets/479b7f64-b5ad-43a4-b24c-43ddfd1e3bbe" />
<img width="609" height="405" alt="image" src="https://github.com/user-attachments/assets/66e2edaa-97e2-40dc-8271-a3835be30e84" />


### 결과 관찰

`customers.head()`, shape, 컬럼 결과에서 직접 확인한 사실을 작성하세요.
총 6개의 열이 있으며 
수치형, 범주형이 골고루 섞여 있음 
총 150개의 정보를 확인할 수 있음 

### 나의 해석과 판단

이 단계까지 성공했다면 어떤 구성 요소가 정상 연결되었다고 판단할 수 있는지 작성하세요.
네 잘 연결되고 데이터도 read할 수 있습니다. 

### 업무·분석적 의미

분석 전에 최소 스모크 테스트를 하는 이유를 작성하세요.
- 전체 데이터를 다 밀어넣기 전에, 극소량의 데이터나 더미 데이터를 활용하여 코드 전체의 에러 발생 여부를 확인하며
- 시간과 비용을 절감하기 위해서 진행함 

### 한계와 추가 확인 사항

현재는 환경 연결만 확인했으며 데이터 품질은 아직 검증하지 않았다는 점을 작성하세요.
- 아직 품질 검증(결측치 여부, 있을 경우 보강 방법)에 대해서는 생각하지 않았습니다. 

---

## 5. 오류 해결 기록

실습 중 오류가 있었다면 작성합니다. 오류가 없었다면 `해당 없음`이라고 적습니다.
- 해당 없음 

### 오류 메시지

```text
 없습니다. 
```

### 원인 후보

1.
2.
3.

### 내가 확인한 순서

1.
2.
3.

### 해결 방법

```text
실제로 적용한 해결 방법
```

### Evidence

![오류 해결 결과](images/step05_troubleshooting.png)

### 나의 해석과 판단

왜 해당 원인이 가장 가능성이 높다고 판단했는지 작성하세요.

### 한계와 추가 확인 사항

보안 정책 변경, 무분별한 삭제처럼 시도하지 않은 조치와 이유를 작성하세요.

---

## 6. Secret 보호 확인

- [ o ] `.env`는 Git 추적 대상이 아닙니다.
- [ o ] 실제 API Key를 코드에 작성하지 않았습니다.
- [ o ] 캡처 화면에 Token/비밀번호가 없습니다.
- [ o ] `.venv`를 Git에 올리지 않습니다.

### Evidence

필요한 경우 `git status`, `.gitignore` 확인 화면을 첨부합니다.

![Secret 보호 확인](images/step06_security.png)

### 나의 해석과 판단

환경 파일과 비밀정보를 분리해야 하는 이유를 작성하세요.
- 보안 침해 방지와 운영의 유연성 확보를 위함 

---

## 7. Chapter 02 최종 회고

### 가장 중요했다고 생각한 환경 설정 1가지
- .venv 가상환경 설정 

```text
이전부터 분석활동을 진행하면서 python에서 library 설치 시.
특히 tensorflow, tensor 등 딥러닝 관련 라이브러리의 버전이 꼬여서 애로사항이 많았음


```

### 그 이유

```text
이번 기회를 통해 가상환경 구축의 중요성에 대해서 다시 한번 인지하였음 
```

### 다음 Chapter에서 재사용할 환경 체크 3가지

1. .venv
2. 폴더 확인 
3. 데이터 업로드 확인 

### 현재 환경의 한계 또는 주의점

```text
앞으로도 재활용을 위해 해당 경로를 활용하겠습니다. 
```

---

## 최종 제출 체크

- [ o ] 핵심 Evidence 4~7장을 첨부했습니다.
- [ o ] 단순 캡처가 아니라 관찰과 판단을 작성했습니다.
- [ o ] Secret/개인정보가 없습니다.
- [ o ] GitHub에서 이미지가 정상 표시됩니다.
- [ o ] 개인 저장소에 `chapter02/chapter02.md`를 업로드했습니다.
- [ p ] 저장소 URL이 아니라 최종 파일 URL을 제출합니다.
