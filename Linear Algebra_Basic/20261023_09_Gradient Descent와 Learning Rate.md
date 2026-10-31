# **[09. Gradient Descent와 Learning Rate]**

<br>

# **1. Gradient Descent를 왜 사용하는가?**

앞에서 학습한 과정은 다음과 같다.

```text
Input
↓
Forward Propagation
↓
Prediction
↓
Loss
↓
Backpropagation
↓
Gradient
```

Backpropagation까지 수행하면 각 Parameter가 Loss에 어떤 영향을 주는지 나타내는 Gradient를 얻을 수 있다.

하지만 Gradient를 계산했다고 Model Parameter가 자동으로 바뀌는 것은 아니다.

실제 학습을 위해서는 Gradient를 이용하여 Parameter를 변경해야 한다.

이때 사용하는 대표적인 Optimization Method가 **Gradient Descent**이다.

Gradient Descent의 목표는

> **Loss Function을 작게 만드는 Parameter를 반복적으로 찾아가는 것**

이다.

전체 흐름은 다음과 같다.

```text
현재 Parameter
↓
Prediction 계산
↓
Loss 계산
↓
Gradient 계산
↓
Parameter Update
↓
새로운 Parameter
↓
다시 반복
```

<br><br>

---

# **2. Gradient Descent의 핵심 원리**

**Gradient**는 현재 지점에서 함수값이 가장 빠르게 증가하는 국소적인 방향을 나타낸다.

> **경사하강법에서 기울기의 의미**

- 현재 위치에서 함수값이 얼마나 빠르게 증가하는지 나타낸다.   

- 함수값이 증가하는 방향을 나타내므로 기울기는 아래에서 위쪽 방향으로 향하는 것으로 정의가 된다.

- 기울기는 손실이 커지는 방향을 알려준다.

따라서 Loss를 감소시키려면 일반적으로 Gradient의 반대 방향으로 이동한다.

예를 들어, 손실함수를

$$
L(w)=(w-5)^2
$$

라고 하자.

도함수(미분)는

$$
\frac{dL}{dw}=2(w-5)
$$

이다.

| `w` | Loss | Gradient | Loss 감소 방향 |
| ---: | ---: | ---: | --- |
| 2 | 9 | -6 | `w` 증가 |
| 4 | 1 | -2 | `w` 증가 |
| 5 | 0 | 0 | 정지점 |
| 6 | 1 | 2 | `w` 감소 |
| 8 | 9 | 6 | `w` 감소 |

Gradient가 음수라면 Parameter를 증가시키는 방향이 Loss 감소 방향이고,    
Gradient가 양수라면 Parameter를 감소시키는 방향이 된다.

<br><br>

---

# **3. Gradient Descent의 Parameter Update 공식**

기본 업데이트 공식은 다음과 같다.

$$
\theta_{new}
=
\theta_{old}
-
\alpha
\nabla L(\theta_{old})
$$

여기서

- `θ`: Model Parameter

- `α`: Learning Rate

- `∇L`: Loss의 Gradient

이다.

가중치 하나라면

$$
w_{new}
=
w_{old}
-
\alpha
\frac{\partial L}{\partial w}
$$

편향은

$$
b_{new}
=
b_{old}
-
\alpha
\frac{\partial L}{\partial b}
$$

로 업데이트한다.

핵심은 `-` 부호이다.

```text
Gradient
= Loss가 증가하는 방향
-Gradient
= Loss가 감소하는 방향
```

<br><br>

---

# **4. Gradient의 부호와 Parameter 변화**

Learning Rate를 양수라고 가정하면 다음과 같이 해석할 수 있다.

| Gradient | 업데이트 식 | Parameter 변화 |
| --- | --- | --- |
| `Gradient < 0` | `θ - α(-Gradient)` | 증가 |
| `Gradient > 0` | `θ - α(+Gradient)` | 감소 |
| `Gradient = 0` | `θ - 0` | 변화 없음 |

하지만 Gradient가 0이라고 해서 반드시 Global Minimum이라는 뜻은 아니다.

다음과 같은 경우에도 Gradient가 0일 수 있다.

- Local Minimum

- Local Maximum

- Saddle Point

- 평평한 영역

따라서 Gradient 값만으로 최적해를 완전히 판단할 수는 없다.

<br><br>

---

# **5. 스포츠 회귀 모델에서 한 번의 Gradient Descent**

GPS Training Load를 이용하여 다음날 선수의 Fatigue Score를 예측하는 간단한 Regression Model을 생각하자.

Model은 다음과 같다.

$$
\hat y=wx+b
$$

현재 한 선수의 값이 다음과 같다고 하자.

```text
GPS Load Index x = 2
실제 Fatigue Score y = 10
w = 3
b = 1
```

Prediction은

$$
\hat y
=
3\times2+1
=
7
$$

이다.

Squared Error를 사용하면

$$
L=(y-\hat y)^2
$$

이므로

$$
L=(10-7)^2=9
$$

이다.

Gradient는

$$
\frac{\partial L}{\partial w}
=
2(\hat y-y)x
$$

$$
=
2(7-10)\times2
=
-12
$$

이고

$$
\frac{\partial L}{\partial b}
=
2(\hat y-y)
=
-6
$$

이다.

따라서

```text
Gradient = [-12, -6]
```

이다.

<br><br>

---

# **6. Learning Rate를 적용한 Parameter Update**

Learning Rate를

```text
α = 0.05
```

라고 하자.

가중치는

$$
w_{new}
=
3-0.05(-12)
=
3.6
$$

이다.

편향은

$$
b_{new}
=
1-0.05(-6)
=
1.3
$$

이다.

새로운 Prediction은

$$
\hat y
=
3.6\times2+1.3
=
8.5
$$

이고 Loss는

$$
L=(10-8.5)^2
=
2.25
$$

이다.

| 구분 | Update 전 | Update 후 |
| --- | ---: | ---: |
| `w` | 3.0 | 3.6 |
| `b` | 1.0 | 1.3 |
| Prediction | 7.0 | 8.5 |
| Loss | 9.0 | 2.25 |

Gradient를 한 번 적용한 결과 Loss가 감소하였다.

<br><br>

---

# **7. Gradient Descent는 반복해서 수행한다**

한 번 Parameter를 Update했다고 Training이 끝나는 것은 아니다.

새로운 Parameter에서는 Loss Function의 위치가 달라졌기 때문에 **Gradient도 다시 계산해야 한다.**

