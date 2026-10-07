
# **[Python : scikit-learn, pandas, joblib] : <br> 재현가능한 END-to End Pipeline 구축 프로젝트**

<br>


# **1. End-to-End Pipeline이 필요한 이유**

> ## **1-1) Pipeline이란?**

머신러닝 프로젝트에서는 일반적으로 모델 하나만 학습하는 것이 아니라 여러 단계가 연속적으로 수행된다.

예를 들어 운동처방 예측 시스템이라면 다음과 같은 과정이 필요할 수 있다.

```text
원본 데이터
    ↓
결측치 처리
    ↓
범주형 변수 Encoding
    ↓
수치형 변수 Scaling
    ↓
Class Imbalance 처리
    ↓
Machine Learning Model
    ↓
Prediction
```

이 과정들을 각각 따로 실행하면 학습할 때와 새로운 데이터를 예측할 때 서로 다른 전처리를 적용하는 실수가 발생할 수 있다.

`Pipeline`은 이러한 작업 순서를 **하나의 객체로 연결하여 동일한 데이터 처리 과정을 반복할 수 있도록 하는 도구**이다.

```text
Pipeline

Preprocessing
      ↓
Resampling
      ↓
Model
      ↓
Prediction
```

`fit()`을 실행하면 각 단계가 순서대로 학습되고,

```text
pipeline.fit(X_train, y_train)
```

`predict()`를 실행하면 학습된 전처리 기준을 이용하여 새로운 데이터를 변환한 뒤 예측한다.

```text
pipeline.predict(X_test)
```

<br>

> ## **1-2) Pipeline의 fit과 predict 동작**

일반적인 `scikit-learn Pipeline`은 다음처럼 동작한다.

```text
fit()

X_train
   ↓
Transformer 1
fit + transform
   ↓
Transformer 2
fit + transform
   ↓
Estimator
fit
```

예측 시에는

```text
predict()

X_new
   ↓
Transformer 1
transform
   ↓
Transformer 2
transform
   ↓
Estimator
predict
```

가 수행된다.

중요한 차이는 새로운 데이터에서 Transformer를 다시 `fit()`하지 않는다는 것이다.

학습 데이터에서 계산했던 통계량과 Mapping을 그대로 사용한다.

<br>

> ## **1-3) 보행분석 Pipeline 기본 예제**

보행 속도, 보폭, Cadence를 이용하여 고위험 보행 패턴을 분류한다고 가정한다.

[ Python : code ] "StandardScaler와 LogisticRegression을 하나의 Pipeline으로 구성"

```python
from sklearn.pipeline import Pipeline
from sklearn.preprocessing import StandardScaler
from sklearn.linear_model import LogisticRegression

pipe = Pipeline([
    ("scaler", StandardScaler()),
    ("classifier", LogisticRegression(max_iter=1000, random_state=42))
])

pipe.fit(X_train, y_train)

pred = pipe.predict(X_test)

print(pred[:5])
```

[ Result ]

```text
[0 1 0 0 1]
```

내부적으로 학습 시 다음 과정이 수행된다.

```text
X_train
   ↓
StandardScaler.fit()
   ↓
StandardScaler.transform()
   ↓
LogisticRegression.fit()
```

Test 데이터에서는

```text
X_test
   ↓
StandardScaler.transform()
   ↓
LogisticRegression.predict()
```

만 수행한다.

즉, Test 데이터의 평균과 표준편차를 새롭게 계산하지 않는다.


<br><br>

---

# **2. Data Leakage(데이터 누수)**

> ## **2-1) Data Leakage란?**

Data Leakage는 **실제 예측 시점에는 알 수 없어야 하는 정보가 모델 학습 과정에 들어가는 문제**이다.

Leakage가 발생하면 Validation이나 Test 성능이 실제보다 높게 나타날 수 있다.

대표적인 예는 전체 데이터를 이용하여 Scaling을 먼저 수행한 뒤 Train / Test를 분리하는 것이다.

```text
잘못된 방법

전체 데이터
    ↓
StandardScaler.fit()
    ↓
전체 데이터 Scaling
    ↓
Train / Test Split
```

이 경우 Scaling 과정에서 이미 Test 데이터의

```text
평균
표준편차
```

가 사용되었다.

즉, Test 데이터의 정보가 학습 과정에 간접적으로 반영된 것이다.

<br>

> ## **2-2) 올바른 처리 순서**

올바른 방식은 먼저 Train과 Test를 분리하는 것이다.

```text
전체 데이터
     ↓
Train / Test Split
     ↓
┌───────────────┐
│ Train         │
│               │
│ Scaler.fit()  │
└───────┬───────┘
        ↓
 학습된 Scaler
        ↓
┌───────┴────────┐
│                │
Train          Test
transform      transform
```

Pipeline에서는 다음처럼 사용할 수 있다.

```python
pipe.fit(X_train, y_train)
pred = pipe.predict(X_test)
```

Pipeline은 **Train 데이터에 대해서만 `fit()`을 호출했다는 전제에서** 전처리 기준을 Test 데이터와 분리할 수 있다.

따라서 Pipeline 자체가 잘못된 Train/Test 설계를 자동으로 수정해주는 것은 아니다.

```text
올바른 Data Split
        +
Pipeline 내부 전처리
        ↓
Leakage 방지
```

가 핵심이다.

<br>

> ## **2-3) Cross Validation에서 Pipeline이 더 중요한 이유**

Cross Validation에서는 매 Fold마다 Train과 Validation이 달라진다.

Pipeline 없이 전체 Train 데이터에 먼저 Scaling을 하면

```text
전체 Train 데이터
      ↓
Scaler.fit()
      ↓
5-Fold CV
```

처럼 Validation Fold 정보도 Scaler 계산에 포함된다.

이 역시 Leakage이다.

Pipeline을 CV에 넣으면 각 Fold마다 다음 과정이 반복된다.

```text
Fold 1

Training Fold
    ↓
Scaler.fit()
    ↓
Model.fit()

Validation Fold
    ↓
Scaler.transform()
    ↓
Validation 평가
```

다음 Fold에서는 Scaler도 다시 학습된다.

```text
Fold 2
→ 새로운 Training Fold에서 Scaler.fit()

Fold 3
→ 새로운 Training Fold에서 Scaler.fit()
```

따라서 **전처리까지 Cross Validation 내부에서 수행하는 것**이 중요하다.


<br><br>

---

# **3. ColumnTransformer를 이용한 전처리 분기**

> ## **3-1) 실제 데이터에는 여러 자료형이 존재한다**

실제 스포츠 데이터는 모든 Feature가 동일한 형식이 아니다.

예를 들어 운동처방 데이터가 다음과 같이 구성될 수 있다.

| Feature | 자료형 | 의미 |
| --- | --- | --- |
| `age` | 수치형 | 연령 |
| `bmi` | 수치형 | BMI |
| `grip_strength` | 수치형 | 악력 |
| `vo2max` | 수치형 | 최대산소섭취량 |
| `sex` | 범주형 | 성별 |
| `life_stage` | 범주형 | 생애주기 |
| `activity_level` | 범주형 | 활동 수준 |

수치형과 범주형에는 서로 다른 전처리가 필요하다.

| 자료형 | 결측치 | 추가 처리 |
| --- | --- | --- |
| 수치형 | Median / Mean | Scaling |
| 범주형 | Mode / Constant | Encoding |

이를 하나의 객체로 처리하는 것이 `ColumnTransformer`이다.

<br>

> ## **3-2) 운동처방 데이터 전처리**

[ Python : code ] "수치형과 범주형 Feature에 서로 다른 전처리 적용"

```python
from sklearn.compose import ColumnTransformer
from sklearn.pipeline import Pipeline
from sklearn.impute import SimpleImputer
from sklearn.preprocessing import StandardScaler, OneHotEncoder

# 수치형(연속형) 피저
numeric_features = [
    "age",
    "bmi",
    "grip_strength",
    "vo2max"
]

# 범주형 피처
categorical_features = [
    "sex",
    "life_stage",
    "activity_level"
]

# 수치형 파이프라인
numeric_pipe = Pipeline([
    ("imputer", SimpleImputer(strategy="median")), # 전처리(결측치 처리 방법-중앙값 대체)
    ("scaler", StandardScaler())
])

# 범주형 파이프라인
categorical_pipe = Pipeline([
    ("imputer", SimpleImputer(strategy="most_frequent")), # 전처리(결측치 처리 방법-최빈값 대체)
    ("onehot", OneHotEncoder(
        handle_unknown="ignore"
    ))
])

# 수치형 + 범주형 파이프라인 통합 (ColumnTransformer)
preprocessor = ColumnTransformer([ 
    ("numeric", numeric_pipe, numeric_features),
    ("categorical", categorical_pipe, categorical_features)
])

X_train_transformed = preprocessor.fit_transform(X_train)

print(X_train_transformed.shape)
```

[ Result ]

```text
(8000, 14)
```

원본 Feature는 7개이지만 범주형 Feature가 One-Hot Encoding되면서 변환 후 Feature 수가 14개가 되었다.

<br>

> ## **3-3) One-Hot Encoder에서 handle_unknown="ignore"가 중요한 이유**

학습 데이터에는 다음과 같은 운동 수준만 있었다고 가정한다.

