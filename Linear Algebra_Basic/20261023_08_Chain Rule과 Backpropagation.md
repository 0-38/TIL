
# [08. Chain Rule과 Backpropagation]

<br>

# **1. 합성함수와 Chain Rule의 수학적 원리**

> ## **1-1) 합성함수(Composite Function)란?**

합성함수는 **앞의 함수에서 계산한 결과가 다음 함수의 입력으로 사용되는 구조**이다.

예를 들어 다음과 같은 함수가 있다고 하자.

```text
z = 3x + 1
f = z²
```

계산 흐름은 다음과 같다.

```text
x
↓
z = 3x + 1
↓
f = z²
```

두 식을 하나로 연결하면

```text
f(x) = (3x + 1)²
```

이 된다.

즉,

```text
x → z → f
```

처럼 여러 연산이 연결된 형태가 합성함수이다.

신경망 역시 같은 구조를 가진다.

```text
입력 X
↓
Linear
↓
Activation
↓
Linear
↓
Prediction
↓
Loss
```

따라서 신경망은 매우 많은 함수가 연결된 **거대한 합성함수**라고 볼 수 있다.

<br>

> ## **1-2) 중간 변수(Intermediate Variable)**

복잡한 함수를 한 번에 계산하지 않고 중간 변수를 정의하면 계산 구조를 쉽게 이해할 수 있다.

예를 들어

```text
f(x) = (3x + 1)²
```

을 다음처럼 분리한다.

```text
z = 3x + 1
f = z²
```

`z`가 중간 변수이다.

`x=2`를 넣으면

```text
z = 3×2 + 1
  = 7
```

이고,

```text
f = 7²
  = 49
```

이다.

| 단계 | 계산 | 결과 |
| --- | --- | ---: |
| 입력 | `x` | 2 |
| 중간 변수 | `z = 3x + 1` | 7 |
| 최종 출력 | `f = z²` | 49 |

이처럼 중간 변수는 **연산과 연산 사이의 값을 전달하는 역할**을 한다.

<br>

> ## **1-3) Chain Rule이 필요한 이유**

최종 함수 `f`는 `z`에 의해 변하고, `z`는 다시 `x`에 의해 변한다.

따라서 `x`가 변할 때 `f`가 얼마나 변하는지를 구하려면 다음 두 변화율을 연결해야 한다.

```text
x가 변할 때 z의 변화
+
z가 변할 때 f의 변화
```

이를 계산하는 규칙이 **Chain Rule(연쇄법칙)** 이다.

Chain Rule은 다음과 같다.

$$
\frac{df}{dx}
=
\frac{df}{dz}
\times
\frac{dz}{dx}
$$

즉,

```text
전체 기울기
=
뒤쪽 연산의 Local Gradient
×
앞쪽 연산의 Local Gradient
```

로 이해할 수 있다.


|수식|	의미|정의|미분 방법|미분 결과|
|---|---|---|---|---|
|$$\frac{df}{dx}$$|`x`에 대한 `f`의 변화율|`x`가 바뀌면 `f`가 얼마나 변할까?|$f=(3x+1)^2$를 `x`로 미분|$6(3x+1)$
|$$\frac{df}{dz}$$|`z`에 대한 `f`의 변화율|`z`가 바뀌면 `f`가 얼마나 변할까?|$f=z^2$ 를 `z`로 미분|$2z$|
|$$\frac{dz}{dx}$$|`x`에 대한 `z`의 변화율|`x`가 바뀌면 `z`가 얼마나 변할까?|$z=3x+1$ 을 `x`로 미분|$3$|



<br>

> ## **1-4) Local Derivative**

Local Derivative는 **하나의 연산에서 입력 변화에 따른 출력 변화율**이다.

다음 함수를 사용하자.

```text
z = 3x + 1
f = z²
```

먼저

$$
\frac{df}{dz}
=
2z
$$

이고,

$$
\frac{dz}{dx}
=
3
$$

이다.

따라서 Chain Rule을 적용하면

$$
\frac{df}{dx}
=
2z \times 3
$$

이다.

`z = 3x+1`을 다시 넣으면

$$
\frac{df}{dx}
=
6(3x+1)
$$

$$
=
18x+6
$$

이 된다.

<br>

> ## **1-5) 특정 지점에서 Chain Rule 계산**

`x=2`라면 먼저 중간값은

```text
z = 3×2 + 1
  = 7
```

이다.

따라서

$$
\frac{df}{dz}
=
2\times7
=
14
$$

이고,

$$
\frac{dz}{dx}
=
3
$$

이므로

$$
\frac{df}{dx}
=
14\times3
=
42
$$

이다.

즉, `x=2`에서 순간 변화율은 `42`이다.

| 계산 항목 | 결과 |
| --- | ---: |
| `x` | 2 |
| `z` | 7 |
| `f` | 49 |
| `df/dz` | 14 |
| `dz/dx` | 3 |
| `df/dx` | 42 |

<br>

> ## **1-6) Python으로 합성함수와 Chain Rule 확인**

[ Python : code ] "합성함수의 순전파와 Chain Rule 기울기 계산"

```python
# STEP 1. 순전파(Forward Propagation)
# 입력값 x를 이용해 -> 중간값 z와 -> 최종 함수값 f 계산
def forward(x):
    z = 3 * x + 1   # 중간값 z 계산
    f = z ** 2      # z를 제곱하여 최종 함수값 f 계산
    return z, f     # 중간값과 최종 함수값 반환


# STEP 2. 역전파(Backpropagation)
# 연쇄법칙(Chain Rule)을 이용해 x에 대한 f의 기울기 계산
# f에서 z, x 방향으로 미분값을 계산
def derivative(x):
    z = 3 * x + 1   # 순전파에서 계산한 중간값 z

    df_dz = 2 * z   # f = z²을 z에 대해 미분 → df/dz = 2z
    dz_dx = 3       # z = 3x + 1을 x에 대해 미분 → dz/dx = 3

    # 연쇄법칙 적용: df/dx = (df/dz) × (dz/dx)
    df_dx = df_dz * dz_dx

    return df_dx    # 입력값 x에 대한 최종 기울기 반환


# STEP 3. 입력 데이터 설정
x = 2.0

# STEP 4. 순전파 실행
z, f = forward(x)

# STEP 5. 역전파 실행 (기울기 계산)
gradient = derivative(x)

# STEP 6. 결과 출력
print("중간값 z:", z)
print("함수값 f:", f)
print("df/dx:", gradient)
```

[ Result ]

```text
중간값 z: 7.0
함수값 f: 49.0
df/dx: 42.0
```

결과를 보면 순전파에서는

