# [ Python pandas ] - 데이터 결합 (merge / concat)

<br>

---


# **1. concat(): 데이터 단순 연결**

`pd.concat()`은 여러 개의 DataFrame을 **행 방향 또는 열 방향으로 이어 붙일 때** 사용하는 함수임.

공통된 Key를 찾아 연결하는 방식이 아니라 데이터의 **인덱스와 컬럼 구조를 기준으로 연결**함.

SQL의 `UNION ALL`과 비슷하게 동일한 구조의 데이터를 위아래로 합칠 때 자주 사용함.

> ## **1-1 ) axis=0 : 행 방향으로 연결**

`axis=0`은 DataFrame을 **위아래 방향으로 연결**함.

월별 경기 기록, 여러 센서에서 수집한 동일한 형식의 데이터처럼 **컬럼 구조가 같은 데이터**를 하나로 합칠 때 적합함.

[ python : code ] "육상 기록 데이터를 행 방향으로 결합"

```python
import pandas as pd

first_half = pd.DataFrame({
    "athlete": ["서준", "하린"],
    "event": ["100m", "200m"],
    "record_sec": [11.2, 24.8]
})

second_half = pd.DataFrame({
    "athlete": ["도윤", "예린"],
    "event": ["100m", "200m"],
    "record_sec": [10.9, 25.1]
})

track_all = pd.concat(
    [first_half, second_half],
    axis=0,
    ignore_index=True
)

print(track_all.to_string(index=False))
```

[ Result ]

```text
athlete event  record_sec
     서준  100m        11.2
     하린  200m        24.8
     도윤  100m        10.9
     예린  200m        25.1
```

`first_half`의 2행 아래에 `second_half`의 2행이 추가되어 총 4행이 됨.

`ignore_index=True`를 사용했기 때문에 기존 인덱스를 버리고 `0, 1, 2, 3` 형태로 새로운 인덱스를 생성함.

<br>

> ## **1-2 ) ignore_index=True**

각 DataFrame은 자체 인덱스를 가지고 있기 때문에 단순히 `concat()`하면 인덱스가 중복될 수 있음.

```python
pd.concat([df1, df2])
```

예를 들어 두 DataFrame이 모두 `0, 1` 인덱스를 가진다면 결합 후 다음처럼 될 수 있음.

```text
0
1
0
1
```

다음처럼 작성하면 새로운 연속 인덱스를 생성함.

```python
pd.concat(
    [df1, df2],
    ignore_index=True
)
```

```text
0
1
2
3
```

행 방향으로 데이터를 단순 연결하면서 기존 인덱스가 중요하지 않다면 `ignore_index=True`를 자주 사용함.

<br>

> ## **1-3 ) 컬럼이 다른 데이터를 axis=0으로 연결**

두 DataFrame의 컬럼이 완전히 같지 않아도 `concat()`은 가능함.

단, 한쪽에 존재하지 않는 컬럼의 값은 `NaN`으로 채워짐.

[ python : code ] "농구 선수 통계의 서로 다른 컬럼을 행 방향으로 결합"

```python
import pandas as pd

team_a = pd.DataFrame({
    "player": ["민재", "수현"],
    "points": [18, 22]
})

team_b = pd.DataFrame({
    "player": ["지훈", "예나"],
    "assists": [7, 9]
})

combined = pd.concat(
    [team_a, team_b],
    axis=0,
    ignore_index=True
)

print(combined.to_string(index=False))
```

[ Result ]

```text
player  points  assists
    민재    18.0      NaN
    수현    22.0      NaN
    지훈     NaN      7.0
    예나     NaN      9.0
```

`team_a`에는 `assists`가 없으므로 해당 값이 `NaN`으로 채워짐.

반대로 `team_b`에는 `points`가 없으므로 `points`가 `NaN`으로 채워짐.

`NaN`이 포함되면서 해당 숫자 컬럼이 `18`이 아닌 `18.0`처럼 실수형으로 표현될 수 있음.

<br>

> ## **1-4 ) axis=1 : 열 방향으로 연결**

`axis=1`은 여러 DataFrame을 **좌우 방향으로 연결**함.

