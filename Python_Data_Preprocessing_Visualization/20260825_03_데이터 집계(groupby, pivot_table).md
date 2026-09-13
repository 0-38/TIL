# [ Python pandas] - 데이터 집계(groupby, pivot_table)

<br>

---

# **1. groupby() <br>  분할 → 적용 → 결합**

`groupby()`는 데이터를 특정 기준으로 묶은 뒤 각 그룹에 합계, 평균, 개수 등의 집계 연산을 적용하는 Pandas의 핵심 기능임.

예를 들어 다음과 같은 질문을 해결할 때 사용할 수 있음.

```text
야구팀별 평균 득점은?
축구 포지션별 평균 슈팅 수는?
테니스 선수별 총 승리 횟수는?
지역별 총매출은?
```

> ## **1-1 ) groupby()의 동작 원리**

`groupby()`는 **Split → Apply → Combine**의 3단계로 이해할 수 있음.

| 단계 | 의미 |
| --- | --- |
| Split | 기준 컬럼을 이용해 데이터를 그룹으로 나눔 |
| Apply | 각 그룹에 `sum()`, `mean()` 등의 연산 적용 |
| Combine | 계산된 그룹별 결과를 하나로 결합 |

예를 들어 야구 선수의 타점 데이터가 다음과 같다고 가정함.

```text
선수    팀     타점
-------------------
서준     A      3
현우     A      5
도윤     B      4
지민     B      2
예린     C      6
```

`team`을 기준으로 그룹화하면 다음과 같이 나뉨.

```text
Split

A → 3, 5
B → 4, 2
C → 6
```

각 그룹에 `sum()`을 적용함.

```text
Apply

A → 3 + 5 = 8
B → 4 + 2 = 6
C → 6
```

결과를 하나로 합침.

```text
Combine

A → 8
B → 6
C → 6
```

<br>

> ## **1-2 ) groupby()로 그룹별 합계 계산**

[ python : code ] "야구팀별 총 타점 계산"

```python
import pandas as pd

baseball = pd.DataFrame({
    "team": ["A", "A", "B", "B", "C"],
    "rbi": [3, 5, 4, 2, 6]
})

result = (
    baseball.groupby("team")["rbi"]
            .sum()
            .reset_index()
)

print(result.to_string(index=False))
```

[ Result ]

```text
team  rbi
   A    8
   B    6
   C    6
```

계산 과정은 다음과 같음.

```text
A팀 = 3 + 5 = 8

B팀 = 4 + 2 = 6

C팀 = 6
```

<br>

> ## **1-3 ) groupby()로 그룹별 평균 계산**

테니스 선수들의 경기별 서브 성공률을 선수별로 평균 낸다고 가정함.

[ python : code ] "테니스 선수별 평균 서브 성공률 계산"

```python
import pandas as pd

tennis = pd.DataFrame({
    "player": [
        "서연", "서연",
        "민재", "민재",
        "도윤"
    ],
    "serve_pct": [
        62.0, 68.0,
        70.0, 74.0,
        66.0
    ]
})

result = (
    tennis.groupby("player", sort=False)["serve_pct"]
          .mean()
          .reset_index()
)

print(result.to_string(index=False))
```

[ Result ]

```text
player  serve_pct
    서연       65.0
    민재       72.0
    도윤       66.0
```

계산 과정은 다음과 같음.

```text
서연
= (62 + 68) / 2
= 130 / 2
= 65%

민재
= (70 + 74) / 2
= 144 / 2
= 72%

도윤
= 66%
```

<br>

> ## **1-4 ) 주요 집계 함수**

| 함수 | 의미 |
| --- | --- |
| `sum()` | 그룹별 합계 |
| `mean()` | 그룹별 평균 |
| `count()` | 결측값을 제외한 값의 개수 |
| `size()` | 결측값 여부와 관계없이 전체 행 개수 |
| `max()` | 그룹별 최댓값 |
| `min()` | 그룹별 최솟값 |
| `median()` | 그룹별 중앙값 |
| `std()` | 그룹별 표준편차 |
| `nunique()` | 그룹별 고유값 개수 |

<br>

> ## **1-5 ) reset_index()의 역할**

`groupby()`를 사용하면 그룹 기준 컬럼이 결과의 인덱스가 되는 경우가 많음.

[ python : code ] "그룹 기준을 인덱스로 유지하여 합계 계산"

```python
result = baseball.groupby("team")["rbi"].sum()

print(result)
```

