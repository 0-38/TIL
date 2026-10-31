# **[05. Likelihood와 Maximum Likelihood]**

<br>

# **1. 확률 모델과 머신러닝 학습의 전체 흐름**

앞선 강의에서는 분류 모델이 다음과 같은 과정을 통해 Prediction을 만드는 것을 살펴보았다.

```text
Input Feature

        ↓

Model

        ↓

Logit(관측값)

        ↓

Sigmoid / Softmax(확률변환 - 이진분류, 다중분류)

        ↓

Predicted Probability

        ↓

Threshold / Argmax

        ↓

Predicted Class
```

이번에는 한 단계 더 나아가 다음 질문을 다룬다.

```text
모델이 계산한 Probability가
실제 관측 데이터와 얼마나 잘 맞는가?

        ↓

어떤 Model Parameter가
관측 데이터를 가장 잘 설명하는가?

        ↓

Likelihood

        ↓

Maximum Likelihood Estimation

        ↓

Log-Likelihood

        ↓

Negative Log-Likelihood

        ↓

Loss Function
```

> 용어 정리

- **확률(Probability)** : 특정 사건이 발생할 가능성을 0과 1사이의 값으로 나타낸 것

- **확률변수(Random Variable)** : 확률적인 결과를 숫자로 표현하는 변수
- **확률분포(Probability Distribution)** : 확률변수가 가질 수 있는 값과 각 값에 부여되는 확률
- **베르누이 분포(Bernoilli Distribution)** : 결과가 0 또는 1인 확률변수를 표현하는 분포
- **범주분포(Categorical Distribution)** : 여러 개의 상호배타적인 클래스 중 하나가 선택되는 확률
- **조건부 확률(Conditional Probability)** : 특정 조건이 주어졌을 때 다른 사건이 발생할 확률
- **머신러닝 활용 방법** : 분류 모델은 입력 데이터가 주어졌을 때 각 클래스가 발생할 조건부 확률을 모델링 할 수 있다.

즉 이번 강의의 핵심은 **Prediction을 만드는 방법이 아니라, 어떤 Parameter가 좋은지를 확률적으로 평가하는 방법**이다.

예를 들어 축구 선수의 슈팅 결과가 다음과 같다고 하자.

```text
Shot 1 → Goal
Shot 2 → Miss
Shot 3 → Goal
Shot 4 → Goal
Shot 5 → Miss
```

이를 `1=Goal`, `0=Miss`로 표현하면

```text
y = [1, 0, 1, 1, 0]
```

이다.

5번의 슈팅 중 3번 성공했으므로 관측 성공 비율은

```math
\frac{3}{5}=0.6
```

이다.

이제 다음 두 Model을 비교할 수 있다.

```text
Model A
Goal Probability = 0.6

Model B
Goal Probability = 0.8
```

둘 중 실제 관측 결과를 더 잘 설명하는 Parameter를 찾는 개념이 **Likelihood와 MLE**이다.

<br><br>

---

# **2. 확률(Probability)의 의미**

Probability는 **Model 또는 조건이 주어졌을 때 특정 결과가 발생할 가능성**을 나타낸다.

사건 \(A\)의 **Probability**는

```math
P(A)
```

로 표현한다.

확률은 항상

```math
0\le P(A)\le1
```

범위를 가진다.

| Probability | 의미 |
| ---: | --- |
| 0 | 발생할 수 없음 |
| 0.2 | 20% 가능성 |
| 0.5 | 50% 가능성 |
| 0.8 | 80% 가능성 |
| 1 | 반드시 발생 |

예를 들어 페널티킥 성공 확률을 0.7이라고 가정하면

```math
P(Y=1)=0.7
```

이고 실패 확률은

```math
P(Y=0)=0.3
```

이다.

두 결과가 전체 가능한 결과를 구성한다면

```math
P(Y=1)+P(Y=0)=1
```

이 성립한다.

<br><br>

---

# **3. 관측 비율과 실제 확률은 구분해야 한다**

실제 Dataset에서 계산한 비율은 Probability를 추정하는 데 활용할 수 있지만 **실제 모집단의 Probability와 반드시 동일하지는 않다.**

예를 들어 양궁 선수의 최근 10발 결과가

```text
10점 성공
→ 7회

10점 미성공
→ 3회
```

라면 관측 성공 비율은

```math
\frac{7}{10}=0.7
```

이다.

그러나 이것이 선수의 진짜 성공 확률이 정확히 0.7이라는 것을 의미하지는 않는다.

표본이 달라지면

```text
최근 10발
→ 0.70

최근 50발
→ 0.64

최근 100발
→ 0.67
```

처럼 달라질 수 있기 때문이다.

따라서 다음을 구분해야 한다.

| 개념 | 의미 |
| --- | --- |
| 실제 Probability | 확률모형의 알려지지 않은 특성 |
| Observed Rate | Sample에서 관측된 비율 |
| Estimate | Sample을 이용해 Probability를 추정한 값 |

[Python Code]"양궁선수의 10점 평균 성공 확률"

```python
import numpy as np

# 1: 성공, 0: 미성공
y = np.array([1, 0, 1, 1, 0])

success_rate = np.mean(y)

print(success_rate)
```

[Result]
```text
0.6
```
이진 데이터에서는 평균을 이용하여 양성 클래스의 관측 비율 계산 가능하다.

<br><br>

---

# **4. 확률변수(Random Variable)**

Random Variable은 확률적으로 발생하는 결과를 숫자로 표현한 변수이다.

예를 들어 야구 타석 결과를 간단히

```text
Y = 1
→ 출루

Y = 0
→ 아웃
```

으로 정의할 수 있다.

이 경우 `Y`는 0 또는 1을 가질 수 있는 Random Variable이다.

출루 Probability가 `p`라면

```math
P(Y=1)=p
```

```math
P(Y=0)=1-p
```

이다.

Random Variable은

```text
어떤 값을 가질 수 있는가?
```

를 표현하고,

Probability Distribution은

```text
각 값에 얼마의 Probability가 배정되는가?
```

를 표현한다.

<br><br>

---

# **5. 확률분포(Probability Distribution)**

확률분포는 확률변수가 가질 수 있는 값과 각 값에 부여되는 Probability의 구조를 나타낸다.

예를 들어 경기 중 Sprint 발생 여부가

```text
Y = 1
→ Sprint 발생

Y = 0
→ Sprint 미발생
```

이고 Sprint Probability가 0.4라면

| $Y$ | Probability |
| ---: | ---: |
| 0 | 0.6 |
| 1 | 0.4 |

이다.

확률분포는 크게 이산형과 연속형으로 구분할 수 있다.

| 구분 |확률변수| 특징 | 스포츠 예시 |
| --- | --- | --- | --- |
| Discrete(이산형) |이산확률변수 |셀 수 있는 개별 값 | 득점 여부, Stroke Class |
| Continuous(연속형) |연속확률변수| 연속적인 값 | 속도, 거리, 점프 높이 |

연속형 Random Variable에서는 특정 한 점의 확률이 아니라 **구간에 대한 Probability**를 계산하며 Probability Density Function을 사용한다.

연속확률변수는 확률밀도함수(Probability Density Function)를 이용하여 분포를 표현하며, 대표적인 예시로 정규분포(Normal Distribution)이 있다.

<br><br>

---

# **6. Bernoulli Distribution(베르누이 분포)**

Bernoulli Distribution은 결과가 두 개인 상황(이진 분류)을 표현한다.

```text
1
→ Success

0
→ Failure
```

스포츠 데이터 예시는 다음과 같다.

| 문제 | 1 | 0 |
| --- | --- | --- |
| 페널티킥 | 성공 | 실패 |
| 서브 | In | Fault |
| 슈팅 | On Target | Off Target |
| 센서 Event | 발생 | 미발생 |

Bernoulli Distribution의 Parameter는

```math
p=P(Y=1)
```

이다.

따라서

```math
P(Y=0)=1-p
```

이다.

<br>

> ## **6-1) Bernoulli Probability Mass Function**

Bernoulli Distribution은 다음 하나의 식으로 0과 1의 Probability를 표현할 수 있다.

```math
P(Y=y)
=
p^y(1-p)^{1-y}
```

`y=1`이면

```math
P(Y=1)
=
p^1(1-p)^0
=
p
```

이다.

`y=0`이면

```math
P(Y=0)
=
p^0(1-p)^1
=
1-p
```

가 된다.

<br><br>

---

# **7. Bernoulli Distribution Python 예제**

슈팅 성공 Probability를

```text
p = 0.7
```

이라고 가정하자.

[ Python : code ] "Bernoulli Distribution에서 성공·실패 Probability 계산"

```python
import numpy as np

p = 0.7

y = np.array([0, 1])

probabilities = (p ** y) * ((1 - p) ** (1 - y))

print(probabilities)
print(probabilities.sum())
```

[ Result ]

```text
[0.3 0.7]
1.0
```

결과는

```text
P(Y=0)
→ 0.3

P(Y=1)
→ 0.7
```

이다.

<br><br>

---

# **8. Categorical Distribution(범주 분포)**

Categorical Distribution은 **여러 개의 상호 배타적인 Class 중 하나가 발생하는 경우(다중 분류)** 를 표현한다.

예를 들어 테니스 Stroke를

```text
Class 0 → Forehand
Class 1 → Backhand
Class 2 → Serve
```

로 분류한다고 하자.

Model Probability가

```text
[0.50, 0.30, 0.20]
```

이라면

```math
P(Y=0)=0.50
```

```math
P(Y=1)=0.30
```

```math
P(Y=2)=0.20
```

이다.

전체 Class Probability는

```math
0.50+0.30+0.20=1
```

이 된다.

따라서 클래스가 K개인 범주분포의 확률은 다음 조건을 만족한다.

`p₁ + p₂ + ... + pₖ = 1`

[Python Code] "테니스 Stroke 범주분포의 확률"
```python
import numpy as np

classes = ["Forhead", "Backhand", "Serve"]

probabilities = np.array([0.5, 0.3, 0.2])

for classes_name, p in zip(classes, probabilities):
        print(classes_name, p)

print("확률 합 :", round(np.sum(probabilities), 6))
```

[Result]
```text
Forhead 0.5
Backhand 0.3
Serve 0.2
확률 합 : 1.0
```

<br>

> ## **8-1) Bernoulli와 Categorical 비교**