```text
x → z → f
```

순서로 값을 계산했다.

반면 미분에서는

```text
df/dz
×
dz/dx
```

를 연결하여 `x`가 최종 출력에 미치는 영향을 계산했다.

<br>

> ## **1-7) 함수가 세 개 이상 연결되는 경우**

Chain Rule은 함수가 여러 개 연결되어 있어도 동일하게 적용된다.

예를 들어

```text
a = 2x + 1
z = a²
L = (z - 5)²
```

라고 하자.

전체 흐름은

```text
x
↓
a
↓
z
↓
L
```

이다.

따라서

$$
\frac{dL}{dx}
=
\frac{dL}{dz}
\times
\frac{dz}{da}
\times
\frac{da}{dx}
$$

이다.

`x=1`이면

```text
a = 2×1 + 1 = 3
z = 3² = 9
L = (9-5)² = 16
```

이다.

각 Local Gradient는

$$
\frac{dL}{dz}
=
2(z-5)
=
8
$$

$$
\frac{dz}{da}
=
2a
=
6
$$

$$
\frac{da}{dx}
=
2
$$

이다.

따라서

$$
\frac{dL}{dx}
=
8\times6\times2
=
96
$$

이다.

<br>

> ## **1-8) Python으로 여러 단계 Chain Rule 계산**

[ Python : code ] "여러 함수가 연결된 경우의 Chain Rule 계산"

```python
x = 1.0

# Forward
a = 2 * x + 1
z = a ** 2
loss = (z - 5) ** 2

# Local Gradient
dL_dz = 2 * (z - 5)
dz_da = 2 * a
da_dx = 2

# Chain Rule
dL_dx = dL_dz * dz_da * da_dx

print("a:", a)
print("z:", z)
print("Loss:", loss)
print("dL/dx:", dL_dx)
```

[ Result ]

```text
a: 3.0
z: 9.0
Loss: 16.0
dL/dx: 96.0
```

복잡한 함수라도

```text
각 연산의 미분
↓
연결
↓
전체 Gradient
```

방식으로 계산할 수 있다.

<br>

> ## **1-9) 머신러닝에서 Chain Rule**

회귀 모델을 생각해 보자.

```text
ŷ = wx + b
L = (y - ŷ)²
```

가중치 `w`가 바뀌면

```text
w
↓
ŷ 변화
↓
L 변화
```

가 발생한다.

따라서 `w`에 대한 손실의 기울기는

$$
\frac{\partial L}{\partial w}
=
\frac{\partial L}{\partial \hat{y}}
\times
\frac{\partial \hat{y}}{\partial w}
$$

이다.

편향도 마찬가지다.

$$
\frac{\partial L}{\partial b}
=
\frac{\partial L}{\partial \hat{y}}
\times
\frac{\partial \hat{y}}{\partial b}
$$

이 원리가 신경망의 Backpropagation까지 그대로 확장된다.

<br>

> ## **1-10) Chain Rule과 신경망**

간단한 DNN을 생각하면 다음과 같다.

```text
X
↓
z₁ = XW₁ + b₁
↓
h₁ = ReLU(z₁)
↓
z₂ = h₁W₂ + b₂
↓
Prediction
↓
Loss
```

첫 번째 Layer의 `W₁`이 Loss에 미치는 영향을 구하려면

```text
W₁
→ z₁
→ h₁
→ z₂
→ Loss
```

전체 경로에 존재하는 미분을 연결해야 한다.

따라서 Deep Neural Network의 학습에서 Chain Rule은 핵심적인 수학 원리이다.


<br><br>

---

# **2. 순전파와 역전파의 원리**

> ## **2-1) Forward Propagation이란?**

Forward Propagation은 **입력에서 시작하여 모델의 예측값과 손실을 계산하는 과정**이다.

회귀 모델이

```text
ŷ = wx + b
```

이고 손실 함수가

```text
L = (y - ŷ)²
```

라면 Forward는

```text
x
↓
ŷ 계산
↓
y와 비교
↓
Loss 계산
```

순서이다.

<br>

> ## **2-2) 회귀 모델 Forward 계산**

다음 값을 사용해보자.

| 항목 | 값 |
| --- | ---: |
| `x` | 2 |
| `y` | 10 |
| `w` | 3 |
| `b` | 1 |

예측값은

$$
\hat{y}
=
3\times2+1
=
7
$$

이다.

손실은

$$
L
=
(10-7)^2
=
9
$$

이다.

즉,

```text
x = 2
↓
ŷ = 7
↓
Loss = 9
```

이다.

<br>

> ## **2-3) Backpropagation이란?**

Backpropagation은 **최종 Loss에서 시작하여 각 Parameter에 대한 Gradient를 계산하는 과정**이다.

Forward가

```text
x
→
ŷ
→
L
```

이라면 Backward는

```text
L
→
ŷ
→
w, b
```

방향으로 진행한다.

여기서 중요한 점은 Backpropagation이 Forward 값을 거꾸로 복원하는 과정이 아니라는 것이다.

```text
Backpropagation
=
Loss에 대한 Gradient를
이전 연산으로 전달하는 과정
```

이다.

<br>

> ## **2-4) Forward와 Backward 비교**

| 구분 | Forward Propagation | Backpropagation |
| --- | --- | --- |
| 방향 | 입력 → 출력 | Loss → 이전 Layer |
| 주요 목적 | Prediction 계산 | Gradient 계산 |
| 사용하는 값 | Parameter와 입력 | Local Gradient |
| 결과 | Prediction, Loss | `∂L/∂θ` |
| Parameter 변경 | X | X |

Backpropagation을 수행했다고 Parameter가 바로 변경되는 것은 아니다.

Parameter Update는 Optimizer가 담당한다.

<br>

> ## **2-5) 손실에서 Prediction으로 Gradient 전달**

손실은

```text
L = (y - ŷ)²
```

이다.

이를 `ŷ`에 대해 미분하면

$$
\frac{\partial L}{\partial \hat{y}}
=
2(\hat{y}-y)
$$

이다.

현재

```text
ŷ = 7
y = 10
```

이므로

$$
\frac{\partial L}{\partial \hat{y}}
=
2(7-10)
=
-6
$$

이다.

<br>

> ## **2-6) 가중치 Gradient 계산**

Prediction은

```text
ŷ = wx + b
```

이다.

가중치에 대한 Local Gradient는

$$
\frac{\partial \hat{y}}{\partial w}
=
x
$$

이다.

현재 `x=2`이므로

$$
\frac{\partial \hat{y}}{\partial w}
=
2
$$

이다.

Chain Rule을 적용하면

