# **[Tableau] : <br> 데이터 결합 - Join, Union, Blending, Relationship**

<br>

# **1. Tableau 데이터 결합의 전체 구조**

> ## **1-1) 먼저 알아야 할 핵심**

실무 데이터는 하나의 테이블에 모든 정보가 들어 있는 경우보다 여러 테이블이나 파일에 나뉘어 있는 경우가 많다.

예를 들어 프로축구팀 데이터가 다음과 같이 관리된다고 가정한다.

```text
players
→ 선수 기본정보

matches
→ 경기 정보

gps_sessions
→ 선수별 경기 GPS 요약

injuries
→ 부상 사건

wellness
→ 수면·피로·근육통

gps_2026_01.csv
gps_2026_02.csv
gps_2026_03.csv
→ 월별 GPS 파일
```

다음 질문에 따라 필요한 데이터도 달라진다.

```text
포지션별 평균 HSR은?
→ players + gps_sessions

경기별 GPS 부하는?
→ matches + gps_sessions

선수별 부상 횟수는?
→ players + injuries

1~3월 GPS를 모두 분석하려면?
→ 월별 GPS 파일
```

Tableau에서는 데이터 구조에 따라 크게 다음 방법을 사용할 수 있다.

| 방법 | 핵심 목적 |
| --- | --- |
| `Join` | 행 수준에서 컬럼을 결합 |
| `Union` | 같은 구조의 행을 추가 |
| `Relationship` | 논리 테이블 사이의 관계 정의 |
| `Blending` | 별도 데이터 원본의 집계 결과를 Worksheet에서 결합 |

가장 중요한 것은 결합 방식의 이름을 외우는 것이 아니다.

```text
분석 질문
    ↓
각 테이블의 Grain 확인
    ↓
공통 Field 확인
    ↓
행 수준 병합이 필요한가?
    ↓
같은 구조를 쌓는 것인가?
    ↓
서로 다른 Grain을 유지해야 하는가?
    ↓
별도 Data Source를 유지해야 하는가?
    ↓
적절한 결합 방법 선택
```

Tableau 공식 데이터 모델은 `Relationship`을 Logical Layer에서, `Join`과 `Union`을 Physical Layer에서 사용하도록 구성한다. Relationship에서는 논리 테이블이 하나의 Flat Table로 미리 합쳐지지 않고 각각의 기본 세부 수준을 유지한다.


<br><br>

---

# **2. Granularity - 데이터 결합 전에 가장 먼저 확인할 것**

> ## **2-1) Granularity란?**

Granularity 또는 Grain은 **데이터 한 행이 무엇을 의미하는지 나타내는 세부 수준**이다.

스포츠 데이터를 예로 들면 다음과 같다.

| Table | 한 행의 의미 |
| --- | --- |
| `players` | 선수 1명 |
| `matches` | 경기 1경기 |
| `gps_sessions` | 선수 × 경기 1건 |
| `gps_1hz` | 선수 × 1초 |
| `injuries` | 부상 사건 1건 |
| `shots` | 슈팅 1회 |

예를 들어

```text
players

P001 | Winger
P002 | Midfielder
```

는

```text
1 Player
= 1 Row
```

이다.

반면

```text
gps_sessions

P001 | M001 | 10420
P001 | M002 | 10180
P002 | M001 | 10870
```

는

```text
1 Player × 1 Match
= 1 Row
```

이다.

두 테이블 모두 `player_id`를 가지고 있지만 **한 행의 의미가 서로 다르다.**

<br>

> ## **2-2) Grain이 다른 데이터를 무작정 Join하면?**

경기 테이블이 다음과 같다고 하자.

```text
matches

match_id | attendance
M001     | 20000
M002     | 18000
```

선수 경기기록은 다음과 같다.

```text
player_match

match_id | player_id
M001     | P001
M001     | P002
M001     | P003
M002     | P001
M002     | P004
```

`match_id`로 Join하면 다음처럼 된다.

```text
M001 | P001 | 20000
M001 | P002 | 20000
M001 | P003 | 20000
M002 | P001 | 18000
M002 | P004 | 18000
```

이 상태에서

```text
SUM(attendance)
```

를 계산하면

```text
20000 + 20000 + 20000 + 18000 + 18000
= 96000
```

이 된다.

하지만 실제 두 경기의 총 관중은

```text
20000 + 18000
= 38000
```

이다.

즉,

```text
Join이 정상적으로 실행됨
≠
Measure 집계도 정확함
```

이다.

따라서 결합 전에는 반드시 다음을 질문해야 한다.

```text
현재 Table의 한 행은 무엇인가?

결합 후 한 행은 무엇이 되는가?

Measure가 반복될 가능성이 있는가?
```


<br><br>

---

# **3. Join - 물리적으로 행을 병합하는 방법**

> ## **3-1) Tableau에서 Join의 정확한 의미**

`Join`은 **공통 Field에 대한 Join Clause와 Join Type을 지정하여 Physical Table들을 하나의 Logical Table 안에서 병합하는 방식**이다.

Tableau에서는 데이터 원본 화면의 `Logical Table`을 두 번 클릭하면 `Physical Layer`로 들어갈 수 있고, 이곳에서 `Join`과 `Union`을 구성한다. Join된 Physical Table은 하나의 Flat한 Logical Table 구조를 만든다.

예를 들어 선수정보와 GPS를 결합한다고 하자.

`players`

```text
player_id | position
P001      | Winger
P002      | Midfielder
```

`gps_sessions`

```text
player_id | match_id | hsr_m
P001      | M001     | 1180
P002      | M001     | 890
P001      | M002     | 1060
```

`player_id`를 기준으로 Join하면

```text
player_id | match_id | hsr_m | position
P001      | M001     | 1180  | Winger
P002      | M001     | 890   | Midfielder
P001      | M002     | 1060  | Winger
```

처럼 된다.

즉 Join은

```text
GPS의 행
+
Player의 Column
```

을 하나의 Row-level 구조로 만든다.

<br>