[ Result ]

```text
team
A    8
B    6
C    6
Name: rbi, dtype: int64
```

`reset_index()`를 사용하면 그룹 기준인 `team`을 다시 일반 컬럼으로 변경할 수 있음.

[ python : code ] "groupby 결과의 그룹 기준을 일반 컬럼으로 복원"

```python
result = (
    baseball.groupby("team")["rbi"]
            .sum()
            .reset_index()
)

print(result.to_string(index=False))
```

[ Result ]

```text
team  rbi
   A    8
   B    6
   C    6
```

따라서 이후 다른 DataFrame과 `merge()`하거나 일반적인 표 형태로 활용할 예정이라면 `reset_index()`가 편리함.

<br>

> ## **1-6 ) groupby() 흐름도**

```text
[원본 데이터]
      │
      ↓
groupby("team")
      │
      ↓
┌─────────────┐
│   Split     │
├─────────────┤
│ A → 3, 5   │
│ B → 4, 2   │
│ C → 6      │
└─────────────┘
      │
      ↓
    sum()
      │
      ↓
┌─────────────┐
│   Apply     │
├─────────────┤
│ A → 8      │
│ B → 6      │
│ C → 6      │
└─────────────┘
      │
      ↓
   Combine
      │
      ↓
[팀별 집계 결과]
```


<br><br>

---

# **2. agg(): 여러 집계 함수를 한 번에 적용**

`agg()`는 하나의 `groupby()` 결과에 여러 집계 함수를 동시에 적용할 때 사용함.

예를 들어 축구팀별 경기 데이터를 이용하여 다음 항목을 한 번에 계산할 수 있음.

```text
총 슈팅 수
평균 슈팅 수
최대 슈팅 수
경기 수
```

> ## **2-1 ) Named Aggregation 문법**

다음 형태로 많이 사용함.

```python
df.groupby("기준컬럼").agg(
    결과컬럼명=("대상컬럼", "집계함수")
)
```

예를 들어 다음 코드는

```python
total_shots=("shots", "sum")
```

다음을 의미함.

```text
shots 컬럼
   ↓
sum() 적용
   ↓
total_shots라는 이름으로 저장
```

<br>

> ## **2-2 ) 축구팀별 다중 집계**

[ python : code ] "축구팀별 슈팅 통계를 여러 집계 함수로 계산"

```python
import pandas as pd

football = pd.DataFrame({
    "team": [
        "Blue", "Blue",
        "Red", "Red", "Red"
    ],
    "match": [1, 2, 1, 2, 3],
    "shots": [12, 18, 10, 15, 20]
})

summary = (
    football
    .groupby("team", sort=False)
    .agg(
        total_shots=("shots", "sum"),
        avg_shots=("shots", "mean"),
        max_shots=("shots", "max"),
        match_count=("match", "count")
    )
    .reset_index()
)

print(summary.to_string(index=False))
```

[ Result ]

```text
team  total_shots  avg_shots  max_shots  match_count
Blue           30       15.0         18            2
 Red           45       15.0         20            3
```

Blue팀 계산.

```text
총 슈팅
= 12 + 18
= 30

평균 슈팅
= 30 / 2
= 15

최대 슈팅
= 18

경기 수
= 2
```

Red팀 계산.

```text
총 슈팅
= 10 + 15 + 20
= 45

평균 슈팅
= 45 / 3
= 15

최대 슈팅
= 20

경기 수
= 3
```

<br>

> ## **2-3 ) groupby() 단일 집계와 agg() 다중 집계 비교**

| 목적 | 사용 방법 |
| --- | --- |
| 하나의 합계만 계산 | `groupby().sum()` |
| 하나의 평균만 계산 | `groupby().mean()` |
| 여러 통계를 동시에 계산 | `groupby().agg()` |
| 결과 컬럼명을 직접 지정 | Named Aggregation |

단순 합계만 필요하다면 다음처럼 작성 가능함.

```python
football.groupby("team")["shots"].sum()
```

여러 통계가 필요하다면 `agg()`를 사용하는 것이 효율적임.

```python
football.groupby("team").agg(
    total=("shots", "sum"),
    average=("shots", "mean"),
    maximum=("shots", "max")
)
```

<br>

> ## **2-4 ) 양궁 선수별 기록 복합 집계**

양궁 선수마다 여러 번의 화살 점수가 측정되었다고 가정함.