$$
\frac{\partial L}{\partial w}
=
\frac{\partial L}{\partial \hat{y}}
\times
\frac{\partial \hat{y}}{\partial w}
$$

$$
=
-6\times2
$$

$$
=
-12
$$

이다.

<br>

> ## **2-7) 편향 Gradient 계산**

편향은

$$
\frac{\partial \hat{y}}{\partial b}
=
1
$$

이다.

따라서

$$
\frac{\partial L}{\partial b}
=
-6\times1
=
-6
$$

이다.

전체 결과는 다음과 같다.

| 계산 대상 | 결과 |
| --- | ---: |
| Prediction `ŷ` | 7 |
| Loss `L` | 9 |
| `∂L/∂ŷ` | -6 |
| `∂L/∂w` | -12 |
| `∂L/∂b` | -6 |

<br>

> ## **2-8) Python 회귀 모델 Forward·Backward 예제**

운동역학 센서에서 하나의 입력 특성을 이용해 특정 지표를 예측한다고 가정하자.

[ Python : code ] "회귀 모델의 순전파와 Chain Rule 기반 역전파"

```python
# 운동역학 측정값
x = 2.0
y = 10.0

# Model Parameter
w = 3.0
b = 1.0

# --------------------
# Forward
# --------------------

y_pred = w * x + b
loss = (y - y_pred) ** 2

# --------------------
# Backward
# --------------------

dL_dy_pred = 2 * (y_pred - y)

dy_pred_dw = x
dy_pred_db = 1.0

dL_dw = dL_dy_pred * dy_pred_dw
dL_db = dL_dy_pred * dy_pred_db

print("Prediction:", y_pred)
print("Loss:", loss)
print("dL/dy_pred:", dL_dy_pred)
print("dL/dw:", dL_dw)
print("dL/db:", dL_db)
```

[ Result ]

```text
Prediction: 7.0
Loss: 9.0
dL/dy_pred: -6.0
dL/dw: -12.0
dL/db: -6.0
```

<br>

> ## **2-9) Gradient 부호의 의미**

현재

```text
dL/dw = -12
```

이다.

Gradient Descent의 기본 업데이트는

$$
w_{new}
=
w_{old}
-
\eta
\frac{\partial L}{\partial w}
$$

이다.

Gradient가 음수이면

```text
w_new
=
w_old - 음수
```

이므로 `w`는 증가한다.

예를 들어 Learning Rate가 `0.01`이라면

$$
w_{new}
=
3
-
0.01(-12)
$$

$$
=
3.12
$$

이다.

마찬가지로

$$
b_{new}
=
1
-
0.01(-6)
=
1.06
$$

이다.

<br>

> ## **2-10) Backpropagation과 Parameter Update의 차이**

| 과정 | 역할 | 대표적인 표현 |
| --- | --- | --- |
| Forward | 예측 계산 | `model(X)` |
| Loss | 오차 계산 | `loss_fn(pred, y)` |
| Backward | Gradient 계산 | `loss.backward()` |
| Update | Parameter 변경 | `optimizer.step()` |

즉,

```text
Backward
≠
Parameter Update
```

이다.

Backpropagation은 기울기를 구한다.

Optimizer는 그 기울기를 이용하여 실제 Parameter를 변경한다.

<br>

> ## **2-11) 여러 Layer가 연결된 경우**

다음과 같은 모델을 생각하자.

```text
a = w₁x + b₁
ŷ = w₂a + b₂
L = (y - ŷ)²
```

전체 흐름은

```text
x
↓
a
↓
ŷ
↓
Loss
```

이다.

다음 값을 사용한다.

| 변수 | 값 |
| --- | ---: |
| `x` | 2 |
| `y` | 12 |
| `w₁` | 2 |
| `b₁` | 1 |
| `w₂` | 2 |
| `b₂` | 0 |

<br>

> ## **2-12) 여러 Layer의 Forward**

첫 번째 연산은

$$
a
=
2\times2+1
=
5
$$

이다.

두 번째 연산은

$$
\hat{y}
=
2\times5
=
10
$$

이다.

Loss는

$$
L
=
(12-10)^2
=
4
$$

이다.

따라서

```text
x = 2
↓
a = 5
↓
ŷ = 10
↓
Loss = 4
```

이다.

<br>

> ## **2-13) 출력 Layer의 Backward**

먼저

$$
\frac{\partial L}{\partial \hat{y}}
=
2(10-12)
=
-4
$$

이다.

두 번째 Weight는

$$
\frac{\partial L}{\partial w_2}
=
\frac{\partial L}{\partial \hat{y}}
\times
\frac{\partial \hat{y}}{\partial w_2}
$$

이다.

`ŷ = w₂a+b₂`이므로

$$
\frac{\partial \hat{y}}{\partial w_2}
=
a
=
5
$$

이다.

따라서

$$
\frac{\partial L}{\partial w_2}
=
-4\times5
=
-20
$$

이다.

편향은

$$
\frac{\partial L}{\partial b_2}
=
-4
$$

이다.

<br>

> ## **2-14) 이전 Layer로 Gradient 전달**

첫 번째 Layer까지 Gradient를 전달하려면 중간 변수 `a`에 대한 기울기를 구해야 한다.

$$
\frac{\partial L}{\partial a}
=
\frac{\partial L}{\partial \hat{y}}
\times
\frac{\partial \hat{y}}{\partial a}
$$

이고

$$
\frac{\partial \hat{y}}{\partial a}
=
w_2
=
2
$$

이므로

$$
\frac{\partial L}{\partial a}
=
-4\times2
=
-8
$$

이다.

이 기울기가 이전 Layer로 전달되는 Gradient이다.

<br>

> ## **2-15) 첫 번째 Layer Parameter Gradient**

첫 번째 연산은

```text
a = w₁x + b₁
```

이다.

따라서

$$
\frac{\partial L}{\partial w_1}
=
\frac{\partial L}{\partial a}
\times x
$$

$$
=
-8\times2
=
-16
$$

이다.

편향은

$$
\frac{\partial L}{\partial b_1}
=
-8
$$

이다.

최종 결과는 다음과 같다.

| Parameter | Gradient |
| --- | ---: |
| `w₁` | -16 |
| `b₁` | -8 |
| `w₂` | -20 |
| `b₂` | -4 |

<br>

> ## **2-16) Python으로 다중 Layer Backpropagation 구현**

[ Python : code ] "두 개의 Affine Layer에서 Gradient를 뒤쪽에서 앞쪽으로 전달하는 방법"