> ## **3-2) SQL로 이해하는 Join**

[ SQL : code ] "축구 GPS 데이터에 선수정보 Left Join"

```sql
SELECT
    g.match_id,
    g.player_id,
    p.position,
    g.total_distance_m,
    g.hsr_distance_m
FROM gps_sessions AS g
LEFT JOIN players AS p
    ON g.player_id = p.player_id;
```

[ Result ]

```text
match_id | player_id | position   | total_distance_m | hsr_distance_m
----------------------------------------------------------------------
M001     | P001      | Winger     | 10420            | 1180
M001     | P002      | Midfielder | 10870            | 890
M002     | P001      | Winger     | 10180            | 1060
```

Tableau Join도 개념적으로 이러한 Row-level 병합과 비슷하게 이해할 수 있다.


<br><br>

---

# **4. Join Type**

> ## **4-1) Inner Join**

Inner Join은 양쪽 Table에서 Join 조건이 일치하는 행만 유지한다.

```text
GPS          Player

P001         P001
P002         P002
P004         P003

      ↓ INNER

P001
P002
```

즉

```text
양쪽 모두 존재
→ 유지

한쪽에만 존재
→ 제거
```

이다.

<br>

> ## **4-2) Left Join**

Left Join은 왼쪽 Table의 행을 모두 유지한다.

```text
GPS          Player

P001         P001
P002         P002
P004         P003
```

결과는

```text
P001 → Player 정보 연결
P002 → Player 정보 연결
P004 → Player 정보 NULL
```

이다.

GPS 기록을 반드시 모두 보존해야 한다면 GPS를 왼쪽 Table로 두는 방식을 고려할 수 있다.

<br>

> ## **4-3) Right Join**

Right Join은 오른쪽 Table의 모든 행을 유지한다.

개념적으로

```text
Left Join의 기준 Table을 반대로 둔 형태
```

로 이해할 수 있다.

<br>

> ## **4-4) Full Outer Join**

Full Outer Join은 양쪽에 존재하는 모든 행을 유지한다.

```text
GPS          Player

P001         P001
P002         P002
P004         P003
```

결과:

```text
P001
P002
P003
P004
```

일치하지 않는 상대 Table의 Field에는 `NULL`이 들어간다.

<br>

> ## **4-5) Join Type 비교**

| Join | 왼쪽 미일치 | 오른쪽 미일치 | 일치 |
| --- | --- | --- | --- |
| Inner | 제거 | 제거 | 유지 |
| Left | 유지 | 제거 | 유지 |
| Right | 제거 | 유지 | 유지 |
| Full Outer | 유지 | 유지 | 유지 |

Join Type을 선택할 때 핵심은

```text
어떤 데이터가 결과에서
반드시 보존되어야 하는가?
```

이다.


<br><br>

---

# **5. Join Clause와 Composite Key**

> ## **5-1) 하나의 Key만으로 충분하지 않을 수 있다**

GPS와 Wellness 데이터가 모두 선수별·날짜별 데이터라고 가정한다.

`gps_daily`

```text
player_id | date       | player_load
P001      | 2026-03-01 | 620
P001      | 2026-03-02 | 655
```

`wellness_daily`

```text
player_id | date       | sleep_hr
P001      | 2026-03-01 | 6.8
P001      | 2026-03-02 | 6.2
```

`player_id`만 Join하면 한 선수의 여러 날짜가 서로 조합될 수 있다.

따라서

```text
player_id
+
date
```

를 함께 사용해야 한다.

[ SQL : code ] "선수와 날짜를 동시에 이용한 Join"

```sql
SELECT
    g.player_id,
    g.session_date,
    g.player_load,
    w.sleep_hr
FROM gps_daily AS g
LEFT JOIN wellness_daily AS w
    ON g.player_id = w.player_id
   AND g.session_date = w.measure_date;
```

[ Result ]

```text
player_id | session_date | player_load | sleep_hr
-------------------------------------------------
P001      | 2026-03-01   | 620         | 6.8
P001      | 2026-03-02   | 655         | 6.2
```

즉 Join에서는 단순히

```text
같은 Column 이름이 있는가?
```

보다

```text
어떤 Field 조합이
동일한 관측값을 식별하는가?
```

가 중요하다.


<br><br>

---

# **6. Join의 가장 중요한 위험 — Duplication과 Filtering**

> ## **6-1) One-to-Many**

선수 Master가

```text
1 Player = 1 Row
```

이고 GPS가

```text
1 Player = 여러 경기
```

라면

```text
Players 1
   ↓
GPS N
```

관계이다.

Join 후 선수정보가 여러 GPS 행에 반복되는 것은 정상이다.

문제는 선수 Table에 존재하는 Measure를 잘못 합계하는 경우이다.

<br>

> ## **6-2) Many-to-Many Join Explosion**

GPS:

```text
P001 | Match1
P001 | Match2
```

부상 기록:

```text
P001 | Injury1
P001 | Injury2
```

`player_id`만으로 Join하면

```text
Match1 × Injury1
Match1 × Injury2
Match2 × Injury1
Match2 × Injury2
```

가 되어

```text
2 × 2
= 4 Rows
```

가 생성된다.

따라서 Many-to-Many 상황에서 Physical Join을 사용할 경우 행 증가와 Measure 중복에 특히 주의해야 한다.

Tableau 공식 문서도 서로 다른 Level of Detail의 Table을 Join하면 데이터가 중복되거나 Join Type에 따라 데이터가 손실될 수 있다고 설명한다.


<br><br>

---

# **7. Union - 행을 아래로 추가하는 방법**

> ## **7-1) Tableau Union의 정확한 의미**

Union은 **한 Table의 행을 다른 Table에 추가하여 두 개 이상의 Table을 하나로 결합하는 방식**이다.

Tableau에서는 Union에 포함되는 Table들이 동일한 Connection에 있어야 하며, 가장 안정적인 결과를 위해서는 Field 수·Field 이름·Data Type 등 구조가 같은 것이 좋다.

예를 들어