| 구분 | Bernoulli | Categorical |
| --- | --- | --- |
| 결과 개수 | 2 | K개 |
| 예시 | 성공 / 실패 | Smash / Drop / Clear |
| Parameter | $p$ | $p_1,\dots,p_K$ |
| 확률 합 | $p+(1-p)=1$ | $\sum_{k=1}^{K} p_k = 1$ |
| 대표 Model Output | **Sigmoid** | **Softmax** |

<br><br>

---

# **9. 조건부 확률(Conditional Probability)**

조건부 확률은 **어떤 조건이 주어졌을 때 특정 사건이 발생할 Probability**이다.

예를 들어 축구 경기에서 다음 두 사건을 정의하자.

```text
A
→ Penalty Area 안에서 Shot

B
→ Shot On Target
```

그러면

```math
P(B\mid A)
```

는

> Penalty Area 안에서 Shot을 했다는 조건에서 Shot이 On Target일 Probability

를 의미한다.

공식은

```math
P(B\mid A)
=
\frac{P(A\cap B)}
{P(A)}
```

이다.

단,

```math
P(A)>0
```

이어야 한다.

<br><br>

---

# **10. 조건부 확률은 방향이 중요하다**

다음 두 Probability는 일반적으로 서로 다르다.

```math
P(B\mid A)
```

```math
P(A\mid B)
```

예를 들어 다음 경기 데이터가 있다고 하자.

| Shot 위치 | On Target | Off Target | 합계 |
| --- | ---: | ---: | ---: |
| Penalty Area 내부 | 12 | 8 | 20 |
| 외부 | 3 | 7 | 10 |
| 합계 | 15 | 15 | 30 |

Penalty Area 내부 Shot이 On Target일 관측 비율은

```math
P(OnTarget\mid Inside)
=
\frac{12}{20}
=
0.60
```

이다.

반대로 On Target Shot 중 Penalty Area 내부였던 비율은

```math
P(Inside\mid OnTarget)
=
\frac{12}{15}
=
0.80
```

이다.

| Conditional Probability | 결과 |
| --- | ---: |
| $P(OnTarget\mid Inside)$ | 0.60 |
| $P(Inside\mid OnTarget)$ | 0.80 |

따라서 조건을 뒤집으면 의미와 분모가 달라진다.

[Python Code] "유효슈팅 성공여부와 팀 승리 여부 간에 조건부확률"

```python
import numpy as np

# X: 유효슈팅 성공 여부 (1: 성공, 0: 실패)
shooting = np.array([
    1, 1, 1, 1, 1, 1, 0, 0, 0, 0
])

# Y: 팀 승리 여부 (1: 승리, 0: 패배)
win = np.array([
    1, 1, 1, 1, 0, 0, 1, 0, 0, 0
])

# 전체 승리 비율
p_win = np.mean(win == 1)

# 유효슈팅 성공 조건에서 승리 비율
p_win_given_shooting = np.mean(
    win[shooting == 1] == 1
)

# 유효슈팅 실패 조건에서 승리 비율
p_win_given_no_shooting = np.mean(
    win[shooting == 0] == 1
)

print("전체 승리 비율:", round(p_win, 3))
print("유효슈팅 성공 시 승리 비율:", round(p_win_given_shooting, 3))
print("유효슈팅 실패 시 승리 비율:", round(p_win_given_no_shooting, 3))
```

[Result]
```
전체 승리 비율: 0.5
유효슈팅 성공 시 승리 비율: 0.667
유효슈팅 실패 시 승리 비율: 0.25
```

<br><br>

---

# **11. 머신러닝에서 조건부 확률**

Classification Model은 일반적으로 다음과 같은 Probability를 학습한다고 볼 수 있다.

```math
P(Y\mid X)
```

즉

> Input Feature $X$가 주어졌을 때 Target $Y$가 특정 Class일 Probability

이다.

예를 들어 선수의 GPS Feature가

```text
X = [
    Sprint Count,
    HSR Distance,
    Max Speed
]
```

라고 하자.

모델이

```math
P(Y=1\mid X=x)=0.82
```

를 출력했다면 입력값 `x`를 가진 Sample이 Class 1일 확률형 출력을 `0.82`로 계산했다는 의미이다.

Binary Classification에서는

$$
P(Y=1\mid X=x;\theta)
=
\sigma(xw+b)
$$

와 같이 표현할 수 있다.

여기서

$$
\theta=(w,b)
$$

는 학습해야 하는 Model Parameter이다.

로지스틱 회귀에서 선형회귀식인 `x · w + b`에 시그모이드를 활용하면 종속변수가 1인 확률을 계산 가능하다.

그에 따른 식은 `P(Y=1 | X=x) = Sigmoid(x · w + b)` 이다. 


<br><br>

---

# **12. Probability와 Likelihood의 차이**

확률과 우도는 같은 확률 모델을 사용하지만 **무엇을 고정하고 무엇을 바꾸는가**가 다르다.

Probability는

$$
P(D\mid\theta)
$$

에서 Parameter θ를 고정하고 데이터가 어떻게 나올지를 생각한다.

Likelihood(우도)는 같은 식을

$$
L(\theta\mid D)
$$

로 보고 **실제 관측 데이터 $D$를 고정한 채 Parameter를 비교**한다.

- **확률** : 모델의 파라미터가 고정되어있을 때, 데이터가 발생할 가능성

- **우도** : 관측 데이터가 고정되어있을 때, 모델 파라미터가 해당 데이터를 얼마나 더 잘 설명하는지

| 구분 | Probability | Likelihood(우도) |
| --- | --- | --- |
| 고정 | Parameter | Observed Data |
| 변화 | 가능한 Data | Parameter(가중치, 편향) |
| 질문 | 이 Model에서 어떤 결과가 가능한가? | 어떤 Parameter가 Data를 잘 설명하는가? |
| 활용 | Probability 계산 / <br> 결과의 발생가능성 계산 | Parameter Estimation / <br> 우도를 최대화하도록 학습 알고리즘이 가중치와 편향을 업데이트 |


```text
학습 데이터 고정
(X ; GPS Feature, y ; 정답)
        ↓
가중치 W, 편향 b로 예측
(예측 확률 계산)
        ↓
우도 계산
(현재 파라미터가 정답 데이터를 얼마나 잘 설명하는가?)
        ↓
W, b 업데이트
(우도가 증가하는 방향으로 조정)
        ↓
⟲ 반복 학습
(가중치 업데이트)
```

중요한 것은

```text
Likelihood = 0.4
```

라고 해서

```text
Parameter가 참일 Probability = 40%
```

라는 의미가 아니라는 점이다.

<br><br>

---

# **13. 하나의 Bernoulli 관측값에 대한 Likelihood**

Bernoulli Probability Mass Function(베르누이 확률질량 함수)은

$$
P(Y=y\mid p)
=
p^y(1-p)^{1-y}
$$

이다.

Observed Data `y`를 고정하고 `p`의 함수로 보면 Likelihood가 된다.

$$
L(p\mid y)
=
p^y(1-p)^{1-y}
$$

실제 결과가 `1`이면

```math
L(p\mid y=1)=p
```

이다.

실제 결과가 `0`이면

```math
L(p\mid y=0)=1-p
```

이다.

즉 Model이 **실제로 관측된 결과에 얼마나 높은 Probability를 부여했는가**가 중요하다.

<br><br>

---

# **14. 여러 관측값의 Likelihood**

> ## **14-1) 여러 관측값의 Likelihood란?**

실제로 머신러닝에서는 하나의 데이터가 아니라 여러 개의 관측 데이터(Sample)를 이용하여 모델을 학습한다.

**Likelihood(우도)** 는 관측된 데이터를 고정한 상태에서 Model Parameter를 변경하면서, 어떤 파라미터가 관측 데이터를 가장 잘 설명하는지 평가하는 것이다.

여러 관측값이 Model Parameter가 주어졌을 때 서로 독립이라고 가정하면, 전체 데이터의 Likelihood(Joint Likelihood)는 **각 관측 결과의 Probability를 모두 곱하여 계산할 수 있다.**

$$
L(\theta \mid D)=\prod_{i=1}^{n}P(y_i\mid\theta)
$$

- $L(\theta\mid D)$ : 전체 관측 데이터의 Likelihood
- $\theta$ : Model Parameter
- $D$ : 관측 데이터
- $n$ : 전체 관측 데이터의 개수
- $y_i$ : i번째 Sample의 실제 관측 결과
- $\prod$ : 각 관측값의 확률을 모두 곱한다는 의미

즉, **각 Sample에서 실제로 관측된 결과가 발생할 확률을 모두 곱한 값이 전체 Likelihood이다.**

<br>

> ## **14-2) Bernoulli Dataset에서의 Likelihood**

Bernoulli Distribution(베르누이 분포)은 결과가 두 가지로 나뉘는 경우에 사용한다.

예를 들어 축구 경기의 승리 여부를 다음과 같이 정의할 수 있다.

```text
Y = 1 → 승리 (Success)
Y = 0 → 패배 (Failure)
```

이때 승리할 확률을 $p$라고 하면 다음과 같다.

$$
P(Y=1)=p
$$

$$
P(Y=0)=1-p
$$

즉,

- 승리할 확률 : $p$
- 패배할 확률 : $1-p$

이다.

베르누이 분포에서는 승리와 패배의 확률을 하나의 수식으로 표현할 수 있다.

$$
P(y_i\mid p)=p^{y_i}(1-p)^{1-y_i}
$$

> **① 승리한 경우 ($y_i=1$)**

$$
p^1(1-p)^0=p
$$

따라서 승리한 Sample의 Probability는 $p$이다.

> **② 패배한 경우 ($y_i=0$)**

$$
p^0(1-p)^1=1-p
$$

따라서 패배한 Sample의 Probability는 $1-p$이다.

이 수식을 전체 관측 데이터에 적용하면 다음과 같다.

$$
L(p\mid D)=\prod_{i=1}^{n}p^{y_i}(1-p)^{1-y_i}
$$

즉, **승리한 Sample에는 $p$를 적용하고 패배한 Sample에는 $1-p$를 적용한 뒤 모든 확률을 곱하는 것이다.**

<br>

> ## 14-3) 실제 관측 데이터를 이용한 Likelihood 계산

예를 들어 축구팀의 5경기 승리 여부를 관측했다고 가정하자.

```python
y = [1, 0, 1, 1, 0]
```

각 데이터는 다음을 의미한다.

| 경기 | 실제 결과 | Probability |
|---|---|---|
| 1경기 | 1 (승리) | $p$ |
| 2경기 | 0 (패배) | $1-p$ |
| 3경기 | 1 (승리) | $p$ |
| 4경기 | 1 (승리) | $p$ |
| 5경기 | 0 (패배) | $1-p$ |

