
# 💡 Data Science TIL

> **Today I Learned**

데이터 사이언티스트를 목표로 매일 새로 배운 내용을 TIL로 작성한다.

데이터 분석, 통계, 머신러닝, 컴퓨터 비전, SQL, 데이터베이스를 공부하면서 그날 이해한 개념과 코드를 Python, SQL 중심으로 직접 정리한다.

배운 내용을 내 말로,  내가 이해할 수 있는 예제로 다시 써 보며, 왜 사용하는지, 어떻게 동작하는지, 결과를 어떻게 해석하는지 스스로 설명할 수 있을 때까지 기록한다.

<br>

---

## 🛠 Tech Stack

<div align="center">

### Language

![Python](https://img.shields.io/badge/Python-3776AB?style=flat-square&logo=python&logoColor=white)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-4169E1?style=flat-square&logo=postgresql&logoColor=white)

### Data Analysis

![NumPy](https://img.shields.io/badge/NumPy-013243?style=flat-square&logo=numpy&logoColor=white)
![Pandas](https://img.shields.io/badge/Pandas-150458?style=flat-square&logo=pandas&logoColor=white)
![Matplotlib](https://img.shields.io/badge/Matplotlib-11557C?style=flat-square&logo=python&logoColor=white)
![Seaborn](https://img.shields.io/badge/Seaborn-4C72B0?style=flat-square&logo=python&logoColor=white)
![SciPy](https://img.shields.io/badge/SciPy-8CAAE6?style=flat-square&logo=scipy&logoColor=white)


### Statistics

![SciPy](https://img.shields.io/badge/SciPy-8CAAE6?style=flat-square&logo=scipy&logoColor=white)
![statsmodels](https://img.shields.io/badge/statsmodels-4051B5?style=flat-square&logo=python&logoColor=white)


### Machine Learning

![scikit-learn](https://img.shields.io/badge/scikit--learn-F7931E?style=flat-square&logo=scikitlearn&logoColor=white)

### Deep Learning

![TensorFlow](https://img.shields.io/badge/TensorFlow-FF6F00?style=flat-square&logo=tensorflow&logoColor=white)
![PyTorch](https://img.shields.io/badge/PyTorch-EE4C2C?style=flat-square&logo=pytorch&logoColor=white)

### Database

![PostgreSQL](https://img.shields.io/badge/PostgreSQL-4169E1?style=flat-square&logo=postgresql&logoColor=white)
![DBeaver](https://img.shields.io/badge/DBeaver-382923?style=flat-square&logo=dbeaver&logoColor=white)

### Generative AI

![Claude](https://img.shields.io/badge/Claude-D97757?style=flat-square&logo=claude&logoColor=white)
![ChatGPT](https://img.shields.io/badge/ChatGPT-000000?style=flat-square&logo=data%3Aimage%2Fsvg%2Bxml%3Bbase64%2CPHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHZpZXdCb3g9IjAgMCAyNTYgMjYwIj48cGF0aCBmaWxsPSIjZmZmIiBkPSJNMjM5LjE4NCAxMDYuMjAzYTY0LjcxNiA2NC43MTYgMCAwIDAtNS41NzYtNTMuMTAzQzIxOS40NTIgMjguNDU5IDE5MSAxNS43ODQgMTYzLjIxMyAyMS43NEE2NS41ODYgNjUuNTg2IDAgMCAwIDUyLjA5NiA0NS4yMmE2NC43MTYgNjQuNzE2IDAgMCAwLTQzLjIzIDMxLjM2Yy0xNC4zMSAyNC42MDItMTEuMDYxIDU1LjYzNCA4LjAzMyA3Ni43NGE2NC42NjUgNjQuNjY1IDAgMCAwIDUuNTI1IDUzLjEwMmMxNC4xNzQgMjQuNjUgNDIuNjQ0IDM3LjMyNCA3MC40NDYgMzEuMzZhNjQuNzIgNjQuNzIgMCAwIDAgNDguNzU0IDIxLjc0NGMyOC40ODEuMDI1IDUzLjcxNC0xOC4zNjEgNjIuNDE0LTQ1LjQ4MWE2NC43NjcgNjQuNzY3IDAgMCAwIDQzLjIyOS0zMS4zNmMxNC4xMzctMjQuNTU4IDEwLjg3NS01NS40MjMtOC4wODMtNzYuNDgzWm0tOTcuNTYgMTM2LjMzOGE0OC4zOTcgNDguMzk3IDAgMCAxLTMxLjEwNS0xMS4yNTVsMS41MzUtLjg3IDUxLjY3LTI5LjgyNWE4LjU5NSA4LjU5NSAwIDAgMCA0LjI0Ny03LjM2N3YtNzIuODVsMjEuODQ1IDEyLjYzNmMuMjE4LjExMS4zNy4zMi40MDkuNTYzdjYwLjM2N2MtLjA1NiAyNi44MTgtMjEuNzgzIDQ4LjU0NS00OC42MDEgNDguNjAxWm0tMTA0LjQ2Ni00NC42MWE0OC4zNDUgNDguMzQ1IDAgMCAxLTUuNzgxLTMyLjU4OWwxLjUzNC45MjEgNTEuNzIyIDI5LjgyNmE4LjMzOSA4LjMzOSAwIDAgMCA4LjQ0MSAwbDYzLjE4MS0zNi40MjV2MjUuMjIxYS44Ny44NyAwIDAgMS0uMzU4LjY2NWwtNTIuMzM1IDMwLjE4NGMtMjMuMjU3IDEzLjM5OC01Mi45NyA1LjQzMS02Ni40MDQtMTcuODAzWk0yMy41NDkgODUuMzhhNDguNDk5IDQ4LjQ5OSAwIDAgMSAyNS41OC0yMS4zMzN2NjEuMzlhOC4yODggOC4yODggMCAwIDAgNC4xOTUgNy4zMTZsNjIuODc0IDM2LjI3Mi0yMS44NDUgMTIuNjM2YS44MTkuODE5IDAgMCAxLS43NjcgMEw0MS4zNTMgMTUxLjUzYy0yMy4yMTEtMTMuNDU0LTMxLjE3MS00My4xNDQtMTcuODA0LTY2LjQwNXYuMjU2Wm0xNzkuNDY2IDQxLjY5NS02My4wOC0zNi42M0wxNjEuNzMgNzcuODZhLjgxOS44MTkgMCAwIDEgLjc2OCAwbDUyLjIzMyAzMC4xODRhNDguNiA0OC42IDAgMCAxLTcuMzE2IDg3LjYzNXYtNjEuMzkxYTguNTQ0IDguNTQ0IDAgMCAwLTQuNC03LjIxM1ptMjEuNzQyLTMyLjY5LTEuNTM1LS45MjItNTEuNjE5LTMwLjA4MWE4LjM5IDguMzkgMCAwIDAtOC40OTIgMEw5OS45OCA5OS44MDhWNzQuNTg3YS43MTYuNzE2IDAgMCAxIC4zMDctLjY2NWw1Mi4yMzMtMzAuMTMzYTQ4LjY1MiA0OC42NTIgMCAwIDEgNzIuMjM2IDUwLjM5MXYuMjA1Wk04OC4wNjEgMTM5LjA5N2wtMjEuODQ1LTEyLjU4NWEuODcuODcgMCAwIDEtLjQxLS42MTRWNjUuNjg1YTQ4LjY1MiA0OC42NTIgMCAwIDEgNzkuNzU3LTM3LjM0NmwtMS41MzUuODctNTEuNjcgMjkuODI1YTguNTk1IDguNTk1IDAgMCAwLTQuMjQ2IDcuMzY3bC0uMDUxIDcyLjY5N1ptMTEuODY4LTI1LjU4IDI4LjEzOC0xNi4yMTcgMjguMTg4IDE2LjIxOHYzMi40MzRsLTI4LjA4NiAxNi4yMTgtMjguMTg4LTE2LjIxOC0uMDUyLTMyLjQzNFoiLz48L3N2Zz4%3D)


### Tools

![Git](https://img.shields.io/badge/Git-F05032?style=flat-square&logo=git&logoColor=white)
![GitHub](https://img.shields.io/badge/GitHub-181717?style=flat-square&logo=github&logoColor=white)
![Jupyter](https://img.shields.io/badge/Jupyter-F37626?style=flat-square&logo=jupyter&logoColor=white)
[![VS Code](https://img.shields.io/badge/VS_Code-007ACC?style=flat&logo=data:image/svg%2Bxml;base64,PHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHZpZXdCb3g9IjAgMCAxMjggMTI4Ij48cGF0aCBmaWxsPSIjZmZmZmZmIiBmaWxsLXJ1bGU9ImV2ZW5vZGQiIGNsaXAtcnVsZT0iZXZlbm9kZCIgZD0iTTkwLjc2NyAxMjcuMTI2YTcuOTY4IDcuOTY4IDAgMCAwIDYuMzUtLjI0NGwyNi4zNTMtMTIuNjgxYTggOCAwIDAgMCA0LjUzLTcuMjA5VjIxLjAwOWE4IDggMCAwIDAtNC41My03LjIxTDk3LjExNyAxLjEyYTcuOTcgNy45NyAwIDAgMC05LjA5MyAxLjU0OGwtNTAuNDUgNDYuMDI2TDE1LjYgMzIuMDEzYTUuMzI4IDUuMzI4IDAgMCAwLTYuODA3LjMwMmwtNy4wNDggNi40MTFhNS4zMzUgNS4zMzUgMCAwIDAtLjAwNiA3Ljg4OEwyMC43OTYgNjQgMS43NCA4MS4zODdhNS4zMzYgNS4zMzYgMCAwIDAgLjAwNiA3Ljg4N2w3LjA0OCA2LjQxMWE1LjMyNyA1LjMyNyAwIDAgMCA2LjgwNy4zMDNsMjEuOTc0LTE2LjY4IDUwLjQ1IDQ2LjAyNWE3Ljk2IDcuOTYgMCAwIDAgMi43NDMgMS43OTNabTUuMjUyLTkyLjE4M0w1Ny43NCA2NGwzOC4yOCAyOS4wNThWMzQuOTQzWiIvPjwvc3ZnPgo=&logoColor=white)](https://code.visualstudio.com/)
</div>

<br>

---

# 🖥️ TIL

## [01. Python Basic](https://github.com/0-38/TIL/tree/main/Python_basic)

Python 기본 문법과 프로그래밍 기초 학습.

- 변수 / 자료형
- 조건문 / 반복문
- List / Dictionary
- Function / Lambda
- Exception Handling
- Class / OOP

<br>

## [02. Python Data Preprocessing & Visualization](https://github.com/0-38/TIL/tree/main/Python_Data_Preprocessing_Visualization)

Pandas와 NumPy를 활용한 데이터 전처리 및 시각화.

- NumPy / Pandas
- DataFrame
- 결측치 / 이상치
- GroupBy / Merge
- 기술통계
- Matplotlib / Seaborn

### 🔄 Data Analysis Workflow

```text
Raw Data
    ↓
Data Structure Check
    ↓
Data Type Check
    ↓
Missing Value Check
    ↓
Duplicate Check
    ↓
Outlier Check
    ↓
Data Preprocessing
    ↓
Feature Engineering
    ↓
EDA
    ↓
Visualization
```

<br>

## [03. Basic Statistics](https://github.com/0-38/TIL/tree/main/Basic_Statistics)

데이터 분석을 위한 기초 통계 학습.

- 평균 / 중앙값
- 분산 / 표준편차
- 분위수 / IQR
- 왜도 / 첨도
- 공분산 / 상관관계
- 확률분포


### 🔄 Basic Statistics Workflow

```text
Data
  ↓
Descriptive Statistics
  ↓
Distribution
  ↓
Variability
  ↓
Relationship
  ↓
Interpretation
```

<br>

## [04. Experimental Design & Inferential Statistics](https://github.com/0-38/TIL/tree/main/Experimental%20Design_Inferential%20Statistics)

표본을 이용한 통계적 추론과 가설검정 학습.

- 가설검정
- p-value
- 신뢰구간
- t-test
- ANOVA
- 카이제곱 검정
- 검정력 / Effect Size

### 🔄 Hypothesis Testing Workflow

```text
Research Question
        ↓
Hypothesis
        ↓
Significance Level
        ↓
Statistical Test
        ↓
Test Statistic
        ↓
p-value
        ↓
Decision
        ↓
Interpretation
```

<br>

## [05. Machine Learning basic](https://github.com/0-38/TIL/tree/main/Machine%20Learning_basic)

데이터를 이용한 예측 모델의 기본 원리와 평가 방법 학습.

- Regression
- Classification
- Linear / Logistic Regression
- Decision Tree
- Random Forest
- Model Evaluation
- Cross Validation
- Hyperparameter Tuning

### 🔄 Machine Learning Workflow

```text
Raw Data
    ↓
EDA
    ↓
Data Preprocessing
    ↓
Feature / Target Split
    ↓
Train / Validation / Test Split
    ↓
Encoding
    ↓
Scaling
    ↓
Baseline Model
    ↓
Model Training
    ↓
Cross Validation
    ↓
Hyperparameter Tuning
    ↓
Best Model
    ↓
Test Evaluation
    ↓
Prediction
    ↓
Interpretation
```

<br>

## [06. SQL & Database](https://github.com/0-38/TIL/tree/main/SQL_Database_basic)

PostgreSQL을 이용한 데이터 조회 및 데이터베이스 학습.

- SELECT / WHERE
- GROUP BY / HAVING
- JOIN
- Subquery / CTE
- Window Function
- Database Normalization

### 🔄 SQL Query Processing

```text
FROM
 ↓
JOIN
 ↓
WHERE
 ↓
GROUP BY
 ↓
HAVING
 ↓
SELECT
 ↓
ORDER BY
 ↓
LIMIT
```

<br>

## [07. SQL Query Optimization](https://github.com/0-38/TIL/tree/main/SQL_query_optimization)

SQL 실행계획을 이용한 쿼리 성능 분석.

- EXPLAIN ANALYZE
- Sequential Scan
- Index Scan
- Hash Join
- Nested Loop
- Sort
- Index
- Query Optimization


### 🔄 Query Optimization Workflow

```text
SQL Query
    ↓
EXPLAIN ANALYZE
    ↓
Execution Plan
    ↓
Scan 확인
    ↓
Join 확인
    ↓
Sort 확인
    ↓
Rows / Cost / Actual Time 확인
    ↓
병목 구간 탐색
    ↓
Index / Query 수정
    ↓
실행계획 재확인
```

단순히 `Cost`만 확인하기보다 다음 항목을 함께 비교한다.

```text
Estimated Rows
Actual Rows
Actual Time
Loops
Buffers
Disk Sort
Scan Method
Join Method
```


<br>

## [참고. Claude Chat](https://github.com/0-38/TIL/tree/main/Claude_Chat)

생성형 AI를 학습과 데이터 분석에 활용하는 방법을 정리한다.

- Context
- Project
- Prompt
- Role / Instruction
- Constraint
- Output Format
- 결과 검증 및 개선

<br>

---

# 📂 Repository Structure

```text
TIL
│
├── Python_basic
│   ├── variable_data_type
│   ├── conditional_statement
│   ├── loop
│   ├── list_dictionary
│   ├── function_lambda
│   ├── exception
│   └── class_oop
│
├── Python_Data_Preprocessing_Visualization
│   ├── numpy
│   ├── pandas
│   ├── dataframe
│   ├── missing_value
│   ├── outlier
│   ├── groupby
│   ├── merge
│   └── visualization
│
├── Basic_Statistics
│   ├── mean_median
│   ├── variance_std
│   ├── quantile_iqr
│   ├── skewness_kurtosis
│   ├── covariance
│   └── correlation
│
├── Experimental Design_Inferential Statistics
│   ├── hypothesis_test
│   ├── confidence_interval
│   ├── t_test
│   ├── anova
│   ├── chi_square
│   ├── effect_size
│   └── statistical_power
│
├── Maching Learning_basic
│   ├── regression
│   ├── classification
│   ├── linear_regression
│   ├── logistic_regression
│   ├── decision_tree
│   ├── random_forest
│   ├── model_evaluation
│   ├── cross_validation
│   └── hyperparameter_tuning
│
├── SQL_Database_basic
│   ├── select_where
│   ├── groupby_having
│   ├── join
│   ├── subquery
│   ├── cte
│   ├── window_function
│   └── normalization
│
├── SQL_query_optimization
│   ├── explain_analyze
│   ├── sequential_scan
│   ├── index_scan
│   ├── nested_loop
│   ├── hash_join
│   ├── sort
│   └── index
│
├── Claude_Chat
│   ├── context
│   ├── project
│   ├── prompt
│   ├── role_instruction
│   ├── constraint
│   └── result_validation
│
└── README.md
```

> Repository Structure의 하위 항목은 실제 폴더 구조라기보다  
> **각 폴더에서 학습한 주요 개념과 함수·문법을 한눈에 보여주기 위한 구조이다.**

<br>

---

# ⚙️ TIL 작성 방식

각 학습 내용은 다음 흐름을 기준으로 정리한다.

```text
개념
 ↓
사용하는 이유
 ↓
핵심 원리
 ↓
코드 / 문법
 ↓
실행 결과
 ↓
결과 해석
 ↓
핵심 정리
```

단순히 코드를 암기하기보다,

```text
왜 사용하는가?

어떻게 동작하는가?

결과는 무엇을 의미하는가?
```

를 중심으로 이해한다.