```text
Low
Moderate
High
```

운영 데이터에서

```text
Very High
```

라는 새로운 범주가 입력될 수 있다.

기본 설정에 따라서는 새로운 범주 때문에 오류가 발생할 수 있다.

```python
OneHotEncoder(handle_unknown="ignore")
```

를 사용하면 학습할 때 보지 못했던 범주가 입력되더라도 해당 One-Hot 영역을 0으로 처리하여 예측 과정을 계속할 수 있다.

운영 환경에서는 **새로운 Category가 나타날 가능성**을 고려해야 한다.


<br><br>

---

# **4. Class Imbalance와 Resampling**

> ## **4-1) 불균형 데이터 문제**

스포츠 손상이나 운동 중 이상반응처럼 발생 빈도가 낮은 사건을 예측하면 클래스 불균형이 발생할 수 있다.

예를 들어 축구 선수 부상 데이터가

```text
정상
8,800건

부상
1,200건
```

이라고 하자.

비율은

```text
정상  = 88%
부상  = 12%
```

이다.

이런 상황에서 모델이 모든 데이터를 정상으로 예측하면 Accuracy는 높아질 수 있지만 부상 선수를 탐지하지 못한다.

```text
Accuracy 높음
≠
소수 Class 예측 성능이 좋음
```

따라서 다음 지표를 함께 확인할 수 있다.

```text
Precision
Recall
F1-score
PR-AUC
Confusion Matrix
```

<br>

> ## **4-2) SMOTE**

SMOTE는 소수 클래스 데이터 사이를 보간하여 새로운 합성 Sample을 생성하는 Oversampling 방법이다.

```text
Minority A ●

          × ← Synthetic Sample

Minority B ●
```

단순히 기존 Sample을 복사하는 것이 아니라 주변 소수 클래스 Sample을 이용하여 새로운 관측값을 만든다.

그러나 SMOTE는 반드시 **Training Data에만 적용해야 한다.**

잘못된 방법은

```text
전체 데이터
    ↓
SMOTE
    ↓
Train / Test Split
```

이다.

합성 데이터를 생성한 뒤 분리하면 원본과 매우 유사한 합성 정보가 Train과 Test 양쪽에 들어갈 수 있다.

이는 심각한 Leakage를 만들 수 있다.

<br>

> ## **4-3) Cross Validation에서 SMOTE 적용**

올바른 구조는 다음과 같다.

```text
Fold Training Data
        ↓
SMOTE
        ↓
Model Training

Fold Validation Data
        ↓
SMOTE 적용 X
        ↓
Model Evaluation
```

이를 구현할 때는 `imbalanced-learn`의 Pipeline을 사용할 수 있다.

<br>

> ## **4-4) 축구 GPS 부상예측 Pipeline**

GPS Feature가 모두 수치형이라고 가정한다.

```text
total_distance_m
hsr_distance_m
sprint_count
accel_count
player_load
sleep_hr
soreness_score
```

[ Python : code ] "SMOTE를 Training Fold에만 적용하는 부상예측 Pipeline"

```python
from imblearn.pipeline import Pipeline as ImbPipeline
from imblearn.over_sampling import SMOTE
from sklearn.impute import SimpleImputer
from sklearn.preprocessing import StandardScaler
from sklearn.ensemble import RandomForestClassifier

pipeline = ImbPipeline([
    ("imputer", SimpleImputer(strategy="median")),
    ("scaler", StandardScaler()),
    ("smote", SMOTE(random_state=42)),
    ("classifier", RandomForestClassifier(
        n_estimators=300,
        random_state=42
    ))
])

pipeline.fit(X_train, y_train)

pred = pipeline.predict(X_test)

print(pred[:10])
```

[ Result ]

```text
[0 0 1 0 0 1 0 0 0 1]
```

학습할 때는

```text
결측치 처리
    ↓
Scaling
    ↓
SMOTE
    ↓
Random Forest
```

가 수행된다.

예측할 때는 SMOTE가 적용되지 않는다.

```text
새로운 데이터
    ↓
결측치 처리
    ↓
Scaling
    ↓
SMOTE 건너뜀
    ↓
Random Forest Prediction
```

Resampling은 **학습용 처리**이지 실제 새로운 선수 데이터를 합성하는 과정이 아니기 때문이다.


<br><br>

---

# **5. 범주형 Feature와 SMOTE 사용 시 주의점**

> ## **5-1) One-Hot Encoding 뒤 일반 SMOTE를 무조건 사용하면 안 되는 이유**

범주형 변수를 One-Hot Encoding하면 다음과 같은 값이 만들어진다.

```text
포지션

GK  DF  MF  FW

0   0   1   0
```

일반 SMOTE는 값 사이를 보간하기 때문에 경우에 따라 다음처럼 실제 Category를 직접 의미하지 않는 중간값이 만들어질 수 있다.

```text
0.0  0.0  0.4  0.6
```

따라서 범주형 Feature가 포함된 데이터에 일반 SMOTE를 적용할 때는 주의해야 한다.

<br>

> ## **5-2) 혼합형 데이터의 대안**

| 상황 | 고려 방법 |
| --- | --- |
| Feature가 모두 수치형 | `SMOTE` |
| 수치형 + 범주형 | `SMOTENC` |
| 단순 복제가 허용됨 | `RandomOverSampler` |
| 모델 자체에서 불균형 조절 | `class_weight` |
| XGBoost Binary Classification | `scale_pos_weight` 등 |

범주형 Feature가 포함된 경우에는 `SMOTENC`처럼 범주형 Feature를 고려하는 방법을 사용할 수 있다.

중요한 것은

```text
SMOTE를 쓰는 것 자체
```

가 목적이 아니라

```text
현재 데이터 구조에 적절한
불균형 처리 방법을 선택
```

하는 것이다.


<br><br>

---

# **6. ColumnTransformer + imbalanced-learn Pipeline 통합**

> ## **6-1) 운동처방 예측 End-to-End 구조**

운동처방 필요 여부를 예측한다고 가정한다.

수치형 Feature는

```text
age
bmi
grip_strength
sit_and_reach_cm
vo2max
```

이고 범주형 Feature는

```text
sex
life_stage
activity_level
```

이다.

Target은

```text
exercise_prescription

0 = 일반 관리
1 = 운동처방 필요
```

라고 가정한다.

범주형 Feature를 One-Hot Encoding한 뒤 합성 보간 방식 대신 `RandomOverSampler`를 사용하면 다음처럼 통합할 수 있다.

[ Python : code ] "ColumnTransformer와 RandomOverSampler를 결합한 End-to-End Pipeline"

```python
from sklearn.compose import ColumnTransformer
from sklearn.pipeline import Pipeline
from sklearn.impute import SimpleImputer
from sklearn.preprocessing import StandardScaler, OneHotEncoder
from sklearn.ensemble import RandomForestClassifier

from imblearn.pipeline import Pipeline as ImbPipeline
from imblearn.over_sampling import RandomOverSampler

# 수치형 피처
numeric_features = [
    "age",
    "bmi",
    "grip_strength",
    "sit_and_reach_cm",
    "vo2max"
]

# 범주형 피처
categorical_features = [
    "sex",
    "life_stage",
    "activity_level"
]

# 수치형 변수 파이프라인 전처리기
numeric_pipe = Pipeline([
    ("imputer", SimpleImputer(strategy="median")),
    ("scaler", StandardScaler())
])

# 범주형 변수 파이프라인 전처리기
categorical_pipe = Pipeline([
    ("imputer", SimpleImputer(strategy="most_frequent")),
    ("onehot", OneHotEncoder(
        handle_unknown="ignore"
    ))
])

# 수치형 + 범주형 변수 통합 전처리기
preprocessor = ColumnTransformer([
    ("num", numeric_pipe, numeric_features),
    ("cat", categorical_pipe, categorical_features)
])

# 불균형 데이터 파이프라인 
model_pipe = ImbPipeline([
    ("preprocessor", preprocessor),
    ("sampler", RandomOverSampler(random_state=42)),
    ("classifier", RandomForestClassifier(
        n_estimators=300,
        random_state=42
    ))
])

# 통합적으로 생성한 불균형 파이프라인에 Train 데이터 학습 
model_pipe.fit(X_train, y_train) # 'classifier'

# Test 데이터 예측
pred = model_pipe.predict(X_test) # 'classifier'

print(pred[:5])

# 새로운 대상자 데이터
new_player = pd.DataFrame({
    "age" : [36],
    "bmi" : [29],
    "grip_strength" : [30],
    "sit_and_reach_cm" : [15],
    "vo2max" : [33]
})

# Raw Data를 그대로 입력
new_pred = pipeline.predict(new_player)

print("\n[새로운 대상자 예측]")
print(new_prediction)
```

[ Result ]

```text
[1 0 0 1 1]

[새로운 대상자 예측]
0
```

이 Pipeline 하나에

```text
결측치 처리
      ↓
Scaling
      ↓
One-Hot Encoding
      ↓
Oversampling
      ↓
Random Forest
```

가 모두 포함되어 있다.

<br>

> ## **6-2) 전체 흐름도**