첫 번째 Update 결과는

```text
w = 3.6
b = 1.3
```

이다.

새로운 Prediction은

```text
ŷ = 8.5
```

이다.

따라서 새로운 Gradient는

$$
\frac{\partial L}{\partial w}
=
2(8.5-10)\times2
=
-6
$$

$$
\frac{\partial L}{\partial b}
=
2(8.5-10)
=
-3
$$

이다.

두 번째 Update는

$$
w
=
3.6-0.05(-6)
=
3.9
$$

$$
b
=
1.3-0.05(-3)
=
1.45
$$

이다.

새로운 Prediction은

$$
\hat y
=
3.9\times2+1.45
=
9.25
$$

이며

$$
L=(10-9.25)^2
=
0.5625
$$

이다.

| Step | `w` | `b` | Prediction | Loss |
| ---: | ---: | ---: | ---: | ---: |
| 0 | 3.000 | 1.000 | 7.000 | 9.0000 |
| 1 | 3.600 | 1.300 | 8.500 | 2.2500 |
| 2 | 3.900 | 1.450 | 9.250 | 0.5625 |

즉 매 Step마다

```text
현재 Parameter
→ Forward
→ Loss
→ Gradient
→ Update
```

를 다시 수행한다.

<br><br>

---

# **8. NumPy로 반복 Gradient Descent 구현하기**

[ Python : code ] "선수 Fatigue Score 회귀 모델의 반복 Gradient Descent"

```python
import numpy as np

# STEP 1. 초기값 설정
x, y = 2.0, 10.0                 # x: 입력값, y: 실제 정답
theta = np.array([3.0, 1.0])    # 학습할 파라미터 [가중치(w), 편향(b)]
learning_rate = 0.05            # 학습률: 가중치와 편향을 얼마나 수정할지 결정


# STEP 2. 순전파: 예측값 계산
def predict(x, w, b):
    return w * x + b            # 선형함수: 예측값 = wx + b


# STEP 3. 손실함수: 예측값과 실제값의 차이 계산
def loss_fn(x, y, w, b):
    y_pred = predict(x, w, b)   # 현재 가중치와 편향으로 예측
    return (y - y_pred) ** 2    # 오차를 제곱하여 손실 계산 (제곱오차)


# STEP 4. 역전파: 가중치와 편향에 대한 기울기 계산
def gradient_fn(x, y, w, b):
    y_pred = predict(x, w, b)   # 예측값 계산
    error = y_pred - y          # 오차 = 예측값 - 실제값

    # 손실함수를 w에 대해 미분 → w가 손실에 미치는 영향
    grad_w = 2 * error * x

    # 손실함수를 b에 대해 미분 → b가 손실에 미치는 영향
    grad_b = 2 * error

    return np.array([grad_w, grad_b])  # [w의 기울기, b의 기울기] 반환


# STEP 5. 경사하강법: 학습 반복
for step in range(6):           # 총 6번 학습 반복
    w, b = theta                # 현재 가중치(w)와 편향(b) 가져오기

    # ① 순전파 및 손실 계산
    loss = loss_fn(x, y, w, b)

    # ② 역전파: 손실을 줄이기 위한 기울기 계산
    grad = gradient_fn(x, y, w, b)

    # ③ 현재 학습 상태 출력
    print(
        f"{step}: w={w:.4f}, "
        f"b={b:.4f}, Loss={loss:.4f}"
    )

    # ④ 경사하강법: 기울기의 반대 방향으로 가중치와 편향 업데이트
    theta = theta - learning_rate * grad
```

[ Result ]

```text
0: w=3.0000, b=1.0000, Loss=9.0000
1: w=3.6000, b=1.3000, Loss=2.2500
2: w=3.9000, b=1.4500, Loss=0.5625
3: w=4.0500, b=1.5250, Loss=0.1406
4: w=4.1250, b=1.5625, Loss=0.0352
5: w=4.1625, b=1.5813, Loss=0.0088
```

Loss가 반복적으로 감소하는 것을 확인할 수 있다.

<br><br>

---

# **9. 반복 학습 코드의 흐름도**

앞의 코드는 다음 순서로 동작한다.

```text
┌──────────────────────────┐
│ Parameter 초기화 w, b    │
└─────────────┬────────────┘
              ↓
┌──────────────────────────┐
│ Forward Propagation      │
│ ŷ = wx + b               │
└─────────────┬────────────┘
              ↓
┌──────────────────────────┐
│ Loss 계산                │
│ L = (y - ŷ)²             │
└─────────────┬────────────┘
              ↓
┌──────────────────────────┐
│ Backpropagation          │
│ dL/dw, dL/db             │
└─────────────┬────────────┘
              ↓
┌──────────────────────────┐
│ Gradient Descent         │
│ θ = θ - α∇L              │
└─────────────┬────────────┘
              ↓
        다음 Step 반복
```

각 단계의 역할을 구분하는 것이 중요하다.

<br><br>

---

# **10. Forward·Backpropagation·Gradient Descent 비교**

| 과정 | 역할 | 결과 |
| --- | --- | --- |
| Forward Propagation | Prediction 계산 | `ŷ` |
| Loss Calculation | Prediction 평가 | `L` |
| Backpropagation | Gradient 계산 | `∂L/∂θ` |
| Gradient Descent | Parameter Update | 새로운 `θ` |

즉

```text
Backpropagation
≠
Gradient Descent
```

이다.

Backpropagation은 **Gradient를 구하는 알고리즘**이고 Gradient Descent는 **그 Gradient를 이용하여 Parameter를 변경하는 Optimization 방법**이다.

<br><br>

---

# **11. Learning Rate란 무엇인가?**

Learning Rate는 Parameter를 한 번 Update할 때 얼마나 이동할지를 조절하는 Hyperparameter이다.

업데이트 공식은

$$
\theta_{new}
=
\theta_{old}
-
\alpha\nabla L
$$

이다.

따라서 실제 Parameter 변화량은

$$
\Delta\theta
=
-\alpha\nabla L
$$

이다.

즉 업데이트 크기는 Learning Rate만으로 결정되지 않는다.

$$
|\Delta\theta|
=
|\alpha\nabla L|
$$

이므로 Gradient의 크기도 함께 영향을 준다.

<br><br>

---

# **12. Learning Rate가 작은 경우**

다음 Loss Function을 사용하자.

$$
L(w)=(10-2w)^2
$$

도함수는

$$
\frac{dL}{dw}=8w-40
$$

