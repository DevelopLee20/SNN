# Brian2, Chapter 1

## Brian2

### Install

```powershell
conda install -c conda-forge brian2
```

```powershell
The following packages will be downloaded:

    package                    |            build
    ---------------------------|-----------------
    brian2-2.5.4               |  py311h12feb9d_2         1.5 MB  conda-forge
    ca-certificates-2023.11.17 |       h56e8100_0         151 KB  conda-forge
    certifi-2023.11.17         |     pyhd8ed1ab_0         155 KB  conda-forge
    cython-3.0.7               |  py311h12c1d0e_0         3.1 MB  conda-forge
    gsl-2.4                    |    hfa6e2cd_1004         1.9 MB  conda-forge
    openssl-3.2.0              |       hcfcfb64_1         7.9 MB  conda-forge
    python_abi-3.11            |          2_cp311           5 KB  conda-forge
    vc14_runtime-14.38.33130   |      h82b7239_18         732 KB  conda-forge
    vs2015_runtime-14.38.33130 |      hcb4865c_18          17 KB  conda-forge
    ------------------------------------------------------------
                                           Total:        15.4 MB
```

- conda의 base 환경에 라이브러리를 설치합니다.
- brian2 는 2.5.4 버전을 사용합니다. 

### Import

```python
from brian2 import *
```

- brian2의 모든 메소드를 사용한다.

```python
%matplotlib inline
```

- 주피터 상에 그래프를 출력할 수 있도록 설정

### Brians units system

#### Dimensions

- 브라이어는 SNN 상에서 사용하는 단위와 계산을 지원합니다.

```python
20 * volt
```

---

$$20.0V$$

---

- volt는 볼트(V) 단위로 매핑

```python
1000 * amp
```

---

$$1.0kA$$

---

- amp는 암페어(A) 단위로 매핑

```python
1e6 * volt
```

---

$$1.0MV$$

---

- 소수점 단위로 처리 가능

```python
1000 * namp
```

---

$$1.0000000000000002μA$$

---

- 마이크로(μ) 단위 또한 지원합니다.

### Caluations

- 두 가지 수식에 대한 계산도 가능합니다.

```python
(10 * amp) + (100 * mA)
```

---

$$10.1A$$

---

- 호환되는 단위(unit)일 경우 연산이 가능

```python
try:
    5*amp + 10*volt
except DimensionMismatchError as e:
    print(e, "\n계산 단위가 맞지 않아 연산할 수 없습니다.")
```

```cmd
Cannot calculate 5. A + 10. V, units do not match (units are A and V). 
계산 단위가 맞지 않아 연산할 수 없습니다.
```

- 단위(unit)이 맞지 않을 경우 DimensionMismatchError가 발생합니다.
- 공식 튜토리얼에 따르면, 단위의 문제와 Brian2의 라이브러리 버그로 인해 해당 오류가 발생할 수 있다고 합니다.

### 간단한 뉴런 모델

- Brian2 에서는 모든 모델이 미분 방정식 시스템으로 정의되어있습니다.

```python
tau = 10*ms             # 시간 상수

# 미분 방정식 정의
eqs = '''
dv/dt = (1-v)/tau : 1
'''
```

- ':1'는 단위(unit) 표시이며 1은 v(volt)를 의미

```python
G = NeuronGroup(1, eqs)
```

- 매개변수(params)는 (뉴런수, 미분방정식) 입니다.
- Brian2에서는 NeuronGroup이라는 class를 사용해 뉴런의 집단을 만듭니다.

```python
eqs = '''
dv/dt = 1-v : 1
'''
G = NeuronGroup(1, eqs)
run(100*ms)
```

- 만약 시간 상수(tau)를 미분방정식에(eqs)넣지 않을 경우 "DimensionMismatchError"를 발생시킵니다.
- 여기서 v는 막전위

```python
G = NeuronGroup(1, eqs)
run(100*ms)
```

- run 메소드의 매개변수는 (시뮬레이션 진행 시간) 을 의미합니다.
- 위의 코드에서는 100ms 동안 시뮬레이션을 진행합니다.

```output
INFO       No numerical integration method specified for group 'neurongroup_1', using method 'exact' (took 0.00s). [brian2.stateupdaters.base.method_choice]
```

- 위 처럼 INFO로 '수치 적분 방법'을 이용하지 않았다는 문장이 뜨는데, 다음 셀에서 명시적으로 수정하는 방법을 배웁니다.

```python
start_scope()

G = NeuronGroup(1, eqs, method='exact')
print('Before v = %s' % G.v[0])
run(100*ms)
print('After v = %s' % G.v[0])
```

```output
Before v = 0.0
After v = 0.9999546000702376
```