따라서 전체 관측 결과는 다음과 같다.

```text
Success (승리)
→ 3회

Failure (패배)
→ 2회

전체 Sample
→ 5개
```

모든 경기의 승리 확률이 동일하고, 

각 경기의 결과가 서로 독립이라고 가정하면 전체 Likelihood는 **각 경기에서 관측된 결과의 확률을 곱하여 계산한다.**

$$
L(p\mid D)=p\times(1-p)\times p\times p\times(1-p)
$$

승리한 경기는 3개이므로 $p$가 3번 곱해지고, 패배한 경기는 2개이므로 $(1-p)$가 2번 곱해진다.

따라서 다음과 같이 정리할 수 있다.

$$
\boxed{L(p\mid D)=p^3(1-p)^2}
$$


- $p^3$ : 승리한 3개 Sample의 확률을 곱한 값
- $(1-p)^2$ : 패배한 2개 Sample의 확률을 곱한 값

<br>

> ## 14-4) Model Parameter를 변경하면서 Likelihood 계산

Likelihood는 **관측 데이터를 고정하고 Model Parameter를 변경하면서 계산한다.**

현재 관측 데이터는 다음과 같다.

```python
y = [1, 0, 1, 1, 0]
```

이 데이터는 변경하지 않고 승리 확률 $p$를 변경해 보자.

> **① 승리 확률이 $p=0.2$인 경우**

$$
L(0.2\mid D)=0.2^3(1-0.2)^2
$$

$$
=0.2^3\times0.8^2
$$

$$
=0.00512
$$

> **② 승리 확률이 $p=0.5$인 경우**

$$
L(0.5\mid D)=0.5^3(1-0.5)^2
$$

$$
=0.5^3\times0.5^2
$$

$$
=0.03125
$$

> **③ 승리 확률이 $p=0.6$인 경우**

$$
L(0.6\mid D)=0.6^3(1-0.6)^2
$$

$$
=0.6^3\times0.4^2
$$

$$
=0.03456
$$

> **④ 승리 확률이 $p=0.9$인 경우**

$$
L(0.9\mid D)=0.9^3(1-0.9)^2
$$

$$
=0.9^3\times0.1^2
$$

$$
=0.00729
$$

계산 결과를 정리하면 다음과 같다.

| Model Parameter ($p$) | Likelihood |
|---|---|
| 0.2 | 0.00512 |
| 0.5 | 0.03125 |
| **0.6** | **0.03456** |
| 0.9 | 0.00729 |

관측 데이터는 동일하지만 $p$의 값에 따라 Likelihood가 달라지는 것을 확인할 수 있다.

이 데이터에서는 $p=0.6$일 때 Likelihood가 가장 높다.

따라서 최대우도추정(MLE)을 통해 구한 파라미터는 다음과 같다.

$$
\hat p=\frac{\text{Success 횟수}}{\text{전체 Sample 수}}
$$

$$
\hat p=\frac35=0.6
$$

즉, **5경기 중 3경기를 승리한 관측 데이터에서는 승리 확률을 0.6으로 설정했을 때 전체 데이터의 Likelihood가 최대가 된다.**

<br>

> ## 14-5) 모델의 가중치와 편향으로 연결

앞선 예제에서는 모든 경기의 승리 확률이 동일한 $p$라고 가정하였다.

하지만 실제 머신러닝 모델에서는 각 Sample의 입력 Feature가 다르기 때문에 예측 확률도 달라질 수 있다.

예를 들어 선수의 GPS Feature가 다음과 같다고 가정하자.

```python
X = [
    Sprint_Count,
    HSR_Distance,
    Max_Speed
]
```

로지스틱 회귀 모델은 각 Feature에 학습된 가중치를 적용하여 다음과 같이 계산한다 (선형모델).

$$
z=w_1x_1+w_2x_2+w_3x_3+b
$$

이후 Sigmoid 함수를 이용하여 승리 확률을 계산한다.

$$
p_i=P(Y_i=1\mid X_i=x_i)=\sigma(w^Tx_i+b)
$$

- $x_i$ : i번째 Sample의 Feature
- $w$ : 모델의 가중치(Weight)
- $b$ : 모델의 편향(Bias)
- $p_i$ : i번째 Sample에 대해 모델이 예측한 승리 확률


각 Sample의 승리 확률이 $p_i$로 달라지므로 전체 Likelihood는 다음과 같다.

$$
L(w,b\mid D)=\prod_{i=1}^{n}p_i^{y_i}(1-p_i)^{1-y_i}
$$

이때 모델은 단순히 `p` 하나를 변경하는 것이 아니라, 

**가중치 `w`와 편향 `b`를 조정하여 전체 데이터의 Likelihood를 최대화하는 파라미터를 찾는다.**

이 과정을 Maximum Likelihood Estimation(MLE)이라고 한다.

<br>

> ### 14-6) 정리

**여러 관측값의 Likelihood란,** 

**각 Sample에서 실제로 관측된 결과가 발생할 확률을 모두 곱하여 계산한 값이다.**

```text
관측 데이터 준비
        ↓
Model Parameter 설정
        ↓
각 Sample의 Probability 계산
        ↓
전체 Probability 곱하기
        ↓
Joint Likelihood 계산
        ↓
Likelihood가 최대인지 평가
        ↓
Model Parameter 조정
        ↓
       ⟲ 반복
```

- **Probability** : 파라미터가 주어졌을 때 데이터가 발생할 확률
- **Likelihood** : 관측 데이터를 고정했을 때 파라미터에 따른 데이터의 가능도
- **Joint Likelihood** : 여러 관측 데이터의 확률을 곱한 값(조건부 독립 가정)
- **MLE** : Likelihood를 최대화하는 파라미터를 찾는 방법
- **Weight / Bias** : 머신러닝 모델에서 학습을 통해 조정하는 주요 파라미터

결국 머신러닝에서는 **관측 데이터를 가장 잘 설명할 수 있도록 가중치와 편향을 조정하는 과정**을 통해 모델을 학습할 수 있다.

<br><br>

---

# **15. Bernoulli Likelihood 직접 계산**

축구 선수의 유효 슈팅 결과가 다음과 같다고 하자.

```text
y = [1, 0, 1, 1, 0]
```

Model A:

```math
p=0.6
```

Model B:

```math
p=0.8
```

<br>

> ## **15-1) Model A**

```math
L(0.6)
=
0.6^3(1-0.6)^2
```

```math
=
0.6^3\times0.4^2
```

```math
=
0.216\times0.16
```

```math
=
0.03456
```

<br>

> ## **15-2) Model B**

```math
L(0.8)
=
0.8^3\times0.2^2
```

```math
=
0.512\times0.04
```

```math
=
0.02048
```

따라서

```math
0.03456>0.02048
```

이므로 이 Dataset에서는 Model A가 더 높은 Likelihood를 가진다.

| Model | \(p\) | Likelihood |
| --- | ---: | ---: |
| A | 0.6 | 0.03456 |
| B | 0.8 | 0.02048 |

Model B는 Success에 높은 Probability를 부여하지만 Failure에 불과 `0.2`의 Probability를 부여하기 때문에 전체 Likelihood가 감소한다.

<br><br>

---

# **16. Likelihood 계산 Python 예제**

[ Python : code ] "Bernoulli Dataset에 대한 여러 Parameter의 Likelihood 비교"

```python
import numpy as np

y = np.array([1, 0, 1, 1, 0])

def likelihood(p, y):
    observed_prob = (p ** y) * ((1 - p) ** (1 - y)) # 베르누이 분포
    return np.prod(observed_prob) # np.prod -> 배열의 모든 원소 곱함 -> observed_prob에서 구한 확률의 곱 : 우도 추정 

for p in [0.4, 0.6, 0.8]:
    print(
        f"p={p:.1f}, "
        f"L={likelihood(p, y):.5f}"
    )
```

[ Result ]

```text
p=0.4, L=0.02304
p=0.6, L=0.03456
p=0.8, L=0.02048
```

비교한 후보 중에서는 `p=0.6`이 가장 크다.

<br><br>

---

# **17. Likelihood에서 독립 가정이 중요한 이유**

여러 Probability를 단순히 곱할 수 있는 것은 다음과 같은 가정을 사용했기 때문이다.

```text
각 관측값이
Parameter가 주어졌을 때
서로 독립
```

즉

```math
P(y_1,\dots,y_n\mid\theta)
=
\prod_iP(y_i\mid\theta)
```

라고 가정한 것이다.

하지만 스포츠 데이터에서는 다음처럼 독립성이 약할 수 있다.

```text
같은 경기의 연속 Rally

같은 선수의 반복 Sprint

같은 Pitcher의 연속 투구

동일 세션의 반복 Jump

시계열 Sensor Sample
```
 
이런 데이터는 시간·선수·세션 구조에 따라 서로 의존할 수 있다.

따라서 실제 Modeling에서는

```text
Time Series Model
Mixed Model
Hierarchical Model
Sequence Model
```

등 다른 확률 구조가 필요할 수도 있다.

Likelihood 자체의 개념보다 **어떤 Probability Model과 Independence Assumption을 사용했는가**가 중요하다.

<br><br>

---

# **18. 개별 Sample마다 Probability가 다른 경우**

실제 Machine Learning에서는 모든 Sample에 동일한 `p`를 사용하지 않는다.

각 Sample마다

```math
p_i = P(Y_i=1\mid X_i;\theta)
```

가 다를 수 있다.

Binary Classification(이진분류)에서는

```math
p_i = \sigma(X_iw+b)
```

로 계산할 수 있다.

Likelihood는

```math
L(\theta\mid D)=\prod_i p_i^{y_i}(1-p_i)^{1-y_i}
```

가 된다.

즉

```text
실제 정답 = 1
→ pᵢ 사용

실제 정답 = 0
→ 1-pᵢ 사용
```

한다.

<br><br>

---

# **19. 경기분석 Model Likelihood 예제**

세 번의 Shot에 대해 실제 결과가

```text
y = [1, 0, 1]
```

이라고 하자.

두 Model이 계산한 Goal Probability는 다음과 같다.

| Shot | Actual | Model A | Model B |
| --- | ---: | ---: | ---: |
| 1 | 1 | 0.80 | 0.60 |
| 2 | 0 | 0.30 | 0.50 |
| 3 | 1 | 0.70 | 0.40 |

Model A는

$$
\begin{aligned}
&0.8 \times (1-0.3) \times 0.7 \\
&= 0.8 \times 0.7 \times 0.7 \\
&= 0.392
\end{aligned}
$$