이때 단순히 행의 위치만 보는 것이 아니라 **인덱스를 기준으로 맞춰서 연결**함.

[ python : code ] "보행 분석 데이터와 추가 측정값을 열 방향으로 결합"

```python
import pandas as pd

gait = pd.DataFrame({
    "name": ["유나", "시우", "재민"],
    "speed": [1.21, 1.34, 1.18]
})

balance = pd.DataFrame({
    "step_width_cm": [8.4, 7.9, 9.1],
    "cadence": [108, 112, 104]
})

result = pd.concat(
    [gait, balance],
    axis=1
)

print(result.to_string(index=False))
```

[ Result ]

```text
name  speed  step_width_cm  cadence
  유나   1.21            8.4      108
  시우   1.34            7.9      112
  재민   1.18            9.1      104
```

두 DataFrame의 인덱스가 모두 `0, 1, 2`이므로 같은 인덱스끼리 좌우로 연결됨.

즉,

```text
gait index 0    +    balance index 0
gait index 1    +    balance index 1
gait index 2    +    balance index 2
```

형태로 결합됨.

인덱스가 서로 다르면 원하는 행과 다른 데이터가 연결되거나 `NaN`이 생길 수 있으므로 `axis=1` 사용 시 인덱스를 확인하는 것이 중요함.


<br><br>

---

# **2. merge(): Key를 기준으로 데이터 병합**

`pd.merge()`는 두 DataFrame의 **공통된 Key 값을 기준으로 데이터를 연결**하는 함수임.

SQL의 `JOIN`과 매우 유사한 방식으로 동작함.

예를 들어 운동선수 기본 정보와 운동처방 데이터가 각각 다른 DataFrame에 존재하지만 두 데이터 모두 `athlete_id`를 가지고 있다면 해당 컬럼을 기준으로 연결할 수 있음.

> ## **2-1 ) merge() 기본 구조**

기본 형태는 다음과 같음.

```python
pd.merge(
    left,
    right,
    on="기준컬럼",
    how="병합방식"
)
```

주요 옵션은 다음과 같음.

| 옵션 | 의미 |
| --- | --- |
| `left` | 왼쪽 DataFrame |
| `right` | 오른쪽 DataFrame |
| `on` | 병합 기준 컬럼 |
| `how='inner'` | 양쪽 모두 존재하는 Key만 유지 |
| `how='left'` | 왼쪽 Key 전체 유지 |
| `how='right'` | 오른쪽 Key 전체 유지 |
| `how='outer'` | 양쪽 Key 전체 유지 |

<br>

> ## **2-2 ) inner merge**

`inner`는 두 DataFrame에 **공통으로 존재하는 Key만 남김**.

[ python : code ] "운동선수 측정 데이터와 운동처방 데이터를 inner 방식으로 병합"

```python
import pandas as pd

performance = pd.DataFrame({
    "athlete_id": [101, 102, 103, 104],
    "name": ["서아", "건우", "다은", "현준"],
    "vo2max": [48.2, 52.1, 45.7, 50.4]
})

prescription = pd.DataFrame({
    "athlete_id": [101, 102, 104, 105],
    "weekly_sessions": [3, 4, 5, 2]
})

inner_result = pd.merge(
    performance,
    prescription,
    on="athlete_id",
    how="inner"
)

print(inner_result.to_string(index=False))
```

[ Result ]

```text
 athlete_id name  vo2max  weekly_sessions
        101   서아    48.2                3
        102   건우    52.1                4
        104   현준    50.4                5
```

두 데이터에 모두 존재하는 `athlete_id`는 다음과 같음.

```text
101
102
104
```

따라서 `103`과 `105`는 결과에서 제외됨.

<br>

> ## **2-3 ) left merge**

`left`는 왼쪽 DataFrame의 데이터를 **모두 유지**하고 오른쪽 DataFrame의 정보를 추가함.

오른쪽에서 일치하는 Key를 찾지 못하면 `NaN`으로 채움.

[ python : code ] "모든 운동선수 정보를 유지하면서 운동처방 데이터 추가"