선수별 화살 수, 평균 점수, 최고 점수를 한 번에 계산할 수 있음.

[ python : code ] "양궁 선수별 평균 점수와 최고 점수 집계"

```python
import pandas as pd

archery = pd.DataFrame({
    "archer": [
        "하진", "하진",
        "서우", "서우", "서우"
    ],
    "score": [9, 10, 8, 9, 10]
})

summary = (
    archery
    .groupby("archer", sort=False)
    .agg(
        arrow_count=("score", "count"),
        avg_score=("score", "mean"),
        max_score=("score", "max")
    )
    .reset_index()
)

print(summary.to_string(index=False))
```

[ Result ]

```text
archer  arrow_count  avg_score  max_score
    하진            2        9.5         10
    서우            3        9.0         10
```

### 결과 해석

하진의 기록.

```text
9 + 10 = 19

19 / 2
= 9.5

최고 점수
= 10
```

서우의 기록.

```text
8 + 9 + 10
= 27

27 / 3
= 9

최고 점수
= 10
```

### 흐름도

```text
[개별 화살 점수]
        │
        ↓
groupby("archer")
        │
        ↓
선수별 기록 분리
        │
        ├── 하진 → 9, 10
        │
        └── 서우 → 8, 9, 10
        │
        ↓
agg()
        │
        ├── count
        ├── mean
        └── max
        │
        ↓
[선수별 요약 결과]
```


<br><br>

---

# **3. pivot_table(): 데이터를 행과 열로 재구성하여 집계**

`pivot_table()`은 데이터를 집계하면서 결과를 **행(index)과 열(columns)의 교차표 형태로 재구성**하는 함수임.

Excel의 피벗 테이블과 비슷한 형태임.

예를 들어 다음과 같은 분석에 활용할 수 있음.

```text
유도 체급 × 성별 평균 승리 수
스키 종목 × 팀별 평균 기록
배드민턴 종목 × 연령대별 경기 수
지역 × 상품별 매출
```

> ## **3-1 ) pivot_table() 주요 옵션**

기본 형태는 다음과 같음.

```python
pd.pivot_table(
    data,
    index="행에 배치할 컬럼",
    columns="열에 배치할 컬럼",
    values="집계할 값",
    aggfunc="집계 함수",
    fill_value=0
)
```

| 옵션 | 역할 |
| --- | --- |
| `data` | 사용할 DataFrame |
| `index` | 결과의 행 기준 |
| `columns` | 결과의 열 기준 |
| `values` | 집계 대상 값 |
| `aggfunc` | 적용할 집계 함수 |
| `fill_value` | 데이터가 없는 조합을 채울 값 |

`aggfunc`를 생략하면 기본적으로 `mean`이 적용됨.

따라서 합계가 필요하다면 `aggfunc="sum"`처럼 직접 지정하는 것이 좋음.

<br>

> ## **3-2 ) 쇼트트랙 거리별·팀별 평균 기록 비교**

[ python : code ] "쇼트트랙 거리와 팀에 따른 평균 기록 피벗 테이블 생성"

```python
import pandas as pd

shorttrack = pd.DataFrame({
    "distance": [
        "500m", "500m", "500m",
        "1000m", "1000m"
    ],
    "team": [
        "A", "A", "B",
        "A", "B"
    ],
    "record_sec": [
        41.2, 40.8, 42.0,
        84.5, 86.0
    ]
})

pivot = pd.pivot_table(
    shorttrack,
    index="distance",
    columns="team",
    values="record_sec",
    aggfunc="mean",
    sort=False
)

print(pivot.to_string())
```

[ Result ]

```text
team         A     B
distance
500m      41.0  42.0
1000m     84.5  86.0
```

500m A팀 평균 기록.

```text
41.2 + 40.8
= 82.0

82.0 / 2
= 41.0초
```

500m B팀은 기록이 하나이므로 `42.0초`.

1000m은 각 팀에 기록이 하나씩 있으므로 다음과 같음.

```text
A팀 → 84.5초
B팀 → 86.0초
```

<br>

> ## **3-3 ) fill_value의 역할**

특정 행과 열의 조합에 데이터가 없다면 기본적으로 `NaN`이 표시됨.

`fill_value=0`을 사용하면 비어 있는 값을 `0`으로 표시할 수 있음.

[ python : code ] "배드민턴 종목별·팀별 승리 수에서 없는 조합을 0으로 표시"