```python
# Input / Target
x = 2.0
y = 12.0

# Layer 1
w1 = 2.0
b1 = 1.0

# Layer 2
w2 = 2.0
b2 = 0.0

# --------------------
# Forward
# --------------------

a = w1 * x + b1
y_pred = w2 * a + b2

loss = (y - y_pred) ** 2

# --------------------
# Backward
# --------------------

# Loss -> Prediction
dL_dy_pred = 2 * (y_pred - y)

# Layer 2
dL_dw2 = dL_dy_pred * a
dL_db2 = dL_dy_pred

# Gradient propagation
dL_da = dL_dy_pred * w2

# Layer 1
dL_dw1 = dL_da * x
dL_db1 = dL_da

print("a:", a)
print("Prediction:", y_pred)
print("Loss:", loss)

print("w1 Gradient:", dL_dw1)
print("b1 Gradient:", dL_db1)
print("w2 Gradient:", dL_dw2)
print("b2 Gradient:", dL_db2)
```

[ Result ]

```text
a: 5.0
Prediction: 10.0
Loss: 4.0
w1 Gradient: -16.0
b1 Gradient: -8.0
w2 Gradient: -20.0
b2 Gradient: -4.0
```

<br>

> ## **2-17) 복잡한 코드 흐름도**

```text
[ Forward ]

x = 2
↓
w₁x + b₁
↓
a = 5
↓
w₂a + b₂
↓
ŷ = 10
↓
Loss = 4


[ Backward ]

Loss
↓
dL/dŷ = -4
↓
┌─────────────────────────┐
│ w₂ Gradient = -20       │
│ b₂ Gradient = -4        │
└─────────────────────────┘
↓
dL/da = -8
↓
┌─────────────────────────┐
│ w₁ Gradient = -16       │
│ b₁ Gradient = -8        │
└─────────────────────────┘
```

Backpropagation에서는 뒤쪽에서 계산한

```text
dL/da
```

를 앞쪽 Layer가 다시 사용한다.

이러한 **Gradient의 재사용**이 Deep Neural Network의 효율적인 Gradient 계산에서 중요하다.

<br>

> ## **2-18) Branch가 있는 계산 그래프**

지금까지는 하나의 경로만 있는 모델을 다뤘다.

하지만 하나의 값이 여러 계산에 동시에 사용될 수도 있다.

예를 들어

```text
      → f₁
x
      → f₂
```

처럼 `x`가 두 경로에 사용되었다면 각 경로에서 전달된 Gradient를 더한다.

즉,

$$
\frac{\partial L}{\partial x}
=
\left(
\frac{\partial L}{\partial x}
\right)_1
+
\left(
\frac{\partial L}{\partial x}
\right)_2
$$

이다.

따라서 Backpropagation의 기본 원칙은

```text
직렬 경로 → Gradient를 곱함
분기 경로 → Gradient를 합함
```

으로 이해할 수 있다.


<br><br>

---

# **3. 신경망의 역전파와 Gradient 계산**

> ## **3-1) 이진 분류 Neural Network**

이번에는 회귀가 아니라 이진 분류를 생각한다.

GPS 센서 정보를 이용하여 특정 구간이 고강도 활동인지 예측한다고 가정하자.

모델은

```text
z = wx + b
p = Sigmoid(z)
L = BCE(y, p)
```

형태이다.

전체 흐름은

```text
입력 x
↓
Linear
↓
Logit z
↓
Sigmoid
↓
Probability p
↓
BCE
↓
Loss
```

이다.

<br>

> ## **3-2) Logit**

Logit은 확률 변환 함수를 적용하기 전의 점수이다.

```text
z = wx + b
```

예를 들어

```text
x = 2
w = 0.5
b = 0
```

이면

$$
z
=
0.5\times2
=
1
$$

이다.

Logit은 확률이 아니므로 음수나 1보다 큰 값도 가질 수 있다.

<br>

> ## **3-3) Sigmoid**

Sigmoid는 Logit을 `0~1` 범위로 변환한다.

$$
\sigma(z)
=
\frac{1}
{1+e^{-z}}
$$

`z=1`이면

$$
p
=
\sigma(1)
\approx
0.731
$$

이다.

따라서 모델은 클래스 `1`일 확률을 약 `73.1%`로 예측한 것이다.

<br>

> ## **3-4) Binary Cross Entropy**

BCE는 다음과 같다.

$$
L
=
-
\left[
y\log(p)
+
(1-y)\log(1-p)
\right]
$$

실제 정답이

```text
y = 1
```

이면 식은

$$
L
=
-\log(p)
$$

가 된다.

현재 `p≈0.731`이므로

$$
L
\approx
0.313
$$

이다.

<br>

> ## **3-5) Sigmoid와 BCE의 Chain Rule**

계산 구조는

```text
z
↓
p = Sigmoid(z)
↓
L = BCE(y, p)
```

이다.

따라서

$$
\frac{\partial L}{\partial z}
=
\frac{\partial L}{\partial p}
\times
\frac{\partial p}{\partial z}
$$

이다.

BCE를 `p`에 대해 미분하면

$$
\frac{\partial L}{\partial p}
=
-
\frac{y}{p}
+
\frac{1-y}{1-p}
$$

이다.

Sigmoid의 미분은

$$
\frac{\partial p}{\partial z}
=
p(1-p)
$$

이다.

두 값을 곱하여 정리하면

$$
\boxed{
\frac{\partial L}{\partial z}
=
p-y
}
$$

가 된다.

이 결과는 **Sigmoid + BCE 조합에서 매우 중요한 Gradient 공식**이다.

<br>

> ## **3-6) 실제 숫자로 Logit Gradient 계산**

현재

```text
p = 0.731
y = 1
```

이므로

$$
\frac{\partial L}{\partial z}
=
0.731-1
$$

$$
\approx
-0.269
$$

이다.

Gradient가 음수라는 것은 현재 지점에서 `z`를 증가시키는 방향으로 움직이면 Loss가 감소할 수 있음을 의미한다.

실제 정답이 `1`이므로 모델이 예측 확률을 더 높이는 방향과 일치한다.

<br>

> ## **3-7) Weight와 Bias Gradient**

Logit은

```text
z = wx + b
```

이다.

따라서

$$
\frac{\partial z}{\partial w}
=
x
$$

이고,

$$
\frac{\partial z}{\partial b}
=
1
$$

이다.

Weight Gradient는

$$
\frac{\partial L}{\partial w}
=
\frac{\partial L}{\partial z}
\times
\frac{\partial z}{\partial w}
$$

$$
=
(p-y)x
$$

이다.

현재 `x=2`이므로

$$
\frac{\partial L}{\partial w}
\approx
-0.269\times2
$$

$$
\approx
-0.538
$$