Model B는

$$
\begin{aligned}
&0.6 \times (1-0.5) \times 0.4 \\
&= 0.6 \times 0.5 \times 0.4 \\
&= 0.12
\end{aligned}
$$


따라서

```text
Model A Likelihood
→ 0.392

Model B Likelihood
→ 0.120
```

즉, Model A가 실제 관측 결과에 더 높은 Probability를 부여했다.

<br><br>

---

# **20. Maximum Likelihood Estimation(MLE)**

MLE는 **Observed Data의 Likelihood를 최대화하는 Parameter를 찾는 방법**이다.

수식으로

```math
\hat{\theta}_{MLE}
=
\arg\max_{\theta}
L(\theta\mid D)
```

라고 표현한다.

여기서 `argmax`는 가장 큰 함수값 자체가 아니라

> 함수값을 가장 크게 만드는 Parameter

를 의미한다.

```text
Parameter 후보

        ↓

각 후보의 Likelihood 계산

        ↓

가장 큰 Likelihood 선택

        ↓

MLE Parameter
```

즉

```text
Good Parameter
=
Observed Data에
높은 Probability를 부여하는 Parameter
```

라는 관점이다.

<br><br>

---

# **21. Bernoulli MLE (이진분류)**

다시 다음 Dataset을 사용하자.

```text
y = [1, 0, 1, 1, 0]
```

Likelihood는

```math
L(p)
=
p^3(1-p)^2
```

이다.

Parameter 후보를 비교하면

| $p$ (가중치) | Likelihood |
| ---: | ---: |
| 0.2 | 0.00512 |
| 0.4 | 0.02304 |
| **0.6** | **0.03456** |
| 0.8 | 0.02048 |


비교한 값 중에서는 `p=0.6`이 최대다.

실제로 Bernoulli Distribution에서 모든 Sample이 동일한 `p`를 가진다고 가정하면 MLE는

```math
\hat p
=
\frac{\sum_{i=1}^{n}y_i}
{n}
```

즉 **Sample Mean과 동일하다.**

현재 Dataset에서는

```math
\hat p = \frac{1+0+1+1+0} {5}
```

```math
= \frac{3}{5}
```

```math
=0.6
```

<br><br>

---

# **22. 왜 Bernoulli MLE가 평균인가**

Bernoulli Log-Likelihood는

```math
\ell(p)
=
\sum_i
\left[
y_i\log p
+
(1-y_i)\log(1-p)
\right]
```

Success 수를 `k`, Sample 수를 `n`이라 하면

```math
\ell(p)
=
k\log p
+
(n-k)\log(1-p)
```

이를 `p`에 대해 미분하면

```math
\frac{d\ell}{dp}
=
\frac{k}{p}
-
\frac{n-k}{1-p}
```


MLE에서는 최댓값 후보를 찾기 위해 이를 0으로 둔다.

```math
\frac{k}{p}
-
\frac{n-k}{1-p}
=
0
```

정리하면

```math
k(1-p)
=
(n-k)p
```

```math
k-kp
=
np-kp
```

따라서

```math
k=np
```

이고

```math
\hat p
=
\frac{k}{n}
```

이 된다.

즉 Bernoulli Parameter의 MLE가 Observed Success Rate가 되는 이유를 수학적으로 확인할 수 있다.

<br><br>

---

# **23. MLE를 Grid Search로 확인하기**

[ Python : code ] "여러 Bernoulli Parameter 후보에서 MLE 찾기"

```python
import numpy as np

y = np.array([1, 0, 1, 1, 0])

# Likelihood 계산 함수 (우도 추정)
def likelihood(p):
    probabilities = (p ** y) * ((1 - p) ** (1 - y))
    return np.prod(probabilities)

# 비교할 구매 확률
## 0.01부터 0.99까지 0.01 간격으로 총 99개의 후보 파라미터를 생성한다.
candidates = np.linspace(0.01, 0.99, 99) 

values = np.array([
    likelihood(p)
    for p in candidates
])

best_index = np.argmax(values)

print(
    "Best p:",
    round(candidates[best_index], 2)
)

print(
    "Max Likelihood:",
    round(values[best_index], 5)
)
```

[ Result ]

```text
Best p: 0.6
Max Likelihood: 0.03456
```

<br>

> ## **23-1) 코드 흐름**

```text
Observed Data

        ↓

p 후보 생성

        ↓

각 p에 대해 Likelihood 계산

        ↓

np.argmax()

        ↓

Maximum Likelihood 후보 선택

        ↓

p = 0.60
```

실제 Deep Learning에서는 Parameter Dimension이 매우 크기 때문에 모든 후보를 Grid Search로 탐색하지 않는다.

대신

```text
Gradient

Gradient Descent

Backpropagation
```

등을 이용한다.

<br><br>

---

# **24. Logistic Regression에서 MLE**

실제 Logistic Regression에서는 하나의 \(p\)를 학습하는 것이 아니다.

각 Sample의 Probability를

```math
p_i
=
\sigma(x_iw+b)
```

로 계산한다.

학습 Parameter는

```text
w
→ Weight

b
→ Bias
```

이다.

따라서

```math
\theta=(w,b)
```

이고

```math
\hat\theta_{MLE}
=
\arg\max_{\theta}
\prod_i
p_i^{y_i}
(1-p_i)^{1-y_i}
```

를 찾는다.

즉 Model은

```text
실제 y = 1인 Sample
→ 높은 pᵢ

실제 y = 0인 Sample
→ 낮은 pᵢ
```

를 만들도록 Parameter를 조정한다.

<br><br>

---

# **25. Likelihood가 크면 반드시 좋은 예측 모델인가**

Training Dataset에서 높은 Likelihood를 갖는다는 것은 해당 Dataset을 잘 설명한다는 뜻이지만

```text
높은 Training Likelihood
=
새로운 Data에서도 무조건 좋은 Model
```

은 아니다.

지나치게 복잡한 Model은 Training Data에 과도하게 맞춰질 수 있다.

이를 **Overfitting**이라고 한다.

따라서 실제 머신러닝에서는

```text
Training Likelihood / Loss

Validation Performance

Test Performance

Regularization
```

을 함께 고려한다.

MLE는 **Parameter를 추정하는 원리**이지 Generalization 성능을 자동으로 보장하는 방법은 아니다.

<br><br>

---

# **26. Logarithm의 핵심 성질**

Likelihood를 실무에서 직접 사용하기 어려운 가장 큰 이유는 많은 Probability를 곱하기 때문이다.

Logarithm은 다음 성질을 가진다.

```math
\log(ab)
=
\log a+\log b
```

또한

```math
\log(a^n)
=
n\log a
```

이다.

따라서

```math
P_1P_2P_3
```

의 Log는

```math
\log P_1
+
\log P_2
+
\log P_3
```

로 바꿀 수 있다.

즉

```text
Probability의 곱

↓

Log 적용

↓

Log Probability의 합
```

으로 변환한다.

<br><br>

---

# **27. Log-Likelihood**

Log-Likelihood(로그 우도)는 **Likelihood에 Logarithm(로그)을 적용한 것**이다.

$$
\ell(\theta)=\log L(\theta)
$$

로그를 적용하는 가장 큰 이유는 **여러 확률의 곱셈을 덧셈으로 변환하여 계산을 쉽게 하기 위해서**이다.

<br>

> ## 27-1) 로그(Log)의 기본 성질

로그에는 다음과 같은 성질이 있다.

$$
\log(a\times b)=\log(a)+\log(b)
$$

즉, **곱셈으로 연결된 숫자에 로그를 적용하면 각각의 로그를 더하는 형태로 바뀐다.**

예를 들어

$$
\log(2\times3)=\log 2+\log 3
$$

이다.

숫자를 세 번 곱하는 경우에도 동일하다.

$$
\log(2\times3\times4)=\log2+\log3+\log4
$$

또한 같은 숫자를 여러 번 곱한 거듭제곱은 다음과 같이 변환할 수 있다.

$$
\log(a^n)=n\log(a)
$$

예를 들어

$$
\log(0.6^3)=3\log(0.6)
$$

이다.

<br>

> ## 27-2) Likelihood에 로그 적용하기

다음과 같은 5개의 관측 데이터가 있다고 가정하자.

```python
y = [1, 0, 1, 1, 0]
```

- Success(1): 3회
- Failure(0): 2회

승리 확률을 $p$라고 하면 Likelihood는 다음과 같다.

$$
L(p\mid D)=p^3(1-p)^2
$$

이제 로그를 적용하는 과정을 단계별로 살펴보면,

> **① Likelihood 전체에 로그를 적용한다.**

$$
\log L(p\mid D)=\log[p^3(1-p)^2]
$$

> **② 곱셈을 덧셈으로 분리한다.**

로그의 성질을 이용하면 다음과 같다.

$$
\log L(p\mid D)=\log(p^3)+\log((1-p)^2)
$$

> **③ 지수를 로그 앞으로 이동시킨다.**

$$
\log L(p\mid D)=3\log p+2\log(1-p)
$$

이렇게 하면 기존의 곱셈이 덧셈으로 변환된다.

<br>

> ## 27-3) 실제 숫자로 계산하기

승리 확률이 $p=0.6$이라고 가정하면,

**① 기존 Likelihood**

$$
\begin{aligned}
L(0.6\mid D)
&=0.6^3\times(1-0.6)^2\\
&=0.6^3\times0.4^2\\
&=0.03456
\end{aligned}
$$

**② Log-Likelihood**

$$
\begin{aligned}
\log L(0.6\mid D)
&=\log(0.6^3\times0.4^2)\\
&=\log(0.6^3)+\log(0.4^2)\\
&=3\log(0.6)+2\log(0.4)\\
&\approx-3.36506
\end{aligned}
$$

여기서 일반적으로 사용하는 로그는 자연로그 $\ln$이며, Python에서는 `np.log()`로 계산한다.

**중요한 점은 0.03456과 -3.36506이 동일한 숫자라는 뜻이 아니라, 로그를 적용하여 표현 방식이 달라진 것이라는 점이다.**

<br>

> ## 27-4) 왜 Log-Likelihood를 사용할까?

Likelihood는 Sample이 많아질수록 수많은 확률을 곱해야 한다.

확률은 0과 1 사이의 값이므로 계속 곱하면 결과가 매우 작아진다.

반면 Log-Likelihood는 곱셈을 덧셈으로 변환하기 때문에 다음과 같은 장점이 있다.