```text
Raw Data
    ↓
ColumnTransformer
    │
    ├─ Numeric Features
    │      ↓
    │   Imputation
    │      ↓
    │   Scaling
    │
    └─ Categorical Features
           ↓
       Imputation
           ↓
       One-Hot Encoding
              │
              ↓
      Feature Matrix 결합
              ↓
         Resampling
              ↓
          Classifier
              ↓
         Prediction
```


<br><br>

---

# **7. Pipeline과 Cross Validation**

> ## **7-1) Pipeline 전체를 CV 대상으로 사용**

Pipeline을 구축한 가장 큰 장점 중 하나는 Pipeline 전체를 `cross_val_score()`에 전달할 수 있다는 것이다.

[ Python : code ] "운동처방 Pipeline을 Stratified 5-Fold CV로 평가"

```python
from sklearn.model_selection import StratifiedKFold, cross_val_score

cv = StratifiedKFold(
    n_splits=5,
    shuffle=True,
    random_state=42
)

scores = cross_val_score(
    model_pipe,
    X_train,
    y_train,
    scoring="f1",
    cv=cv,
    n_jobs=-1
)

print("Fold F1:", scores.round(3))
print("Mean F1:", round(scores.mean(), 3))
print("Std F1:", round(scores.std(), 3))
```

[ Result ]

```text
Fold F1: [0.741 0.759 0.752 0.768 0.747]

Mean F1: 0.753
Std F1: 0.009
```

각 Fold 내부에서는 다음 과정이 독립적으로 반복된다.

```text
Fold Training
      ↓
Imputer.fit()
      ↓
Scaler.fit()
      ↓
OneHotEncoder.fit()
      ↓
Oversampling
      ↓
Model.fit()
      ↓
Fold Validation
      ↓
transform만 수행
      ↓
Prediction
```

이 구조가 Leakage 방지의 핵심이다.


<br><br>

---

# **8. Pipeline Hyperparameter 접근 방법**

> ## **8-1) 단계이름__파라미터명**

Pipeline 안에 들어간 모델의 Hyperparameter를 변경하려면 다음 형식을 사용한다.

```text
단계이름__Hyperparameter
```

Pipeline에서 모델 단계 이름이

```python
"classifier"
```

라면 Random Forest의 `max_depth`는

```python
classifier__max_depth
```

가 된다.

`n_estimators`는

```python
classifier__n_estimators
```

이다.

예를 들어

```python
model_pipe.set_params(
    classifier__n_estimators=500,
    classifier__max_depth=8
)
```

처럼 사용할 수 있다.

[ python : code ] "Pipeline과 하이퍼파라미터 탐색의 결합(승패 예측)"

| 변수 | 의미 | 종류 |
|---|---|---|
| `avg_score` | 최근 경기 평균 득점 | 수치형 |
| `avg_conceded` | 최근 경기 평균 실점 | 수치형 |
| `recent_win_rate` | 최근 경기 승률 | 수치형 |
| `venue` | 홈 / 원정 | 범주형 |
| `opponent_level` | 상대팀 수준 | 범주형 |
| `win` | 승리 1 / 패배 0 | Target |

```python
import pandas as pd
import numpy as np

from sklearn.model_selection import train_test_split, GridSearchCV
from sklearn.impute import SimpleImputer
from sklearn.preprocessing import StandardScaler, OneHotEncoder
from sklearn.compose import ColumnTransformer
from sklearn.pipeline import Pipeline
from sklearn.linear_model import LogisticRegression


# =========================================================
# 1. 경기 데이터 생성
# =========================================================

data = pd.DataFrame({
    # 최근 5경기 평균 득점
    "avg_score": [
        82, 75, 91, 88, 70,
        85, 78, 73, 95, 89,
        76, 92, 80, 72, 90,
        79, 87, 74, 86, 93,
        77, 84, 71, 96, 83,
        69, 88, 81, 94, 75
    ],

    # 최근 5경기 평균 실점
    "avg_conceded": [
        76, 80, 83, 79, 85,
        78, 82, 84, 81, 77,
        83, 80, 79, 86, 82,
        81, 76, 85, 78, 79,
        84, 80, 87, 82, 81,
        88, 77, 80, 79, 83
    ],

    # 최근 5경기 승률
    "recent_win_rate": [
        0.8, 0.4, 0.8, 0.6, 0.2,
        0.8, 0.4, 0.2, 1.0, 0.8,
        0.4, 0.8, 0.6, 0.2, 0.8,
        0.4, 0.8, 0.2, 0.6, 1.0,
        0.4, 0.6, 0.2, 1.0, 0.6,
        0.2, 0.8, 0.6, 1.0, 0.4
    ],

    # 홈 / 원정
    "venue": [
        "Home", "Away", "Home", "Away", "Away",
        "Home", "Away", "Away", "Home", "Home",
        "Away", "Home", "Home", "Away", "Home",
        "Away", "Home", "Away", "Home", "Home",
        "Away", "Home", "Away", "Home", "Away",
        "Away", "Home", "Home", "Home", "Away"
    ],

    # 상대팀 수준
    "opponent_level": [
        "Medium", "Strong", "Strong", "Medium", "Weak",
        "Medium", "Strong", "Medium", "Strong", "Medium",
        "Weak", "Strong", "Medium", "Strong", "Medium",
        "Weak", "Strong", "Medium", "Strong", "Medium",
        "Weak", "Medium", "Strong", "Strong", "Medium",
        "Weak", "Medium", "Strong", "Medium", "Strong"
    ],

    # 결과
    # 승리 = 1
    # 패배 = 0
    "win": [
        1, 0, 1, 1, 0,
        1, 0, 0, 1, 1,
        0, 1, 1, 0, 1,
        0, 1, 0, 1, 1,
        0, 1, 0, 1, 1,
        0, 1, 0, 1, 0
    ]
})


# =========================================================
# 2. 일부 결측치 생성
# =========================================================

data.loc[2, "avg_score"] = np.nan
data.loc[7, "recent_win_rate"] = np.nan
data.loc[5, "opponent_level"] = np.nan


# =========================================================
# 3. Feature / Target 분리
# =========================================================

X = data.drop(columns="win")
y = data["win"]


# =========================================================
# 4. Train / Test 분리
# =========================================================

X_train, X_test, y_train, y_test = train_test_split(
    X,
    y,
    test_size=0.2,
    random_state=42,
    stratify=y
)


# =========================================================
# 5. Feature 종류 구분
# =========================================================

numeric_features = [
    "avg_score",
    "avg_conceded",
    "recent_win_rate"
]

categorical_features = [
    "venue",
    "opponent_level"
]


# =========================================================
# 6. 수치형 Feature 전처리
# =========================================================

numeric_pipeline = Pipeline([
    ("imputer", SimpleImputer(strategy="median")),
    ("scaler", StandardScaler())
])


# =========================================================
# 7. 범주형 Feature 전처리
# =========================================================

categorical_pipeline = Pipeline([
    ("imputer", SimpleImputer(strategy="most_frequent")),
    ("encoder", OneHotEncoder(handle_unknown="ignore"))
])


# =========================================================
# 8. 전처리 결합
# =========================================================

preprocessor = ColumnTransformer([
    ("numeric", numeric_pipeline, numeric_features),
    ("categorical", categorical_pipeline, categorical_features)
])


# =========================================================
# 9. 전처리 + Logistic Regression Pipeline + 모델의 하이퍼파라미터 변경
# =========================================================

pipeline = Pipeline([
    ("preprocessor", preprocessor),
    ("model", LogisticRegression(max_iter=1000))
])


# 파라미터 변경
pipeline.set_params(
    model__C=10,
    model__max_iter=2000
)

print(pipeline)


# =========================================================
# 10. 탐색할 하이퍼파라미터
# =========================================================

param_grid = {"model__C": [0.01, 0.1, 1, 10, 100]}


# =========================================================
# 11. GridSearchCV
# =========================================================

grid_search = GridSearchCV(
    estimator=pipeline,
    param_grid=param_grid,
    cv=3,
    scoring="accuracy"
)


# =========================================================
# 12. 학습
# =========================================================

grid_search.fit(X_train, y_train)


# =========================================================
# 13. 결과 확인
# =========================================================

print("최적 하이퍼파라미터")
print(grid_search.best_params_)

print("\n최고 CV 평균 Accuracy")
print(grid_search.best_score_)

print("\nTest Accuracy")
print(grid_search.score(X_test, y_test))


# =========================================================
# 14. 최종 Pipeline
# =========================================================

best_pipeline = grid_search.best_estimator_

print("\n최종 Pipeline")
print(best_pipeline)


# =========================================================
# 15. 새로운 경기 승패 예측
# =========================================================

new_game = pd.DataFrame({
    "avg_score": [89],
    "avg_conceded": [80],
    "recent_win_rate": [0.8],
    "venue": ["Home"],
    "opponent_level": ["Medium"]
})

# 승패 예측
prediction = best_pipeline.predict(new_game)

# 승리 / 패배 확률
prediction_probability = best_pipeline.predict_proba(new_game)

print("\n새로운 경기 예측")

if prediction[0] == 1:
    print("예측 결과: 승리")
else:
    print("예측 결과: 패배")


print(
    "패배 확률:",
    prediction_probability[0][0]
)

print(
    "승리 확률:",
    prediction_probability[0][1]
)
```
[Result]

