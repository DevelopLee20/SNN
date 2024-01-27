# 4-brian2-2.md

## 불응성

- 불응성: 불응기의 상태에 있는 것, 즉 스파이크와 유사한 단어

## 내화성

- 내화성(Refractoriness)은 특정 기간동안 다른 스파이크를 발사할 수 없음을 의미

```python
eqs = '''
dv/dt = (1-v)/tau : 1 (unless refractory)
'''

G = NeuronGroup(1, eqs, threshold='v>0.8', reset='v = 0', refractory=5*ms, method='exact')
```

- 소스 코드 작성시 refractory 라는 속성을 이용해 내화성을 추가한다.
- 내화성은 휴식기 또는 휴지기를 의미하기도 한다.
- 미분방정식에 (unless refractory) 를 꼭 넣어주어야 한다.

## unless refractory를 넣지 않으면?

```python
eqs = '''
dv/dt = (1-v)/tau : 1
'''

G = NeuronGroup(1, eqs, threshold='v>0.8', reset='v = 0', refractory=15*ms, method='exact')
```

- 위와 같이 refractory 코드를 작성하였지만 (unless refractory) 코드는 작성하지 않았을 때, 휴식기 일때 임계점(threshold)를 넘어도 스파이크가 발생하지 않는다.
- unless refractory는 휴식기에 막전위 누수 여부이다. 코드를 넣으면 누수 발생에 의해 막전위 그래프가 reset 전위에 고정, 아니면 증가한다.

## 다중 뉴런

```python
start_scope()

N = 100                 # 뉴런수
tau = 10*ms             # 시간상수
# 미분방정식
eqs = '''
dv/dt = (2-v)/tau : 1
'''

G = NeuronGroup(N, eqs, threshold='v>1', reset='v=0', method='exact')

# 0과 1 사이의 균일한 무작위 값으로 각 뉴런 초기화
G.v = 'rand()'

spikemon = SpikeMonitor(G)

run(50*ms)

plot(spikemon.t/ms, spikemon.i, '.k')
xlabel('Time (ms)')
ylabel('Neuron index');
```

- 그림은 raster plot을 나타냄
- 각 뉴런에 스파이크가 발생되는 시점을 나타내는 그래프

```python
start_scope()       # Brian2 라이브러리 시작

# 매개변수 지정
N = 100             # 뉴런의 수
tau = 10*ms         # 뉴런의 시간 상수
v0_max = 3.         # 최대 초기화 전위 설정
duration = 1000*ms  # 시뮬레이션 시간
sigma = 0.2         # 생성되는 난수의 표준편자(확률 변수)

# 미분방정식, 뉴런 모델 정의
eqs = '''
dv/dt = (v0-v)/tau+sigma*xi*tau**-0.5 : 1 (unless refractory)
v0 : 1
'''

G = NeuronGroup(N, eqs, threshold='v>1', reset='v=0', refractory=5*ms, method='euler')
M = SpikeMonitor(G) # 뉴런의 스파이크 발생을 기록

# 초기화 전압을 각 뉴런마다 다르게 적용
G.v0 = 'i*v0_max/(N-1)'

run(duration)

figure(figsize=(12,4))
subplot(121)
plot(M.t/ms, M.i, '.k')
xlabel('Time (ms)')
ylabel('Neuron index')
subplot(122)
plot(G.v0, M.count/duration)
xlabel('v0')
ylabel('Firing rate (sp/s)');
```

$$
\frac{dv}{dt} = \frac{v_0 - v}{\tau} + \sigma \xi \frac{1}{\sqrt{\tau}}
$$

- 첫번째 그림은 단위 시간 각 뉴런의 스파이크 발생 여부이며,
- 두번째 그림은 v0(초기화 전압)에 따른 스파이크 발생 비율이다.

## Synapse(가장 간단한 시냅스)

```python
start_scope()

# LIF 뉴런 모델
eqs = '''
dv/dt = (I-v)/tau : 1
I : 1
tau : second
'''

# 2개의 NeuronGroup 생성  
G = NeuronGroup(2, eqs, threshold='v>1', reset='v = 0', method='exact')
G.I = [2, 0]            # 입력 전류 설정
G.tau = [10, 100]*ms    # 뉴런의 시간 상수(뉴런의 막의 변화량 지정)

# Comment these two lines out to see what happens without Synapses
S = Synapses(G, G, on_pre='v_post += 0.2')  # 시냅스 연결 설정(0 뉴런에서 1 뉴런으로 시냅스 연결해 막전위 0.2 증가)
S.connect(i=0, j=1)                         # 시냅스 뉴런 0->1 연결 설정

M = StateMonitor(G, 'v', record=True)       # 상태 추적

# 뉴런 1이 약간 내려가는데, 이것은 누수(leaky)를 의미

run(100*ms)

plot(M.t/ms, M.v[0], label='Neuron 0')
plot(M.t/ms, M.v[1], label='Neuron 1')
xlabel('Time (ms)')
ylabel('v')
legend();
```