```python
left_result = pd.merge(
    performance,
    prescription,
    on="athlete_id",
    how="left"
)

print(left_result.to_string(index=False))
```

[ Result ]

```text
 athlete_id name  vo2max  weekly_sessions
        101   서아    48.2              3.0
        102   건우    52.1              4.0
        103   다은    45.7              NaN
        104   현준    50.4              5.0
```

`performance`에 존재하는 `101`, `102`, `103`, `104`가 모두 유지됨.

하지만 `prescription`에는 `athlete_id=103`이 없기 때문에 `weekly_sessions`가 `NaN`이 됨.

즉,

```text
왼쪽 데이터는 반드시 유지
        ↓
오른쪽에서 Key 검색
        ↓
있음 → 값 연결
없음 → NaN
```

형태로 이해할 수 있음.

<br>

> ## **2-4 ) right merge**

`right`는 오른쪽 DataFrame의 Key를 모두 유지함.

[ python : code ] "운동처방 대상자를 기준으로 선수 정보 병합"

```python
right_result = pd.merge(
    performance,
    prescription,
    on="athlete_id",
    how="right"
)

print(right_result.to_string(index=False))
```

[ Result ]

```text
 athlete_id name  vo2max  weekly_sessions
        101   서아    48.2                3
        102   건우    52.1                4
        104   현준    50.4                5
        105  NaN     NaN                2
```

`athlete_id=105`는 `prescription`에는 존재하지만 `performance`에는 없음.

따라서 `name`, `vo2max`가 `NaN`으로 나타남.

<br>

> ## **2-5 ) outer merge**

`outer`는 두 DataFrame에 존재하는 **모든 Key를 유지**함.

[ python : code ] "양쪽 데이터의 모든 운동선수 ID를 유지하여 병합"

```python
outer_result = pd.merge(
    performance,
    prescription,
    on="athlete_id",
    how="outer"
)

print(outer_result.to_string(index=False))
```

[ Result ]

```text
 athlete_id name  vo2max  weekly_sessions
        101   서아    48.2              3.0
        102   건우    52.1              4.0
        103   다은    45.7              NaN
        104   현준    50.4              5.0
        105  NaN     NaN              2.0
```

전체 Key는 다음과 같음.

```text
performance  → 101, 102, 103, 104
prescription → 101, 102, 104, 105

outer 결과   → 101, 102, 103, 104, 105
```

<br>

> ## **2-6 ) merge 방식 비교**

| 방식 | 유지되는 Key | 특징 |
| --- | --- | --- |
| `inner` | 양쪽에 모두 존재 | 공통 데이터만 분석 |
| `left` | 왼쪽 전체 | 왼쪽 데이터를 기준으로 정보 추가 |
| `right` | 오른쪽 전체 | 오른쪽 데이터를 기준으로 정보 추가 |
| `outer` | 양쪽 전체 | 불일치 데이터까지 모두 확인 |

스포츠 데이터 분석에서 선수 명단을 기준으로 측정 결과를 붙이고 싶다면 `left`가 유용함.

두 데이터에 실제로 모두 존재하는 선수만 분석한다면 `inner`를 사용할 수 있음.

데이터 누락 여부 자체를 점검하려면 `outer`가 유용함.


<br><br>

---

# **3. join(): 인덱스를 기준으로 결합**

`DataFrame.join()`은 기본적으로 **인덱스를 기준으로 DataFrame을 연결**함.

결합 대상이 이미 같은 ID를 인덱스로 가지고 있다면 `merge()`보다 간단하게 작성할 수 있음.

> ## **3-1 ) join() 기본 사용**

농구 선수의 기본 정보와 경기 통계가 선수 ID를 인덱스로 가지고 있다고 가정함.

[ python : code ] "선수 ID 인덱스를 기준으로 농구 데이터 결합"

```python
import pandas as pd

profile = pd.DataFrame(
    {
        "name": ["가온", "예준", "소율"],
        "position": ["G", "F", "C"]
    },
    index=[201, 202, 203]
)

stats = pd.DataFrame(
    {
        "points": [21, 18, 15],
        "rebounds": [4, 7, 11]
    },
    index=[201, 202, 203]
)

result = profile.join(
    stats,
    how="left"
)

print(result.to_string())
```