이다.

초기값은

```text
w = 3
```

이다.

현재 Gradient는

$$
8(3)-40=-16
$$

이다.

Learning Rate가 `0.01`이라면

$$
w_{new}
=
3-0.01(-16)
=
3.16
$$

이다.

Loss는

$$
(10-2(3.16))^2
=
13.5424
$$

이다.

기존 Loss는 `16`이었으므로 감소했지만 이동량이 작다.

Learning Rate가 지나치게 작으면

- Training이 느려질 수 있고

- 많은 Iteration이 필요하며

- 제한된 Epoch에서는 충분히 학습되지 못할 수 있다.

<br><br>

---

# **13. 적절한 Learning Rate**

같은 조건에서

```text
Learning Rate = 0.05
```

라고 하자.

첫 Update는

$$
w
=
3-0.05(-16)
=
3.8
$$

이다.

Loss는

$$
L(3.8)=5.76
$$

이다.

다시 Gradient를 계산하면

$$
8(3.8)-40
=
-9.6
$$

이다.

두 번째 Update는

$$
w
=
3.8-0.05(-9.6)
=
4.28
$$

이고 Loss는

$$
L(4.28)=2.0736
$$

이다.

| Step | `w` | Loss |
| ---: | ---: | ---: |
| 0 | 3.00 | 16.0000 |
| 1 | 3.80 | 5.7600 |
| 2 | 4.28 | 2.0736 |

이 예제에서는 비교적 안정적으로 Minimum에 접근한다.

<br><br>

---

# **14. Overshooting(오버슈팅)**

Learning Rate가 커지면 Parameter가 Minimum을 지나 반대쪽으로 이동할 수 있다.

또한 학습률이 커지면 한 번의 업데이트에서 파라미터가 최솟값을 지나칠 수도 있다.

이를 **Overshooting(오버슈팅)** 이라고 한다.

예를 들어

```text
Learning Rate = 0.20
```

이면

$$
w
=
3-0.2(-16)
=
6.2
$$

이다.

실제 Minimum은

```text
w = 5
```

이므로 6.2까지 지나갔다.

하지만 Loss는

$$
L(6.2)
=
5.76
$$

이다.

기존 Loss `16`보다 작다.

따라서

> Overshooting이 발생했다고 해서 반드시 Loss가 증가하는 것은 아니다.

Minimum을 지나갔더라도 이전 위치보다 좋은 위치에 도달할 수 있다.

<br><br>

---

# **15. Oscillation(진동)**

Learning Rate가 더 커지면 Minimum 주변을 반복적으로 오갈 수 있다.

예를 들어

```text
Learning Rate = 0.25
```

를 적용하자.

초기값

```text
w = 3
```

에서

$$
w_{new}
=
3-0.25(-16)
=
7
$$

이다.

다음 Gradient는

$$
8(7)-40=16
$$

이다.

다음 Update는

$$
7-0.25(16)
=
3
$$

이다.

따라서

```text
3 → 7 → 3 → 7 → 3 ...
```

처럼 반복된다.

Loss 역시 계속

```text
16
```

으로 유지된다.

Minimum에 접근하지 못하고 주변을 반복해서 오가는 현상을 **Oscillation(진동)** 이라고 한다.

<br><br>

---

# **16. Divergence(발산)**

Learning Rate가 지나치게 크면 Loss가 점점 증가할 수 있다.

예를 들어

```text
Learning Rate = 0.30
```

이라고 하자.

| Step | `w` | Loss |
| ---: | ---: | ---: |
| 0 | 3.000 | 16.0000 |
| 1 | 7.800 | 31.3600 |
| 2 | 1.080 | 61.4656 |
| 3 | 10.488 | 120.4726 |

Parameter가 Minimum에서 점점 멀어지고 Loss도 증가한다.

이를 **Divergence(발산)** 라고 한다.

<br><br>

---

# **17. Learning Rate에 따른 차이**

| Learning Rate | Training 특성 |
| --- | --- |
| 너무 작음 | 안정적일 수 있지만 매우 느림 |
| 적절함 | 비교적 빠르고 안정적인 수렴 |
| 조금 큼 | Overshooting 가능 |
| 너무 큼 | Oscillation 가능 |
| 매우 큼 | Divergence 가능 |

따라서

```text
Learning Rate가 크다
=
무조건 학습이 빠르다
```

는 잘못된 해석이다.

<br><br>

---

# **18. Learning Rate 표기법**

Machine Learning에서는 작은 Learning Rate를 Scientific Notation으로 자주 표현한다.

| 표현 | 실제 값 |
| --- | ---: |
| `1e-1` | 0.1 |
| `5e-2` | 0.05 |
| `1e-2` | 0.01 |
| `1e-3` | 0.001 |
| `5e-4` | 0.0005 |
| `1e-4` | 0.0001 |
| `5e-5` | 0.00005 |

예를 들어

```python
learning_rate = 5e-5
```

는

```python
learning_rate = 0.00005
```

와 같다.

<br><br>

---

# **19. Learning Rate별 Loss를 NumPy로 비교하기**

[ Python : code ] "서로 다른 Learning Rate에서 Gradient Descent의 안정성 비교"

```python
import numpy as np

# STEP 1. 손실함수 정의
# 예측값(2w)과 실제값(10)의 차이를 제곱하여 손실 계산
def loss_fn(w):
    return (10 - 2 * w) ** 2


# STEP 2. 기울기 계산
# 손실함수를 w에 대해 미분 → 기울기 = 8w - 40
# w를 변화시켰을 때 손실이 얼마나 변하는지 계산
def gradient_fn(w):
    return 8 * w - 40


# STEP 3. 학습률 설정
# 학습률에 따라 손실이 어떻게 달라지는지 비교
learning_rates = [0.01, 0.05, 0.20, 0.25, 0.30]


# STEP 4. 학습률별 경사하강법 실행
for lr in learning_rates:
    w = 3.0                   # 모든 학습률에서 동일한 초기 가중치 사용
    losses = [loss_fn(w)]     # 학습 전 초기 손실 저장

    # STEP 5. 경사하강법 3번 반복
    for _ in range(3):
        grad = gradient_fn(w)  # ① 현재 w에서 기울기 계산

        w -= lr * grad         # ② 기울기의 반대 방향으로 w 업데이트

        losses.append(loss_fn(w))  # ③ 업데이트 후 손실 저장

    # STEP 6. 학습률별 손실 변화 출력
    print(
        f"LR={lr:.2f}:",
        np.round(losses, 4)    # 손실값을 소수점 넷째 자리까지 반올림
    )
```

