<div align="center">

# 🍪 Cookie Cats A/B Test Analysis
### 모바일 게임 게이트 위치 변경(레벨 30 → 40)이 유저 유지율에 미치는 영향 분석

<img src="https://img.shields.io/badge/Python-3776AB?style=for-the-badge&logo=python&logoColor=white">
<img src="https://img.shields.io/badge/Jupyter-F37626?style=for-the-badge&logo=jupyter&logoColor=white">
<img src="https://img.shields.io/badge/Pandas-150458?style=for-the-badge&logo=pandas&logoColor=white">
<img src="https://img.shields.io/badge/SciPy-8CAAE6?style=for-the-badge&logo=scipy&logoColor=white">
<img src="https://img.shields.io/badge/Statsmodels-4051B5?style=for-the-badge">
<img src="https://img.shields.io/badge/Matplotlib-11557C?style=for-the-badge">

</div>

---

# 📖 프로젝트 소개

모바일 퍼즐 게임 **Cookie Cats**의 A/B 테스트 데이터를 활용하여, 게임 속 **게이트(기다리거나 결제해야 다음 레벨로 진행하는 장치)** 의 위치를 레벨 30에서 40으로 옮겼을 때 유저 유지율이 달라지는지 검증한 프로젝트입니다.

단순한 가설 검정에 그치지 않고 **실험 설계 → 데이터 품질 점검 → 빈도주의·베이지안 검정 → 세그먼트 분석 → 의사결정**까지 하나의 흐름으로 정리하여, 실제 데이터 분석가가 실험 결과를 비즈니스 결정으로 연결하는 과정을 담았습니다.

---

# 1. Project Overview

### 📌 주제
게이트 위치 변경(gate_30 → gate_40)이 유저 유지율에 미치는 영향 분석

### 🎯 목표
- 사전 표본 크기·검정력 계산을 포함한 **실험 설계**
- SRM, 이상치 점검을 통한 **실험 데이터 신뢰도 확인**
- 빈도주의와 베이지안 두 관점의 **A/B 검정**
- 효과가 어떤 유저 구간에서 나타나는지 **세그먼트 Deep Dive**
- 검정 결과를 바탕으로 한 **의사결정과 액션 아이템** 도출

---

# 2. Dataset

### 데이터 구성
Kaggle의 *Mobile Games A/B Testing - Cookie Cats* 데이터셋을 사용합니다. 총 **90,189명**의 유저가 무작위로 gate_30(기존) 또는 gate_40(변경) 그룹에 배정되었습니다.

데이터 출처 : https://www.kaggle.com/datasets/mursideyarkin/mobile-games-ab-testing-cookie-cats

| 컬럼명 | 설명 |
| :--- | :--- |
| userid | 유저 식별자 |
| version | 실험 그룹 (gate_30: 기존 / gate_40: 변경) |
| sum_gamerounds | 설치 후 14일 동안 플레이한 라운드 수 |
| retention_1 | 설치 1일 뒤 재접속 여부 |
| retention_7 | 설치 7일 뒤 재접속 여부 |

> ⚠️ 결제/매출 데이터가 없어 게이트의 **수익 측면은 평가할 수 없습니다.**

---

# 3. 프로젝트 구조

```text
cookie-cats-ab-test
├── cookie_cats_ab_test.ipynb   # 전체 분석 노트북
├── cookie_cats.csv             # 데이터 (Kaggle에서 다운로드)
├── .gitignore
└── README.md
```

---

# 4. 분석 내용

### 🧪 실험 설계
- 가설: H0(두 그룹의 유지율은 차이가 없다) / H1(두 그룹의 유지율은 차이가 있다), 양측 검정
- 주 지표: 1일·7일 유지율 / 가드레일: 14일 누적 라운드 수
- 주 지표 2개에 대한 **Bonferroni 보정** (개별 α = 0.025)
- 표본 크기: 1%p 차이 탐지에 그룹당 약 28,000명 필요 → 실제 표본으로 탐지 가능한 최소 차이(MDE) 약 0.8%p

### 🔍 데이터 점검
- 결측치·중복 확인
- **SRM(Sample Ratio Mismatch)** 카이제곱 검정으로 그룹 배정 비율 점검
- 극단값 처리: 14일 동안 물리적으로 불가능한 수준(10,000라운드 초과) 1건 제거

### 📊 A/B 검정
- 유지율: 두 비율 z-검정 + 부트스트랩 95% 신뢰구간
- 라운드 수: 치우친 분포를 고려한 Mann-Whitney U 검정
- 베이지안: Beta-Binomial 모델로 P(gate_40이 더 높을 확률), 기대 손실, 95% 신용구간 산출

### 🧩 세그먼트 Deep Dive
- 플레이량 구간별(0 / 1~29 / 30~39 / 40~99 / 100+) 효과 비교
- `sum_gamerounds`는 레벨 수가 아니라 플레이한 판 수(실패·재도전 포함)
  - 30판 미만이면 레벨 30에 도달할 수 없어 **게이트 전 구간**으로 보고, 두 그룹의 차이가 없어야 한다는 점을 확인
  - 30판 이상은 실제 도달 레벨을 알 수 없어 "게이트를 만났을 가능성이 있는 구간"으로 탐색
- 0라운드 제외, 30라운드 이상만 포함한 **민감도 분석**

---

# 5. Analysis Pipeline