```python
import pandas as pd

badminton = pd.DataFrame({
    "event": [
        "단식",
        "단식",
        "복식"
    ],
    "team": [
        "A",
        "B",
        "A"
    ],
    "wins": [
        5,
        3,
        4
    ]
})

pivot = pd.pivot_table(
    badminton,
    index="event",
    columns="team",
    values="wins",
    aggfunc="sum",
    fill_value=0,
    sort=False
)

print(pivot.to_string())
```

[ Result ]

```text
team   A  B
event
단식     5  3
복식     4  0
```

`복식 + B팀` 데이터는 존재하지 않기 때문에 `fill_value=0`으로 인해 `0`으로 표시됨.

단, 실제 분석에서는 다음 두 의미를 구분해야 함.

```text
0
→ 실제 측정값이 0

NaN
→ 데이터 자체가 없음
```

따라서 데이터가 없는 상태와 실제 0이 다른 의미라면 `fill_value=0`을 신중하게 사용해야 함.

<br>

> ## **3-4 ) pivot_table() 동작 흐름**

```text
[원본 쇼트트랙 데이터]

distance   team   record
500m        A      41.2
500m        A      40.8
500m        B      42.0
1000m       A      84.5
1000m       B      86.0

          │
          ↓

index="distance"
columns="team"

          │
          ↓

행 → 경기 거리
열 → 팀

          │
          ↓

aggfunc="mean"

          │
          ↓

             A      B
500m       41.0   42.0
1000m      84.5   86.0
```


<br><br>

---

# **4. groupby()와 pivot_table() 비교**

`groupby()`와 `pivot_table()`은 모두 **그룹별 데이터를 집계**할 수 있지만 결과 구조와 활용 목적에서 차이가 있음.

> ## **4-1 ) 핵심 차이**

| 구분 | `groupby()` | `pivot_table()` |
| --- | --- | --- |
| 주요 목적 | 그룹별 통계 계산 | 그룹별 통계 + 교차표 구성 |
| 기본 결과 | 세로형 데이터 | 행 × 열 형태 |
| 그룹 기준 | 하나 이상의 컬럼 | `index`, `columns` |
| 여러 집계 | `agg()` 사용 | `aggfunc` 사용 |
| 비교 분석 | 가능 | 두 기준을 비교할 때 편리 |
| 대표 사용 | 선수별 평균 점수 | 팀 × 종목 평균 점수 |

<br>

> ## **4-2 ) 같은 데이터를 groupby()로 집계**

앞서 사용한 쇼트트랙 데이터를 `groupby()`로 집계함.

[ python : code ] "거리별·팀별 평균 기록을 groupby로 계산"

```python
grouped = (
    shorttrack
    .groupby(
        ["distance", "team"],
        sort=False
    )["record_sec"]
    .mean()
    .reset_index()
)

print(grouped.to_string(index=False))
```

[ Result ]

```text
distance team  record_sec
    500m    A        41.0
    500m    B        42.0
   1000m    A        84.5
   1000m    B        86.0
```

`groupby()` 결과는 세로 방향의 데이터임.

<br>

> ## **4-3 ) 같은 데이터를 pivot_table()로 집계**

[ python : code ] "거리별·팀별 평균 기록을 피벗 테이블로 계산"

```python
pivot = pd.pivot_table(
    shorttrack,
    index="distance",
    columns="team",
    values="record_sec",
    aggfunc="mean",
    sort=False
)

print(pivot.to_string())
```

[ Result ]

```text
team         A     B
distance
500m      41.0  42.0
1000m     84.5  86.0
```

같은 데이터를 집계했지만 결과의 형태가 다름.

```text
groupby()

distance | team | record
------------------------
500m     | A    | 41.0
500m     | B    | 42.0
1000m    | A    | 84.5
1000m    | B    | 86.0
```

```text
pivot_table()

          A      B
500m     41.0   42.0
1000m    84.5   86.0
```

따라서 다음과 같이 판단할 수 있음.

```text
단순 그룹별 집계
        ↓
groupby()

두 기준을 교차해서 비교
        ↓
pivot_table()
```


<br><br>

---

# **5. 복합 예제: 보행 데이터를 그룹별로 집계하고 재구성**

보행 분석에서 연령 그룹과 보행 조건에 따라 평균 보행 속도가 어떻게 달라지는지 분석한다고 가정함.

> ## **5-1 ) groupby() + agg()로 보행 데이터 요약**