```text
gps_2026_01
gps_2026_02
gps_2026_03
```

가 동일한 Schema라면 다음처럼 쌓을 수 있다.

```text
January GPS
      ↓
February GPS
      ↓
March GPS
      ↓
Full GPS Dataset
```

Join이 주로 Column을 늘린다면 Union은 Row를 늘린다.

<br>

> ## **7-2) 월별 GPS 예제**

1월:

```text
player_id | date       | hsr_m
P001      | 2026-01-05 | 1030
P002      | 2026-01-05 | 870
```

2월:

```text
player_id | date       | hsr_m
P001      | 2026-02-04 | 1090
P003      | 2026-02-04 | 780
```

Union 후:

```text
player_id | date       | hsr_m
P001      | 2026-01-05 | 1030
P002      | 2026-01-05 | 870
P001      | 2026-02-04 | 1090
P003      | 2026-02-04 | 780
```

<br>

> ## **7-3) SQL과 Tableau의 Union을 구분하기**

SQL에는

```text
UNION
UNION ALL
```

이 존재한다.

일반 SQL에서 `UNION`은 중복 Row를 제거하고 `UNION ALL`은 모두 유지한다.

Tableau Desktop의 Union은 기본적으로 **여러 Table의 Row를 쌓는 데이터 결합 기능**으로 이해하는 것이 적절하다.

따라서 Tableau Union을 SQL `UNION`의 “중복 제거” 동작과 동일하다고 기억하면 안 된다.


<br><br>

---

# **8. Wildcard Union**

> ## **8-1) Tableau Desktop의 Wildcard Union**

다음처럼 월별 파일명이 일정한 경우를 생각해보자.

```text
gps_2026_01.csv
gps_2026_02.csv
gps_2026_03.csv
...
```

Tableau Desktop은 Wildcard Union에서 `*` 패턴을 사용해 조건과 일치하는 Table이나 File을 자동으로 Union 대상에 포함시킬 수 있다.

예를 들면 개념적으로

```text
gps_2026_*
```

와 같은 이름 패턴을 사용할 수 있다.

Wildcard 검색은 선택한 Connection을 기준으로 작동하며, Connector 종류에 따라 File이나 Workbook을 가로질러 검색할 수도 있다.

<br>

> ## **8-2) Wildcard Union 사용 시 주의**

자동으로 새로운 파일이 포함될 수 있기 때문에 다음을 관리해야 한다.

```text
Schema 변경

Field 이름 변경

단위 변경

잘못된 File 이름

Data Type 변경
```

따라서 자동 Union을 사용할수록 **Schema 관리가 중요하다.**


<br><br>

---

# **9. Join과 Union의 차이**

> ## **9-1) 가장 간단한 비교**

| 구분 | Join | Union |
| --- | --- | --- |
| 목적 | 다른 정보를 연결 | 같은 종류의 데이터를 추가 |
| 방향 | 가로 | 세로 |
| 핵심 기준 | Join Clause | 호환되는 Schema |
| 주로 증가 | Column | Row |
| 예 | GPS + 선수정보 | 1월 GPS + 2월 GPS |
| 주요 위험 | 중복 / 누락 | Schema 불일치 |

```text
다른 정보가 필요하다.
→ Join

같은 구조의 관측값을 더 추가한다.
→ Union
```


<br><br>

---

# **10. Tableau Data Model - Logical Layer와 Physical Layer**

> ## **10-1) Tableau 데이터 모델은 두 Layer로 구성된다**

현재 Tableau의 Data Model은 크게 두 Layer로 나뉜다.

```text
Logical Layer
→ Relationship

Physical Layer
→ Join / Union
```

Data Source 화면에서 처음 보이는 Top-level Canvas가 Logical Layer이다.

여기에 배치되는 Table을 **Logical Table**이라고 한다.

Logical Table을 두 번 클릭하면 Physical Layer로 이동하며 내부의 Physical Table을 Join하거나 Union할 수 있다.

<br>

> ## **10-2) Logical Table은 Container 역할을 한다**

Logical Table 하나는

```text
Physical Table 1개
```

만 포함할 수도 있고,

```text
Physical Table A
JOIN
Physical Table B
UNION
Physical Table C
```

처럼 여러 Physical Table로 구성될 수도 있다.

따라서 개념적으로

```text
Logical Layer

[GPS] ↔ [Players] ↔ [Matches]
```

처럼 Relationship을 정의할 수 있고,

`GPS` Logical Table 내부에는

```text
January GPS
UNION
February GPS
UNION
March GPS
```

가 들어 있을 수도 있다.


<br><br>

---

# **11. Relationship - Tableau의 논리적 데이터 연결**

> ## **11-1) Relationship의 정확한 의미**

Relationship은 **두 Logical Table이 어떤 Field로 관련되어 있는지 정의하지만, Table을 하나로 미리 병합하지 않는 방식**이다.

관계를 설정해도 Logical Table은 각각 독립된 상태로 유지되며 고유한 Level of Detail과 Domain을 유지한다.

예:

```text
Players
   ↕ player_id
GPS Sessions
   ↕ match_id
Matches
```

여기에서 세 Table을 미리 하나의 거대한 Flat Table로 만들지 않는다.

<br>

> ## **11-2) Relationship에서는 Join Type을 직접 지정하지 않는다**

Relationship에서는

```text
Inner
Left
Right
Full Outer
```

같은 Join Type을 사용자가 미리 고정하지 않는다.

Worksheet에서 어떤 Dimension과 Measure를 사용하는지에 따라 Tableau가 분석 시점에 필요한 Query와 Contextual Join을 결정한다.

따라서

```text
Relationship
= 특정 Join Type의 다른 이름
```

이 아니다.

<br>

> ## **11-3) Relationship을 왜 사용하는가?**

서로 다른 Grain의 Table을 Physical Join하면

```text
Measure 반복
Row Duplication
Unmatched Row Filtering
NULL 추가
```

등의 문제가 생길 수 있다.

