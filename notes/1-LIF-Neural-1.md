# LIF-Neural-1

## 영단어 정리

- 막 전위: membrane potential
- 활동 전위: action potential
- 점화 임계치: firing threshold
- 임계 전위: threshold voltage
- 스파이크: spike, ***action potlential와 동의어**
- 리셋 전위: reset voltage
- 막 전위 전하: membrane potential voltage

---

## LIF Model

### LIF Model 이란?

![LIF model flow](../image/LIF-Neural-1-LIF%20model%20flow.jpg)

$$I(t) = C \frac{dV}{dt} + \frac{V - E_L}{R}$$

$$T_m \frac{d}{dt} V(t) = E_L - V(t) + R \cdot I(t)$$

$$if V(t) \leq V_{\text{th}} -> V(t) = V_{\text{reset}}, otherwise -> continue$$

$$V(t), V_m: membrane-potential$$

$$T_m: membrane-time-constant$$

$$E_L: leaky-potential$$

$$R: membrane-resistance$$

$$I(t):syanpse-input-voltage$$

$$V_th: firing-threshold$$

$$V_reset: reset-voltage$$

- Leaky Integrate and Fire Model
- 막 전위가 임계 전위를 넘으면 스파이크를 생성하고, 리셋 전위로 초기화된다.
- 막 전위 전하는 지속적으로 **leak(누수)**가 일어난다.
- 뉴런의 **활동 전위(action potential)**을 묘사한 모델 중 하나
  - **모델링: 자연현상을 수학적으로 나타낼 수 있도록 가공하는 과정**
  - 뉴런의 활동 전위(action potential)을 가장 잘 묘사한 모델은 호지킨-헉슬리 모델(Hodgkin-Huxley model)이다.
- 상미분 방정식(ordinary differential equation)
  - ***상미분 방정식: 구하려는 함수가 하나의 독립 변수만을 가지고 있는 미분 방정식**
  - 반의어: 편미분 방정식(partial differential equation)

## 소스 코드 상에서의 LIF Model 계산

### 랜덤 시냅스 입력 전류

$$I(t) = I_{\text{mean}} \left(1 + 0.1 \cdot \sqrt{\left(\frac{t_{\text{max}}}{\Delta t}\right)} \sim E(t)\right) with E(t) \sim u(-1, 1)$$

```python
random_num = 2 * np.random.random() - 1                   # 랜덤
i = i_mean * (1 + 0.1 * (t_max / dt)**(0.5) * random_num) # 입력 전류
```

- 최종적으로 아래의 수식으로 사용된다.

### 막전위 계산: 오일러 방법(Euler's method)

$$T_m \frac{d}{dt} V(t) = E_L - V(t) + R \cdot I(t)$$

$$T_m \cdot \frac{V(t + \Delta t) - V(t)}{\Delta t} = E_L - V(t) + R \cdot I(t)$$

$$V(t + \Delta t) = V(t) + \frac{\Delta t}{T_m} \left( E_L - V(t) + R \cdot I(t) \right)$$

```python
v = v + dt / tau * (el - v + r * i)
```

- 최종적으로 아래의 수식으로 사용된다.