[ Result ]

```text
    name position  points  rebounds
201   가온        G      21         4
202   예준        F      18         7
203   소율        C      15        11
```

두 DataFrame의 인덱스가 다음과 같이 동일함.

```text
profile index → 201, 202, 203
stats index   → 201, 202, 203
```

따라서 같은 인덱스끼리 연결됨.

```text
201 → 가온 + 21점 + 4리바운드
202 → 예준 + 18점 + 7리바운드
203 → 소율 + 15점 + 11리바운드
```

<br>

> ## **3-2 ) 일반 컬럼을 인덱스로 바꾼 후 join()**

`customer_id`, `athlete_id`, `player_id` 등이 일반 컬럼으로 존재한다면 `set_index()`를 이용하여 인덱스로 만든 후 `join()`할 수 있음.

```python
left = left.set_index("athlete_id")
right = right.set_index("athlete_id")

result = left.join(
    right,
    how="left"
)
```

필요하다면 병합 후 `reset_index()`로 다시 일반 컬럼으로 되돌릴 수 있음.

```python
result = result.reset_index()
```

흐름은 다음과 같음.

```text
athlete_id 일반 컬럼
        ↓
set_index("athlete_id")
        ↓
athlete_id를 인덱스로 변경
        ↓
join()
        ↓
동일한 인덱스끼리 결합
        ↓
reset_index()
        ↓
다시 일반 컬럼으로 복원
```


<br><br>

---

# **4. concat(), merge(), join() 비교**

세 함수는 모두 DataFrame을 결합하지만 **결합 기준이 다름**.

> ## **4-1 ) 핵심 차이**

| 구분 | `concat()` | `merge()` | `join()` |
| --- | --- | --- | --- |
| 주요 기준 | 행·열 방향 | Key 컬럼 | 인덱스 |
| 대표 사용 목적 | 데이터 이어붙이기 | 관계 있는 데이터 연결 | 인덱스 기준 연결 |
| SQL과 비교 | `UNION ALL`과 유사 | `JOIN`과 유사 | 인덱스 기반 `JOIN`과 유사 |
| `axis` 사용 | 가능 | 사용하지 않음 | 사용하지 않음 |
| `inner`, `left` 등 | `join` 옵션의 개념은 다름 | 사용 가능 | 사용 가능 |
| 대표 상황 | 월별 경기 데이터 합치기 | 선수 정보 + 측정 데이터 | 선수 ID가 인덱스인 두 데이터 |

<br>

> ## **4-2 ) 어떤 함수를 선택해야 하는가**

### 같은 구조의 데이터를 위아래로 쌓기

```text
1월 경기 기록
2월 경기 기록
3월 경기 기록
```

→ `concat()`

```python
pd.concat(
    [january, february, march],
    ignore_index=True
)
```

### 선수 ID를 기준으로 서로 다른 정보 연결

```text
선수 정보
athlete_id | name

체력 측정
athlete_id | vo2max
```

→ `merge()`

```python
pd.merge(
    athletes,
    fitness,
    on="athlete_id",
    how="left"
)
```

### 선수 ID가 이미 인덱스인 경우

```text
선수 정보 index = athlete_id
경기 기록 index = athlete_id
```

→ `join()`

```python
athletes.join(
    stats,
    how="left"
)
```

<br>

> ## **4-3 ) SQL과 Pandas 결합 비교**

| SQL | Pandas | 의미 |
| --- | --- | --- |
| `UNION ALL` | `pd.concat(..., axis=0)` | 행 방향으로 데이터 연결 |
| `INNER JOIN` | `pd.merge(..., how='inner')` | 공통 Key만 유지 |
| `LEFT JOIN` | `pd.merge(..., how='left')` | 왼쪽 데이터 유지 |
| `RIGHT JOIN` | `pd.merge(..., how='right')` | 오른쪽 데이터 유지 |
| `FULL OUTER JOIN` | `pd.merge(..., how='outer')` | 양쪽 데이터 모두 유지 |