```
실험 설계 (가설·지표·표본 크기 / 검정력)
↓
데이터 점검 (결측·중복 / SRM / 이상치 처리)
↓
A/B 검정 (빈도주의 + 베이지안)
↓
세그먼트 Deep Dive (구간별 효과 / 게이트 전 구간 점검 / 민감도 분석)
↓
결론 · 액션 아이템 · 한계점
```

---

# 6. 사용 기법

| 목적 | 기법 | 선택 이유 |
| :--- | :--- | :--- |
| 유지율 비교 | 두 비율 z-검정, 부트스트랩 CI | 이진 지표의 차이와 불확실성 확인 |
| 참여도 비교 | Mann-Whitney U 검정 | 라운드 수가 한쪽으로 크게 치우친 분포 |
| 다중 비교 | Bonferroni 보정 | 주 지표 2개, 구간 5개 비교의 오류 통제 |
| 의사결정 | 베이지안 (Beta-Binomial) | "gate_40이 더 나쁠 확률"로 직접 해석 가능 |
| 실험 신뢰도 | SRM 카이제곱 검정 | 그룹 배정 이상 여부 확인 |

---

# 7. 핵심 결과

| 구분 | 결과 |
| :--- | :--- |
| **7일 유지율** | **19.02% → 18.20% (−0.82%p)**, p = 0.0016 (Bonferroni 기준 유의), 95% CI [−1.33, −0.34] |
| 1일 유지율 | 44.82% → 44.23% (−0.59%p), p = 0.074 (유의하지 않음) |
| 베이지안 | gate_40이 더 높을 확률 **0.1%**, 차이 95% 신용구간 [−1.33, −0.31]%p |
| 14일 참여도 | 평균·중앙값 차이 미미 (Mann-Whitney p = 0.051) |
| 세그먼트 (탐색적) | 차이가 **40~99라운드 구간에 집중** (32.9% → 29.4%, −3.5%p), 게이트 전 구간·100+ 구간은 차이 없음 |

### ✅ 결론: gate_30 유지 (gate_40 도입 비권장)
- gate_40은 7일 유지율을 낮추며, 이는 통계적으로 유의하고 방향도 일관적입니다.
- 100라운드 이상의 핵심 유저는 영향이 없고, 차이는 중간 플레이량(40~99라운드) 유저에게서 나타납니다. 라운드 수는 레벨 수가 아니고 처치의 영향을 받는 변수라 탐색적 결과로만 해석합니다.

### 🚀 액션 아이템
1. gate_40 전면 적용을 보류하고 gate_30을 유지한다.
2. 감소 폭이 큰 40~99라운드 구간 유저의 이탈 원인을 분석한다.
3. 위치 변경 대신 게이트 보상·대기 시간 등 다른 변수를 실험한다.
4. 레벨 도달 로그(게이트를 만난 시점)와 1~3일 차 일자별 로그를 확보해 게이트 직후의 이탈을 직접 확인한다.

---

# 8. Tech Stack

| 분야 | 기술 |
| :--- | :--- |
| Language | Python |
| Environment | Jupyter Notebook (VS Code) |
| Data | Pandas, NumPy |
| Statistics | SciPy, statsmodels |
| Visualization | Matplotlib |

---

# 9. 실행 방법

```bash
git clone https://github.com/ikhyun00/cookie-cats-ab-test.git
cd cookie-cats-ab-test

# 1. 패키지 설치
pip install pandas numpy scipy statsmodels matplotlib jupyter

# 2. 데이터 준비
#    Kaggle의 Cookie Cats 데이터(cookie_cats.csv)를 노트북과 같은 폴더에 둡니다.

# 3. 노트북 실행
jupyter notebook cookie_cats_ab_test.ipynb
```

---

# 10. 프로젝트 특징

- ✅ 분석 전 **표본 크기·검정력 계산**으로 실험 설계 단계부터 접근
- ✅ **SRM 점검**으로 실험 데이터 신뢰도 확인
- ✅ 빈도주의와 베이지안 **두 관점의 결과 교차 검증**
- ✅ 게이트 전 구간 점검과 민감도 분석으로 **결과의 견고성 확인**
- ✅ 검정 결과가 의사결정과 액션 아이템으로 이어지는 **결론 중심 구성**
- ✅ 한계점과 다음 실험까지 정리

---

# 11. 한계점 및 향후 개선 사항

### 한계점
- **매출 지표 부재**: 유지율이 오르내려도 수익 영향은 알 수 없습니다.
- **관측 기간**: 7일 유지율까지만 있어 장기(LTV) 효과를 판단할 수 없습니다.
- **그룹 비율**: gate_30 44,700명 / gate_40 45,489명으로 SRM p = 0.0086 (엄격한 기준 0.001에는 미달하나 완전한 50:50은 아님).
- **처치 후 변수**: 플레이량 기반 세그먼트는 게이트의 영향을 받은 변수를 사용하므로 인과 해석에 한계가 있습니다.
- **라운드 수 ≠ 레벨**: 레벨 도달 정보가 없어 유저가 실제로 게이트를 만났는지는 직접 확인할 수 없습니다.

---

# 👨‍💻 Developer

| 이름 | 역할 |
| :--- | :--- |
| 조익현 | 실험 설계, 데이터 분석, 통계 검정 |

Data Analysis · A/B Testing · Statistics

GitHub: https://github.com/ikhyun00