- **계산의 단순화:** 곱셈을 덧셈으로 변환한다.
- **수치적 안정성:** 매우 작은 확률을 반복해서 곱하는 문제를 줄인다.
- **미분의 편리함:** 기울기 계산과 파라미터 최적화가 쉬워진다.

또한 로그는 단조 증가 함수이므로 **Likelihood를 최대화하는 파라미터와 Log-Likelihood를 최대화하는 파라미터는 동일하다.**

$$
\arg\max_\theta L(\theta)
=
\arg\max_\theta\log L(\theta)
$$

따라서 머신러닝에서는 계산하기 편리한 Log-Likelihood를 주로 사용한다.

<br><br>

---

# **28. Bernoulli Log-Likelihood**

Bernoulli Likelihood가

```math
L(p)
=
\prod_i
p^{y_i}(1-p)^{1-y_i}
```

이면 Log-Likelihood는

```math
\ell(p)
=
\sum_i
\left[
y_i\log p
+
(1-y_i)\log(1-p)
\right]
```

이다.

예를 들어

```text
y = [1, 0, 1, 1, 0]

p = 0.6
```

이면

```math
\ell(p)
=
3\log(0.6)
+
2\log(0.4)
```

이다.

```math
\log(0.6)\approx-0.5108
```

```math
\log(0.4)\approx-0.9163
```

이므로

```math
\ell(p)
\approx
3(-0.5108)
+
2(-0.9163)
```

```math
\approx-3.3651
```

이다.

<br><br>

---

# **29. Likelihood와 Log - Likelihood는 같은 Parameter를 선택한다**

Logarithm은 양수 범위에서 단조 증가 함수이다.

따라서

```math
L_A>L_B
```

이면

```math
\log L_A>\log L_B
```

이다.

예를 들어

| Model | Likelihood | Log-Likelihood |
| --- | ---: | ---: |
| A | 0.03456 | -3.3651 |
| B | 0.02048 | -3.8883 |

Log-Likelihood는 음수이지만

```math
-3.3651>-3.8883
```

이므로 Model A가 더 크다.

따라서

```math
\arg\max_\theta
L(\theta)
=
\arg\max_\theta
\log L(\theta)
```

이다.

<br><br>

---

# **30. Log-Likelihood가 필요한 이유 - Underflow**

Probability는 대부분 0과 1 사이이다.

예를 들어

```math
0.5^{2000}
```

은 극도로 작은 값이다.

Computer에서는 이 값이 표현 가능한 범위보다 작아져 `0`처럼 처리될 수 있다.

이를 **Underflow**라고 한다.

[ Python : code ] "Probability 곱의 Underflow와 Log-Likelihood 비교"

```python
import numpy as np

p = np.full(2000, 0.5)

likelihood = np.prod(p)

log_likelihood = np.sum(np.log(p)) # 로그 우도 (로그를 씌운 값을 모두 더함)

print("Likelihood:", likelihood)
print(
    "Log-Likelihood:",
    round(log_likelihood, 3)
)
```

[ Result ]

```text
Likelihood: 0.0
Log-Likelihood: -1386.294
```

Likelihood는 계산 중 `0`이 되었지만 Log-Likelihood는 유한한 값으로 유지된다.

따라서 Log-Likelihood는

```text
계산 단순화

+

Numerical Stability 개선
```

이라는 장점이 있다.

<br><br>

---

# **31. Negative Log-Likelihood (NLL)**

> ## 31-1) Negative Log-Likelihood란?

MLE(Maximum Likelihood Estimation)의 목표는 **관측 데이터를 가장 잘 설명하는 Model Parameter를 찾아 Log-Likelihood를 최대화하는 것**이다.

$$
\max_{\theta}\ell(\theta)
$$

하지만 머신러닝에서는 일반적으로 Loss Function(손실함수)을 **최소화하는 방향**으로 모델을 학습한다.

따라서 Log-Likelihood에 음수(-)를 붙여 Negative Log-Likelihood(NLL)로 변환한다.

$$
NLL(\theta)=-\ell(\theta)
$$

> ## 31-2) 왜 음수(-)를 붙일까?

**음수를 붙이는 이유는 최대화 문제를 최소화 문제로 바꾸기 위해서이다.**

예를 들어 Log-Likelihood가 다음과 같다고 가정하자.

```text
Parameter A → Log-Likelihood = -5
Parameter B → Log-Likelihood = -2
```

Log-Likelihood는 값이 클수록 좋으므로 `-2`가 `-5`보다 좋은 결과이다.

여기에 음수를 붙이면 다음과 같이 변한다.

```text
Parameter A → NLL = 5
Parameter B → NLL = 2
```

NLL은 값이 작을수록 좋으므로 `2`가 `5`보다 좋은 결과이다.

즉,

| Parameter | Log-Likelihood | NLL |
|---|---:|---:|
| A | -5 | 5 |
| **B** | **-2 (최대)** | **2 (최소)** |

**음수를 붙이면 값의 부호가 반대로 바뀌기 때문에, Log-Likelihood를 최대화하는 것과 NLL을 최소화하는 것은 동일한 Parameter를 선택하게 된다.**

> ## 31-3) 실제 Likelihood에 적용하면

예를 들어 관측 데이터가 다음과 같다고 가정하자.

```python
y = [1, 0, 1, 1, 0]
```

승리 3회, 패배 2회이므로 Likelihood는 다음과 같다.

$$
L(p\mid D)=p^3(1-p)^2
$$

승리 확률이 $p=0.6$이라면 Log-Likelihood는

$$
\begin{aligned}
\ell(p)
&=3\log(0.6)+2\log(0.4)\\
&\approx-3.36506
\end{aligned}
$$

이다.

여기에 음수를 붙이면

$$
\begin{aligned}
NLL(p)
&=-\ell(p)\\
&=-(-3.36506)\\
&=3.36506
\end{aligned}
$$

이 된다.

확률이 0과 1 사이에 있으므로 Log-Likelihood는 일반적으로 0 이하의 값을 가진다. 

따라서 음수를 붙인 NLL은 0 이상의 값이 된다.

<br>

> ## 31-4) MLE와 NLL의 관계

Log-Likelihood 최대화는

$$
\max_{\theta}\ell(\theta)
$$

이고, Negative Log-Likelihood 최소화는

$$
\min_{\theta}NLL(\theta)
$$

두 방법은 동일한 최적의 Parameter를 선택한다.

전체 흐름은 다음과 같다.

```text
Likelihood 최대화
        ↕
Log-Likelihood 최대화
        ↕
Negative Log-Likelihood 최소화
```

머신러닝에서는 NLL을 Loss Function(손실함수)으로 사용하여 역전파(Backpropagation)로 기울기를 계산하고,

옵티마이저(Optimizer)를 통해 가중치(Weight)와 편향(Bias)을 업데이트할 수 있다.

```text
NLL (Loss) 계산
        ↓
Backpropagation
        ↓
Gradient 계산
        ↓
Optimizer
        ↓
Weight, Bias 업데이트
        ⟲ 반복 학습
```

<br><br>

---

# **32. 한 Sample의 Binary NLL**

Binary Classification에서 실제 정답이 `y`, Model Probability가 `p`라면

```math
NLL
=
-
\left[
y\log p
+
(1-y)\log(1-p)
\right]
```

<br>

> ## **32-1) 실제 정답이 1인 경우**

`y=1`이면

```math
NLL=-\log p
```

예를 들어,

```math
p=0.9
```

이면

```math
-\log(0.9)
\approx0.105
```


반면,

```math
p=0.1
```

이면

```math
-\log(0.1)
\approx2.303
```

이다.

즉 실제 정답에 높은 Probability를 부여할수록 Loss가 작다.

<br>

> ## **32-2) 실제 정답이 0인 경우**

\(y=0\)이면

```math
NLL
=
-\log(1-p)
```

이다.

Model이

```text
p = 0.1
```

이라고 Prediction했다면 실제 Class 0에 부여한 Probability는

```text
1 - 0.1
= 0.9
```

이므로 Loss가 작다.

<br><br>

---

# **33. NLL은 잘못된 확신을 크게 벌점 준다**

다음 상황을 비교해보자.

실제 정답은 모두 `1`이다.

| Prediction (`p`) | NLL |
| ---: | ---: |
| 0.99 | 0.010 |
| 0.90 | 0.105 |
| 0.70 | 0.357 |
| 0.50 | 0.693 |
| 0.10 | 2.303 |
| 0.01 | 4.605 |

Model이 실제 정답에 매우 낮은 Probability를 부여하면 Loss가 급격하게 커진다.

따라서 NLL 계열 Loss는

```text
정답을 틀림

+

틀린 Prediction을 강하게 확신함
```

에 큰 Penalty를 준다.

<br><br>

---

# **34. 스포츠 센서 Binary NLL 예제**

IMU Sensor 기반으로 특정 Movement Event의 발생 여부를 예측한다고 하자.

실제 정답과 Probability는 다음과 같다.

```text
y = [1, 0, 1, 0]

p = [0.85, 0.25, 0.70, 0.10]
```

[ Python : code ] "Binary Sensor Event Prediction의 NLL 계산"

```python
import numpy as np

y = np.array([1, 0, 1, 0])

p = np.array([0.85, 0.25, 0.70, 0.10])

# NLL = -(실제 정답에 부여한 예측 확률의 로그값)
# 실제 정답이 1이면 -log(p), 0이면 -log(1-p)
nll_each = -(y * np.log(p) + (1 - y) * np.log(1 - p)) 

total_nll = np.sum(nll_each)
mean_nll = np.mean(nll_each)

print(
    "NLL:",
    np.round(nll_each, 3)
)

print(
    "Total:",
    round(total_nll, 3)
)

print(
    "Mean:",
    round(mean_nll, 3)
)
```

[ Result ]

```text
NLL: [0.163 0.288 0.357 0.105]

Total: 0.913
Mean: 0.228
```

각 Sample에서 Model이 실제 정답에 높은 Probability를 부여했기 때문에 Loss가 비교적 작다.

<br><br>

---

# **35. Total NLL과 Mean NLL**

Dataset 전체 Loss는 Sum으로 계산할 수도 있고 Mean으로 계산할 수도 있다.

Total NLL:

```math
-\sum_{i=1}^{n}
\left[
y_i\log p_i
+
(1-y_i)\log(1-p_i)
\right]
```

Mean NLL:

```math
-\frac{1}{n}
\sum_{i=1}^{n}
\left[
y_i\log p_i
+
(1-y_i)\log(1-p_i)
\right]
```

고정된 Dataset에서는 Mean이 Total을 단순히 `n`으로 나눈 것이므로 같은 Parameter에서 최솟값이 발생한다.

