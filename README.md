### Link

**Source code**

https://github.com/merware4969/LG-AI-Hackathon-5th-Detecting-product-anomalies/blob/main/code.ipynb

# 상세 내용

## 1. 데이터 전처리

### 1) PCA

다양한 위치 정보와 팔레트 데이터를 그룹별로 구분한 뒤, 각 그룹에 대해 주성분 분석(PCA)을 적용하였습니다. 

데이터의 주요 분산을 1차원으로 요약하면서, 정보 보존율과 데이터 분포 특성을 파악하였고, 이를 통해 데이터의 구조적 특징과 변수 간 상관관계를 시각적으로 확인할 수 있었습니다.

![image.png](attachment:f31db8d7-9d36-48f0-9232-f3a138f55fcf:image.png)

![image.png](attachment:7029fcab-cb5b-4d17-b2ec-f484af394718:image.png)

### 2) 덴드로그램

**고상관 피처 간 덴드로그램 기반 군집화**

- 상호 유사한 피처 간 **거리 기반 계층적 군집화(Hierarchical Clustering)** 수행
- `ward` 방법은 군집 내 분산 최소화를 기반으로 함
- **덴드로그램(Dendrogram)** 을 그려서 시각적으로 피처 간 유사도 및 군집 구조를 확인함

**임계값 기준 군집 수 설정 (임계값 1.5 → 군집 6개)**

- 덴드로그램의 수직 거리(높이)를 기준으로 **임계값(threshold)을 1.5로 지정**
- 이에 따라 **총 6개의 피처 군집**이 형성됨

![image.png](attachment:19431fb2-8b6f-46bb-84cd-daa71629379f:image.png)

다수의 공정 변수 중 상관계수가 1인 피처들을 식별하여, 정보 중복성과 과적합 가능성을 줄이고자 계층적 군집 분석과 PCA 기반 차원 축소를 수행하였습니다. 

덴드로그램을 통해 군집을 6개로 나눈 후 각 군집에 대해 주성분 분석을 적용하였고, 그 결과 대부분 98% 이상의 분산을 설명할 수 있어 효율적인 피처 축소가 가능했습니다.

이를 통해 분석 성능 저하 없이 데이터 구조를 간결하게 만들고, 모델 학습 효율을 높일 수 있었습니다.

### 3) EDA

머신러닝 모델(RandomForest)을 학습시켜, 각 피처의 **F score**를 기반으로 중요도를 산출하였습니다.

- F score는 해당 피처가 모델 트리 구조에서 얼마나 자주 사용되었는지를 나타내는 지표입니다.

![KakaoTalk_20240812_201345470.png](attachment:6dc0bb5c-fa0b-4cc7-9def-cc5709fa39e8:KakaoTalk_20240812_201345470.png)

**피처 중요도 해석**

- 상위 변수는 다음과 같으며, 생산 품질에 높은 영향을 미치는 것으로 보임
    - `Production Qty Collect Result_Dam`: **생산 수량**
    - `1st/2nd/3rd Pressure Collect Result_AutoClave`: **압력 관련 변수**
    - `PalletID Collect Result_Dam / Fill1 / Fill2`: **팔레트 ID**
    - `Chamber Temp. Collect Result_AutoClave`: **챔버 온도**
    - `Machine Tact time`: **기계 작동 시간**
- 반면, 중요도가 낮은 변수(예: `Chamber Temp. Unit Time`, `Production Qty Fill2`)는 예측 성능에 큰 영향을 주지 않는 것으로 나타남

### 4) SMOTE 기법(클래스 불균형 처리)

- `target` 변수의 클래스 분포가 `Normal >> AbNormal`로 **심하게 불균형(imbalanced)** 할 경우, 모델이 다수 클래스만 예측하게 됨.
- SMOTE는 소수 클래스(AbNormal)의 데이터를 **합성(Synthetic)하여 샘플 수를 늘려줌**

기대 효과

- 모델이 **양쪽 클래스 모두를 균형 있게 학습**하도록 유도
- 과소표집(undersampling)과 달리 기존 데이터를 버리지 않고 **데이터 손실 없이 균형 확보**
- 최종적으로 `Normal: 38156`, `AbNormal: 38156`으로 **완전 균형 클래스 구성**

## 2. 모델

### 1) 랜덤포레스트 모델

`target` 변수(Normal vs AbNormal)를 예측하는 **이진 분류 모델**을 구축하고, **예측 성능 및 주요 영향 변수**를 평가하고자 함.