이다.

Bias Gradient는

$$
\frac{\partial L}{\partial b}
=
p-y
\approx
-0.269
$$

이다.

| Parameter | Gradient |
| --- | ---: |
| `w` | -0.538 |
| `b` | -0.269 |

<br>

> ## **3-8) GPS 고강도 활동 분류 Python 예제**

[ Python : code ] "Sigmoid와 BCE를 사용하는 이진 분류 모델의 Forward와 Backward"

```python
import numpy as np

# GPS 기반 입력 Feature
x = 2.0

# 실제 Label
# 1 = High Intensity
y = 1.0

# Model Parameter
w = 0.5
b = 0.0

# --------------------
# Forward
# --------------------

z = w * x + b

p = 1 / (1 + np.exp(-z))

# Numerical stable BCE
loss = np.logaddexp(0, z) - y * z

# --------------------
# Backward
# --------------------

dL_dz = p - y

dL_dw = dL_dz * x
dL_db = dL_dz

print("Logit:", round(z, 3))
print("Probability:", round(p, 3))
print("BCE:", round(float(loss), 3))

print("dL/dz:", round(dL_dz, 3))
print("dL/dw:", round(dL_dw, 3))
print("dL/db:", round(dL_db, 3))
```

[ Result ]

```text
Logit: 1.0
Probability: 0.731
BCE: 0.313
dL/dz: -0.269
dL/dw: -0.538
dL/db: -0.269
```

<br>

> ## **3-9) 결과 흐름**

```text
x = 2
↓
z = 1
↓
Sigmoid
↓
p = 0.731
↓
BCE
↓
Loss = 0.313


Backward

Loss
↓
dL/dz = p-y
↓
-0.269
↓
┌────────────────────┐
│ dL/dw = -0.538     │
│ dL/db = -0.269     │
└────────────────────┘
```

<br>

> ## **3-10) 은닉층이 있는 Neural Network**

이번에는 Hidden Layer를 추가한다.

```text
a = w₁x + b₁
h = ReLU(a)

z = w₂h + b₂
p = Sigmoid(z)

L = BCE(y, p)
```

전체 흐름은

```text
x
↓
Linear 1
↓
a
↓
ReLU
↓
h
↓
Linear 2
↓
z
↓
Sigmoid
↓
p
↓
BCE
↓
Loss
```

이다.

<br>

> ## **3-11) Hidden Layer Forward**

다음 값을 사용하자.

| 변수 | 값 |
| --- | ---: |
| `x` | 2 |
| `y` | 1 |
| `w₁` | 1 |
| `b₁` | 0 |
| `w₂` | 0.5 |
| `b₂` | 0 |

첫 번째 Linear 결과는

$$
a
=
1\times2
=
2
$$

이다.

ReLU는

```text
ReLU(2) = 2
```

이므로

```text
h = 2
```

이다.

출력 Layer의 Logit은

$$
z
=
0.5\times2
=
1
$$

이다.

Sigmoid를 적용하면

```text
p ≈ 0.731
```

이고 BCE는

```text
L ≈ 0.313
```

이다.

<br>

> ## **3-12) 출력 Layer Backpropagation**

먼저

$$
\frac{\partial L}{\partial z}
=
p-y
$$

이므로

$$
\frac{\partial L}{\partial z}
\approx
-0.269
$$

이다.

출력 Weight는

$$
\frac{\partial L}{\partial w_2}
=
\frac{\partial L}{\partial z}
\times h
$$

$$
=
-0.269\times2
$$

$$
\approx
-0.538
$$

이다.

Bias는

$$
\frac{\partial L}{\partial b_2}
\approx
-0.269
$$

이다.

<br>

> ## **3-13) Hidden Layer로 Gradient 전달**

출력 Layer는

```text
z = w₂h + b₂
```

이다.

따라서

$$
\frac{\partial z}{\partial h}
=
w_2
$$

이다.

Chain Rule을 사용하면

$$
\frac{\partial L}{\partial h}
=
\frac{\partial L}{\partial z}
\times
w_2
$$

$$
=
-0.269\times0.5
$$

$$
\approx
-0.134
$$

이다.

즉, 출력 Layer에서 계산한 Gradient가 Hidden Layer까지 전달되었다.

<br>

> ## **3-14) ReLU Gradient**

ReLU는

$$
ReLU(a)
=
\max(0,a)
$$

이다.

도함수는 일반적으로

$$
ReLU'(a)
=
\begin{cases}
1, & a>0 \\
0, & a<0
\end{cases}
$$

로 이해할 수 있다.

`a=2`이므로 현재 ReLU Gradient는 `1`이다.

따라서

$$
\frac{\partial L}{\partial a}
=
\frac{\partial L}{\partial h}
\times1
$$

$$
\approx
-0.134
$$

이다.

`a=0`에서의 미분은 수학적으로 특별한 처리가 필요하며, 딥러닝 프레임워크에서는 구현상 특정 Subgradient를 사용한다.

<br>

> ## **3-15) 첫 번째 Layer Gradient**

첫 번째 Layer는

```text
a = w₁x + b₁
```

이다.

따라서

$$
\frac{\partial L}{\partial w_1}
=
\frac{\partial L}{\partial a}
\times x
$$

$$
=
-0.134\times2
$$

$$
\approx
-0.269
$$

이다.

Bias Gradient는

$$
\frac{\partial L}{\partial b_1}
\approx
-0.134
$$

이다.

전체 결과를 정리하면 다음과 같다.

| Parameter | Gradient |
| --- | ---: |
| `w₁` | -0.269 |
| `b₁` | -0.134 |
| `w₂` | -0.538 |
| `b₂` | -0.269 |

<br>

> ## **3-16) Python으로 Hidden Layer Backpropagation 구현**

[ Python : code ] "ReLU 은닉층과 Sigmoid 출력층을 가진 신경망의 역전파"

```python
import numpy as np


def sigmoid(z):
    return 1 / (1 + np.exp(-z))


def relu(x):
    return np.maximum(0, x)


# Input / Target
x = 2.0
y = 1.0

# Hidden Layer
w1 = 1.0
b1 = 0.0

# Output Layer
w2 = 0.5
b2 = 0.0

# --------------------
# Forward
# --------------------

a = w1 * x + b1
h = relu(a)

z = w2 * h + b2
p = sigmoid(z)

loss = np.logaddexp(0, z) - y * z

# --------------------
# Backward: Output
# --------------------

dL_dz = p - y

dL_dw2 = dL_dz * h
dL_db2 = dL_dz

# --------------------
# Backward: Hidden
# --------------------

dL_dh = dL_dz * w2

relu_grad = (1.0 if a > 0 else 0.0)

dL_da = dL_dh * relu_grad

dL_dw1 = dL_da * x
dL_db1 = dL_da

print("a:", round(a, 3))
print("h:", round(float(h), 3))
print("z:", round(z, 3))
print("Probability:", round(p, 3))
print("BCE:", round(float(loss), 3))

print("w1 Gradient:", round(dL_dw1, 3))
print("b1 Gradient:", round(dL_db1, 3))
print("w2 Gradient:", round(dL_dw2, 3))
print("b2 Gradient:", round(dL_db2, 3))
```