Relationship은 Table을 별도로 유지하고 View에 필요한 Field를 기준으로 Query를 구성하기 때문에 이러한 물리적 Join의 중복·필터링 문제를 피하기 쉽다. Tableau 역시 Relationship을 다중 테이블 데이터 결합의 기본 접근으로 권장한다.


<br><br>

---

# **12. Relationship과 Join의 정확한 비교**

> ## **12-1) 핵심 차이**

| 구분 | Relationship | Join |
| --- | --- | --- |
| Data Model | Logical Layer | Physical Layer |
| Table 처리 | 독립적으로 유지 | 하나의 Table로 병합 |
| Join Type | 미리 지정하지 않음 | 직접 지정 |
| Query | Viz Context에 따라 구성 | 모든 Query에 물리 구조 반영 |
| Grain | 각 Logical Table의 기본 LOD 유지 | 병합된 행 수준 |
| 다른 Grain 분석 | 적합 | 중복 주의 |
| 명시적인 Row-level 구조 | 제한적 | 적합 |

Tableau 공식 문서에서는 Relationship이 **Viz에서 실제로 필요한 Table만 Query**하고, 분석 Context에 맞는 Join을 형성하는 반면, Physical Join은 만들어진 Join 구조가 Query에 반영된다고 설명한다.

<br>

> ## **12-2) Join이 더 적합한 경우도 있다**

Relationship이 항상 Join보다 “좋다”라고 이해하면 안 된다.

다음처럼 명시적인 Row-level 병합 자체가 분석의 목적이면 Join이 적절할 수 있다.

```text
반드시 Left Join 결과가 필요함

두 Table을 하나의 고정된 Row Structure로 만들어야 함

Physical Table 수준의 계산을 사용해야 함

Join의 Filtering / Duplication 효과가 의도된 분석임
```

Tableau 역시 명시적인 특정 Join Type이나 물리적 병합이 필요한 경우 Join을 사용할 수 있다고 설명한다.


<br><br>

---

# **13. Relationship 실무 예제**

> ## **13-1) 축구 퍼포먼스 데이터 모델**

Table 구조:

```text
players
→ 1 Player = 1 Row

matches
→ 1 Match = 1 Row

gps_sessions
→ 1 Player × Match = 1 Row

injuries
→ 1 Injury Event = 1 Row
```

Logical Model을 다음과 같이 설계할 수 있다.

```text
             Matches
                ↕
             match_id
                ↕
Players ↔ GPS Sessions
   ↕
player_id
   ↕
Injuries
```

이를 이용하면

```text
포지션별 평균 HSR

경기별 평균 Player Load

선수별 부상 건수

포지션별 부상 현황
```

등을 한 Data Source에서 분석할 수 있다.

<br>

> ## **13-2) View Context에 따라 사용하는 Table이 달라진다**

질문:

```text
포지션별 평균 HSR
```

이면 주로

```text
Players
+
GPS Sessions
```

가 필요하다.

질문:

```text
상대팀별 관중 수
```

이면

```text
Matches
```

만으로 충분할 수도 있다.

즉 Relationship Data Model의 장점 중 하나는 모든 Logical Table을 항상 물리적으로 결합하지 않는다는 것이다.


<br><br>

---

# **14. Cardinality(관계) - Relationship Performance Option**

> ## **14-1) Tableau에서 Cardinality의 의미**

Relationship에는 선택적인 **Performance Options**가 있다.

그중 Cardinality는 관계 Field 값이 각 Table에서

```text
One
또는
Many

; 일대일, 일대다, 다대다의 관계
```

인지 Tableau에 알려준다.

예:

```text
Players
player_id Unique
→ One

GPS Sessions
player_id Repeated
→ Many
```

따라서

```text
Players
One : Many
GPS Sessions
```

관계라고 표현할 수 있다.

<br>

> ## **14-2) Cardinality는 Query 최적화에 영향을 준다**

Tableau에서 Cardinality 설정은 관련 Table을 자동으로 결합하는 과정에서 **집계를 Join 전 또는 후에 수행하는 방식**에 영향을 줄 수 있다.

`Many`는 Field가 Unique하지 않거나 확신이 없을 때 사용하는 안전한 선택이다.

`One`은 실제로 해당 Relationship Key가 Unique하다고 확신할 때 사용한다. `One`으로 잘못 지정했는데 실제 값이 중복되어 있다면 View에서 Aggregate가 중복되어 잘못 표시될 수 있다.

따라서

```text
Unique한 것 같다.
→ One
```

처럼 추측해서 변경하면 안 된다.

<br>

> ## **14-3) 모르면 기본값을 유지한다**

Tableau가 구조를 자동으로 감지하지 못했을 때 안전한 기본 방향은

```text
Cardinality
→ Many-to-Many

Referential Integrity
→ Some Records Match
```

이다.

데이터 구조를 확실하게 알고 있을 때만 Performance Options를 조정하는 것이 좋다.


<br><br>

---

# **15. Referential Integrity(참조 무결성)**

> ## **15-1) Referential Integrity란?**

Referential Integrity는 관계의 한쪽 Record가 다른 Table에서 **항상 일치하는 Record를 가지고 있는지**를 나타낸다.

예를 들어 모든 GPS 기록의 `player_id`가 반드시 선수 Master에 존재한다면

```text
GPS
→ Players에 항상 Match
```

하는 구조이다.

<br>

> ## **15-2) Tableau의 설정**

대표 설정은 다음과 같이 이해할 수 있다.

```text
Some Records Match
→ 모든 Record가 Match한다고 보장할 수 없음

All Records Match
→ 반드시 Match한다고 확신
```

`Some Records Match` 상황에서는 Tableau가 unmatched Measure도 보존할 수 있도록 Outer Join 계열의 처리를 사용할 수 있다.

`All Records Match`를 지정하면 Tableau가 더 간단한 Query를 사용할 수 있지만 실제 unmatched 값이 존재하면 데이터가 View에서 누락될 수 있다.

따라서 Referential Integrity는 단순한 설명용 Metadata가 아니라 **잘못 지정하면 실제 분석 결과가 달라질 수 있는 Performance Option**이다.