```text
[최적 하이퍼파라미터]
{'model__C': 0.1}

[최고 CV 평균 Accuracy]
0.9583333333333334

[Test Accuracy]
1.0

[최종 Pipeline]
Pipeline(steps=[('preprocessor',
                 ColumnTransformer(transformers=[('numeric',
                                                  Pipeline(steps=[('imputer',
                                                                   SimpleImputer(strategy='median')),
                                                                  ('scaler',
                                                                   StandardScaler())]),
                                                  ['avg_score', 'avg_conceded',
                                                   'recent_win_rate']),
                                                 ('categorical',
                                                  Pipeline(steps=[('imputer',
                                                                   SimpleImputer(strategy='most_frequent')),
                                                                  ('encoder',
                                                                   OneHotEncoder(handle_unknown='ignore'))]),
                                                  ['venue',
                                                   'opponent_level'])])),
                ('model', LogisticRegression(C=0.1, max_iter=2000))])

[새로운 경기 예측]
예측 결과: 승리
패배 확률: 0.18797445704848115
승리 확률: 0.8120255429515189
```



<br>

> ## **8-2) 여러 단계의 Parameter에 접근**

| Pipeline 단계 | 실제 Parameter 접근 |
| --- | --- |
| `classifier`의 `max_depth` | `classifier__max_depth` |
| `classifier`의 `n_estimators` | `classifier__n_estimators` |
| `sampler`의 sampling strategy | `sampler__sampling_strategy` |
| `preprocessor` 내부 수치형 imputer | `preprocessor__num__imputer__strategy` |
| 범주형 encoder | `preprocessor__cat__onehot__handle_unknown` |

Pipeline이 여러 단계로 중첩되어 있어도 `__`를 이용해 내부 Parameter까지 접근할 수 있다.


<br><br>

---

# **9. Optuna를 이용한 Pipeline 튜닝**

> ## **9-1) Optuna란?**

Grid Search는 모든 조합을 탐색하고 Random Search는 무작위 조합을 선택한다.

`Optuna`는 이전 Trial 결과를 활용하여 **성능이 좋을 가능성이 높은 영역을 효율적으로 탐색하는 Hyperparameter Optimization Framework**이다.

```text
Trial 1
   ↓
성능 확인

Trial 2
   ↓
성능 확인

Trial 3
   ↓
이전 Trial 정보를 활용

...

좋은 영역 집중 탐색
```

모든 경우에 반드시 Random Search보다 우수하다는 뜻은 아니지만, 탐색 공간이 크고 모델 학습 비용이 높은 상황에서 유용하다.

<br>

> ## **9-2) 축구 부상예측 Optuna 튜닝**

[ Python : code ] "Pipeline 전체를 Stratified CV와 Optuna로 튜닝"

```python
import optuna

from sklearn.base import clone
from sklearn.model_selection import StratifiedKFold, cross_val_score

cv = StratifiedKFold(
    n_splits=5,
    shuffle=True,
    random_state=42
)

def objective(trial):
    trial_pipe = clone(model_pipe)

    trial_pipe.set_params(
        classifier__n_estimators=trial.suggest_int(
            "n_estimators", 100, 600
        ),
        classifier__max_depth=trial.suggest_int(
            "max_depth", 3, 15
        ),
        classifier__min_samples_leaf=trial.suggest_int(
            "min_samples_leaf", 1, 8
        ),
        classifier__max_features=trial.suggest_categorical(
            "max_features",
            ["sqrt", "log2"]
        )
    )

    scores = cross_val_score(
        trial_pipe,
        X_train,
        y_train,
        scoring="f1",
        cv=cv,
        n_jobs=-1
    )

    return scores.mean()

study = optuna.create_study(direction="maximize")

study.optimize(objective, n_trials=30)

print(study.best_params)
print(round(study.best_value, 3))
```

[ Result ]

```text
{
 'n_estimators': 437,
 'max_depth': 9,
 'min_samples_leaf': 3,
 'max_features': 'sqrt'
}

Best CV F1: 0.781
```

여기서 중요한 점은 Optuna가 Test Data를 보지 않는다는 것이다.

```text
Optuna
→ X_train 내부 CV만 사용

X_test
→ 마지막 최종 평가까지 보관
```

<br>

> ## **9-3) 코드 흐름도**

```text
Optuna Trial 시작
        ↓
Hyperparameter 제안
        ↓
Pipeline Clone
        ↓
Pipeline Parameter 설정
        ↓
Stratified 5-Fold CV
        ↓
각 Fold 내부
        │
        ├─ Preprocessing
        ├─ Resampling
        └─ Model Training
        ↓
Mean F1 계산
        ↓
Optuna에 Score 반환
        ↓
다음 Trial
        ↓
30회 반복
        ↓
Best Hyperparameter 결정
```


<br><br>

---

# **10. 최적 Hyperparameter로 최종 Pipeline 학습**

> ## **10-1) Best Parameter 적용**

Optuna 탐색이 끝나면 최적 Parameter를 Pipeline에 적용한다.

[ Python : code ] "Optuna 최적값을 Pipeline에 적용하고 전체 Train Data로 재학습"

```python
from sklearn.base import clone

best_pipe = clone(model_pipe)

best_pipe.set_params(
    classifier__n_estimators=
        study.best_params["n_estimators"],

    classifier__max_depth=
        study.best_params["max_depth"],

    classifier__min_samples_leaf=
        study.best_params["min_samples_leaf"],

    classifier__max_features=
        study.best_params["max_features"]
)

best_pipe.fit(X_train, y_train)

print("Final Pipeline Training Complete")
```

[ Result ]

```text
Final Pipeline Training Complete
```

이때 전체 `X_train`에 대해

```text
전처리 학습
    ↓
Resampling
    ↓
최종 Model 학습
```

이 다시 수행된다.


<br><br>

---

# **11. Hold-out Test 최종 평가**

> ## **11-1) Test Data는 마지막에 사용한다**

최종 Pipeline이 결정된 뒤 처음부터 분리해 두었던 Hold-out Test를 사용한다.

[ Python : code ] "최종 운동처방 Pipeline의 Test 성능 평가"

```python
from sklearn.metrics import (
    accuracy_score,
    precision_score,
    recall_score,
    f1_score,
    confusion_matrix
)

test_pred = best_pipe.predict(X_test)

print(
    "Accuracy:",
    round(accuracy_score(y_test, test_pred), 3)
)

print(
    "Precision:",
    round(precision_score(y_test, test_pred), 3)
)

print(
    "Recall:",
    round(recall_score(y_test, test_pred), 3)
)

print(
    "F1:",
    round(f1_score(y_test, test_pred), 3)
)

print(
    "Confusion Matrix:\n",
    confusion_matrix(y_test, test_pred)
)
```

[ Result ]

```text
Accuracy: 0.862
Precision: 0.796
Recall: 0.744
F1: 0.769

Confusion Matrix:
[[1540  110]
 [ 154  446]]
```

이 결과에서 단순 Accuracy만 보지 않는다.

소수 Class 탐지가 중요한 문제라면

```text
Recall
F1-score
False Negative
```

도 함께 확인해야 한다.

<br>

> ## **11-2) Confusion Matrix 해석**

위 결과를 정리하면 다음과 같다.

| 구분 | 예측 0 | 예측 1 |
| --- | ---: | ---: |
| 실제 0 | 1540 | 110 |
| 실제 1 | 154 | 446 |

따라서

```text
True Negative  = 1540
False Positive = 110
False Negative = 154
True Positive  = 446
```

이다.

운동처방이 필요한 사람을 놓치는 것이 중요한 문제라면 특히

```text
False Negative = 154
```

를 주의해서 분석해야 한다.


<br><br>

---

# **12. 예제 데이터에서 Pipeline과 Group CV**

> ## **12-1) 동일 선수 반복 측정 문제**

스포츠 연구에서는 동일 선수가 여러 경기나 측정 세션에 반복적으로 등장할 수 있다.

예를 들어 보행 센서 데이터가 다음과 같다고 하자.

```text
Subject 01
→ Trial 1
→ Trial 2
→ Trial 3

Subject 02
→ Trial 1
→ Trial 2
→ Trial 3
```

일반 Random CV를 사용하면

```text
Train
→ Subject 01 Trial 1, 2

Validation
→ Subject 01 Trial 3
```

이 될 수 있다.

모델이 이미 Subject 01의 개인 특성을 학습했기 때문에 성능이 과대평가될 수 있다.

<br>

> ## **12-2) GroupKFold Pipeline**

[ Python : code ] "보행분석에서 피험자를 완전히 분리한 Pipeline 평가"

```python
from sklearn.model_selection import GroupKFold, cross_val_score

group_cv = GroupKFold(
    n_splits=5
)

scores = cross_val_score(
    model_pipe,
    X,
    y,
    groups=subject_id,
    cv=group_cv,
    scoring="f1"
)

print("Group CV:", scores.round(3))
print("Mean F1:", round(scores.mean(), 3))
```

[ Result ]

```text
Group CV: [0.742 0.718 0.755 0.731 0.726]

Mean F1: 0.734
```

일반 Random CV보다 점수가 낮아질 수 있지만, 연구 목적이

```text
처음 보는 새로운 선수에게
모델이 얼마나 잘 작동하는가?
```