### 2) confusion matrix

혼동 행렬(Confusion Matrix)을 통해 실제 vs 예측 간 분포 확인

![image.png](attachment:1076d90b-c748-4aeb-9e82-90a66e29ca7e:image.png)

### 3) PCA(차원축소)가 의미있었는가?

![image.png](attachment:70a1750b-831d-4089-992f-e5c9d5128a95:image.png)

PCA 축소된 변수(`PCA_Group_10_PC1`)도 높은 중요도를 가짐 → **차원축소가 의미 있음**

## 3. 결과

### **1) 실전 평가 결과 요약**

| 항목 | 내용 |
| --- | --- |
| 참가팀 수 | 총 740팀 |
| 본인 순위 | **141위 (상위 약 19%)** |
| 사용 모델 | RandomForestClassifier |
| 핵심 기법 | PCA 기반 차원 축소, SMOTE 오버샘플링, 원핫 인코딩, 피처 중요도 분석 등 |
| 🎯 최종 평가 지표 | 점수: **0.188235** |

![final.png](attachment:d3cc8153-0cf7-47e6-8864-1c02a23afa4b:final.png)

### 2) 사용 기술 및 라이브러리

| 구분 | 도구/기술 |
| --- | --- |
| 📦 데이터 전처리 | `pandas`, `numpy`, `sklearn.preprocessing` |
| 🧪 차원 축소 | `PCA` from `sklearn.decomposition` |
| 🧠 모델 학습 | `RandomForestClassifier` from `sklearn.ensemble` |
| ⚖️ 클래스 불균형 처리 | `SMOTE` from `imblearn.over_sampling` |
| 📊 평가 지표 | `f1_score`, `classification_report`, `confusion_matrix` from `sklearn.metrics` |
| 📈 시각화 | `matplotlib`, `seaborn`, `dendrogram` from `scipy.cluster.hierarchy` |
| 🧩 기타 기법 | One-Hot Encoding (`pd.get_dummies`), 피처 중요도 추출, 덴드로그램 기반 군집화 |

### 3) 담당한 부분

- 데이터 전처리
- EDA
- SMOTE

### 4) 배운 점

**데이터 불균형 문제에 대한 실전 이해**

프로젝트 초반, 클래스 불균형 문제를 해결하기 위한 **언더샘플링과 오버샘플링** 기법을 비교하며 고민했습니다. 언더샘플링은 정상 클래스(다수 클래스)의 데이터 손실을 유발하고, 그로 인해 **학습 성능 저하 및 정보 손실**이 크다고 판단했습니다.

→ "오버피팅은 조절 가능하지만, 정보 손실은 되돌릴 수 없다"는 생각으로 팀 내 합의하에 **오버샘플링(SMOTE)** 전략을 선택했습니다.

**차원 축소(PCA)에 대한 실제 경험**

PCA는 그동안 이론적으로만 알고 있었던 기법이었으나, 본 프로젝트에서는 **복잡한 공정 데이터(3차원 위치 좌표 등)를 효과적으로 요약**하는 데 활용했습니다. 

각 그룹에서 PCA의 설명 분산 비율이 대부분 98% 이상을 기록하며, **데이터의 대표성을 유지하면서도 차원 축소가 가능함**을 실제로 확인했습니다. 또한, PCA를 통해 생성한 주성분이 **최종 모델의 주요 피처로 선정**되는 것을 보며, 단순 축소를 넘어서 **유의미한 특징 추출 도구**가 될 수 있음을 체감했습니다.

**모델 성능 평가 및 해석 역량 향상**

F1 Score, Confusion Matrix, Feature Importance 시각화를 통해 **단순한 정확도 평가를 넘는 종합적인 모델 성능 해석**을 경험했습니다.

특히, 랜덤포레스트 기반 피처 중요도 분석을 통해, **정량적으로 중요한 공정 변수 도출**이 가능하다는 점에서 데이터 기반 인사이트 발굴에 대한 실질적 감을 익혔습니다.

**실전 평가 환경의 경험**

실제 제출용 평가 데이터셋으로 검증한 결과 **0.188점**, 전체 740팀 중 141위(상위 약 19%)를 기록하며 실전 모델의 성능을 수치로 확인할 수 있었습니다.

**데이터 전처리 → 피처 생성 → 학습 → 예측 → 평가**까지의 전 과정을 반복하며, **모델 개발의 A to Z를 처음부터 끝까지 스스로 경험**했다는 점에서 값진 프로젝트였습니다.