[ Result ]

```text
LR=0.01: [16.     13.5424 11.4623  9.7017]
LR=0.05: [16.      5.76    2.0736  0.7465]
LR=0.20: [16.      5.76    2.0736  0.7465]
LR=0.25: [16. 16. 16. 16.]
LR=0.30: [ 16.      31.36    61.4656 120.4726]
```

이 예제에서는 Learning Rate에 따라

```text
느린 감소
안정적 감소
진동
발산
```

이 모두 발생할 수 있음을 확인할 수 있다.

<br><br>

---

# **20. Feature Scale도 Gradient에 영향을 준다**

실무 데이터에서는 Feature Scale이 매우 다를 수 있다.

예를 들어 축구 GPS Dataset이 다음과 같다고 하자.

| 변수 | 일반적인 Scale 예시 |
| --- | ---: |
| `total_distance_m` | 8,000~12,000 |
| `max_speed_kmh` | 25~36 |
| `player_load_au` | 300~700 |
| `sprint_count` | 5~40 |

Scale 차이가 매우 크면 Parameter별 Gradient 크기도 크게 달라질 수 있다.

이 때문에 Optimization이 불안정하거나 특정 방향으로만 크게 움직일 수 있다.

따라서 실제 Model에서는

- Standardization

- Normalization

- 적절한 Initialization

등이 Optimization Stability에 영향을 줄 수 있다.

<br><br>

---
# 경사하강법의 종류

<br>

# **21. Batch Gradient Descent <br> (배치 경사하강법)**

Training Sample이 `N`개라고 하자.

Batch Gradient Descent는 **전체 Training Dataset을 사용해서 한 번의 Gradient를 계산**한다.

$$
\nabla L
=
\frac{1}{N}
\sum_{i=1}^{N}
\nabla L_i
$$

이다.

전체 Dataset이 10,000개라면

```text
10,000개 Forward
↓
전체 Loss
↓
전체 Gradient
↓
Parameter Update 1번
```

이 된다.

장점은 같은 Parameter 상태에서 전체 Dataset의 평균 Gradient를 사용한다는 것이다.

하지만 Dataset이 매우 크면

- Memory 사용량 증가

- Update까지 시간이 오래 걸림

- 대규모 Neural Network에서 비효율적일 수 있음

등의 문제가 발생할 수 있다.




<br><br>

---

# **22. Stochastic Gradient Descent(SGD) <br> (확률적 경사하강법)**

이론적인 SGD는 Sample 하나를 이용하여 Gradient를 계산하고 바로 Parameter를 Update한다.

Batch Size는

```text
1
```

이다.

Dataset이 100개라면 1 Epoch에서

```text
100번 Gradient 계산
100번 Parameter Update
```

가 발생한다.

Gradient는 개별 Sample에 크게 영향을 받기 때문에 Update 방향의 변동성이 크다.

장점은

- 빠르게 Update 가능

- Memory 사용량이 작음

등이 있다.

반면

- Gradient Noise가 큼

- Loss가 많이 흔들릴 수 있음

등의 특징이 있다.

<br><br>

---

# **23. Mini-Batch Gradient Descent <br> (미니 배치 경사하강법)**

Mini-Batch Gradient Descent는 여러 Sample을 작은 묶음으로 만들어 Gradient를 계산한다.

예를 들어

```text
Training Data = 1,000
Batch Size = 100
```

이라면

```text
Batch 1: 1~100
Batch 2: 101~200
...
Batch 10: 901~1000
```

으로 나누어 학습할 수 있다.

각 Batch마다

```text
Forward
↓
Loss
↓
Backward
↓
Update
```

를 수행한다.

따라서 1 Epoch에 Parameter가 10번 Update된다.

실제 Deep Learning에서 가장 일반적으로 사용되는 방식이다.

<br><br>

---

# **24. Batch GD·SGD·Mini-Batch GD 비교**

| 구분 | **Batch GD** | **SGD** | **Mini-Batch** |
| --- | --- | --- | --- |
| Batch Size | 전체 Dataset | 1 | 중간 크기 |
| 1 Epoch Update 수 | 적음 | 많음 | 중간 |
| Gradient 변동 | 작음 | 큼 | 중간 |
| Memory 요구 | 큼 | 작음 | 조절 가능 |
| 병렬 계산 | 가능 | 제한적 | 효율적 |
| Deep Learning 활용 | 제한적 | 개념적 | 매우 일반적 |

실무에서는 `SGD Optimizer`라고 부르면서 실제 입력은 Mini-Batch를 사용하는 경우도 많다.

즉 이름이 SGD라고 해서 반드시 Batch Size가 1이라는 뜻으로 사용되는 것은 아니다.


> **예제 :**   
> 학생 1,000명의 데이터를 이용해서 합격 여부를 예측하는 DNN을 학습한다고 가정해본다.

> **1. 배치 경사하강법(Batch GD)**   
- 1,000명의 데이터를 모두 사용하여 평균 기울기를 계산하고 가중치를 1번 업데이트함.

- 장점 : 업데이트가 안정적인 편   
- 단점 : 데이터가 많으면 계산과 메모리 부담이 큼

> **2. 확률적 경사하강법**
- 학생 1명의 데이터로 기울기를 계산하고 가중치를 한 번 업데이트한다.

- 특징 :   
        - 업데이트가 매우 빈번함   
        - 기울기의 변동이 커서 진동할 수 있음

> **3. 미니배치 경사하강법**
- 학생 100명씩 묶어서 평균 기울기를 계산하고 업데이트한다.

- 특징 :    
        - 계산 효율과 안정성 사이의 균형   
        - 주로 딥러닝에서 널리 사용하는 방식 


<img src="https://blog.kakaocdn.net/dna/nigG1/btr1QyBOkoX/AAAAAAAAAAAAAAAAAAAAALDoVK6sstdmmoGxjJE7-I0AMtoXTQ7jSdApJTFIM--S/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1793458799&allow_ip=&allow_referer=&signature=Y2FdhsU9P1IeHTF6lC51iMmjtUM%3D" width="400">


<br><br>

---

# **25. 같은 Dataset이어도 Batch Size에 따라 Update 횟수가 달라진다**

Training Dataset이

```text
100개
```

라고 하자.