라면 Group 단위 평가가 더 현실적인 성능 추정이 될 수 있다.


<br><br>

---

# **13. 시간 순서가 있는 데이터 Pipeline**

> ## **13-1) Wearable·IoT 데이터**

웨어러블 센서에서는 하루 또는 세션 단위 데이터가 시간 순서대로 축적된다.

예를 들어

```text
1월 → 2월 → 3월 → 4월 → 5월
```

데이터로 6월의 피로도를 예측한다고 가정한다.

Random Split을 하면 미래 정보가 과거 Training에 들어갈 수 있다.

잘못된 구조는

```text
Train
→ 1월, 3월, 5월

Validation
→ 2월, 4월
```

이다.

실제 운영에서는 5월 데이터를 이용해 2월을 예측할 수 없기 때문이다.

시간 구조를 유지하면

```text
Train → 1월
Valid → 2월

Train → 1~2월
Valid → 3월

Train → 1~3월
Valid → 4월
```

과 같은 평가가 가능하다.

이를 위해 `TimeSeriesSplit`을 고려할 수 있다.


<br><br>

---

# **14. SHAP을 이용한 최종 모델 해석**

> ## **14-1) Pipeline과 SHAP**

SHAP을 사용할 때도 Pipeline의 전처리를 무시하면 안 된다.

Pipeline 내부의 최종 Tree 모델을 해석하려면 학습된 Preprocessor로 데이터를 먼저 변환한 뒤 Feature 이름을 복원해야 할 수 있다.

```text
Raw Validation Data
        ↓
Fitted Preprocessor
        ↓
Transformed Features
        ↓
Final Tree Model
        ↓
SHAP
```

<br>

> ## **14-2) Feature 이름 추출**

[ Python : code ] "학습된 Pipeline의 변환 Feature 이름 확인"

```python
preprocessor = best_pipe.named_steps["preprocessor"]

feature_names = preprocessor.get_feature_names_out()

print(feature_names[:10])
```

[ Result ]

```text
[
 'num__age'
 'num__bmi'
 'num__grip_strength'
 'num__sit_and_reach_cm'
 'num__vo2max'
 'cat__sex_F'
 'cat__sex_M'
 'cat__life_stage_adult'
 'cat__life_stage_elderly'
 'cat__activity_level_high'
]
```

<br>

> ## **14-3) SHAP Global Importance**

Tree 기반 모델이라고 가정하면 다음과 같이 접근할 수 있다.

[ Python : code ] "최종 Random Forest 모델의 SHAP Global Importance 분석"

```python
import numpy as np
import pandas as pd
import shap

preprocessor = best_pipe.named_steps["preprocessor"]

classifier = best_pipe.named_steps["classifier"]

X_test_transformed = (preprocessor.transform(X_test)


feature_names = preprocessor.get_feature_names_out()

# SHAP
explainer = shap.TreeExplainer(classifier)

shap_values = explainer(X_test_transformed)

print(
    "Transformed Shape:",
    X_test_transformed.shape
)
```

[ Result ]

```text
Transformed Shape: (2250, 14)
```

SHAP 결과 형식은 SHAP 버전과 분류 출력 형태에 따라 차이가 있을 수 있으므로 배열의 Shape을 먼저 확인한 뒤 분석하는 것이 안전하다.

SHAP 결과는

```text
모델이 무엇을 이용해
이런 Prediction을 만들었는가?
```

를 설명하는 것이며 현실의 인과관계를 증명하는 것은 아니다.


<br><br>

---

# **15. 최종 Pipeline 저장**

> ## **15-1) 모델만 저장하면 안 되는 이유**

다음처럼 최종 Classifier만 저장했다고 가정한다.

```python
joblib.dump(
    classifier, 
    "model.pkl" 
)
```

그러면 새로운 데이터 예측 시

```text
결측치 처리 방법?
Scaling 평균?
Scaling 표준편차?
One-Hot Category 순서?
```

를 다시 정확하게 재현해야 한다.

따라서 실무에서는 **전처리를 포함한 전체 Pipeline을 저장하는 것이 훨씬 안전하다.**

<br>

> ## **15-2) 전체 Pipeline 저장**

[ Python : code ] "전처리와 모델이 포함된 최종 Pipeline 저장"

```python
import joblib

joblib.dump(
    best_pipe, # 저장할 파이프라인 설정
    "exercise_prescription_pipeline.pkl" # 저장할 파일 이름.pkl 
)
```

[ Result ]

```text
['exercise_prescription_pipeline.pkl']
```

이 파일에는

```text
Imputer
Scaler
OneHotEncoder
Resampling 이후 학습된 Classifier 구조
```

등 예측에 필요한 학습 상태가 포함된다.

리샘플러 자체는 예측 시 새로운 데이터를 늘리는 용도로 실행되지 않는다.

`.pkl` 파일을 모델과 함께 입력 스키마(metadata)를 같이 저장하는 방식을 사용하면 새로운 데이터에 넣어야할 컬럼과 정보를 알 수 있다.

[ python : code ] "승패 예측 모델 파일의 저장 입력 스키마"

```python
import pandas as pd
import numpy as np

from sklearn.model_selection import train_test_split, GridSearchCV
from sklearn.impute import SimpleImputer
from sklearn.preprocessing import StandardScaler, OneHotEncoder
from sklearn.compose import ColumnTransformer
from sklearn.pipeline import Pipeline
from sklearn.linear_model import LogisticRegression

# ▶ 모델 전처리기 파이프라인 + 모델 학습
# =========================================================
# 1. 경기 데이터 생성
# =========================================================

data = pd.DataFrame({
    # 최근 5경기 평균 득점
    "avg_score": [
        82, 75, 91, 88, 70,
        85, 78, 73, 95, 89,
        76, 92, 80, 72, 90,
        79, 87, 74, 86, 93,
        77, 84, 71, 96, 83,
        69, 88, 81, 94, 75
    ],

    # 최근 5경기 평균 실점
    "avg_conceded": [
        76, 80, 83, 79, 85,
        78, 82, 84, 81, 77,
        83, 80, 79, 86, 82,
        81, 76, 85, 78, 79,
        84, 80, 87, 82, 81,
        88, 77, 80, 79, 83
    ],

    # 최근 5경기 승률
    "recent_win_rate": [
        0.8, 0.4, 0.8, 0.6, 0.2,
        0.8, 0.4, 0.2, 1.0, 0.8,
        0.4, 0.8, 0.6, 0.2, 0.8,
        0.4, 0.8, 0.2, 0.6, 1.0,
        0.4, 0.6, 0.2, 1.0, 0.6,
        0.2, 0.8, 0.6, 1.0, 0.4
    ],

    # 홈 / 원정
    "venue": [
        "Home", "Away", "Home", "Away", "Away",
        "Home", "Away", "Away", "Home", "Home",
        "Away", "Home", "Home", "Away", "Home",
        "Away", "Home", "Away", "Home", "Home",
        "Away", "Home", "Away", "Home", "Away",
        "Away", "Home", "Home", "Home", "Away"
    ],

    # 상대팀 수준
    "opponent_level": [
        "Medium", "Strong", "Strong", "Medium", "Weak",
        "Medium", "Strong", "Medium", "Strong", "Medium",
        "Weak", "Strong", "Medium", "Strong", "Medium",
        "Weak", "Strong", "Medium", "Strong", "Medium",
        "Weak", "Medium", "Strong", "Strong", "Medium",
        "Weak", "Medium", "Strong", "Medium", "Strong"
    ],

    # 결과
    # 승리 = 1
    # 패배 = 0
    "win": [
        1, 0, 1, 1, 0,
        1, 0, 0, 1, 1,
        0, 1, 1, 0, 1,
        0, 1, 0, 1, 1,
        0, 1, 0, 1, 1,
        0, 1, 0, 1, 0
    ]
})


# =========================================================
# 2. 일부 결측치 생성
# =========================================================

data.loc[2, "avg_score"] = np.nan
data.loc[7, "recent_win_rate"] = np.nan
data.loc[5, "opponent_level"] = np.nan


# =========================================================
# 3. Feature / Target 분리
# =========================================================

X = data.drop(columns="win")
y = data["win"]


# =========================================================
# 4. Train / Test 분리
# =========================================================

X_train, X_test, y_train, y_test = train_test_split(
    X,
    y,
    test_size=0.2,
    random_state=42,
    stratify=y
)


# =========================================================
# 5. Feature 종류 구분
# =========================================================

numeric_features = [
    "avg_score",
    "avg_conceded",
    "recent_win_rate"
]

categorical_features = [
    "venue",
    "opponent_level"
]


# =========================================================
# 6. 수치형 Feature 전처리
# =========================================================

numeric_pipeline = Pipeline([
    ("imputer", SimpleImputer(strategy="median")),
    ("scaler", StandardScaler())
])


# =========================================================
# 7. 범주형 Feature 전처리
# =========================================================

categorical_pipeline = Pipeline([
    ("imputer", SimpleImputer(strategy="most_frequent")),
    ("encoder", OneHotEncoder(handle_unknown="ignore"))
])


# =========================================================
# 8. 전처리 결합
# =========================================================

preprocessor = ColumnTransformer([
    ("numeric", numeric_pipeline, numeric_features),
    ("categorical", categorical_pipeline, categorical_features)
])


# =========================================================
# 9. 전처리 + Logistic Regression Pipeline + 모델의 하이퍼파라미터 변경
# =========================================================

pipeline = Pipeline([
    ("preprocessor", preprocessor),
    ("model", LogisticRegression(max_iter=1000))
])


# 파라미터 변경
pipeline.set_params(
    model__C=10,
    model__max_iter=2000
)

print(pipeline)


# =========================================================
# 10. 탐색할 하이퍼파라미터
# =========================================================

param_grid = {"model__C": [0.01, 0.1, 1, 10, 100]}


# =========================================================
# 11. GridSearchCV
# =========================================================

grid_search = GridSearchCV(
    estimator=pipeline,
    param_grid=param_grid,
    cv=3,
    scoring="accuracy"
)


# =========================================================
# 12. 학습
# =========================================================

grid_search.fit(X_train, y_train)


# =========================================================
# 13. 결과 확인
# =========================================================

print("최적 하이퍼파라미터")
print(grid_search.best_params_)

print("\n최고 CV 평균 Accuracy")
print(grid_search.best_score_)

print("\nTest Accuracy")
print(grid_search.score(X_test, y_test))


# =========================================================
# 14. 최종 Pipeline
# =========================================================

best_pipeline = grid_search.best_estimator_

print("\n최종 Pipeline")
print(best_pipeline)


# =========================================================
# 15. 새로운 경기 승패 예측
# =========================================================

new_game = pd.DataFrame({
    "avg_score": [89],
    "avg_conceded": [80],
    "recent_win_rate": [0.8],
    "venue": ["Home"],
    "opponent_level": ["Medium"]
})

# 승패 예측
prediction = best_pipeline.predict(new_game)

# 승리 / 패배 확률
prediction_probability = best_pipeline.predict_proba(new_game)

print("\n새로운 경기 예측")

if prediction[0] == 1:
    print("예측 결과: 승리")
else:
    print("예측 결과: 패배")


print(
    "패배 확률:",
    prediction_probability[0][0]
)

print(
    "승리 확률:",
    prediction_probability[0][1]
)

# =========================
# ▶ 모델 파일 저장
# =========================
import joblib

# 만약 최적 모델을 best_pipeline으로 설정했다면,
best_pipeline = grid_search.best_estimator_

print(best_pipeline.feature_names_in_) # 원본 입력 컬럼


model_package = {
    "model": best_pipeline,

    # 피처명 저장
    "feature_columns": [
        "avg_score",
        "avg_conceded",
        "recent_win_rate",
        "venue",
        "opponent_level"
    ],

    # 수치형 변수 컬럼명 저장
    "numeric_features": [
        "avg_score",
        "avg_conceded",
        "recent_win_rate"
    ],

    # 범주형 변수 컬럼명 저장
    "categorical_features": [
        "venue",
        "opponent_level"
    ],

    # 변수 설명 저장
    "feature_description": {
        "avg_score": "최근 5경기 평균 득점",
        "avg_conceded": "최근 5경기 평균 실점",
        "recent_win_rate": "최근 5경기 승률 (0~1)",
        "venue": "Home 또는 Away",
        "opponent_level": "Weak / Medium / Strong"
    },

    "target": {0: "패배", 1: "승리"}
}

# .pkl 파일 저장
joblib.dump(
    model_package,
    "win_prediction_model.pkl"
)
```