<br><br>

---

# **16. Relationship 설정에서 중요한 원칙**

> ## **16-1) 기본값은 안전한 선택이다**

Relationship Performance Option은 성능 최적화를 위한 기능이다.

```text
Cardinality를 정확히 알고 있음
+
Referential Integrity를 정확히 알고 있음
```

인 경우에만 보다 구체적으로 설정하는 것이 좋다.

모른다면

```text
Many

Some Records Match
```

계열의 기본값을 유지하는 것이 안전하다.

잘못된 Cardinality는 Aggregate 중복을, 잘못된 Referential Integrity는 unmatched Record 누락을 발생시킬 수 있다.


<br><br>

---

# **17. Blending - 별도 Data Source 결과의 View 수준 결합**

> ## **17-1) Blending의 정확한 의미**

Data Blending은 **여러 Tableau Data Source를 각각 독립적으로 Query하고 그 집계 결과를 Worksheet의 View에서 연결하는 방식**이다.

즉,

```text
Table A Row
+
Table B Row
```

를 직접 병합하는 방식이 아니다.

```text
Primary Source Query
        ↓
Aggregate Result

Secondary Source Query
        ↓
Aggregate Result

        ↓

Linking Field를 기준으로
View에서 결합
```

하는 구조이다.

<br>

> ## **17-2) Primary와 Secondary**

Blending은 Sheet별로 동작한다.

View에서 먼저 사용되는 Data Source가 Primary가 되고, 이후 추가되는 Data Source가 Secondary가 된다.

Tableau Desktop에서는 Primary Source가 파란 Check Mark, Secondary Source가 주황 Check Mark로 표시된다.

<br>

> ## **17-3) Linking Field**

두 Data Source를 연결할 공통 Dimension을 Linking Field라고 한다.

예를 들어

```text
경기매출 Data Source
→ match_date, revenue

마케팅 Excel
→ match_date, ad_cost
```

라면

```text
match_date
```

를 Linking Field로 사용할 수 있다.

Tableau는 이름이 동일한 Field를 자동 Linking 후보로 인식할 수 있고, 이름이 다르면 수동으로 Blend Relationship을 정의할 수 있다.


<br><br>

---

# **18. Blending은 Left Join처럼 동작한다**

> ## **18-1) 중요한 특성**

Blending은 관계형 Row-level Left Join 자체는 아니지만 **결과 보존 방식은 Primary Source를 기준으로 하는 Left Join과 유사하게 동작한다.**

따라서 Secondary Source에만 존재하고 Primary Source에는 존재하지 않는 값은 View에서 표시되지 않을 수 있다.

예:

Primary:

```text
Jan
Feb
```

Secondary:

```text
Jan
Feb
Mar
```

라면 Primary가 기준이므로

```text
Jan
Feb
```

가 View의 기준이 되고 `Mar`는 나타나지 않을 수 있다.

<br>

> ## **18-2) Secondary Data는 Aggregate되어 사용된다**

Blending에서는 Secondary Data Source의 Field를 계산에 사용할 때 항상 집계된 형태로 사용해야 하는 등의 제약이 있다.

또한 `COUNTD`, `MEDIAN` 같은 일부 Non-additive Aggregate에 제한이 있을 수 있다.

따라서 Blending은 일반 Join보다 계산 측면의 제약이 더 많을 수 있다.


<br><br>

---

# **19. Relationship과 Blending의 정확한 차이**

> ## **19-1) Blending을 단순히 "서로 다른 파일 연결"로 외우지 않는다**

과거에는

```text
다른 Data Source
→ Blending
```

으로 설명하는 경우가 많았지만 현재 Tableau에서는 지나치게 단순한 설명이다.

현재는 여러 Connection의 Table을 하나의 Tableau Data Source 안에서 구성할 수 있는 경우 Relationship이나 Cross-database Join을 사용할 수도 있다.

Blending은 특히

```text
Data Source를 독립적으로 유지해야 함

Sheet마다 Linking Field가 달라져야 함

별도 Published Data Source를 활용함

특정 Cube / Relational 조합을 다룸
```

등의 상황에서 고려할 수 있다. Tableau는 일반적인 Multi-table 분석에서는 Relationship을 우선적인 접근으로 설명한다.

<br>

> ## **19-2) Relationship vs Blending**

| 구분 | Relationship | Blending |
| --- | --- | --- |
| 범위 | 한 Tableau Data Source의 Logical Model | 여러 Tableau Data Source |
| 동작 | Analysis Context에서 Query 생성 | Source별 Query 후 View에서 결합 |
| 설정 지속성 | Workbook Data Model | Worksheet별 |
| Join 보존 방식 | Context에 따라 Tableau가 결정 | Primary 기준 Left Join 유사 |
| Grain | 각 Logical Table의 LOD 유지 | 각 Source Aggregate 결과 |
| 계산 제약 | 상대적으로 적음 | Secondary 계산 제약 존재 |
| 권장 상황 | 일반적인 Multi-table 분석 | 별도 Source를 유지해야 하는 특수 상황 |


<br><br>

---

# **20. Cross - Database Join과 Blending 구분**

> ## **20-1) Database가 다르다고 무조건 Blending은 아니다**

Tableau는 지원되는 Connector 조합에서 **Cross-database Join**을 구성할 수도 있다.

즉,

```text
SQL Server
+
Excel
```

또는 서로 다른 Database Connection이라고 해서 항상 Blending만 가능한 것은 아니다.

하나의 Tableau Data Source 안에서 Join할 수 있다면 Cross-database Join이라는 선택지가 존재한다.

다만 모든 Data Source가 Cross-database Join을 지원하는 것은 아니며 Published Tableau Data Source 등에는 제한이 있다.

따라서 올바른 사고는

```text
Data Source가 다름
→ 무조건 Blend
```

가 아니라

```text
하나의 Tableau Data Model로
통합할 수 있는가?

Relationship이 가능한가?

Cross-database Join이 필요한가?

독립 Data Source를 유지해야 하는가?
→ 그때 Blending 검토
```