| 방법 | Batch Size | 1 Epoch Update |
| --- | ---: | ---: |
| SGD | 1 | 100 |
| Mini-Batch | 20 | 5 |
| Mini-Batch | 32 | 4 |
| Mini-Batch | 50 | 2 |
| Batch GD | 100 | 1 |

Batch Size가 32일 때는

```text
32 + 32 + 32 + 4
```

로 네 개의 Batch가 만들어진다.

따라서 마지막 불완전 Batch까지 사용하는 경우 Update는 4번이다.

<br><br>

---

# **26. Epoch이란 무엇인가?**

Epoch은 **전체 Training Dataset을 한 번 학습에 사용한 상태**를 의미한다.

예를 들어 Training Dataset이 1,000개라면

```text
1 Epoch
=
1,000개 Sample을 한 번씩 학습에 사용
```

한다는 의미이다.

5 Epoch이면 전체 Dataset을 총 다섯 번 반복하여 사용한다.

단, 각 Epoch에서 Sample 순서는 Shuffle될 수 있다.

<br><br>

---

# **27. Iteration이란 무엇인가?**

Iteration은 **한 Batch에 대해 Parameter Update를 한 번 수행한 것**을 의미한다.

즉

```text
1 Iteration
=
1 Batch Forward
+
Loss
+
Backward
+
Parameter Update
```

이다.

따라서 Batch Size에 따라 한 Epoch의 Iteration 수가 달라진다.

<br><br>

---

# **28. Epoch과 Iteration의 관계**

마지막 Batch를 버리지 않는다고 가정하면

$$
Iterations\ per\ Epoch
=
\left\lceil
\frac{N}{BatchSize}
\right\rceil
$$

이다.

전체 Training 동안 Update 횟수는

$$
Total\ Iterations
=
Epochs
\times
Iterations\ per\ Epoch
$$

이다.

예를 들어

```text
Dataset = 1,000
Batch Size = 128
Epoch = 10
```

이면

$$
\left\lceil
\frac{1000}{128}
\right\rceil
=
8
$$

이므로

```text
1 Epoch = 8 Iterations
```

이고

```text
10 Epoch = 80 Updates
```

이다.

<img src="https://blog.kakaocdn.net/dna/BHLeA/btsaJVbaFdA/AAAAAAAAAAAAAAAAAAAAAGKTsgvhzshvFL4UCBDK3kAsvKBWzcu0el52l_u0YZXX/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1793458799&allow_ip=&allow_referer=&signature=Uj7PmKFnNKBk0xR46s%2Fp59DOHes%3D" width="400">


<br><br>

---

# **29. Batch Size별 Iteration 계산**

| Dataset | Batch Size | Iteration / Epoch |
| ---: | ---: | ---: |
| 1,000 | 1 | 1,000 |
| 1,000 | 10 | 100 |
| 1,000 | 32 | 32 |
| 1,000 | 100 | 10 |
| 1,000 | 128 | 8 |
| 1,000 | 1,000 | 1 |

`1000 / 32 = 31.25`이므로 마지막 Batch까지 사용하면 32번 Update한다.

다만 Training 설정에서 마지막 불완전 Batch를 버리는 `drop_last`와 같은 설정을 사용하면 실제 Iteration 수는 달라질 수 있다.

<br><br>

---

# **30. NumPy로 Iteration 수 계산하기**

[ Python : code ] "Dataset Size와 Batch Size에 따른 Iteration 계산"

```python
import numpy as np

# STEP 1. 전체 데이터 개수 설정
n_samples = 1000  # 전체 학습 데이터 1000개

# STEP 2. 배치 크기별 반복 횟수 비교
for batch_size in [1, 10, 32, 100, 128]:

    # 전체 데이터를 배치 크기로 나누어 반복 횟수 계산
    # ceil(): 소수점이 나오면 올림 (남은 데이터도 한 번 처리)
    iterations = int(np.ceil(n_samples / batch_size))

    # STEP 3. 결과 출력
    print(
        f"Batch Size={batch_size}, "
        f"Iterations={iterations}"
    )
```

[ Result ]

```text
Batch Size=1, Iterations=1000
Batch Size=10, Iterations=100
Batch Size=32, Iterations=32
Batch Size=100, Iterations=10
Batch Size=128, Iterations=8
```

<br><br>

---

# **31. 스포츠 GPS 데이터 Mini-Batch 학습 예제**

이번에는 선수 Training Load와 다음날 Fatigue Score의 관계를 간단한 Linear Regression으로 학습하자.

실무에서는 GPS Load 자체의 Scale이 크기 때문에 전처리된 `gps_load_z`를 Feature로 사용한다고 가정한다.

| 선수 Session | `gps_load_z` | 다음날 Fatigue Score |
| --- | ---: | ---: |
| 1 | -1.5 | 3.2 |
| 2 | -0.5 | 4.4 |
| 3 | 0.5 | 5.7 |
| 4 | 1.5 | 7.0 |

Model은

$$
\hat y=wx+b
$$

이다.

초기 Parameter는

```text
w = 0
b = 5
```

로 설정한다.

<br><br>

---

# **32. 전체 Dataset으로 Gradient 계산하기**

MSE는

$$
MSE
=
\frac{1}{N}
\sum_{i=1}^{N}
(y_i-\hat y_i)^2
$$

이다.

Gradient는

$$
\frac{\partial L}{\partial w}
=
\frac{2}{N}
\sum_{i=1}^{N}
(\hat y_i-y_i)x_i
$$

$$
\frac{\partial L}{\partial b}
=
\frac{2}{N}
\sum_{i=1}^{N}
(\hat y_i-y_i)
$$

이다.

초기 Parameter

```text
w = 0
b = 5
```

에서 계산하면

```text
MSE = 2.0225
Gradient w = -3.175
Gradient b = -0.150
```

이다.

Learning Rate가 `0.1`이라면

```text
w_new = 0 - 0.1(-3.175)
      = 0.3175
b_new = 5 - 0.1(-0.15)
      = 5.015
```

가 된다.

<br><br>

---

# **33. 실무형 Regression Training Loop**

[ Python : code ] "GPS Load를 이용한 다음날 Fatigue Score Gradient Descent 학습"