| 구분 | 특징 |
| --- | --- |
| Total | Sample 수가 많으면 값도 커짐 |
| Mean | Sample당 평균 Loss |
| 최적 Parameter | 고정 Dataset에서는 동일 |

실제 Deep Learning Library에서는 **평균 Loss**를 기본으로 사용하는 경우가 많다.

<br><br>

---

# **36. Likelihood와 Accuracy는 다른 정보를 본다**

두 Model이 동일한 Class Prediction을 하더라도 Probability는 다를 수 있다.

실제 정답:

```text
y = 1
```

Model A:

```text
p = 0.51
→ Class 1
```

Model B:

```text
p = 0.95
→ Class 1
```

Threshold가 `0.5`이면 두 Model 모두 Accuracy 관점에서는 Correct이다.

하지만 NLL은

Model A:

```math
-\log(0.51)\approx0.673
```

Model B:

```math
-\log(0.95)\approx0.051
```

이다.

따라서 Probability 기반 Loss는 단순 Class Correctness보다 더 많은 정보를 사용한다.

```text
Accuracy
→ 최종 Class만 비교

NLL
→ 정답에 부여한 Probability까지 평가
```

<br><br>

---

# **37. Categorical Likelihood**

Multiclass Classification에서는 Categorical Distribution을 사용한다.

예를 들어 배드민턴 Stroke Prediction에서

```text
Smash
Drop
Clear
```

의 Probability가

```text
[0.70, 0.20, 0.10]
```

이고 실제 Class가 `Smash`라면 해당 Sample의 Likelihood는

```text
0.70
```

이다.

실제 Class가 `Drop`이었다면

```text
0.20
```

을 사용한다.

즉 Multiclass에서도 핵심 원리는 같다.

```text
실제 정답 Class에
Model이 얼마나 높은 Probability를 주었는가?
```

<br><br>

---

# **38. Multiclass NLL**

실제 Class를 One-Hot Vector \(y\), Model Probability를 \(p\)라고 하면

```math
NLL
=
-\sum_{k=1}^{K}
y_k\log p_k
```

이다.

예를 들어

```text
Actual Class
→ Drop

One-Hot
→ [0, 1, 0]

Probability
→ [0.20, 0.70, 0.10]
```

이라면

```math
NLL
=
-
[
0\log0.20
+
1\log0.70
+
0\log0.10
]
```

이므로

```math
NLL
=
-\log0.70
```

```math
\approx0.357
```

이다.

이 식은 다음 강의에서 **Categorical Cross Entropy**와 직접 연결된다.

<br><br>

---

# **39. MLE와 Cross Entropy의 연결**

Binary Classification에서 Mean NLL은

```math
-\frac{1}{n}
\sum_i
\left[
y_i\log p_i
+
(1-y_i)\log(1-p_i)
\right]
```

이다.

이것이 바로 Binary Cross Entropy에서 사용하는 형태와 같다.

즉

```text
Bernoulli Probability Model

        ↓

Likelihood

        ↓

Log-Likelihood

        ↓

Negative Log-Likelihood

        ↓

Binary Cross Entropy
```

로 연결된다.

Multiclass에서도

```text
Categorical Distribution

        ↓

Likelihood

        ↓

NLL

        ↓

Categorical Cross Entropy
```

로 이어진다.

<br><br>

---

# **40. Regression의 MLE와 MSE 연결**

MLE는 Classification에서만 사용하는 개념이 아니다.

Regression에서 Target이

```math
y_i
=
f(x_i;\theta)
+
\epsilon_i
```

이고 Noise가

```math
\epsilon_i
\sim
N(0,\sigma^2)
```

즉 평균 0의 Normal Distribution을 따른다고 가정하면

```math
Y_i\mid X_i
\sim
N(\hat y_i,\sigma^2)
```

로 Modeling할 수 있다.

이때 \(\sigma^2\)가 고정되어 있다고 가정하면 Log-Likelihood를 최대화하는 것은

```math
\sum_i
(y_i-\hat y_i)^2
```

를 최소화하는 것과 연결된다.

즉

```text
Normal Distribution 가정

        ↓

Maximum Likelihood

        ↓

Negative Log-Likelihood

        ↓

Squared Error

        ↓

MSE
```

로 연결할 수 있다.

따라서

```text
Classification
→ Cross Entropy

Regression + Gaussian Assumption
→ MSE
```

모두 확률 모델과 MLE 관점에서 설명할 수 있다.

<br><br>

---

# **41. Probability 0 또는 1에서 발생하는 문제**

Logarithm은

```math
\log0
```

을 정의할 수 없다.

따라서 실제 정답이 1인데 Model Probability가 정확히 0이라면

```math
-\log0
```

이 되어 Loss가 무한대로 발산한다.

기초 구현에서는 다음처럼 Probability를 Clip할 수 있다.

[ Python : code ] "Logarithm 계산을 위한 Probability Clipping"

```python
import numpy as np

p = np.array([
    0.0,
    0.9,
    1.0
])

eps = 1e-7

safe_p = np.clip(
    p,
    eps,
    1 - eps
)

print(safe_p)
print(np.log(safe_p))
```

[ Result ]

```text
[1.000000e-07 9.000000e-01 9.999999e-01]

[-16.11809565  -0.10536052  -0.0000001 ]
```

다만 실무 Framework에서는 Probability를 직접 Clip하기보다 **Logit에서 수치적으로 안정적인 Loss 함수를 계산하는 방식**을 많이 사용한다.

<br><br>

---

# **42. Logit 기반 Loss가 실무에서 중요한 이유**

Binary Classification에서 다음 두 방식이 있다고 하자.

```text
방법 A

Logit
→ Sigmoid
→ Probability
→ log()
→ Loss
```

```text
방법 B

Logit
→ Stable BCE Loss
```

방법 B는 Sigmoid와 Log 계산을 내부적으로 합쳐 수치적으로 안정적으로 처리할 수 있다.

예를 들어 PyTorch에서는 개념적으로

```text
Sigmoid
+
Binary Cross Entropy
```

를 따로 수행하기보다 `BCEWithLogitsLoss`와 같은 방식을 사용할 수 있다.

핵심은

> 수학적으로 같은 목적이라도 구현에서는 Numerical Stability를 고려해야 한다.

는 것이다.

<br><br>

---

# **43. MLE와 MAP의 차이**

MLE는 Data의 Likelihood만 이용한다.

```math
\hat\theta_{MLE}
=
\arg\max_\theta
P(D\mid\theta)
```

반면 MAP(Maximum A Posteriori)는 Parameter에 대한 Prior까지 고려한다.

```math
\hat\theta_{MAP}
=
\arg\max_\theta
P(\theta\mid D)
```

Bayes Rule을 이용하면

```math
P(\theta\mid D)
\propto
P(D\mid\theta)
P(\theta)
```

이므로 MAP는

```text
Likelihood

+

Prior
```

를 이용한다고 이해할 수 있다.

| 구분 | MLE | MAP |
| --- | --- | --- |
| Data Likelihood | 사용 | 사용 |
| Parameter Prior | 사용하지 않음 | 사용 |
| 목표 | Data를 가장 잘 설명 | Data + Prior를 함께 고려 |

초기 단계에서는 MLE를 먼저 이해하고, 이후 Bayesian Statistics에서 MAP로 확장하면 된다.

<br><br>

---

# **44. Likelihood와 이항분포 Probability를 구분하기**

다음 Data를 생각해보자.

```text
5번 시도 중
3번 Success
```

순서가 고정된 실제 관측값

```text
[1, 0, 1, 1, 0]
```

의 Likelihood는

```math
p^3(1-p)^2
```

이다.

반면

```text
5번 중 정확히 3번 Success
```

라는 **횟수만** 관측하는 Binomial Probability는

```math
\binom{5}{3}
p^3(1-p)^2
```

이다.

차이는 Combination Term

```math
\binom{5}{3}
```

이다.

다만 이 Combination Term은 \(p\)에 의존하지 않으므로 \(p\)에 대한 MLE를 구할 때는 동일한 최적값을 만든다.

<br><br>

---

# **45. 실무 종합 예제 — 야구 투구 결과 Model**

투수의 각 Pitch가 Strike인지 Ball인지 예측한다고 하자.

Feature:

```text
Release Speed
Horizontal Movement
Vertical Movement
Release Height
Count
```

Target:

```text
1 → Strike
0 → Ball
```

Model은 Logistic Regression 형태라고 가정한다.

```math
p_i
=
\sigma(x_iw+b)
```

전체 Training 흐름은 다음과 같다.

```text
Pitch Sensor Data

        ↓

Feature Matrix X

        ↓

Logit
Xw + b

        ↓

Sigmoid

        ↓

Strike Probability pᵢ

        ↓

Actual Target yᵢ와 비교

        ↓

Likelihood

        ↓

Log-Likelihood

        ↓

Negative Log-Likelihood

        ↓

Gradient 계산

        ↓

w, b Update
```

이것이 Classification Model Learning의 확률적 관점이다.

<br><br>

---

# **46. 실무 종합 Python 예제**

[ Python : code ] "센서 Feature → Sigmoid Probability → Likelihood → NLL 계산"

```python
import numpy as np

# 정규화된 투구 Feature
X = np.array([
    [ 0.8,  0.2,  0.6],
    [-0.3,  0.7, -0.2],
    [ 1.0, -0.4,  0.9],
    [-0.5,  0.5, -0.3]
])

# 실제 Strike 여부
y = np.array([1, 0, 1, 0])

# 예제 Parameter
w = np.array([1.1, -0.8, 0.6])

b = -0.2

# Logit
logits = X @ w + b # 선형회귀(행렬 곱)

# Sigmoid
p = 1 / (1 + np.exp(-logits)) # Logit을 Sigmoid 함수로 적용하여 0~1사이의 확률로 변환
                              # p는 모델이 예측한 Strike 확률
                              # 첫 번째 투구는 Strike일 확률이 약 70.68%라는 의미

# 실제 관측 결과에 부여한 Probability
observed_prob = (p ** y) * ((1 - p) ** (1 - y)) # 베르누이 분포(실제 관측된 정답에 모델이 부여한 확률)

likelihood = np.prod(observed_prob) # likelihood -> 관측된 네 결과의 확률을 모두 곱함 

log_likelihood = np.sum(np.log(observed_prob)) # log-likelihood -> likelihood에 로그를 적용하고 더함

nll = -log_likelihood # 경사하강법을 사용하기 위해 NLL 적용

print(
    "Logits:",
    np.round(logits, 3)
)

print(
    "Probabilities:",
    np.round(p, 3)
)

print(
    "Observed Prob:",
    np.round(observed_prob, 3)
)

print(
    "Likelihood:",
    round(likelihood, 5)
)

print(
    "Log-Likelihood:",
    round(log_likelihood, 3)
)

print(
    "NLL:",
    round(nll, 3)
)
```