이다.


<br><br>

---

# **21. 네 가지 방법을 정확하게 비교**

> ## **21-1) 핵심 비교표**

| 항목 | Join | Union | Relationship | Blending |
| --- | --- | --- | --- | --- |
| 목적 | Column 결합 | Row 추가 | Logical Table 연결 | Data Source 결과 연결 |
| Layer | Physical | Physical | Logical | Worksheet / Data Source 간 |
| 결합 시점 | 분석 전 구조 고정 | 분석 전 구조 고정 | 분석 Context에서 Query | View에서 Aggregate 결과 결합 |
| Grain | 병합 Row 수준 | 기존 Schema 수준 | 각 Logical Table의 LOD 유지 | 각 Source 집계 결과 |
| Join Type | 직접 지정 | 해당 없음 | 직접 지정하지 않음 | Primary 기준 Left 유사 |
| 주요 위험 | Duplicate / Data Loss | Schema 불일치 | 잘못된 Relationship Option | Secondary 누락 / 집계 제약 |
| 대표 상황 | GPS + Player | 월별 GPS | GPS + Match + Injury | 독립 Published Sources |

<br>

> ## **21-2) 선택 흐름**

```text
같은 구조의 Row를 추가할 것인가?
        ↓ YES
       Union
```

```text
Row-level로 직접 Column을 붙일 것인가?
        ↓ YES
       Join
```

```text
여러 Table이 서로 다른 Grain이고
하나의 Tableau Data Model에서
분석해야 하는가?
        ↓ YES
 Relationship 우선 검토
```

```text
독립된 Tableau Data Source를
그대로 유지하면서 하나의 Sheet에서
결과를 연결해야 하는가?
        ↓ YES
    Blending 검토
```


<br><br>

---

# **22. Tableau Prep Builder**

> ## **22-1) Tableau Prep의 정확한 역할**

Tableau Prep Builder는 **분석 전에 데이터를 연결하고, 정제하고, 구조를 변환하고, 결합하여 분석 가능한 Output을 만드는 데이터 준비 도구**이다.

Flow Pane에서 단계가 왼쪽에서 오른쪽으로 연결되며 데이터가 어떤 연산을 거쳤는지 확인할 수 있다.

예:

```text
Input
   ↓
Clean
   ↓
Union
   ↓
Join
   ↓
Aggregate
   ↓
Pivot
   ↓
Output
```

<br>

> ## **22-2) 주요 Step**

| Step | 역할 |
| --- | --- |
| Input | Data 연결 |
| Clean | Field / Value 정리 |
| Join | Column 방향 결합 |
| Union | Row 방향 결합 |
| Aggregate | Grain 변경 및 집계 |
| Pivot | Row / Column 구조 변경 |
| Output | 결과 저장 |

Tableau Prep은 Flow Output을 `.hyper`, `.csv`, `.xlsx`, Published Data Source, Database 등으로 출력할 수 있다.

<br>

> ## **22-3) Aggregate Step은 매우 중요하다**

Join하기 전에 Grain을 맞춰야 하는 경우가 있다.

예:

```text
GPS
→ 선수 × 초

Wellness
→ 선수 × 일
```

Raw GPS와 Wellness를 바로 Join하기보다

```text
GPS 1Hz
    ↓
Aggregate

선수 × 일 GPS
    ↓
Join

Wellness 선수 × 일
```

처럼 Grain을 맞출 수 있다.

Tableau Prep 공식 문서도 Aggregate Step을 데이터 양을 줄이거나 Join/Union할 다른 데이터와 Level of Detail을 맞추는 용도로 설명한다.


<br><br>

---

# **23. 실무형 종합 프로젝트**

> ## **23-1) 축구 퍼포먼스 데이터 모델**

원본 데이터:

```text
players
→ 선수 1명 = 1 Row

matches
→ 경기 1개 = 1 Row

gps_2026_01
gps_2026_02
gps_2026_03
→ 선수 × 경기 = 1 Row

injuries
→ 부상 Event = 1 Row

marketing.xlsx
→ 월별 구단 캠페인 비용
```

설계 과정:

```text
January GPS
February GPS
March GPS
       ↓
     Union
       ↓
GPS Sessions Logical Table
```

그다음

```text
Players ↔ GPS Sessions ↔ Matches
   ↕
Injuries
```

를 Relationship으로 구성할 수 있다.

마케팅 데이터가 독립된 Published Data Source 등으로 유지되어야 하고 동일 Sheet에서 결합해야 하는 특수 상황이라면 Blending을 추가적으로 고려할 수 있다.

<br>

> ## **23-2) 전체 구조**

```text
Raw Data
    ↓
Granularity 확인
    ↓
같은 Schema?
    │
    └─ Union
    ↓
Physical Row-level 병합 필요?
    │
    └─ Join
    ↓
Logical Tables 구성
    ↓
Relationships
    ↓
필요 시 독립 Source
    ↓
Blending
    ↓
Worksheet
    ↓
Dashboard
```


<br><br>

---

# **24. 데이터 결합 결과 검증**

> ## **24-1) 오류 없이 연결되었다고 끝이 아니다**

Join이나 Data Preparation이 성공적으로 완료되어도 결과가 분석적으로 잘못될 수 있다.

다음 항목을 확인하는 것이 좋다.

```text
Row Count

Distinct Key Count

NULL Count

Measure 합계

Data Type

단위

날짜 Grain

중복 Row

예상 Granularity
```

<br>

> ## **24-2) Join 전후 검증 예제**

Join 전:

```text
Rows
= 12,400

Distinct Player-Match
= 12,400
```

Join 후:

```text
Rows
= 17,850

Distinct Player-Match
= 12,400
```

이라면

```text
Row는 증가했지만
실제 분석 단위의 고유 Key는 그대로
```

이므로 Join Duplication을 의심할 수 있다.

[ SQL : code ] "분석 Grain과 중복 여부 확인"