<br>

> ## **15-3) 저장한 Pipeline 불러오기**

> ### **15-3-1) Pipeline 불러오기**

[ Python : code ] "저장된 Pipeline 불러오기"

```python
import joblib

model = joblib.load(
    "win_prediction_model.pkl"
)

print(model)
```

[ Result ] "기본 예시"

```text
Pipeline(
    steps=[
        ('preprocessor', ColumnTransformer(...)),
        ('model', LogisticRegression(...))
    ]
)
```

[ Result ] "win_prediction_model.pkl 파일에 저장된 파이프라인"

```text
{'model': Pipeline(steps=[('preprocessor',
                 ColumnTransformer(transformers=[('numeric',
                                                  Pipeline(steps=[('imputer',
                                                                   SimpleImputer(strategy='median')),
                                                                  ('scaler',
                                                                   StandardScaler())]),
                                                  ['avg_score', 'avg_conceded',
                                                   'recent_win_rate']),
                                                 ('categorical',
                                                  Pipeline(steps=[('imputer',
                                                                   SimpleImputer(strategy='most_frequent')),
                                                                  ('encoder',
                                                                   OneHotEncoder(handle_unknown='ignore'))]),
                                                  ['venue',
                                                   'opponent_level'])])),
                ('model', LogisticRegression(C=0.1, max_iter=2000))]), 'feature_columns': ['avg_score', 'avg_conceded', 'recent_win_rate', 'venue', 'opponent_level'], 'numeric_features': ['avg_score', 'avg_conceded', 'recent_win_rate'], 'categorical_features': ['venue', 'opponent_level'], 'feature_description': {'avg_score': '최근 5경기 평균 득점', 'avg_conceded': '최근 5경기 평균 실점', 'recent_win_rate': '최근 5경기 승률 (0~1)', 'venue': 'Home 또는 Away', 'opponent_level': 'Weak / Medium / Strong'}, 'target': {0: '패배', 1: '승리'}}
```

<br>

> ### **15-3-2) 학습에 사용한 컬럼 확인**

[ Python : code ] "Pipeline 학습에 사용된 Feature 컬럼 확인"

```python
print(model.feature_names_in_)
```

[ Result ]

```text
[
    'avg_score'
    'avg_conceded'
    'recent_win_rate'
    'venue'
    'opponent_level'
]
```

저장된 Pipeline의 `feature_names_in_`을 사용하면 모델 학습에 사용된 원본 컬럼을 확인할 수 있다.

<br>

> ### **15-3-3) 수치형 / 범주형 컬럼 확인**

[ Python : code ] "ColumnTransformer에 설정된 컬럼 확인"

```python
preprocessor = model.named_steps["preprocessor"]

for name, transformer, columns in preprocessor.transformers_:
    print(name)
    print(columns)
```

[ Result ]

```text
numeric
['avg_score', 'avg_conceded', 'recent_win_rate']

categorical
['venue', 'opponent_level']
```

Pipeline 내부의 `ColumnTransformer`를 확인하면 어떤 Feature가 수치형이고 어떤 Feature가 범주형인지 확인할 수 있다.

<br>

> ### **15-3-4) 범주형 Feature 학습 값 확인**

[ Python : code ] "OneHotEncoder가 학습한 범주 확인"

```python
encoder = (
    model
    .named_steps["preprocessor"]
    .named_transformers_["categorical"]
    .named_steps["encoder"]
)

print(encoder.categories_)
```

[ Result ]

```text
[
    ['Away', 'Home'],
    ['Medium', 'Strong', 'Weak']
]
```

범주형 Feature에 어떤 값들이 사용되었는지도 저장된 `OneHotEncoder`를 통해 확인할 수 있다.

```text
venue
→ Away / Home

opponent_level
→ Medium / Strong / Weak
```

<br>

> ### **15-3-5) 새로운 경기 데이터 생성**

[ Python : code ] "Pipeline에 입력할 새로운 경기 데이터"

```python
import pandas as pd

new_subjects = pd.DataFrame({
    "avg_score": [
        88,
        74,
        92,
        70
    ],
    "avg_conceded": [
        79,
        85,
        80,
        87
    ],
    "recent_win_rate": [
        0.8,
        0.4,
        1.0,
        0.2
    ],
    "venue": [
        "Home",
        "Away",
        "Home",
        "Away"
    ],
    "opponent_level": [
        "Medium",
        "Strong",
        "Medium",
        "Strong"
    ]
})

print(new_subjects)
```

[ Result ]

```text
   avg_score  avg_conceded  recent_win_rate venue opponent_level
0         88            79              0.8  Home         Medium
1         74            85              0.4  Away         Strong
2         92            80              1.0  Home         Medium
3         70            87              0.2  Away         Strong
```

<br>

> ### **15-3-6) 저장된 Pipeline으로 새로운 승/패 예측**

[ Python : code ] "저장된 Pipeline으로 새로운 승/패 예측"

```python
new_pred = model.predict(new_subjects)

print(new_pred)
```

[ Result ]

```text
[1 0 1 0]
```

```text
1 → 승리
0 → 패배
```

Pipeline 전체를 저장했기 때문에 새로운 데이터는 `StandardScaler`나 `OneHotEncoder`로 직접 변환하지 않고 원본 Feature 형태로 입력하면 된다.

```text
새로운 경기 데이터
        ↓
결측치 처리
        ↓
StandardScaler
        ↓
OneHotEncoder
        ↓
LogisticRegression
        ↓
승 / 패 예측
```

<br>

> ### **15-3-7) 승리 확률 확인**

[ Python : code ] "각 경기의 승리 확률 확인"

```python
new_proba = model.predict_proba(new_subjects)

print(new_proba)
```

[ Result ]

```text
[[0.25 0.75]
 [0.71 0.29]
 [0.18 0.82]
 [0.80 0.20]]
```

각 행에서