[ Result ]

```text
Logits: [ 0.88 -1.21  1.76 -1.33]

Probabilities: [0.707 0.230 0.853 0.209]

Observed Prob: [0.707 0.770 0.853 0.791]

Likelihood: 0.36757

Log-Likelihood: -1.001

NLL: 1.001
```

<br>

> ## **46-1) 코드 결과 해석**

첫 번째 Pitch는 실제 Strike이고 Model이 `0.707`의 Strike Probability를 부여했다.

두 번째 Pitch는 실제 Ball이므로 Model의 Strike Probability `0.230`이 아니라

```math
1-0.230=0.770
```

을 사용한다.

따라서 실제 관측 결과에 부여한 Probability는

```text
[0.707, 0.770, 0.853, 0.791]
```

이다.

이를 모두 곱한 값이 Likelihood이고,

Log를 취한 값이 Log-Likelihood이며,

부호를 바꾼 값이 NLL이다.

<br><br>

---

# **47. Sigmoid와 Softmax의 차이**

> ## 47-1) Sigmoid와 Softmax란?

Sigmoid와 Softmax는 모델이 계산한 **Logit(원시 예측값)을 0~1 사이의 확률로 변환하는 함수**이다.

두 함수 모두 분류(Classification) 문제에 사용하지만, 확률을 계산하는 방식이 다르다.

- **Sigmoid:** 각 Class의 확률을 개별적으로 계산한다.

- **Softmax:** 여러 Class의 확률을 비교하여 전체 합이 1이 되도록 계산한다.

<br>

> ## 47-2) Sigmoid

Sigmoid는 Logit을 0~1 사이의 값으로 변환하는 함수이다.

$$
\sigma(z)=\frac{1}{1+e^{-z}}
$$


예를 들어, 야구 투수의 투구 데이터를 이용하여 다음 3가지 특성을 예측한다고 가정하자.

```text
Label 0 → Strike 여부
Label 1 → 빠른 공 여부
Label 2 → 높은 회전수 여부
```

모델이 각 Label에 Sigmoid를 적용하여 다음과 같은 확률을 출력했다고 가정하자.

```text
Strike 확률    → 0.8 (80%)
빠른 공 확률   → 0.7 (70%)
높은 회전수 확률 → 0.9 (90%)
```

이때 확률의 합은

$$
0.8+0.7+0.9=2.4
$$

이다.

즉, **Sigmoid는 각 Label의 확률을 개별적으로 계산하기 때문에 전체 확률의 합이 1이 될 필요가 없다.**

그 이유는 하나의 투구가 Strike이면서 빠른 공이고, 동시에 회전수가 높은 공일 수도 있기 때문이다.

반면 **Strike와 Ball만 구분하는 이진 분류**에서 Sigmoid 출력 하나를 사용하면 다음과 같다.

```text
Strike 확률 → 0.8 (80%)
Ball 확률   → 0.2 (20%)

전체 합계   → 1.0 (100%)
```

이 경우에는 Strike와 Ball이 서로 반대되는 결과이므로 확률의 합이 1이 된다.

<br>

> ## 47-3) Softmax

Softmax는 여러 개의 Logit을 확률로 변환하는 함수이다.

$$
P(Y=k\mid X)=\frac{e^{z_k}}{\sum_j e^{z_j}}
$$

Softmax의 가장 큰 특징은 **모든 Class의 예측 확률을 합하면 1이 된다는 것**이다.

예를 들어, 야구 투수의 구종을 예측한다고 가정하자.

```text
Class 0 → 직구
Class 1 → 슬라이더
Class 2 → 커브
```

Softmax 출력 결과가 다음과 같을 수 있다.

```text
직구      → 0.7 (70%)
슬라이더  → 0.2 (20%)
커브      → 0.1 (10%)

전체 합계 → 1.0 (100%)
```

이 경우 모델은 직구일 확률이 가장 높다고 예측한다.

**즉, Softmax는 여러 Class 중 하나를 선택하는 다중 클래스 분류에 적합하다.**

<br>

> ## 47-4) Sigmoid를 다중 분류에 사용할 수 있을까?

사용할 수 있다. 다만 다중 클래스 분류와 다중 레이블 분류를 구분해야 한다.

> ### ① 다중 클래스 분류 (Multi-class Classification)

**여러 Class 중 하나만 정답인 경우**이다.

예를 들어 구종을 예측한다면 한 투구는 직구, 슬라이더, 커브 중 하나로 분류된다.

```text
직구      → 0.7
슬라이더  → 0.2
커브      → 0.1
```

이 경우에는 일반적으로 **Softmax**를 사용한다.

> ### ② 다중 레이블 분류 (Multi-label Classification)

**하나의 Sample이 여러 Label에 동시에 속할 수 있는 경우**이다.

예를 들어 선수의 상태를 예측한다고 가정하자.

```text
피로 상태  → 0.9
근육 통증  → 0.8
부상 위험  → 0.7
```

한 선수가 피로하면서 근육 통증도 있고 부상 위험도 높을 수 있다.

따라서 여러 Label을 동시에 예측해야 하므로 **Sigmoid**를 사용한다.

Sigmoid는 각 Label을 개별적으로 처리하기 때문에 확률의 합이 1일 필요가 없다.

<br>

> ## 47-5) Sigmoid와 Softmax 비교

| 구분 | Sigmoid | Softmax |
|---|---|---|
| 출력 범위 | 0~1 | 0~1 |
| 확률 계산 | 각 출력 독립 계산 | Class 간 상대적 계산 |
| 확률 합계 | 1일 필요 없음 | 항상 1 |
| 대표적인 분류 | 이진 / 다중 레이블 | 다중 클래스 |
| 여러 Label 동시 예측 | 가능 | 일반적으로 부적합 |
| 대표 손실함수 | Binary Cross Entropy | Cross Entropy |

**참고 :**    
Sigmoid도 다중 클래스 분류에 사용할 수 있고, Softmax도 이진 분류에 사용할 수 있다. 하지만 일반적으로 위와 같이 구분하여 사용한다.

```text
분류(Classification)
       │
       ├── 이진 분류 (Binary)
       │       └── Sigmoid
       │
       ├── 다중 클래스 분류 (Multi-class) -> 여러 클래스 중 하나만 예측
       │       └── Softmax
       │
       └── 다중 레이블 분류 (Multi-label) -> 여러 클래스를 동시에 예측
               └── Sigmoid
```

<br>

## 47-6) 실제 머신러닝 모델에서는?

PyTorch에서는 일반적으로 다음과 같이 사용한다.

```python
import torch.nn as nn

# 이진 분류 (Sigmoid + Binary Cross Entropy)
criterion = nn.BCEWithLogitsLoss()

# 다중 클래스 분류 (Log-Softmax + Negative Log-Likelihood)
criterion = nn.CrossEntropyLoss()

# 다중 레이블 분류 (각 출력에 Sigmoid + Binary Cross Entropy)
criterion = nn.BCEWithLogitsLoss()
```

`BCEWithLogitsLoss`와 `CrossEntropyLoss`는 필요한 확률 변환을 내부적으로 처리하므로 학습 시 Sigmoid나 Softmax를 미리 적용하지 않고 Logit을 입력한다.


<br><br>

---

# **48. 전체 계산 흐름도**

머신러닝에서 모델이 확률을 예측하고, 그 예측이 정답에 가까워지도록 가중치와 편향을 학습하는 전체과정이다.


```text
Observed Dataset
(X, y)

        ↓

Model Parameter θ

        ↓

Logit 계산(모델이 계산한 원시 예측값)

z = f(X; θ)

        ↓

Probability 계산

Binary
→ Sigmoid (0 ~ 1)
      → 각 Logit값을 독립적으로 확률로 변환 (각 클래스의 합이 반드시 1일 필요는 없다)

Multiclass
→ Softmax (0 ~ 1) 
      → 여러 클래스의 확률을 합해서 1이 되도록 변환 (각 클래스의 합은 반드시 1)

        ↓

실제 정답에 부여한
Probability 선택

        ↓

Likelihood

∏ P(yᵢ | xᵢ, θ)

        ↓

Log

Σ log P(yᵢ | xᵢ, θ)

        ↓

Negative

NLL

        ↓

Loss 최소화

        ↓

Parameter Update
```

이 흐름을 이해하면 다음의 관계가 자연스럽게 연결된다.


```text
Probability Model (입력 피처로 예측 확률 계산)

→ Likelihood (실제 관측결과인 로짓값에 부여한 확률 계산)

→ MLE (우도가 가장 커지는 파라미터를 찾는 것이 목표)

→ Log-Likelihood (확률의 곱셈을 로그의 덧셈으로 변환)

→ NLL (최대화 문제를 최소화 문제로 변환)

→ Cross Entropy (정답과 예측 확률 사이의 손실 계산)

→ Gradient Descent (가중치와 편향을 실제로 업데이트)

→ 파라미터 업데이트 후 반복 학습 ⟲
```

<br><br>

---

# **49. 자주 발생하는 실수**

> ## **49-1) Likelihood를 Parameter의 Probability로 해석**

잘못된 해석이다.

Likelihood는

```text
Observed Data가 주어졌을 때
Parameter를 비교하는 함수
```

이다.

<br>

> ## **49-2) 가장 높은 p가 항상 좋은 Bernoulli Model이라고 생각**

Failure Data도 존재한다면 지나치게 높은 \(p\)는 Failure 관측값에 매우 낮은 Probability를 부여한다.

<br>

> ## **49-3) Likelihood를 무조건 직접 곱하기**

Sample 수가 많으면 Underflow가 발생할 수 있다.

따라서 Log-Likelihood를 사용하는 것이 일반적이다.

<br>

> ## **49-4) Log-Likelihood가 -3보다 -4가 더 좋다고 생각**

Log-Likelihood는 **더 큰 값**이 좋다.

```math
-3>-4
```

이다.

NLL에서는 반대로 더 작은 값이 좋다.

<br>

> ## **49-5) Accuracy와 NLL을 같은 Metric으로 생각**

Accuracy는 Class Correctness를 본다.

NLL은 Actual Class에 부여한 Probability까지 평가한다.

<br>

> ## **49-6) Independence Assumption을 확인하지 않음**