```sql
SELECT
    COUNT(*) AS row_count,
    COUNT(
        DISTINCT CONCAT(player_id, '-', match_id)
    ) AS unique_player_match
FROM joined_gps;
```

[ Result ]

```text
row_count           = 17850
unique_player_match = 12400
```

이 결과만으로도

```text
17850 - 12400
= 5450 Rows
```

만큼 추가 조합이 만들어졌음을 확인할 수 있다.


<br><br>

---

# **25. 자주 잘못 이해하는 Tableau 결합 개념**

> ## **25-1) Relationship = 자동 Left Join이 아니다**

Relationship에서는 Join Type을 미리 설정하지 않는다.

Tableau가 현재 Viz의 Field와 Relationship 설정을 이용하여 필요한 Contextual Join을 구성한다.

```text
Relationship
≠
Left Join 자동화
```

이다.

<br>

> ## **25-2) Relationship = 무조건 중복이 없는 것은 아니다**

Relationship은 Join으로 인한 일반적인 Row Duplication 문제를 피하는 데 유리하지만 Relationship Field 자체가 잘못되었거나 Performance Options를 잘못 지정하면 결과가 잘못될 수 있다.

<br>

> ## **25-3) Blending = 다른 파일끼리 결합하는 일반적인 방법이 아니다**

현재 Tableau에서는 서로 다른 Table이나 Connection을 하나의 Data Model 안에서 Relationship 또는 Cross-database Join으로 다룰 수도 있다.

Blending은 **별도 Data Source를 유지하면서 Worksheet에서 Aggregate 결과를 연결하는 별도의 방식**이다.

<br>

> ## **25-4) Union = 중복 제거가 아니다**

Tableau Union의 핵심은

```text
Rows 추가
```

이다.

SQL의 `UNION`이 수행하는 중복 제거 개념과 혼동하지 않는다.

<br>

> ## **25-5) Cardinality 설정은 단순 설명용 Label이 아니다**

`One`, `Many` 설정은 Tableau의 Aggregate와 Join 최적화에 영향을 준다.

실제 중복 Key를 가지고 있는데 `One`이라고 잘못 선언하면 값이 중복 집계될 수 있다.

<br>

> ## **25-6) All Records Match를 추측해서 설정하면 안 된다**

실제로 unmatched Record가 있는데 `All Records Match`를 지정하면 일부 Record가 View에서 누락될 수 있다.

따라서 데이터 구조에 확신이 없다면 기본값을 유지한다.


<br><br>

---

# **26. Python과 SQL 개념으로 연결하기**

> ## **26-1) Pandas Merge**

[ Python : code ] "GPS와 선수정보 Join"

```python
result = gps.merge(
    players,
    on="player_id",
    how="left"
)

print(result)
```

[ Result ]

```text
  player_id match_id  hsr_m    position
0      P001     M001   1180      Winger
1      P002     M001    890  Midfielder
2      P001     M002   1060      Winger
```

개념적으로

```text
pandas.merge()
→ Join
```

과 연결해 생각할 수 있다.

<br>

> ## **26-2) Pandas Concat**

[ Python : code ] "월별 수영 기록 Union"

```python
result = pd.concat(
    [january, february, march],
    ignore_index=True
)

print(result)
```

[ Result ]

```text
  athlete_id  race_month  race_time
0       S001           1      53.82
1       S002           1      54.31
2       S001           2      53.41
3       S003           3      52.98
```

개념적으로

```text
pd.concat(axis=0)
→ Union
```

으로 이해할 수 있다.

단, Tableau `Relationship`과 `Blending`은 Pandas `merge()` 하나로 단순 대응되는 개념이 아니다.


<br><br>

---

# **27. Advanced 추가 학습**

> ## **27-1) Star Schema**

BI 데이터 모델에서는 Fact와 Dimension을 분리한 Star Schema를 많이 활용한다.

```text
               Player
                 │
                 │
Date ───── GPS Fact ───── Match
                 │
                 │
              Position
```

Fact Table:

```text
distance
hsr
sprint
player_load
```

Dimension Table:

```text
Player
Match
Date
Position
```

처럼 설계할 수 있다.

Tableau Relationship은 이러한 다중 Table 분석 구조와 잘 맞는다.

<br>

> ## **27-2) Bridge Table**

Many-to-Many 관계를 명확하게 표현하기 위해 중간 Bridge Table을 사용할 수 있다.

```text
Player
   ↓
Player-Team Membership
   ↓
Team
```

처럼 선수의 소속 이력이 시간에 따라 여러 팀과 연결되는 구조를 관리할 수 있다.

<br>

> ## **27-3) Pre-Aggregation**

1Hz GPS Sensor처럼 데이터가 지나치게 세밀하면

```text
수백만 Row Raw Sensor
```

를 그대로 Dashboard에 사용하는 대신

```text
Raw 1Hz GPS
    ↓
Aggregate

Player × Match
    ↓
Tableau
```

처럼 분석 목적에 적합한 Grain으로 미리 집계할 수 있다.


<br><br>

---

# **28. 핵심정리**

1. Tableau에서 데이터 결합을 결정할 때 가장 먼저 확인해야 하는 것은 **Granularity**, 즉 각 Table의 한 행이 무엇을 의미하는가이다.

2. 데이터 결합 전에는 다음 질문에 답할 수 있어야 한다.

```text
한 행은 무엇인가?

Join / Relationship Field는 무엇인가?

Field는 Unique한가?

결합 후 예상 Grain은 무엇인가?

Measure가 반복될 가능성은 없는가?
```

3. Join은 Tableau Data Model의 **Physical Layer**에서 사용하며 Join Clause와 Join Type을 이용해 Physical Table을 하나의 Logical Table로 병합한다.

4. Join된 Table은 하나의 Row-level 구조가 되므로 서로 다른 Grain의 Table을 Join하면 데이터 중복이나 손실이 발생할 수 있다.

5. Join Type은 다음처럼 구분한다.