[ Result ]

```text
a: 2.0
h: 2.0
z: 1.0
Probability: 0.731
BCE: 0.313
w1 Gradient: -0.269
b1 Gradient: -0.134
w2 Gradient: -0.538
b2 Gradient: -0.269
```

<br>

> ## **3-17) 코드 결과 흐름도**

```text
[ Forward ]

x = 2
↓
w₁x + b₁
↓
a = 2
↓
ReLU
↓
h = 2
↓
w₂h + b₂
↓
z = 1
↓
Sigmoid
↓
p = 0.731
↓
BCE
↓
Loss = 0.313


[ Backward ]

Loss
↓
dL/dz = p-y
↓
-0.269
↓
Output Layer
├─ dL/dw₂ = -0.538
└─ dL/db₂ = -0.269
↓
dL/dh = -0.134
↓
ReLU Gradient = 1
↓
dL/da = -0.134
↓
Hidden Layer
├─ dL/dw₁ = -0.269
└─ dL/db₁ = -0.134
```

<br>

> ## **3-18) ReLU가 Gradient 전달에 미치는 영향**

ReLU 입력이 양수이면

```text
ReLU'(a) = 1
```

이므로 뒤에서 전달된 Gradient가 그대로 통과한다.

반대로 입력이 음수이면

```text
ReLU'(a) = 0
```

이므로

```text
이전 Layer로 전달되는 Gradient = 0
```

이 될 수 있다.

따라서 활성화 함수도 Backpropagation 과정에서 매우 중요한 역할을 한다.

<br>

> ## **3-19) Forward와 Backward에서 저장해야 하는 값**

Backward를 계산하기 위해 Forward 단계의 일부 값이 필요하다.

예를 들어

```text
dL/dw₂ = dL/dz × h
```

를 계산하려면 Forward에서 계산했던 `h`가 필요하다.

마찬가지로 ReLU Gradient를 계산하려면 `a`가 양수였는지 확인해야 한다.

따라서 딥러닝 Framework는 Forward 과정에서 Backward에 필요한 중간값을 관리한다.

| Forward 값 | Backward 활용 |
| --- | --- |
| `x` | Weight Gradient |
| `a` | Activation 미분 |
| `h` | 다음 Layer Weight Gradient |
| `p` | Loss Gradient |

<br>

> ## **3-20) Computational Graph**

신경망의 연산 관계를 그래프로 표현한 것을 Computational Graph라고 한다.

예를 들어

```text
x ----┐
      × w
      ↓
      +
      ↑
      b
      ↓
      z
      ↓
   Sigmoid
      ↓
      p
      ↓
     BCE
      ↓
      L
```

Forward에서는 Graph를 따라 앞쪽으로 값을 계산한다.

Backward에서는 Graph를 반대 방향으로 따라가며 Gradient를 계산한다.

<br>

> ## **3-21) Automatic Differentiation**

실제 DNN에는 수많은 Parameter가 존재한다.

따라서

```text
dL/dw₁
dL/db₁
dL/dw₂
dL/db₂
...
```

를 사람이 직접 계산하지 않는다.

PyTorch와 같은 Framework는 각 연산의 미분 규칙을 알고 있으며 Computational Graph를 따라 자동으로 Chain Rule을 적용한다.

이를 **Automatic Differentiation(자동 미분)**이라고 한다.

<br>

> ## **3-22) PyTorch Autograd로 Gradient 확인**

[ Python : code ] "PyTorch Autograd를 이용한 Hidden Layer Gradient 자동 계산"

```python
import torch

# Input / Target
x = torch.tensor(2.0)
y = torch.tensor(1.0)

# Trainable Parameter
w1 = torch.tensor(1.0, requires_grad=True)
b1 = torch.tensor(0.0, requires_grad=True)

w2 = torch.tensor(0.5, requires_grad=True)
b2 = torch.tensor(0.0, requires_grad=True)

# --------------------
# Forward
# --------------------

a = w1 * x + b1
h = torch.relu(a)

z = w2 * h + b2

loss = torch.nn.functional.binary_cross_entropy_with_logits(z, y)

# --------------------
# Backward
# --------------------

loss.backward()

print(
    "Loss:",
    round(loss.item(), 3)
)

print(
    "w1 Gradient:",
    round(w1.grad.item(), 3)
)

print(
    "b1 Gradient:",
    round(b1.grad.item(), 3)
)

print(
    "w2 Gradient:",
    round(w2.grad.item(), 3)
)

print(
    "b2 Gradient:",
    round(b2.grad.item(), 3)
)
```

[ Result ]

```text
Loss: 0.313
w1 Gradient: -0.269
b1 Gradient: -0.134
w2 Gradient: -0.538
b2 Gradient: -0.269
```

직접 Chain Rule로 계산한 결과와 PyTorch Autograd가 계산한 결과가 동일하다.

즉,

```text
사람이 직접 수행한 Chain Rule
=
Autograd가 자동 수행한 Chain Rule
```

이라고 이해할 수 있다.

<br>

> ## **3-23) Backpropagation과 Autograd의 관계**

두 개념은 동일한 용어가 아니다.

| 개념 | 의미 |
| --- | --- |
| Chain Rule | 연결된 함수의 미분 규칙 |
| Backpropagation | 신경망에서 Chain Rule을 효율적으로 적용해 Gradient를 뒤로 전달하는 방법 |
| Autograd | 연산 Graph와 미분 규칙을 이용해 Gradient 계산을 자동화하는 시스템 |

즉,

```text
Chain Rule
↓
Backpropagation
↓
Autograd로 자동 수행
```

의 관계로 이해하면 쉽다.

<br>

> ## **3-24) 전체 학습 과정과 연결**

실제 Neural Network Training은 다음 순서로 반복된다.

```text
1. 현재 Parameter
↓
2. Forward Propagation
↓
3. Prediction
↓
4. Loss 계산
↓
5. Backpropagation
↓
6. Gradient 계산
↓
7. Optimizer Update
↓
8. 새로운 Parameter
↓
9. 다시 Forward
```