```python
import numpy as np

# STEP 1. 학습 데이터 준비
gps_load_z = np.array([-1.5, -0.5, 0.5, 1.5])  # 입력값(x): 표준화된 GPS 운동 부하
fatigue_score = np.array([3.2, 4.4, 5.7, 7.0]) # 실제값(y): 피로도 점수

# STEP 2. 초기값 설정
w = 0.0               # 가중치: GPS 부하가 피로도에 미치는 영향
b = 5.0               # 편향: GPS 부하가 0일 때 예측 피로도
learning_rate = 0.1   # 학습률: 가중치와 편향을 얼마나 수정할지 결정


# STEP 3. 순전파: 예측값 계산
def forward(x, w, b):
    return w * x + b  # 선형함수: 예측 피로도 = wx + b


# STEP 4. 손실함수: 예측값과 실제값의 차이 계산
def mse_loss(y_true, y_pred):
    return np.mean(
        (y_true - y_pred) ** 2  # 오차를 제곱
    )                          # 전체 오차 제곱의 평균(MSE) 계산


# STEP 5. 기울기 계산: 가중치와 편향에 대한 손실의 미분값
def gradients(x, y_true, y_pred):
    error = y_pred - y_true          # 오차 = 예측값 - 실제값

    grad_w = np.mean(2 * error * x)  # w에 대한 손실의 기울기
    grad_b = np.mean(2 * error)      # b에 대한 손실의 기울기

    return grad_w, grad_b            # 계산한 기울기 반환


# STEP 6. 경사하강법: 총 6번 학습 반복
for step in range(6):

    # ① 순전파: 현재 w, b로 피로도 예측
    y_pred = forward(gps_load_z, w, b)

    # ② 손실 계산: 실제 피로도와 예측 피로도의 차이
    loss = mse_loss(fatigue_score, y_pred)

    # ③ 역전파 원리: 손실을 w, b에 대해 미분하여 기울기 계산
    grad_w, grad_b = gradients(
        gps_load_z,
        fatigue_score,
        y_pred
    )

    # ④ 현재 학습 상태 출력 (업데이트 전)
    print(
        f"Step {step}: "
        f"w={w:.4f}, "
        f"b={b:.4f}, "
        f"MSE={loss:.4f}"
    )

    # ⑤ 경사하강법: 손실이 줄어드는 방향으로 w, b 업데이트
    w -= learning_rate * grad_w
    b -= learning_rate * grad_b
```

[ Result ]

```text
Step 0: w=0.0000, b=5.0000, MSE=2.0225
Step 1: w=0.3175, b=5.0150, MSE=1.1384
Step 2: w=0.5556, b=5.0270, MSE=0.6410
Step 3: w=0.7342, b=5.0366, MSE=0.3611
Step 4: w=0.8682, b=5.0443, MSE=0.2035
Step 5: w=0.9686, b=5.0504, MSE=0.1149
```

반복적으로 Gradient Descent를 수행하면서 MSE가 감소한다.

<br><br>

---

# **34. Training Loop 코드 결과 해석**

앞의 결과를 보면

```text
MSE
2.0225
↓
1.1384
↓
0.6410
↓
0.3611
↓
0.2035
↓
0.1149
```

로 감소한다.

동시에 `w`는

```text
0
↓
0.3175
↓
0.5556
↓
0.7342
↓
0.8682
↓
0.9686
```

로 변경된다.

즉 Model이 GPS Load가 증가할수록 다음날 Fatigue Score도 증가하는 방향의 관계를 학습하고 있다.

다만 이 Dataset은 Gradient Descent 원리를 설명하기 위한 Synthetic Example이며 실제 선수 피로 Prediction에서는 훨씬 많은 Feature와 Validation 과정이 필요하다.

<br><br>

---

# **35. Training Loop의 상세 흐름도**

```text
            Training Data
                 │
                 ↓
        ┌────────────────┐
        │ Mini-Batch 선택 │
        └───────┬────────┘
                ↓
        ┌────────────────┐
        │ Forward Pass   │
        │ ŷ = f(X; θ)    │
        └───────┬────────┘
                ↓
        ┌────────────────┐
        │ Loss 계산      │
        │ L(y, ŷ)        │
        └───────┬────────┘
                ↓
        ┌────────────────┐
        │ Backward Pass  │
        │ ∇θL 계산       │
        └───────┬────────┘
                ↓
        ┌────────────────┐
        │ Optimizer      │
        │ θ ← θ - α∇L    │
        └───────┬────────┘
                ↓
        다음 Mini-Batch
                │
                ↓
        모든 Batch 완료
                │
                ↓
             1 Epoch
                │
                ↓
        다음 Epoch 반복
```

<br><br>

---

# **36. Batch Size의 실무적 의미**

Batch Size는 단순히 Update 횟수만 변경하는 것이 아니다.

| 작은 Batch | 큰 Batch |
| --- | --- |
| Gradient Noise가 상대적으로 큼 | Gradient가 상대적으로 안정적 |
| Update 횟수가 많음 | Update 횟수가 적음 |
| Memory 요구량이 작음 | Memory 요구량이 큼 |
| GPU 활용률이 낮을 수 있음 | 병렬 연산 활용 가능 |
| 더 자주 Parameter Update | Update 간격이 길어짐 |

따라서 Batch Size는

```text
Memory
+
계산 속도
+
Gradient 특성
+
Model Generalization
```

등에 영향을 줄 수 있는 Hyperparameter이다.

<br><br>

---

# **37. Mini-Batch에서는 Shuffling이 중요하다**

Dataset이 특정 순서로 정렬되어 있다면 Batch가 편향될 수 있다.

예를 들어 Football GPS Dataset이

```text
GK
GK
GK
Defender
Defender
...
Forward
Forward
```

순으로 정렬되어 있다고 하자.

그 상태에서 작은 Mini-Batch를 사용하면 특정 Position만 포함된 Batch가 만들어질 수 있다.

일반적인 독립 Sample Dataset에서는 Epoch마다 Training Data를 Shuffle하여 다양한 Sample 조합으로 Batch를 만드는 방식을 자주 사용한다.

단,

- Time Series

- Sequence Model

- 시계열 종속성이 중요한 Dataset

에서는 무조건 Shuffle해서는 안 된다.

<br><br>

---

# **38. Convergence(수렴)란 무엇인가?**

Convergence는 Training을 반복하면서 Loss 또는 Parameter가 일정한 값에 가까워지는 현상을 의미한다.

예를 들어

$$
L(w)=(w-5)^2
$$

이고 초기값이

```text
w = 1
```

Learning Rate가

```text
0.1
```

이라고 하자.