- 기본적으로 모든 변수(여기선 v) 값은 0으로 시작합니다.
- 미분 방정식(dv/dt=(1-v)/tau) 에 따라 v는 1에 가까워질 것으로 예상됩니다.
- v가 1-e^(-t/tau) 값을 가질 것으로 예상하며, 그 결과는 아래에 있습니다.

```python
print('Expected value of v = %s' % (1-exp(-100*ms/tau)))
```

```output
Expected value of v = 0.9999546000702375
```

- 위의 방법으로 미분방정식에 따른 v 값을 알 수 있습니다.

```python
start_scope()

G = NeuronGroup(1, eqs, method='exact') # method는 미분방정식의 해를 정확하게 풀이하겠다는 의미
M = StateMonitor(G, 'v', record=True)

run(30*ms)

plot(M.t/ms, M.v[0])
xlabel('Time (ms)')
ylabel('v')
```

- 위의 코드를 실행시키면 점차적으로 v가 증가하는 것을 알 수 있습니다.
- StateMonitor에 의해 막전위(v)의 변화를 모니터링 해 후에 그래프로 출력할 수 있습니다.
- 모든 뉴런의 값을 기록하기 위해서는 많은 RAM(Random Access Memory)를 사용하기 때문에 지정하는 것이 좋습니다.
- record=0 은 뉴런 0의 모든 값을 기록한다는 의미입니다.

```python
start_scope()

tau = 10*ms
# 미분방정식을 다른 식으로 변경
eqs = '''
dv/dt = (sin(2*pi*100*Hz*t)-v)/tau : 1
'''

# Change to Euler method because exact integrator doesn't work here
G = NeuronGroup(1, eqs, method='euler') # euler 방식 사용
M = StateMonitor(G, 'v', record=0)

G.v = 5 # initial value

run(60*ms)  # 60ms 까지만 시뮬레이션 진행

plot(M.t/ms, M.v[0])
xlabel('Time (ms)')
ylabel('v')
```

- 미분방정식이 다른 전체적인 실행 코드

### 스파이크 동작 추가

```python
start_scope()

tau = 10*ms
eqs = '''
dv/dt = (1-v)/tau : 1
'''

G = NeuronGroup(1, eqs, threshold='v>0.8', reset='v = 0', method='exact')

M = StateMonitor(G, 'v', record=0)
run(50*ms)
plot(M.t/ms, M.v[0])
xlabel('Time (ms)')
ylabel('v');
```

- 스파이크 동작을 추가하기 위해 임계점(threshold) 매개변수와 초기점(reset)을 추가했습니다.
- 막전위(v)가 0.8을 넘을 경우 스파이크를 발생시켜, 0(reset v)으로 초기화 합니다.

```python
start_scope()

G = NeuronGroup(1, eqs, threshold='v>0.8', reset='v = 0', method='exact')

spikemon = SpikeMonitor(G)

run(50*ms)

print('Spike times: %s' % spikemon.t[:])
```

```output
Spike times: [16.  32.1 48.2] ms
```

- Brian2 에서는 스파이크 발생 시점을 저장합니다.

### 휴식기

```python
start_scope()

G = NeuronGroup(1, eqs, threshold='v>0.8', reset='v = 0', refractory=5*ms, method='exact')

statemon = StateMonitor(G, 'v', record=0)
spikemon = SpikeMonitor(G)

run(50*ms)

plot(statemon.t/ms, statemon.v[0])
for t in spikemon.t:
    axvline(t/ms, ls='--', c='C1', lw=3)
xlabel('Time (ms)')
ylabel('v');
```

- 위의 코드를 통해 스파이크의 발생 시점을 그래프로 출력할 수 있습니다.
- 스파이크 발생 후 일정 시간 동안 스파이크가 발생하지 않는 휴지상태(huge) 또는 휴식기는 refractory로 설정합니다.
- 위의 코드에서는 5ms 동안 막전위는 0을 유지합니다.

```python
start_scope()

tau = 10*ms
eqs = '''
dv/dt = (1-v)/tau : 1 (unless refractory)
'''

G = NeuronGroup(1, eqs, threshold='v>0.8', reset='v = 0', refractory=5*ms, method='exact')

statemon = StateMonitor(G, 'v', record=0)
spikemon = SpikeMonitor(G)

run(50*ms)

plot(statemon.t/ms, statemon.v[0])
for t in spikemon.t:
    axvline(t/ms, ls='--', c='C1', lw=3)
xlabel('Time (ms)')
ylabel('v');
```

- 미분방정식 작성시 (unless refractory) 는 작성시 휴식기(refractory)에 막전위는 0으로 고정된다.
- (unless refractory) 를 작성하지 않을 경우, 막전위는 증가하지만, 임계값(threshold)을 넘겨도 스파이크(spike)가 작동하지 않습니다.

### 다중 뉴런