즉, 한 번 Backpropagation을 수행하고 끝나는 것이 아니다.

현재 Parameter를 업데이트한 후 새로운 Parameter를 기준으로 다시 Forward와 Backward를 수행한다.

<br>

> ## **3-25) Parameter Update Python 예제**

[ Python : code ] "Backpropagation으로 계산한 Gradient를 이용한 한 번의 Gradient Descent 업데이트"

```python
# 현재 Parameter
w = 3.0
b = 1.0

# 이미 계산한 Gradient
dL_dw = -12.0
dL_db = -6.0

learning_rate = 0.01

# Gradient Descent
w_new = w - learning_rate * dL_dw

b_new = b - learning_rate * dL_db

print("기존 w:", w)
print("업데이트 w:", w_new)

print("기존 b:", b)
print("업데이트 b:", b_new)
```

[ Result ]

```text
기존 w: 3.0
업데이트 w: 3.12
기존 b: 1.0
업데이트 b: 1.06
```

역전파가 Parameter를 직접 변경한 것이 아니다.

```text
Backpropagation
→ Gradient 계산

Gradient Descent
→ Parameter 변경
```

으로 역할을 구분해야 한다.

<br>

> ## **3-26) 실무 학습 Loop의 핵심**

PyTorch 학습 코드에서는 일반적으로 다음 구조를 반복한다.

[ Python : code ] "PyTorch 신경망 학습에서 Forward, Backward, Parameter Update의 기본 순서"

```python
for epoch in range(num_epochs):

    # 1. 이전 Gradient 초기화
    optimizer.zero_grad()

    # 2. Forward
    prediction = model(X)

    # 3. Loss
    loss = loss_fn(prediction, y)

    # 4. Backward
    loss.backward()

    # 5. Parameter Update
    optimizer.step()
```

[ Result ]

```text
각 Epoch마다

Gradient 초기화
→ Forward
→ Loss
→ Backward
→ Parameter Update

과정이 반복됨
```

이 구조는 이후 CNN, RNN, Transformer를 포함한 대부분의 Neural Network Training에서 공통적으로 사용된다.


<br><br>

---

# **4. Advanced 추가 학습**

> ## **4-1) Vector와 Matrix의 Chain Rule**

실제 Neural Network에서는 Scalar가 아니라 Vector와 Matrix를 사용한다.

예를 들어

```text
z = XW + b
```

에서

```text
X : Matrix
W : Matrix
b : Vector
z : Matrix
```

이다.

따라서 실제 Backpropagation에서는 Scalar 미분을 확장한 Vector·Matrix 미분이 사용된다.

하지만 기본 원리는 동일하다.

```text
뒤쪽 Gradient
×
현재 연산의 Local Gradient
→
앞쪽 Gradient
```

이다.

<br>

> ## **4-2) Jacobian**

Vector를 Vector에 대해 미분하면 하나의 숫자가 아니라 여러 편미분을 포함하는 Matrix가 필요할 수 있다.

이를 **Jacobian Matrix**라고 한다.

예를 들어

```text
y = f(x)
```

에서 `x`와 `y`가 Vector라면 Jacobian은 각 출력이 각 입력에 얼마나 민감한지를 표현한다.

실제 Deep Learning Framework는 이러한 계산을 효율적으로 처리하므로 대부분 Jacobian 전체를 직접 만들 필요는 없다.

<br>

> ## **4-3) Numerical Gradient와 Gradient Check**

Gradient가 올바르게 계산되었는지 확인하기 위해 수치 미분과 비교할 수 있다.

중앙 차분은 다음과 같다.

$$
f'(x)
\approx
\frac{
f(x+h)-f(x-h)
}{
2h
}
$$

이를 Analytical Gradient와 비교하는 과정을 Gradient Checking이라고 한다.

주로 직접 만든 Layer나 미분 구현을 검증할 때 사용할 수 있다.

<br>

> ## **4-4) Sigmoid + BCEWithLogitsLoss**

실제 PyTorch에서는

```text
Sigmoid
+
BCE
```

를 따로 적용하기보다

```python
nn.BCEWithLogitsLoss()
```

를 사용하는 경우가 많다.

이 함수는 Logit을 직접 입력받아 수치적으로 안정적인 방식으로 BCE를 계산한다.

따라서 모델 Training에서는

```text
Logit
→ BCEWithLogitsLoss
```

방식을 사용할 수 있고,

확률이 필요할 때만

```python
torch.sigmoid(logits)
```

를 별도로 적용할 수 있다.


<br><br>

---

# **5. 핵심정리**

1. 합성함수는 한 함수의 출력이 다른 함수의 입력으로 사용되는 구조이다.

2. Neural Network는 Linear, Activation, Loss 등의 많은 함수가 연결된 거대한 합성함수이다.

3. 여러 함수가 연결되어 있을 때 입력이 최종 출력에 미치는 영향을 계산하기 위해 Chain Rule을 사용한다.

4. Chain Rule의 기본 형태는 다음과 같다.

$$
\frac{df}{dx}
=
\frac{df}{dz}
\times
\frac{dz}{dx}
$$

5. 각 연산 하나에서 계산되는 미분값을 Local Derivative라고 이해할 수 있다.

6. 여러 연산이 직렬로 연결되어 있다면 Local Gradient들을 곱하여 전체 Gradient를 계산한다.

7. 하나의 변수가 여러 계산 경로로 분기되었다면 각 경로에서 전달된 Gradient를 합산한다.

8. Forward Propagation은 입력에서 출력 방향으로 값을 계산하는 과정이다.

9. Forward에서는 현재 Parameter를 이용하여 Prediction과 Loss를 계산한다.

10. Backpropagation은 최종 Loss에서 시작하여 이전 연산으로 Gradient를 전달하는 과정이다.

11. Backpropagation은 Forward 값을 거꾸로 복원하는 과정이 아니다.

12. Backpropagation의 목적은 각 Parameter가 Loss에 얼마나 영향을 미치는지 계산하는 것이다.

13. 회귀 모델

```text
ŷ = wx + b
L = (y - ŷ)²
```

에서는 다음 Chain Rule을 사용할 수 있다.

$$
\frac{\partial L}{\partial w}
=
\frac{\partial L}{\partial \hat{y}}
\times
\frac{\partial \hat{y}}{\partial w}
$$

14. 같은 모델에서 Bias Gradient는 다음과 같다.

$$
\frac{\partial L}{\partial b}
=
\frac{\partial L}{\partial \hat{y}}
\times
\frac{\partial \hat{y}}{\partial b}
$$

15. 여러 Layer가 연결된 Neural Network에서는 출력 Layer의 Gradient를 먼저 계산한다.