```text
Inner
→ 양쪽 모두 Match

Left
→ Left 모두 유지

Right
→ Right 모두 유지

Full Outer
→ 양쪽 모두 유지
```

6. Join에서는 하나의 Field뿐 아니라

```text
player_id
+
date
```

같은 여러 Field를 Join 조건으로 사용할 수도 있다.

7. Many-to-Many Join에서는 Row Multiplication이 발생할 수 있으므로 Join 전후 Row Count와 Distinct Grain Key를 반드시 확인해야 한다.

8. Union은 같은 또는 호환되는 구조의 Table에서 **Row를 추가하는 방법**이며 Physical Layer에서 사용한다.

9. Tableau Union을 SQL `UNION`의 중복 제거와 동일하게 이해하면 안 된다. Tableau에서는 주로 여러 Table의 Row를 쌓는 데이터 결합 기능으로 이해한다.

10. Wildcard Union은 일정한 이름 규칙을 가진 여러 Table이나 File을 자동으로 Union 대상으로 검색할 수 있다.

11. Tableau Data Model은 크게 다음 두 Layer로 구성된다.

```text
Logical Layer
→ Relationship

Physical Layer
→ Join / Union
```

12. Relationship은 Logical Table을 실제로 하나의 Flat Table로 미리 합치지 않고 **Table이 어떻게 관련되어 있는지를 정의한다.**

13. Relationship으로 연결된 Logical Table은 각자의 기본 Level of Detail과 Domain을 유지한다.

14. Relationship에서는 사용자가 Join Type을 미리 지정하지 않는다. Tableau가 현재 Viz에서 사용된 Dimension과 Measure의 Context를 이용하여 필요한 Query와 Join 형태를 구성한다.

15. 따라서

```text
Relationship = 자동 Left Join
```

으로 이해하면 안 된다.

16. 일반적인 Multi-table 분석에서는 서로 다른 Grain을 보다 자연스럽게 다룰 수 있기 때문에 Tableau는 Relationship을 우선적인 접근으로 권장한다.

17. Join은 특정 Join Type을 반드시 사용하거나 명시적인 Flat Row Structure가 필요한 경우 여전히 중요하다.

18. Relationship의 Cardinality는 Relationship Field가 각 Table에서 `One`인지 `Many`인지에 대한 정보이며 Tableau가 Join 전후 Aggregation을 최적화하는 데 사용한다.

19. 실제로 Unique하지 않은 Field를 `One`이라고 잘못 지정하면 Aggregate가 중복되어 잘못된 결과가 나타날 수 있다.

20. Referential Integrity는 한 Table의 Record가 다른 Table에서 반드시 Match하는지에 대한 정보이다.

21. `All Records Match`를 잘못 지정하면 실제 unmatched Record가 View에서 제거될 수 있으므로 확신이 없으면 기본값을 유지한다.

22. 데이터 구조를 잘 모르는 경우 안전한 기본 방향은 다음과 같다.

```text
Cardinality
→ Many-to-Many

Referential Integrity
→ Some Records Match
```

23. Blending은 서로 다른 Tableau Data Source를 각각 Query한 뒤 **Aggregate된 결과를 Worksheet에서 Linking Field로 연결하는 방식**이다.

24. Blending은 Sheet 단위로 동작하며 먼저 사용된 Data Source가 Primary, 추가된 Source가 Secondary가 된다.

25. Blending 결과는 Primary Source를 기준으로 하는 Left Join과 유사하게 동작하므로 Secondary에만 존재하는 값은 표시되지 않을 수 있다.

26. Blending은 Row-level Join이 아니며 Secondary Data Source의 값은 Linking Field 수준에 맞게 Aggregate되어 사용된다.

27. 따라서 다음처럼 외우면 부정확하다.

```text
다른 파일
→ 무조건 Blending
```

더 정확한 판단은 다음과 같다.

```text
하나의 Tableau Data Model로
Relationship을 구성할 수 있는가?

Cross-database Join을 사용할 수 있는가?

Data Source를 독립적으로 유지해야 하는가?

Sheet별 Linking이 필요한가?
```

28. 지원되는 환경에서는 서로 다른 Database Connection도 Cross-database Join으로 하나의 Tableau Data Source 안에서 결합할 수 있다.

29. Tableau Prep Builder는 복잡한 데이터 정제와 결합을

```text
Input
→ Clean
→ Join / Union
→ Aggregate
→ Pivot
→ Output
```

형태의 Flow로 관리하는 데이터 준비 도구이다.

30. Grain이 서로 다른 데이터를 결합하기 전에 Tableau Prep의 Aggregate Step 등을 이용하여 Level of Detail을 맞추는 방법도 중요하다.

31. Tableau의 데이터 결합 방법은 다음처럼 기억하면 가장 정확하다.

```text
같은 구조의 Row를 추가
→ Union

명시적인 Row-level Column 병합
→ Join

하나의 Tableau Data Model 안에서
여러 Grain의 Logical Table 연결
→ Relationship

독립 Data Source의 Aggregate 결과를
Worksheet에서 연결
→ Blending
```

32. 실제 데이터 결합 후에는 반드시 다음을 검증한다.

```text
Row Count
Distinct Key
NULL
Duplicate
Measure Total
Data Type
Unit
Date Grain
```

33. 데이터 결합의 최종 흐름은 다음과 같다.

```text
Analysis Question
        ↓
필요 Data 확인
        ↓
Granularity 정의
        ↓
Relationship Key / Join Key 확인
        ↓
Join / Union /
Relationship / Blending 선택
        ↓
결합
        ↓
Row Count 검증
        ↓
Distinct Grain 검증
        ↓
Measure 중복 검증
        ↓
Visualization
        ↓
해석
```

34. 결국 Tableau 데이터 결합에서 가장 중요한 질문은

```text
"어떤 버튼을 눌러 데이터를 합칠까?"
```

가 아니라

```text
"각 Table의 한 행은 무엇을 의미하고,

분석 시 이 Table들의 관계를
어떤 Grain에서 유지해야 하는가?"
```

이다.
