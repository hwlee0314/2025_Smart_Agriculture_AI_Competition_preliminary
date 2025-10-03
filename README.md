# 2025 스마트 농업 AI 경진대회 사전테스트

### **2025-08-08 ~ 2025.08.09**
### [Competition Link](https://agrichallenge.ai)
- 온실 환경 제어의 정밀도를 높이기 위해서는 내부 환경을 사전 예측하는 기술이 필요함
- 작물은 생육 단계에 따라 다른환경을 필요로 하기 때문에 생육 단계를 정확히 아는것은 스마트 재배 자동화를 위해 중요함
- 개발된 모델을 사람의 개입 없이 운용하는 기술은 자율 재배의 필수요소

- 평가 산식: RMSE (Root Mean Squared Error, 평균 제곱근 오차)  

- 13th out of 30 teams

- 주최: 농림축산식품부  
주관: 한국농업기술진흥원, 농협중앙회
---

### **주요 특징**

- **5-Fold Cross Validation**: **XGBoost** 모델을 5-fold 교차 검증으로 학습하여 각 target별로 5개의 모델을 저장하고 앙상블 평균으로 예측합니다.
- **시계열 피처 엔지니어링**: 시간 데이터에서 hour, day_of_week, month를 추출하고, sin/cos 변환을 통해 주기적 특성을 반영했습니다.
- **상호작용 피처**: 온도-습도, 태양복사-온도, 환풍구 총합, 커튼 평균 등 도메인 지식을 활용한 상호작용 변수를 생성했습니다.
- **데이터 전처리**: 무한값 및 결측치를 체계적으로 처리하고, linear interpolation을 통해 시계열 데이터의 연속성을 보장했습니다.

---

### **개발 환경**

- **운영체제**: macos 26.0
- **언어**: Python 3.11

---

### **프로젝트 구조**

```
2025_Smart_Agriculture_AI_Competition_preliminary/
├── notebooks/           
│   ├── inference.ipynb
│   └── train-ml.ipynb
├── outputs/
│   ├── models/
│   │   ├── model-temperature-{0-4}.pkl
│   │   ├── model-humidity-{0-4}.pkl
│   │   └── model-co2-{0-4}.pkl
│   └── submission.csv
├── README.md           
└── requirements.txt    
```

---

### **모델링 접근법**

온실 환경 데이터의 3개 타겟 변수(온도, 습도, CO2)를 예측하기 위해 XGBoost 기반의 회귀 모델을 구축했습니다.

***

### 1. 데이터 전처리 파이프라인
- **결측치 및 이상치 처리**: 무한값을 결측치로 변환 후 선형 보간법으로 처리
- **시계열 피처 생성**: 시간 데이터에서 hour, day_of_week, month 추출 및 sin/cos 변환
- **상호작용 피처**: 온도-습도, 태양복사-온도, 환풍구 관련 변수 등 도메인 지식 기반 파생 변수 생성

### 2. 모델 학습 전략
- **XGBoost Regressor** 사용 (각 타겟별 독립 학습)
- **5-Fold Cross Validation**으로 모델 안정성 확보
- 각 fold별로 모델을 저장하여 최종 예측 시 앙상블 평균 적용

### 3. 주요 하이퍼파라미터
```python
xgb_params = {
    'objective': 'reg:squarederror',
    'max_depth': 6,
    'learning_rate': 0.1,
    'n_estimators': 100,
    'subsample': 0.8,
    'colsample_bytree': 0.8,
    'random_state': 42
}
```

---

### **파일 설명**

#### `notebooks/train-ml.ipynb`
- **데이터 로딩 및 EDA**: 학습/테스트 데이터 확인 및 시각화
- **GreenhousePredictor 클래스**: 전체 머신러닝 파이프라인을 담은 클래스
  - 데이터 전처리 (`clean_data`)
  - 시간 피처 생성 (`create_time_features`) 
  - 상호작용 피처 생성 (`create_interaction_features`)
  - XGBoost 모델 학습 (`train_xgboost_models`)
- **5-fold CV 학습**: 각 타겟(temperature, humidity, co2)별로 5개 모델 저장

#### `notebooks/inference.ipynb`  
- **저장된 모델 로딩**: `model-{target}-{fold}.pkl` 파일들을 불러오기
- **테스트 데이터 전처리**: 학습과 동일한 피처 엔지니어링 적용
- **예측 및 앙상블**: 각 타겟별 5개 모델의 예측 결과를 평균내어 최종 예측
- **제출 파일 생성**: `submission.csv` 형태로 결과 저장

#### `outputs/models/`
- `model-temperature-{0-4}.pkl`: 온도 예측 모델 (5개)
- `model-humidity-{0-4}.pkl`: 습도 예측 모델 (5개) 
- `model-co2-{0-4}.pkl`: CO2 예측 모델 (5개)