```text
첫 번째 값 → 패배 확률
두 번째 값 → 승리 확률
```

을 의미한다.

승리 확률만 확인하려면 다음과 같이 사용할 수 있다.

[ Python : code ] "승리 확률만 추출"

```python
win_probability = model.predict_proba(new_subjects)[:, 1]


print(win_probability)
```

[ Result ]

```text
[0.75 0.29 0.82 0.20]
```

<br>

> ### **15-3-8) 모델과 입력 정보 함께 저장**

`.pkl` 파일에 Pipeline뿐만 아니라 입력 컬럼에 대한 설명도 함께 저장할 수 있다.

[ Python : code ] "Pipeline과 입력 Feature 정보 함께 저장"

```python
model_package = {
    "model": best_pipeline,

    "feature_columns": [
        "avg_score",
        "avg_conceded",
        "recent_win_rate",
        "venue",
        "opponent_level"
    ],

    "numeric_features": [
        "avg_score",
        "avg_conceded",
        "recent_win_rate"
    ],

    "categorical_features": [
        "venue",
        "opponent_level"
    ],

    "feature_description": {
        "avg_score":
            "최근 5경기 평균 득점",

        "avg_conceded":
            "최근 5경기 평균 실점",

        "recent_win_rate":
            "최근 5경기 승률 (0~1)",

        "venue":
            "Home 또는 Away",

        "opponent_level":
            "Weak / Medium / Strong"
    },

    "target": {
        0: "패배",
        1: "승리"
    }
}

joblib.dump(
    model_package,
    "win_prediction_model.pkl"
)
```

[ Result ]

```text
['win_prediction_model.pkl']
```

<br>

> ### **15-3-9) 모델 Package 불러오기**

[ Python : code ] "Pipeline과 Feature 정보 함께 불러오기"

```python
package = joblib.load("win_prediction_model.pkl")

model = package["model"]

print(package["feature_columns"])

print(package["feature_description"])

print(package["target"])
```

[ Result ]

```text
[
    'avg_score',
    'avg_conceded',
    'recent_win_rate',
    'venue',
    'opponent_level'
]

{
    'avg_score': '최근 5경기 평균 득점',
    'avg_conceded': '최근 5경기 평균 실점',
    'recent_win_rate': '최근 5경기 승률 (0~1)',
    'venue': 'Home 또는 Away',
    'opponent_level': 'Weak / Medium / Strong'
}

{
    0: '패배',
    1: '승리'
}
```

Pipeline만 저장하면 어떤 컬럼을 사용했는지는 확인할 수 있지만, 각 컬럼이 어떤 의미인지까지 완전히 알기는 어렵다.

따라서 실제 모델을 다시 사용해야 하는 경우에는 다음과 같이 저장하는 것이 좋다.

```text
win_prediction_model.pkl
│
├── model
│   ├── 전처리 Pipeline
│   └── LogisticRegression
│
├── feature_columns
│
├── numeric_features
│
├── categorical_features
│
├── feature_description
│
└── target
```

<br>

> ### **15-3-10) 입력 컬럼 검증**

새로운 데이터가 모델이 요구하는 컬럼을 모두 가지고 있는지 예측 전에 확인할 수 있다.

[ Python : code ] "필수 Feature 존재 여부 확인"

```python
required_columns = package[
    "feature_columns"
]

missing_columns = [
    column
    for column in required_columns
    if column not in new_subjects.columns
]

if missing_columns:
    raise ValueError(f"필요한 컬럼이 없습니다: {missing_columns}")
```

필요한 컬럼만 학습 당시 순서대로 정렬할 수도 있다.

[ Python : code ] "입력 Feature 순서 맞추기"

```python
new_subjects = new_subjects[package["feature_columns"]]

new_pred = model.predict(new_subjects)

print(new_pred)
```

[ Result ]

```text
[1 0 1 0]
```

Pipeline 전체를 저장하면 학습 당시 사용했던 전처리 과정도 함께 저장되기 때문에 새로운 데이터에 별도로 `StandardScaler`, `SimpleImputer`, `OneHotEncoder`를 적용할 필요가 없다.

```text
학습할 때
원본 데이터
    ↓
Pipeline
    ↓
전처리
    ↓
모델 학습
    ↓
.pkl 저장


예측할 때
원본 데이터
    ↓
.pkl Pipeline 불러오기
    ↓
동일한 전처리 자동 적용
    ↓
승 / 패 예측
```




<br><br>

---

# **16. 신규 데이터 예측 함수**

> ## **16-1) 운영용 함수 만들기**

실무에서는 새로운 DataFrame을 입력하면 바로 Prediction을 반환하도록 함수를 만들 수 있다.

[ Python : code ] "입력 Feature 검증과 예측 확률을 포함한 운영용 함수"

```python
def predict_exercise_prescription(pipeline, new_data):

    required_columns = [
        "age",
        "bmi",
        "grip_strength",
        "sit_and_reach_cm",
        "vo2max",
        "sex",
        "life_stage",
        "activity_level"
    ]

    missing = [
        col for col in required_columns
        if col not in new_data.columns
    ]

    if missing:
        raise ValueError(f"Missing columns: {missing}")

    data = new_data[required_columns].copy()

    pred = pipeline.predict(data)
    prob = pipeline.predict_proba(data)[:, 1]

    result = data.copy()
    result["prediction"] = pred
    result["probability"] = prob

    return result
```

실행 예시는

```python
result = predict_exercise_prescription(pipeline, new_subjects)

print(result[["prediction", "probability"]])
```

[ Result ]

```text
   prediction  probability
0           1        0.842
1           0        0.231
2           1        0.763
3           0        0.184
```

이렇게 하면 입력 Feature 누락 여부도 먼저 확인할 수 있다.


<br><br>

---

# **17. 재현 가능한 프로젝트를 위한 Metadata**

> ## **17-1) random_state만 고정하면 충분한가?**

`random_state`를 고정하는 것은 중요하지만 이것만으로 모든 환경에서 완전히 동일한 결과가 보장되는 것은 아니다.

결과에는 다음 요소도 영향을 줄 수 있다.

```text
Python Version
scikit-learn Version
imbalanced-learn Version
Optuna Version
NumPy Version
운영체제
병렬 연산
사용 알고리즘
Hardware
```

따라서 연구 재현성을 높이려면 환경 정보도 함께 기록하는 것이 좋다.

<br>

> ## **17-2) 관리해야할 정보**

| 항목 | 예 |
| --- | --- |
| Project | Exercise Prescription Prediction |
| Model | RandomForestClassifier |
| Data Version | v1.3 |
| Feature Version | v2 |
| CV | StratifiedKFold, 5 folds |
| Metric | F1 |
| Random Seed | 42 |
| Tuning | Optuna, 30 Trials |
| Python | 프로젝트 사용 버전 |
| sklearn | 프로젝트 사용 버전 |
| Pipeline Version | v1.0 |
| Test F1 | 0.769 |

[ Python : code ] "최종 모델 Metadata 구성"

```python
metadata = {
    "model": "RandomForestClassifier",
    "cv": "StratifiedKFold(n_splits=5)",
    "metric": "f1",
    "random_state": 42,
    "optimizer": "Optuna",
    "n_trials": 30,
    "cv_best_f1": round(study.best_value, 3),
    "test_f1": 0.769,
    "best_params": study.best_params
}

print(metadata)
```

[ Result ]

```text
{
 'model': 'RandomForestClassifier',
 'cv': 'StratifiedKFold(n_splits=5)',
 'metric': 'f1',
 'random_state': 42,
 'optimizer': 'Optuna',
 'n_trials': 30,
 'cv_best_f1': 0.781,
 'test_f1': 0.769,
 'best_params': {
     'n_estimators': 437,
     'max_depth': 9,
     'min_samples_leaf': 3,
     'max_features': 'sqrt'
 }
}
```


<br><br>

---

# **18. Pipeline 방법을 기술하는 예**

> ## **18-1) 연구 절차**

논문에서 머신러닝 분석 절차는 다음처럼 구조화할 수 있다.

```text
Raw Data
    ↓
Train / Test Split
    ↓
Training Data
    ↓
Cross Validation
    ↓
각 Training Fold에서
    │
    ├─ Missing Value Imputation
    ├─ Scaling
    ├─ Encoding
    ├─ Resampling
    └─ Model Training
    ↓
Validation Fold 평가
    ↓
Hyperparameter Optimization
    ↓
Best Model 선정
    ↓
전체 Training Data Refit
    ↓
Hold-out Test
    ↓
최종 성능 평가
    ↓
SHAP Interpretation
```

<br>

> ## **18-2) 연구 결과 보고 시 포함할 항목**

| 구분 | 보고 내용 |
| --- | --- |
| 데이터 분할 | Train / Test 비율 |
| CV | Fold 수와 방식 |
| 전처리 | 결측치, Scaling, Encoding |
| 불균형 처리 | SMOTE, class weight 등 |
| 탐색 방법 | Grid, Random, Optuna |
| 평가 지표 | F1, Recall, ROC-AUC 등 |
| 최적 Parameter | 최종 Hyperparameter |
| CV 성능 | Mean ± SD |
| Test 성능 | 독립 Hold-out 결과 |
| 해석 | Feature Importance, SHAP |
| 재현성 | Random Seed, Software Version |