| Step | `w` | Loss |
| ---: | ---: | ---: |
| 0 | 1.0000 | 16.0000 |
| 1 | 1.8000 | 10.2400 |
| 2 | 2.4400 | 6.5536 |
| 3 | 2.9520 | 4.1943 |
| 4 | 3.3616 | 2.6844 |

Parameter가 `5`에 가까워지고 Loss가 `0`에 가까워진다.

이러한 상태를 **수렴하고 있다**고 표현할 수 있다.


<br><br>

---

# **39. Convergence와 Global Minimum은 다르다**

Loss가 더 이상 크게 감소하지 않는다고 해서 반드시 최적의 Model을 찾았다는 의미는 아니다.

특히 복잡한 Neural Network에서는

- Local Minimum

- Saddle Point

- Flat Region

- Poor Learning Rate

- Gradient Vanishing

등 다양한 상황이 존재할 수 있다.

따라서

```text
Loss가 멈춤
=
반드시 Global Minimum
```

은 아니다.

<br><br>

---

# **40. Training Loss만 보면 안 되는 이유**

Training Loss가 계속 감소해도 Validation Performance는 나빠질 수 있다.

예를 들어 다음과 같은 상황이 가능하다.

```text
Epoch
  │
  ├── Training Loss ↓
  │
  └── Validation Loss ↓
              ↓
          어느 시점
              ↓
      Training Loss 계속 ↓
      Validation Loss 다시 ↑
```

이는 Overfitting의 신호일 수 있다.

따라서 실제 학습에서는

- Training Loss

- Validation Loss

- Validation Metric

을 함께 확인하는 것이 중요하다.

<br><br>

---

# **41. Loss Curve로 학습 상태 판단하기**

| 현상 | 가능한 해석 |
| --- | --- |
| Loss가 안정적으로 감소 | 정상 학습 가능성 |
| Loss가 매우 느리게 감소 | Learning Rate가 작을 가능성 |
| Loss가 크게 위아래로 움직임 | LR이 크거나 Mini-Batch Noise 영향 |
| Loss가 급격히 증가 | Divergence 가능성 |
| Training Loss만 감소 | Overfitting 가능성 |
| Train/Validation 모두 정체 | Optimization 또는 Model Capacity 점검 |

하나의 현상만으로 원인을 확정하면 안 된다.

실제 원인은

- Data

- Scaling

- Initialization

- Model Architecture

- Batch Size

- Optimizer

- Learning Rate

등 여러 요소가 결합되어 나타날 수 있다.

<br><br>

---

# **42. Epoch이 많으면 항상 좋은가?**

Epoch 수를 증가시키면 Training Dataset을 더 많이 반복하게 된다.

그러나 너무 많은 Epoch은 Overfitting으로 이어질 수 있다.

| Epoch 상태 | 가능성 |
| --- | --- |
| 너무 적음 | Underfitting |
| 적절함 | 충분한 학습 |
| 지나치게 많음 | Overfitting 가능 |

따라서 단순히 최대한 많은 Epoch을 사용하는 것이 아니라 Validation Performance를 함께 확인해야 한다.

<br><br>

---

# **43. Gradient Descent 전체 학습 구조**

전체 Machine Learning Training Process는 다음과 같다.

```text
1. Training Data 준비
        ↓
2. Parameter 초기화
        ↓
3. Mini-Batch 구성
        ↓
4. Forward Propagation
        ↓
5. Prediction 계산
        ↓
6. Loss 계산
        ↓
7. Backpropagation
        ↓
8. Gradient 계산
        ↓
9. Optimizer가 Parameter Update
        ↓
10. 다음 Batch
        ↓
11. 전체 Batch 완료
        ↓
12. 1 Epoch 완료
        ↓
13. Validation 평가
        ↓
14. 다음 Epoch
```

<br><br>

---

# **44. 지금까지 배운 개념의 연결**

이전 강의들과 연결하면 전체 흐름은 다음과 같다.

```text
Regression / Classification
        ↓
Prediction
        ↓
MSE / Cross Entropy
        ↓
Loss Function
        ↓
Differentiation
        ↓
Gradient
        ↓
Chain Rule
        ↓
Backpropagation
        ↓
Gradient Descent
        ↓
Parameter Update
        ↓
Learning
```

즉 지금까지 각각 분리되어 보였던 수학 개념들이 실제 Model Training에서는 하나의 과정으로 연결된다.

<br><br>

---

# **45. 자주 혼동하는 개념**

> ## **45-1) Gradient Descent 한 번 = Epoch 한 번인가?**

아니다.

Mini-Batch Training이라면 하나의 Batch에 대한 Update가 보통 한 Iteration이다.

여러 Iteration이 모여 하나의 Epoch이 된다.

<br>

> ## **45-2) Backpropagation이 Parameter를 바꾸는가?**

아니다.

Backpropagation은 Gradient를 계산한다.

Parameter를 실제로 변경하는 것은 Optimizer의 역할이다.

<br>

> ## **45-3) Learning Rate가 작으면 무조건 안전한가?**

반드시 그렇지는 않다.

너무 작으면 제한된 Training 시간 안에 충분히 학습하지 못할 수 있다.

<br>

> ## **45-4) Loss는 매 Iteration마다 반드시 감소하는가?**

아니다.

특히 Mini-Batch나 SGD에서는 Batch가 달라지면서 Loss가 일시적으로 증가할 수 있다.

전체 추세를 확인하는 것이 중요하다.

<br>

> ## **45-5) Batch Size를 두 배로 하면 Learning Rate도 무조건 두 배로 해야 하는가?**

항상 그런 것은 아니다.

Batch Size와 Learning Rate 사이에는 실무적인 조정 전략이 존재하지만 Model과 Optimizer에 따라 달라진다.

<br><br>

---

# **46. Advanced 추가 학습**

> ## **46-1) Momentum**

이전 Gradient의 이동 방향을 일부 누적하여 Optimization을 더 빠르고 안정적으로 진행하도록 돕는 방법이다.

<br>

> ## **46-2) Learning Rate Scheduler**

Training이 진행됨에 따라 Learning Rate를 변화시키는 방법이다.

예를 들어 처음에는 크게 학습하고 후반에는 Learning Rate를 줄일 수 있다.

<br>

> ## **46-3) Adam Optimizer**

Gradient뿐 아니라 Gradient의 1차·2차 통계량을 이용하여 Parameter별 Update 크기를 조절하는 Optimizer이다.

<br>

> ## **46-4) Early Stopping**

Validation Performance가 일정 기간 개선되지 않으면 Training을 중단하여 불필요한 학습과 Overfitting을 줄이는 방법이다.