반복 측정, 선수별 Time Series, 센서 Sequence는 서로 독립이 아닐 수 있다.

<br>

> ## **49-7) Training Likelihood가 크면 Generalization도 좋다고 생각**

Training Data에 과적합될 수 있으므로 Validation/Test 평가가 별도로 필요하다.

<br><br>

---

# **50. Advanced 추가 학습**

> ## **50-1) MAP**

MLE에 Parameter Prior를 추가하면 MAP Estimation으로 확장할 수 있다.

<br>

> ## **50-2) Regularization과 Prior**

L2 Regularization은 특정 조건에서 Gaussian Prior를 사용한 MAP와 연결해서 해석할 수 있다.

L1 Regularization도 Laplace Prior와 연결할 수 있다.

<br>

> ## **50-3) Log-Sum-Exp**

Softmax와 Multiclass Cross Entropy를 안정적으로 계산할 때 `Log-Sum-Exp` 기법이 중요하다.

<br>

> ## **50-4) Mini-Batch Loss**

Deep Learning에서는 전체 Dataset의 NLL을 매번 계산하지 않고 Mini-Batch의 평균 Loss를 이용해 Parameter를 반복적으로 Update한다.

<br><br>

---

# **51. 핵심정리**

1. Probability는 주어진 Model 또는 조건에서 특정 결과가 발생할 가능성을 나타낸다.

2. Probability는 0과 1 사이의 값을 가진다.

3. Random Variable은 확률적으로 발생하는 결과를 숫자로 표현한 변수이다.

4. Probability Distribution은 Random Variable이 가질 수 있는 값과 각 값에 부여된 Probability를 표현한다.

5. Discrete Random Variable은 구분되는 개별 값을 가진다.

6. Continuous Random Variable은 연속적인 값을 가진다.

7. Bernoulli Distribution은 결과가 0 또는 1인 상황을 표현한다.

8. Bernoulli Distribution의 Parameter는 일반적으로

```math
p=P(Y=1)
```

이다.

9. 반대 Class의 Probability는

```math
1-p
```

이다.

10. Bernoulli Probability Mass Function은 다음과 같다.

```math
P(Y=y)
=
p^y(1-p)^{1-y}
```

11. Categorical Distribution은 K개의 상호 배타적인 Class 중 하나가 발생하는 상황을 표현한다.

12. Categorical Distribution의 모든 Class Probability 합은 1이다.

13. Binary Classification의 Sigmoid Output은 Bernoulli Parameter로 해석할 수 있다.

14. Multiclass Classification의 Softmax Output은 Categorical Distribution의 Parameter로 해석할 수 있다.

15. Conditional Probability는 특정 조건이 주어졌을 때 사건이 발생할 Probability이다.

16. 조건부 확률은

```math
P(B\mid A)
=
\frac{P(A\cap B)}
{P(A)}
```

로 계산한다.

17. 일반적으로

```math
P(B\mid A)
\neq
P(A\mid B)
```

이다.

18. Classification Model은 입력 Feature가 주어졌을 때 Target Class가 발생할 Conditional Probability를 모델링할 수 있다.

19. Binary Model은

```math
P(Y=1\mid X=x;\theta)
```

와 같은 Probability를 계산할 수 있다.

20. Probability와 Likelihood는 같은 Probability Model을 사용하지만 관점이 다르다.

21. Probability에서는 Parameter를 고정하고 가능한 Data를 생각한다.

22. Likelihood에서는 Observed Data를 고정하고 Parameter를 비교한다.

23. Likelihood를

```math
L(\theta\mid D)
=
P(D\mid\theta)
```

로 표현할 수 있다.

24. Likelihood는 Parameter 자체가 참일 Probability가 아니다.

25. Bernoulli 한 Sample의 Likelihood는 실제 정답이 1이면 \(p\), 0이면 \(1-p\)이다.

26. 여러 Sample이 Parameter가 주어졌을 때 독립이라면 Joint Likelihood는 각 Probability의 곱이다.

27. Bernoulli Dataset의 Likelihood는 다음과 같다.

```math
L(p)
=
\prod_i
p^{y_i}
(1-p)^{1-y_i}
```

28. Success가 \(k\)개이고 전체 Sample이 \(n\)이면

```math
L(p)
=
p^k(1-p)^{n-k}
```

로 표현할 수 있다.

29. MLE는 Likelihood를 최대화하는 Parameter를 찾는 방법이다.

30. MLE는

```math
\hat\theta_{MLE}
=
\arg\max_\theta
L(\theta\mid D)
```

로 표현한다.

31. `argmax`는 최대 함수값이 아니라 그 값을 만드는 Parameter를 의미한다.

32. 동일 Probability를 갖는 Bernoulli Model의 MLE는 Sample Mean이다.

```math
\hat p
=
\frac{1}{n}
\sum_i y_i
```

33. 따라서 Bernoulli Model에서는 관측 성공 비율이 MLE와 일치한다.

34. Logistic Regression에서는 하나의 \(p\)가 아니라 Weight와 Bias를 MLE로 추정한다.

35. 각 Sample의 Probability는

```math
p_i
=
\sigma(x_iw+b)
```

로 계산할 수 있다.

36. Logistic Regression의 Likelihood는

```math
\prod_i
p_i^{y_i}
(1-p_i)^{1-y_i}
```

이다.

37. Model은 실제 Positive Sample에는 높은 \(p_i\), Negative Sample에는 낮은 \(p_i\)를 주도록 Parameter를 학습한다.

38. Training Likelihood가 높다고 해서 Test Performance가 자동으로 보장되지는 않는다.

39. Logarithm은 곱셈을 덧셈으로 바꾸는 성질을 가진다.

```math
\log(ab)
=
\log a+\log b
```

40. Log-Likelihood는 Likelihood에 Log를 적용한 값이다.

```math
\ell(\theta)
=
\log L(\theta)
```

41. Product 형태 Likelihood는 Log-Likelihood에서 Sum으로 변한다.

42. Bernoulli Log-Likelihood는

```math
\ell
=
\sum_i
[
y_i\log p_i
+
(1-y_i)\log(1-p_i)
]
```

이다.

43. Logarithm은 양수 범위에서 단조 증가하므로 Likelihood와 Log-Likelihood는 같은 최적 Parameter를 가진다.

44. 따라서

```math
\arg\max L
=
\arg\max\log L
```

이다.

45. Log-Likelihood를 사용하면 작은 Probability를 반복해서 곱하면서 발생하는 Underflow 문제를 줄일 수 있다.

46. Negative Log-Likelihood는

```math
NLL=-\ell
```

이다.

47. Likelihood Maximum과 NLL Minimum은 같은 Parameter를 선택한다.

48. 따라서

```text
Likelihood 최대화
↔
Log-Likelihood 최대화
↔
NLL 최소화
```

이다.

49. Binary 한 Sample의 NLL은

```math
-
[
y\log p
+
(1-y)\log(1-p)
]
```

이다.

50. 실제 정답이 1이면

```math
NLL=-\log p
```

이다.

51. 실제 정답이 0이면

```math
NLL=-\log(1-p)
```

이다.

52. Model이 실제 정답에 높은 Probability를 부여할수록 NLL은 작아진다.

53. Model이 실제 정답에 매우 낮은 Probability를 부여하면 NLL은 크게 증가한다.

54. NLL은 틀린 Prediction을 강하게 확신하는 경우 큰 Penalty를 준다.

55. 전체 NLL은 Sample별 NLL의 합으로 계산할 수 있다.

56. Mean NLL은 이를 Sample 수로 나눈 평균이다.

57. 고정된 Dataset에서는 Total NLL과 Mean NLL의 최적 Parameter가 동일하다.

58. Accuracy는 최종 Class만 평가하지만 NLL은 Probability까지 평가한다.

59. 따라서 동일한 Accuracy를 가진 두 Model도 NLL은 다를 수 있다.

60. Multiclass Classification에서는 실제 Class에 부여한 Softmax Probability를 Likelihood 계산에 사용한다.

61. Multiclass NLL은

```math
-\sum_k
y_k\log p_k
```

로 표현할 수 있다.

62. One-Hot Target에서는 실제 Class의 Probability 항만 남는다.

63. Binary Bernoulli NLL은 Binary Cross Entropy와 같은 형태로 연결된다.

64. Categorical NLL은 Multiclass Cross Entropy와 연결된다.

65. Regression에서도 Probability Distribution을 가정하면 MLE를 사용할 수 있다.

66. Gaussian Noise를 가정한 Regression에서 고정된 Variance 조건 아래 MLE는 Squared Error 최소화와 연결된다.

67. 따라서 MSE와 Cross Entropy는 서로 완전히 독립적인 공식이 아니라 서로 다른 Probability Model에서 유도할 수 있다.

68. Probability가 정확히 0이면 \(\log0\)을 계산할 수 없다.

69. 따라서 실제 구현에서는 Probability Clipping 또는 Logit 기반 Stable Loss를 사용할 수 있다.

70. 실무에서는 Sigmoid와 BCE를 따로 계산하기보다 Logit에서 직접 안정적인 Loss를 계산하는 구현을 사용할 수 있다.

71. 반복 측정·시계열·센서 데이터에서는 Sample Independence Assumption이 맞는지 확인해야 한다.

72. MLE는 Data Likelihood만 고려한다.

73. MAP는 Likelihood와 Parameter Prior를 함께 고려한다.

74. MLE와 MAP는 같은 개념이 아니다.

75. Bernoulli Sequence의 Likelihood와 Binomial Count Probability는 Combination Term의 유무가 다르다.

76. 하지만 해당 Combination Term이 Parameter와 무관하면 MLE 최적값에는 영향을 주지 않는다.

77. 머신러닝 학습을 확률 관점으로 보면 전체 흐름은 다음과 같다.

```text
Input X

        ↓

Model Parameter θ

        ↓

Prediction Probability

        ↓

Observed Target y

        ↓

Likelihood

        ↓

Log-Likelihood

        ↓

Negative Log-Likelihood

        ↓

Loss

        ↓

Gradient

        ↓

Parameter Update
```

78. 결국 MLE의 핵심은 **관측된 데이터를 가장 그럴듯하게 만드는 Parameter를 선택하는 것**이다.

79. Log-Likelihood는 이 목적을 바꾸지 않으면서 계산을 더 안정적이고 편리하게 만든다.

80. Negative Log-Likelihood는 확률 모델의 Parameter Estimation을 머신러닝에서 사용하는 **Loss Minimization 문제**로 연결하는 핵심 개념이다.