`concat()`과 SQL `UNION ALL`은 비슷한 목적을 가지지만 완전히 동일한 기능이라고 보기보다는 **행 방향으로 데이터를 합친다는 개념이 유사함**으로 이해하는 것이 좋음.


<br><br>

---

# **5. 복합 예제: 사이클 선수 데이터 결합**

선수 기본 정보와 여러 번 측정한 파워 데이터를 결합한다고 가정함.

한 선수의 파워 측정값이 여러 개 존재하므로 먼저 `groupby()`로 선수별 통계를 계산하고 이후 `merge()`를 수행함.

> ## **5-1 ) 선수별 파워 측정값 집계 후 병합**

[ python : code ] "사이클 선수별 평균·최대 파워 계산 후 선수 정보와 병합"

```python
import pandas as pd

riders = pd.DataFrame({
    "rider_id": [1, 2, 3, 4],
    "name": ["태민", "윤서", "도현", "채원"],
    "team": ["A", "A", "B", "B"]
})

power = pd.DataFrame({
    "rider_id": [1, 1, 2, 3, 3, 3],
    "power_w": [250, 270, 260, 240, 255, 265]
})

power_summary = (
    power
    .groupby("rider_id", as_index=False)
    .agg(
        test_count=("power_w", "count"),
        avg_power=("power_w", "mean"),
        max_power=("power_w", "max")
    )
)

final = pd.merge(
    riders,
    power_summary,
    on="rider_id",
    how="left"
)

print(final.to_string(index=False))
```

[ Result ]

```text
 rider_id name team  test_count  avg_power  max_power
        1   태민    A         2.0 260.000000      270.0
        2   윤서    A         1.0 260.000000      260.0
        3   도현    B         3.0 253.333333      265.0
        4   채원    B         NaN        NaN        NaN
```

### 결과 해석

`rider_id=1`인 태민의 측정값은 `250W`, `270W`임.

```text
측정 횟수 = 2회

평균 파워
= (250 + 270) / 2
= 520 / 2
= 260W

최대 파워
= 270W
```

`rider_id=2`인 윤서의 측정값은 `260W` 하나임.

```text
측정 횟수 = 1회
평균 파워 = 260W
최대 파워 = 260W
```

`rider_id=3`인 도현의 측정값은 `240W`, `255W`, `265W`임.

```text
측정 횟수 = 3회

평균 파워
= (240 + 255 + 265) / 3
= 760 / 3
= 253.333333...W

최대 파워
= 265W
```

`rider_id=4`인 채원은 `riders`에는 존재하지만 `power` 데이터에는 측정 기록이 없음.

`how="left"`를 사용했기 때문에 채원은 결과에 유지되지만 통계값은 `NaN`으로 표시됨.

### 코드 흐름도

```text
[riders 선수 기본 정보]
1 태민
2 윤서
3 도현
4 채원
        │
        │
        │
        └─────────────────────────┐
                                  │
[power 측정 데이터]                │
1 → 250, 270                      │
2 → 260                           │
3 → 240, 255, 265                 │
                                  │
        ↓                         │
groupby("rider_id")               │
        ↓                         │
선수별 파워 통계 계산              │
        ↓                         │
1 → 2회 / 평균 260 / 최대 270      │
2 → 1회 / 평균 260 / 최대 260      │
3 → 3회 / 평균 253.33 / 최대 265   │
        │                         │
        └───────── merge ─────────┘
                  │
                  ↓
          how="left"
                  │
                  ↓
모든 riders 유지 + 파워 통계 추가
                  │
                  ↓
4번 채원 → 측정 기록 없음 → NaN
```

이처럼 실무 데이터에서는 **먼저 필요한 단위로 데이터를 가공한 후 다른 데이터와 병합**하는 과정이 자주 사용됨.


<br><br>

---

# **6. Advanced 추가 학습**

> ## **6-1 ) merge 후 행 개수가 늘어나는 이유**

`merge()`에서는 Key가 중복되어 있으면 결과 행 수가 예상보다 많아질 수 있음.