<br><br>

---

# **19. End-to-End Pipeline 전체 흐름**

> ## **19-1) 실무 프로젝트 전체 구조**

```text
Raw Dataset
      ↓
Schema / Feature 확인
      ↓
Train / Hold-out Test Split
      │
      └──────── Test 보관
      ↓
Training Dataset
      ↓
Cross Validation
      ↓
각 Fold 내부 Pipeline
      │
      ├─ Missing Value Imputation
      │
      ├─ Numerical Scaling
      │
      ├─ Categorical Encoding
      │
      ├─ Resampling
      │
      └─ Model Training
      ↓
Validation Performance
      ↓
Optuna / Grid / Random Search
      ↓
Best Hyperparameter
      ↓
전체 Train Data Refit
      ↓
Hold-out Test
      ↓
Performance Report
      │
      ├─ Accuracy
      ├─ Precision
      ├─ Recall
      ├─ F1
      └─ Confusion Matrix
      ↓
Model Interpretation
      │
      ├─ Feature Importance
      └─ SHAP
      ↓
전체 Pipeline 저장
      ↓
Metadata / Version 저장
      ↓
New Data
      ↓
pipeline.predict()
```


<br><br>

---

# **20. Pipeline 사용 전후 비교**

> ## **20-1) 직접 전처리와 Pipeline 비교**

| 구분 | 직접 처리 | Pipeline |
| --- | --- | --- |
| 전처리 순서 | 코드 관리 필요 | 순서 고정 |
| CV 내부 전처리 | 직접 구현 필요 | 자동 처리 가능 |
| Test Leakage 위험 | 상대적으로 높음 | 올바른 사용 시 감소 |
| 신규 데이터 처리 | 각 단계 직접 실행 | `predict()`로 연결 |
| Hyperparameter Tuning | 여러 객체 관리 | Pipeline 전체 탐색 |
| Resampling | 적용 위치 주의 필요 | imblearn Pipeline으로 관리 |
| 모델 저장 | 전처리 객체도 별도 저장 | 전체 Pipeline 저장 가능 |
| 재현성 | 관리가 복잡 | 상대적으로 높음 |
| 운영 적용 | 실수 가능성 증가 | 일관된 흐름 유지 |


<br><br>

---

# **21. Advanced 추가 학습**

> ## **21-1) Pipeline 내부 Feature Selection**

Feature Selection도 Pipeline 내부에 넣을 수 있다.

```text
Preprocessing
      ↓
Feature Selection
      ↓
Model
```

이렇게 해야 Feature Selection 과정에서도 Validation 정보가 Training에 들어가는 Leakage를 줄일 수 있다.

<br>

> ## **21-2) Nested Cross Validation**

데이터가 작고 연구에서 모델의 일반화 성능을 더 엄격하게 추정해야 한다면 Nested CV를 사용할 수 있다.

```text
Outer CV
→ 최종 성능 추정

Inner CV
→ Hyperparameter Optimization
```

Hyperparameter 탐색으로 인한 낙관적 Bias를 줄이는 데 유용하다.

<br>

> ## **21-3) Threshold Tuning**

불균형 분류에서는 기본 Threshold `0.5`가 항상 최적이지 않을 수 있다.

```text
Probability
      ↓
Decision Threshold
      ↓
Class Prediction
```

Recall이 매우 중요한 문제에서는 Validation Data를 이용하여 Threshold를 별도로 조정할 수 있다.

단, **Test Data를 보면서 Threshold를 선택하면 안 된다.**


<br><br>

---

# **22. 핵심정리**

1. **Pipeline은 전처리부터 모델 예측까지의 처리 순서를 하나의 객체로 연결하여 학습과 운영 환경에서 동일한 처리 과정을 재현하도록 한다.**

2. Pipeline을 사용한다고 무조건 Data Leakage가 사라지는 것은 아니다. 먼저 Train/Test를 적절하게 분리하고, **Pipeline의 `fit()`을 Training Data에만 적용하는 구조**가 필요하다.

3. Cross Validation에서 Pipeline을 사용하면 각 Fold의 Training 부분에서만 Imputation, Scaling, Encoding 등의 `fit()`이 수행되고 Validation에는 학습된 변환만 적용된다.

4. `ColumnTransformer`는 수치형과 범주형 Feature처럼 서로 다른 자료형에 **서로 다른 전처리 Pipeline을 적용한 뒤 결과를 하나로 결합**한다.

5. `OneHotEncoder(handle_unknown="ignore")`를 사용하면 학습 시 존재하지 않았던 새로운 Category가 운영 데이터에 나타났을 때 발생할 수 있는 오류를 줄일 수 있다.

6. Class Imbalance를 처리하기 위한 Resampling은 전체 데이터에 미리 적용하면 안 되고 **각 Training Fold 내부에서만 수행해야 한다.**

7. `imbalanced-learn Pipeline`을 사용하면

```text
Preprocessing
→ Resampling
→ Model
```

을 하나의 학습 과정으로 연결하고 Validation/Test에는 Resampling을 적용하지 않을 수 있다.

8. 모든 Feature가 수치형이라면 일반적인 `SMOTE`를 사용할 수 있지만, 수치형과 범주형이 함께 존재한다면 `SMOTENC` 또는 다른 적절한 방법을 검토해야 한다.

9. One-Hot Encoding된 범주형 변수에 일반 SMOTE를 무조건 적용하면 보간 과정에서 실제 Category를 직접 의미하지 않는 중간값이 생성될 수 있으므로 주의해야 한다.

10. Pipeline 내부 Hyperparameter는 다음 규칙으로 접근한다.

```text
단계이름__파라미터명
```

예를 들어

```text
classifier__max_depth
classifier__n_estimators
```

와 같이 표현한다.

11. Pipeline 전체를 `GridSearchCV`, `RandomizedSearchCV`, Optuna 등의 Hyperparameter 탐색 대상으로 사용할 수 있으며 **모든 전처리와 Resampling 과정도 CV Fold 내부에서 수행되어야 한다.**

12. Optuna 같은 최적화 도구를 사용하더라도 Test Data는 탐색 과정에 사용하지 않고, Train Data 내부의 Cross Validation 성능만 Objective로 사용해야 한다.

13. 최적 Hyperparameter가 결정되면 전체 Training Data에 Pipeline을 다시 학습한 뒤 **한 번도 모델 선택에 사용되지 않은 Hold-out Test에서 최종 성능을 평가**한다.

14. 불균형 데이터에서는 Accuracy만 사용하지 말고 문제 목적에 따라 다음 지표를 함께 확인해야 한다.

```text
Precision
Recall
F1-score
PR-AUC
ROC-AUC
Confusion Matrix
```

15. 동일 선수의 반복 측정값이 존재하는 스포츠과학 데이터에서는 일반 Random CV가 동일 선수 정보를 Train과 Validation에 동시에 배치할 수 있으므로 `GroupKFold`와 같은 Group 기반 분할을 고려해야 한다.

16. Wearable, GPS, IoT처럼 시간 순서가 중요한 데이터에서는 미래 정보가 Training에 포함되지 않도록 시간 구조를 유지한 Validation을 사용해야 한다.

17. 최종 모델을 저장할 때 Classifier 하나만 저장하기보다 **전처리가 포함된 전체 Pipeline을 저장하는 것이 운영 환경에서 더 안전하다.**

18. SHAP을 적용할 때도 Pipeline 내부 전처리를 고려해야 하며, One-Hot Encoding 후 생성된 Feature 이름과 변환된 데이터를 최종 모델에 맞게 연결해야 한다.

19. `random_state`를 고정하는 것은 재현성 확보에 중요하지만 이것만으로 모든 환경에서 완전히 동일한 결과가 보장되는 것은 아니다. Python과 Library Version, 데이터 버전, Feature 정의, 병렬 연산 조건 등도 함께 관리하는 것이 좋다.

20. 최종 모델과 함께 다음 Metadata를 관리하면 논문 재현성과 실제 운영 관리에 도움이 된다.

```text
Data Version
Feature Version
Preprocessing
Model
Hyperparameter
Cross Validation Method
Evaluation Metric
CV Mean ± SD
Hold-out Test Score
Random Seed
Library Version
Pipeline Version
```

21. 재현 가능한 머신러닝 프로젝트의 핵심 구조는 다음과 같이 정리할 수 있다.

```text
Raw Data
    ↓
Train / Hold-out Test Split
    ↓
Training Data
    ↓
Cross Validation
    ↓
각 Fold 내부
    │
    ├─ Imputation
    ├─ Scaling
    ├─ Encoding
    ├─ Resampling
    └─ Model Training
    ↓
Hyperparameter Optimization
    ↓
Best Configuration
    ↓
전체 Train Data Refit
    ↓
Hold-out Test
    ↓
Model Interpretation
    ↓
Pipeline + Metadata 저장
    ↓
New Data Prediction
```

22. 결국 End-to-End Pipeline의 핵심 목적은 단순히 코드를 짧게 만드는 것이 아니라 다음 세 가지를 동시에 확보하는 것이다.

```text
Data Leakage 방지
        +
동일한 처리 과정 재현
        +
연구·운영 환경의 일관성
        ↓
Reproducible Machine Learning Pipeline
```