<br>

> ## **46-5) Gradient Clipping**

Gradient가 지나치게 커지는 상황에서 Gradient 크기를 제한하여 Training을 안정화하는 방법이다.

<br><br>

---

# **47. 핵심정리**

1. Optimization은 Loss Function을 작게 만드는 Model Parameter를 찾는 과정이다.

2. Gradient Descent는 Gradient를 이용하여 Parameter를 반복적으로 Update하는 대표적인 Optimization 방법이다.

3. 기본 Update 공식은

$$
\theta_{new}
=
\theta_{old}
-
\alpha\nabla L(\theta_{old})
$$

이다.

4. Gradient는 현재 지점에서 Loss가 가장 빠르게 증가하는 국소적인 방향을 나타낸다.

5. 따라서 Loss를 줄이기 위해 일반적으로 Negative Gradient 방향으로 이동한다.

6. Gradient가 음수이면 기본 Gradient Descent에서는 Parameter가 증가한다.

7. Gradient가 양수이면 Parameter가 감소한다.

8. Gradient가 0이면 기본 Update에서는 Parameter가 움직이지 않는다.

9. Gradient가 0이라고 해서 반드시 Global Minimum이라는 의미는 아니다.

10. Backpropagation과 Gradient Descent는 서로 다른 과정이다.

11. Backpropagation은 각 Parameter의 Gradient를 계산한다.

12. Gradient Descent는 계산된 Gradient를 이용하여 Parameter를 실제로 변경한다.

13. 전체 Training의 기본 흐름은 다음과 같다.

```text
Forward
↓
Loss
↓
Backward
↓
Gradient
↓
Parameter Update
```

14. Parameter가 Update되면 새로운 위치에서 Gradient를 다시 계산해야 한다.

15. Learning Rate는 Gradient를 이용한 Parameter Update 크기를 조절한다.

16. 실제 Update 크기는

$$
|\alpha\nabla L|
$$

에 의해 결정되므로 Learning Rate와 Gradient 크기를 함께 고려해야 한다.

17. Learning Rate가 너무 작으면 Training이 매우 느릴 수 있다.

18. Learning Rate가 적절하면 Loss가 비교적 빠르고 안정적으로 감소할 수 있다.

19. Learning Rate가 크면 Minimum을 지나가는 Overshooting이 발생할 수 있다.

20. Overshooting이 발생했다고 해서 반드시 Loss가 증가하는 것은 아니다.

21. Learning Rate가 지나치게 크면 Parameter가 Minimum 주변을 반복하는 Oscillation이 발생할 수 있다.

22. 더 큰 Learning Rate에서는 Loss가 점점 증가하는 Divergence가 발생할 수 있다.

23. Learning Rate는 `1e-3`, `5e-5`와 같은 Scientific Notation으로 자주 표현한다.

24. Feature Scale은 Gradient 크기와 Optimization Stability에 영향을 줄 수 있다.

25. Gradient Descent는 Gradient 계산에 사용하는 데이터 범위에 따라 구분할 수 있다.

26. Batch Gradient Descent는 전체 Training Dataset을 이용하여 Gradient를 계산한다.

27. 이론적인 SGD는 Sample 하나마다 Gradient를 계산하고 Parameter를 Update한다.

28. Mini-Batch Gradient Descent는 일부 Sample 묶음마다 Gradient를 계산한다.

29. 실제 Deep Learning에서는 Mini-Batch 방식이 매우 일반적이다.

30. Batch Size는 한 번의 Forward·Backward에 사용하는 Sample 수이다.

31. Batch Size가 작으면 일반적으로 한 Epoch의 Parameter Update 횟수가 증가한다.

32. Batch Size가 커지면 한 Epoch의 Update 횟수가 감소한다.

33. Epoch은 전체 Training Dataset을 한 번 학습에 사용하는 단위이다.

34. Iteration은 하나의 Batch에 대해 한 번 Parameter Update하는 단계이다.

35. 마지막 Batch까지 사용하면 일반적으로

$$
Iterations
=
\left\lceil
\frac{N}{BatchSize}
\right\rceil
$$

로 계산할 수 있다.

36. 전체 Update 횟수는

$$
Epochs
\times
Iterations\ per\ Epoch
$$

으로 계산할 수 있다.

37. 마지막 불완전 Batch를 버리는 설정에서는 Iteration 수가 달라질 수 있다.

38. Mini-Batch Dataset에서는 일반적으로 Training Sample을 Shuffle하여 특정 순서의 편향을 줄인다.

39. 다만 Time Series처럼 순서가 중요한 데이터는 무조건 Shuffle해서는 안 된다.

40. Convergence는 반복 학습을 통해 Loss나 Parameter가 일정한 값에 가까워지는 현상이다.

41. Convergence가 반드시 Global Minimum 도달을 의미하지는 않는다.

42. Training Loss가 감소한다고 해서 반드시 Generalization Performance가 좋아지는 것은 아니다.

43. 따라서 Training Loss와 Validation Loss를 함께 관찰해야 한다.

44. Training Loss는 계속 감소하지만 Validation Loss가 증가하면 Overfitting 가능성을 고려할 수 있다.

45. Loss가 크게 진동한다면 Learning Rate, Batch Noise, Feature Scale 등을 점검할 수 있다.

46. Loss가 급격하게 증가한다면 Divergence 가능성을 확인해야 한다.

47. Epoch을 지나치게 많이 수행한다고 항상 Model이 좋아지는 것은 아니다.

48. 실제 Training에서는 Learning Rate, Batch Size, Epoch, Optimizer를 함께 조절한다.

49. Gradient Descent의 핵심은 단순히 Gradient를 계산하는 것이 아니라 **현재 Parameter에서 Gradient를 계산하고, Update한 뒤 다시 Gradient를 계산하는 반복 과정**이다.

50. 지금까지 학습한 전체 구조는 다음과 같이 연결된다.

```text
Model
↓
Prediction
↓
Loss Function
↓
Differentiation
↓
Gradient
↓
Chain Rule
↓
Backpropagation
↓
Optimizer
↓
Gradient Descent
↓
Parameter Update
↓
다음 Iteration
```

51. 결국 Model Training은 **Loss를 기준으로 Gradient를 계산하고, Parameter를 반복적으로 수정하여 관측 데이터를 더 잘 설명하는 Parameter를 찾아가는 Optimization 과정**이라고 이해할 수 있다.