[ python : code ] "연령 그룹과 보행 조건별 측정 횟수와 평균 속도 계산"

```python
import pandas as pd

gait = pd.DataFrame({
    "group": [
        "Adult",
        "Adult",
        "Adult",
        "Senior",
        "Senior",
        "Senior"
    ],
    "condition": [
        "Normal",
        "Normal",
        "Fast",
        "Normal",
        "Fast",
        "Fast"
    ],
    "speed_mps": [
        1.20,
        1.30,
        1.50,
        0.90,
        1.10,
        1.20
    ]
})

summary = (
    gait
    .groupby(
        ["group", "condition"],
        sort=False
    )
    .agg(
        test_count=("speed_mps", "count"),
        avg_speed=("speed_mps", "mean")
    )
    .reset_index()
)

print(summary.to_string(index=False))
```

[ Result ]

```text
 group condition  test_count  avg_speed
 Adult    Normal           2       1.25
 Adult      Fast           1       1.50
Senior    Normal           1       0.90
Senior      Fast           2       1.15
```

### 결과 해석

Adult 그룹의 Normal 조건.

```text
1.20 + 1.30
= 2.50

2.50 / 2
= 1.25 m/s
```

Adult 그룹의 Fast 조건.

```text
1.50 / 1
= 1.50 m/s
```

Senior 그룹의 Normal 조건.

```text
0.90 / 1
= 0.90 m/s
```

Senior 그룹의 Fast 조건.

```text
1.10 + 1.20
= 2.30

2.30 / 2
= 1.15 m/s
```

<br>

> ## **5-2 ) pivot_table()로 보행 조건 비교**

같은 데이터를 교차표 형태로 바꾸면 연령 그룹 간 차이를 한눈에 확인하기 쉬움.

[ python : code ] "연령 그룹과 보행 조건별 평균 속도를 피벗 테이블로 변환"

```python
pivot = pd.pivot_table(
    gait,
    index="group",
    columns="condition",
    values="speed_mps",
    aggfunc="mean",
    sort=False
)

print(pivot.to_string())
```

[ Result ]

```text
condition  Normal  Fast
group
Adult        1.25  1.50
Senior       0.90  1.15
```

### 코드 흐름도

```text
[원본 보행 데이터]
        │
        ↓
group + condition 기준으로 그룹화
        │
        ↓
groupby()
        │
        ↓
agg()
        │
        ├── count
        └── mean
        │
        ↓
[세로형 집계 결과]
        │
        │
        ↓
pivot_table()
        │
        ↓
[행 × 열 교차표]

           Normal   Fast
Adult       1.25    1.50
Senior      0.90    1.15
```

`groupby() + agg()`는 각 그룹의 통계값을 자세하게 확인하는 데 적합함.

`pivot_table()`은 같은 데이터를 행과 열 형태로 재구성하여 그룹 간 비교를 쉽게 하는 데 적합함.


<br><br>

---

# **6. Advanced 추가 학습**

> ## **6-1 ) as_index=False**

`groupby()`에서 매번 `reset_index()`를 사용하지 않고 `as_index=False`를 설정할 수도 있음.

[ python : code ] "그룹 기준 컬럼을 일반 컬럼으로 유지"

```python
result = (
    baseball.groupby(
        "team",
        as_index=False
    )["rbi"]
    .sum()
)

print(result.to_string(index=False))
```

[ Result ]

```text
team  rbi
   A    8
   B    6
   C    6
```

다음 두 코드는 비슷한 형태의 결과를 반환함.

```python
df.groupby("team")["score"].sum().reset_index()
```

```python
df.groupby(
    "team",
    as_index=False
)["score"].sum()
```

<br>

> ## **6-2 ) count()와 size() 차이**

`count()`는 결측값을 제외하고 개수를 계산함.

`size()`는 결측값이 있어도 실제 행의 개수를 계산함.

[ python : code ] "count와 size의 결측값 처리 차이 확인"

```python
import pandas as pd
import numpy as np

judo = pd.DataFrame({
    "weight_class": [
        "-60kg",
        "-60kg",
        "-60kg"
    ],
    "score": [
        10,
        np.nan,
        8
    ]
})

print(
    judo.groupby("weight_class")["score"].count()
)

print(
    judo.groupby("weight_class").size()
)
```

[ Result ]

```text
weight_class
-60kg    2
Name: score, dtype: int64

weight_class
-60kg    3
dtype: int64
```