예를 들어,

```text
왼쪽
player_id
1
1

오른쪽
player_id
1
1
1
```

이라면 `player_id=1`끼리 모든 조합이 만들어져 결과가 다음과 같이 `2 × 3 = 6행`이 될 수 있음.

```text
왼쪽 1번째 × 오른쪽 1번째
왼쪽 1번째 × 오른쪽 2번째
왼쪽 1번째 × 오른쪽 3번째

왼쪽 2번째 × 오른쪽 1번째
왼쪽 2번째 × 오른쪽 2번째
왼쪽 2번째 × 오른쪽 3번째
```

따라서 병합 전에는 Key의 중복 여부를 확인하는 것이 중요함.

```python
df["player_id"].duplicated().sum()
```

또는

```python
df["player_id"].value_counts()
```

등으로 확인할 수 있음.

<br>

> ## **6-2 ) merge의 validate 옵션**

예상하는 데이터 관계가 맞는지 검증하기 위해 `validate` 옵션을 사용할 수 있음.

```python
pd.merge(
    left,
    right,
    on="athlete_id",
    how="left",
    validate="one_to_one"
)
```

대표적인 값은 다음과 같음.

| 옵션 | 의미 |
| --- | --- |
| `one_to_one` | 양쪽 Key 모두 중복 없음 |
| `one_to_many` | 왼쪽은 고유, 오른쪽은 중복 가능 |
| `many_to_one` | 왼쪽은 중복 가능, 오른쪽은 고유 |
| `many_to_many` | 양쪽 모두 중복 가능 |

예상한 관계와 실제 데이터가 다르면 오류가 발생하므로 잘못된 병합을 빠르게 발견하는 데 도움됨.

<br>

> ## **6-3 ) append() 대신 concat()**

과거 Pandas에서는 다음과 같이 `append()`를 사용하는 코드가 존재했음.

```python
df1.append(df2)
```

하지만 `DataFrame.append()`는 Pandas 2.0에서 제거되었으므로 현재는 `pd.concat()`을 사용해야 함.

```python
pd.concat(
    [df1, df2],
    ignore_index=True
)
```


<br><br>

---

# **7. 핵심정리**

1. `pd.concat()`은 여러 DataFrame을 단순히 행 또는 열 방향으로 연결할 때 사용함.

2. `axis=0`은 행 방향, `axis=1`은 열 방향 연결이며 `axis=1`에서는 인덱스를 기준으로 데이터가 정렬됨.

3. 행 방향 `concat()`에서 기존 인덱스가 필요하지 않다면 `ignore_index=True`를 사용하여 인덱스를 다시 생성할 수 있음.

4. 컬럼 구조가 다른 DataFrame을 `axis=0`으로 결합하면 존재하지 않는 값은 `NaN`으로 채워짐.

5. `pd.merge()`는 공통 Key 컬럼을 기준으로 데이터를 연결하며 SQL의 `JOIN`과 유사함.

6. `inner`는 양쪽 공통 Key, `left`는 왼쪽 전체, `right`는 오른쪽 전체, `outer`는 양쪽 전체 Key를 유지함.

7. `join()`은 기본적으로 인덱스를 기준으로 데이터를 연결하므로 두 DataFrame에서 결합 기준이 이미 인덱스로 설정된 경우 유용함.

8. 결합 방법은 다음 기준으로 선택하면 됨.

| 원하는 작업 | 사용 함수 |
| --- | --- |
| 같은 구조의 경기 데이터를 위아래로 합치기 | `concat()` |
| 선수 ID와 같은 Key를 기준으로 정보 연결 | `merge()` |
| 동일한 인덱스를 기준으로 정보 연결 | `join()` |

9. `merge()` 전에는 Key 중복 여부와 데이터 관계를 확인해야 예상하지 못한 행 증가를 방지할 수 있음.

10. 데이터 결합의 핵심은 함수를 암기하는 것이 아니라 **단순히 데이터를 이어붙일 것인지, 특정 Key를 기준으로 연결할 것인지, 인덱스를 기준으로 연결할 것인지 먼저 판단하는 것**임.