16. 출력 Layer에서 계산한 Gradient는 이전 Hidden Layer의 입력 Gradient로 다시 사용된다.

17. 이러한 Gradient 재사용을 통해 많은 Parameter의 Gradient를 효율적으로 계산할 수 있다.

18. Backpropagation은 출력층에서 입력층 방향으로 진행하지만 Parameter 업데이트 자체를 수행하지는 않는다.

19. Backpropagation은 Gradient 계산이고 Optimizer는 Parameter Update를 수행한다.

20. Gradient Descent의 기본 Parameter Update는 다음과 같다.

$$
\theta_{new}
=
\theta_{old}
-
\eta
\frac{\partial L}{\partial\theta}
$$

21. Gradient가 음수이면 Gradient Descent 업데이트에서는 해당 Parameter가 증가하는 방향으로 이동할 수 있다.

22. Gradient가 양수이면 해당 Parameter가 감소하는 방향으로 이동할 수 있다.

23. Gradient의 부호만으로 실제 Loss 변화 전체를 판단할 수는 없으며 Gradient는 현재 지점의 Local Information이다.

24. 이진 분류 모델은 다음 구조로 표현할 수 있다.

```text
z = wx + b
p = Sigmoid(z)
L = BCE(y, p)
```

25. `z`는 Logit이고 `p`는 클래스 `1`의 예측 확률이다.

26. Sigmoid는 다음과 같다.

$$
\sigma(z)
=
\frac{1}
{1+e^{-z}}
$$

27. Sigmoid의 미분은 다음과 같다.

$$
\sigma'(z)
=
p(1-p)
$$

28. Binary Cross Entropy는 다음과 같다.

$$
L
=
-
\left[
y\log p
+
(1-y)\log(1-p)
\right]
$$

29. Sigmoid와 BCE를 연결하여 Chain Rule을 적용하면 다음과 같이 정리된다.

$$
\boxed{
\frac{\partial L}{\partial z}
=
p-y
}
$$

30. 따라서 Logistic 형태의 단일 Weight 모델에서는 다음과 같다.

$$
\frac{\partial L}{\partial w}
=
(p-y)x
$$

31. Bias Gradient는 다음과 같다.

$$
\frac{\partial L}{\partial b}
=
p-y
$$

32. `p-y`는 Sigmoid와 BCE가 결합된 경우의 Logit Gradient이다.

33. 다른 Activation이나 Loss를 사용하면 Gradient 공식도 달라질 수 있다.

34. Hidden Layer가 추가되어도 Chain Rule의 원리는 동일하다.

35. ReLU Hidden Layer에서는 출력 Gradient에 ReLU의 Local Derivative를 곱해 이전 Layer로 전달한다.

36. ReLU의 입력이 양수이면 일반적으로 Local Gradient는 `1`이다.

37. ReLU의 입력이 음수이면 Local Gradient는 `0`이다.

38. 따라서 ReLU가 음수 영역에 있으면 해당 경로를 통해 이전 Parameter로 전달되는 Gradient가 0이 될 수 있다.

39. Neural Network의 Forward 연산 관계를 Computational Graph로 표현할 수 있다.

40. Forward에서는 Computational Graph의 입력에서 출력 방향으로 값을 계산한다.

41. Backward에서는 Computational Graph를 반대 방향으로 이동하며 Gradient를 계산한다.

42. Backward 계산에는 Forward에서 계산했던 중간값이 필요할 수 있다.

43. 예를 들어 Linear Layer의 Weight Gradient 계산에는 해당 Layer의 입력값이 필요하다.

44. Activation의 Gradient 계산에는 Forward에서 계산한 활성화 이전 또는 이후 값이 필요할 수 있다.

45. 실제 Deep Neural Network에는 매우 많은 Parameter가 있기 때문에 모든 Gradient를 직접 계산하지 않는다.

46. Automatic Differentiation은 연산의 미분 규칙을 이용하여 Chain Rule 계산을 자동화한다.

47. PyTorch에서는 `loss.backward()`를 이용하여 Autograd 기반 Backpropagation을 수행할 수 있다.

48. 계산된 Gradient는 일반적으로 Parameter의 `.grad`에 저장된다.

49. Chain Rule, Backpropagation, Autograd는 서로 관련 있지만 동일한 개념은 아니다.

50. Chain Rule은 수학적 미분 규칙이다.

51. Backpropagation은 Chain Rule을 계산 그래프에 효율적으로 적용하여 Gradient를 전달하는 알고리즘이다.

52. Autograd는 이러한 Gradient 계산을 자동으로 처리하는 Framework 기능이다.

53. Neural Network Training의 전체 흐름은 다음과 같다.

```text
현재 Parameter
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
↓
Optimizer
↓
Parameter Update
↓
다음 Forward
```

54. 학습은 한 번의 Forward와 Backward로 끝나는 것이 아니라 Parameter를 계속 갱신하면서 반복한다.

55. 현재 Parameter가 변경되면 다음 Forward에서 Prediction과 Loss도 달라진다.

56. 따라서 매 학습 Step마다 새로운 Parameter를 기준으로 Gradient도 다시 계산해야 한다.

57. Gradient를 이전 Step과 의도하지 않게 누적하지 않도록 Framework에서는 일반적으로 Gradient 초기화 과정이 필요하다.

58. PyTorch의 대표적인 학습 흐름은 다음과 같다.

```text
optimizer.zero_grad()
↓
model(X)
↓
loss 계산
↓
loss.backward()
↓
optimizer.step()
```

59. Vector와 Matrix로 구성된 실제 Neural Network에서도 Chain Rule의 기본 원리는 동일하다.

60. Vector 미분에서는 Jacobian 등의 개념이 등장하지만 Framework가 이를 효율적으로 처리한다.

61. Numerical Gradient는 Analytical Gradient를 검증하는 Gradient Check에 활용할 수 있다.

62. 실제 Binary Classification에서는 Sigmoid와 BCE를 별도로 계산하는 대신 `BCEWithLogitsLoss`와 같이 수치적으로 안정적인 함수를 사용할 수 있다.

63. Backpropagation을 이해할 때 가장 중요한 흐름은 다음과 같다.

```text
Forward:
입력 → 중간값 → Prediction → Loss

Backward:
Loss → Prediction Gradient
     → Hidden Gradient
     → Parameter Gradient
```

64. 결국 Deep Learning의 학습은 **Forward에서 값을 계산하고, Backward에서 Chain Rule로 Gradient를 계산한 뒤, Optimizer가 Parameter를 갱신하는 과정을 반복하는 것**이다.