| 함수 | 결측값 처리 | 결과 |
| --- | --- | ---: |
| `count()` | `NaN` 제외 | 2 |
| `size()` | `NaN` 포함 | 3 |

<br>

> ## **6-3 ) transform()**

`agg()`는 여러 행을 그룹별 결과로 축약함.

반면 `transform()`은 **원본 행 수를 유지하면서 그룹별 계산 결과를 각 행에 추가**할 수 있음.

[ python : code ] "탁구 선수의 경기 점수에 팀 평균 점수 추가"

```python
import pandas as pd

table_tennis = pd.DataFrame({
    "team": [
        "A", "A",
        "B", "B"
    ],
    "player": [
        "태윤", "서현",
        "건호", "채린"
    ],
    "score": [
        8, 12,
        9, 15
    ]
})

table_tennis["team_avg"] = (
    table_tennis
    .groupby("team")["score"]
    .transform("mean")
)

print(
    table_tennis.to_string(index=False)
)
```

[ Result ]

```text
team player  score  team_avg
   A     태윤      8      10.0
   A     서현     12      10.0
   B     건호      9      12.0
   B     채린     15      12.0
```

계산 과정.

```text
A팀 평균
= (8 + 12) / 2
= 20 / 2
= 10

B팀 평균
= (9 + 15) / 2
= 24 / 2
= 12
```

`transform()`은 선수 개인 기록과 소속팀 평균의 차이를 계산하는 등의 작업에 활용할 수 있음.


<br><br>

---

# **7. 핵심정리**

1. `groupby()`는 특정 컬럼을 기준으로 데이터를 그룹화한 뒤 그룹별 통계를 계산하는 기능임.

2. `groupby()`의 기본 동작은 다음 3단계로 이해할 수 있음.

```text
Split
  ↓
Apply
  ↓
Combine
```

3. 주요 집계 함수는 다음과 같음.

| 함수 | 의미 |
| --- | --- |
| `sum()` | 합계 |
| `mean()` | 평균 |
| `count()` | `NaN`을 제외한 값의 개수 |
| `size()` | 그룹의 전체 행 개수 |
| `max()` | 최댓값 |
| `min()` | 최솟값 |
| `median()` | 중앙값 |
| `std()` | 표준편차 |
| `nunique()` | 고유값 개수 |

4. `groupby()` 결과에서 그룹 기준 컬럼이 인덱스가 되었다면 `reset_index()`로 다시 일반 컬럼으로 변경할 수 있음.

5. `as_index=False`를 사용하면 처음부터 그룹 기준을 일반 컬럼으로 유지할 수 있음.

6. `agg()`는 여러 집계 함수를 한 번에 적용할 때 사용함.

```python
df.groupby("team").agg(
    total=("score", "sum"),
    average=("score", "mean"),
    maximum=("score", "max")
)
```

7. `pivot_table()`은 데이터를 집계하면서 행과 열 형태의 교차표로 재구성하는 함수임.

```text
index
→ 행 기준

columns
→ 열 기준

values
→ 집계할 값

aggfunc
→ 적용할 집계 함수
```

8. `pivot_table()`에서 `aggfunc`를 생략하면 기본적으로 평균(`mean`)이 사용됨.

9. `fill_value=0`은 데이터가 없는 조합을 0으로 표시하지만 **실제 값 0과 데이터 없음이 다른 의미라면 주의해야 함.**

10. `groupby()`와 `pivot_table()`의 가장 큰 차이는 다음과 같음.

| 필요한 분석 | 적합한 방법 |
| --- | --- |
| 선수별 평균 기록 | `groupby()` |
| 팀별 총 득점 | `groupby()` |
| 여러 통계를 한 번에 계산 | `groupby() + agg()` |
| 팀 × 종목 평균 비교 | `pivot_table()` |
| 그룹 × 조건 비교 | `pivot_table()` |

11. `count()`는 결측값을 제외한 값의 개수를 계산하고 `size()`는 결측값과 관계없이 실제 행 수를 계산함.

12. `transform()`은 그룹별 통계값을 계산하면서도 원본 DataFrame의 행 수를 유지해야 할 때 유용함.

13. 데이터 집계에서는 함수부터 선택하기보다 먼저 다음 세 가지를 결정하는 것이 중요함.

```text
무엇을 기준으로 묶을 것인가?
          ↓
어떤 값을 계산할 것인가?
          ↓
어떤 집계 함수를 사용할 것인가?
```


