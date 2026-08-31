---
title: "Joint Edge Server Selection and Data Set Management for Federated-Learning-Enabled Mobile Traffic Prediction"
author: [d.Kim, s.Shin, Joe2357, j17.lee]
categories: [iMES, Publication]
tags: [Publication]
published_at: 2023-08-02 00:00:00 +0900
last_modified_at: 2023-08-02 00:00:00 +0900
description: "- Published in IEEE Internet of Things Journal (Vol.11, Issue.3)"
math: true
---

## Abstract

- 6G 및 Mobile Edge Computing (MEC) 환경에서 지능적인 네트워크 제어를 수행하려면 미래의 **mobile traffic volume을 정확하게 예측**하는 과정이 중요
  - traffic prediction 결과는 network slicing, resource allocation, routing, QoS control과 같은 proactive network management에 활용될 수 있음
  - 기존 AI 기반 traffic prediction 연구의 대부분은 여러 Base Station (BS) 또는 MEC server에서 수집한 raw traffic data를 중앙 서버로 전송하여 모델을 학습하는 **중앙집중식 학습 방식**을 사용
    - 대규모 데이터 전송에 따른 네트워크 지연과 통신 비용 발생
    - 중앙 학습 서버의 연산 및 저장 병목현상 발생
    - raw traffic data가 중앙에 수집되기 때문에 privacy 측면의 우려가 존재
- 본 논문에서는 MEC server의 분산·병렬 연산 능력을 활용하여 mobile traffic prediction model을 빠르고 안전하게 학습하기 위한 **Federated Learning (FL) 기반 framework** 제안
  - 각 MEC server는 자신이 수집한 local traffic data로 local model을 학습
  - Core Network (CN)의 root NWDAF는 local model update만 수집하여 global model을 생성
  - raw traffic data는 MEC server 외부로 전송하지 않음
- 단순히 FL을 적용하는 것에 그치지 않고, 매 aggregation interval마다 다음 두 요소를 공동으로 최적화
  - local training에 참여할 **MEC server selection**
  - 선택된 각 MEC server가 local training에 사용할 **data set utilization ratio**
- global model의 정확도와 FL training cost 사이의 tradeoff를 수학적으로 모델링
  - training cost는 local training latency와 computation energy consumption을 함께 고려
  - 정확도 추정 모델은 두 가지 형태로 구성
    - 제한된 data utilization 범위에서 활용 가능한 **linear accuracy model**
    - 일반적인 증가·포화 특성을 나타내는 **concave accuracy model**
- linear accuracy model을 사용한 문제는 Mixed-Integer Nonlinear Programming (MINLP) 형태로 구성되며 일반적으로 NP-hard
  - binary relaxation과 auxiliary variable을 통해 문제를 변환
  - block coordinate descent 방식으로 반복적으로 푸는 Linear Programming (LP) 기반 near-optimal algorithm 제안
- concave accuracy model을 사용한 일반 문제는 relaxation 이후에도 비선형성이 강하게 남으므로 **genetic-based heuristic algorithm** 제안
  - traffic pattern similarity를 고려한 priority-aware selection
  - population diversity에 따라 crossover/mutation probability를 조절하는 adaptive mechanism 적용
- 실제 Milano 및 Trento mobile traffic data를 사용한 실험 결과
  - baseline과 유사한 prediction accuracy를 유지
  - 전체 MEC server 중 약 66%만 참여
  - FL training energy consumption을 약 45% 절감
  - 논문의 결론에서는 약 40% 수준의 energy reduction을 달성한 것으로 정리



## I. Introduction

- 6G와 MEC 기술이 발전하면서 Artificial Intelligence (AI) / Machine Learning (ML)은 네트워크 운영을 자동화하기 위한 핵심 기술로 간주되고 있음
  - traffic prediction
  - application classification
  - intrusion / anomaly detection
  - resource scheduling 및 allocation
  - routing
- 3rd Generation Partnership Project (3GPP)는 AI 기반 network control을 지원하기 위해 **Network Data Analytics Function (NWDAF)** 을 표준화
  - NWDAF는 network function에 필요한 분석 결과를 제공
  - ML model training을 담당하는 Model Training Logical Function (MTLF)
  - inference와 analytics를 담당하는 Analytics Logical Function (AnLF)
- 다양한 device와 service가 공존하는 6G-MEC 환경에서는 traffic demand가 시간과 공간에 따라 크게 달라짐
  - smartphones, sensors, vehicles, drones, industrial machines 등 heterogeneous device가 존재
  - 미래 traffic을 미리 예측하면 필요한 radio / computing / network resource를 사전에 배치할 수 있음
- 기존 mobile traffic prediction 방식의 한계
  - 여러 BS에서 수집된 방대한 raw data를 중앙 training server로 전송
  - 중앙 서버에서 하나의 prediction model을 학습
  - 발생 가능한 문제
    - data transmission latency
    - centralized processing bottleneck
    - network and storage cost
    - privacy leakage
- FL은 이러한 중앙집중식 학습의 문제를 완화할 수 있는 대표적인 distributed learning paradigm
  - 각 client가 local data로 local model을 병렬 학습
  - 중앙 서버는 local data가 아니라 gradient 또는 model weight를 aggregation
  - distributed computation을 활용하므로 training speed와 energy efficiency를 향상시킬 가능성이 있음
  - raw data를 직접 공유하지 않기 때문에 privacy 측면에서도 이점이 있음
- 하지만 FL을 MEC 기반 mobile traffic prediction에 적용할 때 모든 MEC server가 항상 모든 data를 사용하도록 하면 비용이 과도하게 증가
  - MEC server마다 CPU frequency, local data size, processing cost가 다름
  - 느린 server가 하나라도 포함되면 synchronous FL round의 latency가 증가
  - 많은 data를 사용할수록 accuracy는 향상될 수 있지만 local training latency와 energy consumption도 함께 증가
- 기존 연구들은 mobile device selection, bandwidth allocation, CPU/GPU frequency control 등을 고려했지만, MEC server 기반 traffic prediction에서 다음 두 요소를 함께 최적화한 연구는 부족
  - 어느 MEC server를 FL에 참여시킬 것인가?
  - 참여한 MEC server가 local data 중 어느 정도를 사용할 것인가?
- 본 논문의 핵심 목적
  - prediction accuracy를 유지하면서 FL operation cost를 줄일 수 있도록 **MEC server selection과 data set management를 공동 최적화**
  - standardized distributed NWDAF architecture에 적용 가능한 형태로 설계

- Contribution
  - FL-enabled mobile traffic prediction을 위한 distributed NWDAF 기반 system model 제안
  - accuracy와 latency를 결합한 Learning Efficiency (LE), 그리고 computation energy consumption을 포함하는 cost function 설계
  - linear accuracy estimation model을 사용한 simplified problem formulation
    - binary relaxation
    - max latency term의 affine transformation
    - block coordinate descent + LP 기반 near-optimal solution
  - concave accuracy estimation model을 사용한 general problem formulation
    - traffic-pattern-aware selection
    - adaptive crossover / mutation
    - genetic-based heuristic solution
  - FedAvg, FedDA, Data set and Computation Management (DCM) 방식과 비교
  - 실제 Milano / Trento mobile traffic data에서 competitive accuracy와 energy reduction을 확인



## II. Related Work

### A. AI-Driven Network Management in 3GPP

- 3GPP는 zero-touch network control을 목표로 5G Core Network에 NWDAF를 도입
  - Network Function (NF)은 NWDAF로부터 두 가지 service를 이용할 수 있음
    - `Nnwdaf_EventsSubscription` : 특정 analytics event를 구독
    - `Nnwdaf_AnalyticsInfo` : 필요한 analytics information을 직접 요청
- NWDAF 내부의 주요 논리 기능
  - MTLF : ML model training
  - AnLF : trained model을 이용한 inference / analytics
- NWDAF 적용 예시
  - Policy Control Function (PCF)이 QoS parameter를 결정하도록 지원
  - Network Slice Selection Function (NSSF)의 slice selection 지원
  - service experience prediction
  - load analytics
  - UE behavior / mobility pattern prediction
- 본 논문은 single NWDAF가 모든 데이터와 연산을 독점하는 구조가 아니라, CN의 root NWDAF와 MEC의 leaf NWDAF가 협력하는 **distributed NWDAF** 구조를 활용

### B. Mobile Traffic Prediction

- mobile traffic prediction 방식은 크게 세 종류로 구분

**Simple Method**

- historical average를 기반으로 미래 traffic을 예측
  - Exponentially Weighted Moving Average (EWMA) 등이 대표적
- traffic 변화가 느리고 완만할 때는 효과적
- 복잡한 spatio-temporal pattern과 sudden burst를 표현하기 어려움

**Parametric Method**

- traffic이 특정 probability distribution 또는 statistical model을 따른다고 가정
  - ARIMA와 그 변형
  - Markov model
  - entropy-based model
  - covariance function
  - alpha-stable model
- regular pattern이 강한 traffic에는 높은 성능을 보일 수 있음
- random variation이 커질수록 성능이 저하될 수 있음

**Nonparametric Method**

- distribution에 대한 명시적인 가정 없이 data로부터 prediction model을 학습
  - LSTM
  - CNN
  - auto-encoder
  - multitask learning
  - Federated Learning
- time-series traffic의 temporal dependency를 학습하기 위해 LSTM이 자주 사용됨
- spatial dependency를 함께 반영하기 위해 CNN, attention, clustering 등의 기법이 결합됨
- FedDA는 mobile traffic pattern과 geographical information을 활용한 clustering 및 dual-attention aggregation을 통해 FL convergence speed를 향상시킨 방식

### C. Resource Management in Federated Learning

- FL은 local training과 aggregation을 반복하는 distributed processing system이므로 resource management가 매우 중요
- 주요 최적화 지표
  - energy consumption
  - learning convergence time
  - prediction accuracy
- 지표 사이에는 본질적인 tradeoff가 존재
  - 참여 client 수와 data utilization을 줄이면 energy는 절감
  - 반면 accuracy와 convergence performance는 저하될 수 있음
- 기존 접근
  - 특정 accuracy / latency / energy requirement를 constraint로 부여
  - 여러 지표를 weighted sum 형태의 objective로 결합
  - client selection, bandwidth, CPU frequency, data selection 등을 최적화
- 본 논문은 MEC server participation과 local data utilization을 동시에 결정하면서 **Learning Efficiency와 energy consumption을 직접 tradeoff**하는 것이 차별점



## III. Motivating Example for Estimating Accuracy Model

- 일반적으로 client가 local training에 사용하는 data의 비율이 증가할수록 global model accuracy도 증가
- 하지만 accuracy는 data가 늘어나는 만큼 계속 선형적으로 증가하지 않고 일정 수준에서 포화되는 **increasing concave tendency**를 보임
- 논문에서는 Milano와 Trento data set으로 data utilization ratio와 training accuracy 사이의 관계를 직접 측정
  - 7주 분량은 training data
  - 마지막 1주는 test data
  - experiment를 50회 반복

![figure1](./assets/figure1.png)

- Fig. 1(a)
  - 각 client가 전체 local data의 10%에서 90%까지 사용하는 상황을 실험
  - root function이 concave accuracy behavior를 매우 잘 근사
  - RMSE
    - Milano : 0.068%
    - Trento : 0.159%
- Fig. 1(b)
  - data utilization 범위를 10% ~ 30%로 좁히면 linear model도 매우 정확한 근사 가능
  - RMSE
    - Milano : 0.021%
    - Trento : 0.024%
- 이를 바탕으로 문제를 두 단계로 분리
  - **Simplified case** : narrow utilization range를 가정한 linear model
  - **General case** : 전체적인 saturation behavior를 반영한 concave model



## IV. System Model and Problem Formulation

**Distributed NWDAF-Based FL Architecture**

![figure2](./assets/figure2.png)

- 전체 system은 centralized Core Network와 여러 MEC server로 구성
  - MEC server 집합 : $\mathsf{K}$
  - MEC server 수 : $K = |\mathsf{K}|$
  - 각 MEC server는 하나의 BS와 직접 연결되어 UE traffic data를 수집
- CN에는 **root NWDAF** 배치
  - Communication Module
  - Learning Module
    - Aggregation Module
    - Management Module
- 각 MEC server에는 **leaf NWDAF** 배치
  - Communication Module
  - MTLF Module : local training
  - AnLF Module : traffic prediction inference
  - Data Storage
- 각 MEC server $k$의 local data set

$$
\mathsf{D}_k = \left\{D_k^1, D_k^2, \cdots, D_k^Z\right\}
$$

- $D_k$ : MEC server $k$의 total local training data size
- $x_k \in \{0,1\}$ : MEC server $k$의 FL participation 여부
  - $x_k=1$ : selected
  - $x_k=0$ : not selected
- $v_k$ : MEC server $k$가 local training에 사용하는 data set ratio
  - 실제 사용 data amount : $v_kD_k$

**Overall FL Procedure**

**Step 1. Status Collection and Configuration**

- root NWDAF가 모든 leaf NWDAF로부터 상태정보 수집
  - local data size $D_k$
  - CPU frequency $f_k$
  - CPU cycle requirement $c_k$
  - energy consumption profile
- Management Module이 MEC server selection $x_k$와 data utilization $v_k$를 결정
- root NWDAF는 initial global model $w^{(0)}$와 local training configuration을 selected MEC server에 전달

**Step 2. Local Training**

- selected MEC server $k$는 $v_kD_k$만큼의 local data를 사용하여 local model을 학습
- round $r$에서 local loss function은 $L_k(w_k^{(r)})$

$$
w_k^{(r)*} = \arg\min_{w_k^{(r)}} L_k\left(w_k^{(r)}\right)
$$

- 논문에서는 neural network regression에서 널리 사용되는 Mean Squared Error (MSE)를 loss function으로 사용
- local training 후 model update $w_k^{(r)*}$를 root NWDAF로 전송

**Step 3. Global Aggregation**

- selected MEC server 집합을 $\mathsf{K}'$라고 할 때 global loss function

$$
L\left(w^{(r)}\right)
= \frac{1}{K'} \sum_{k\in\mathsf{K}'} L\left(w_k^{(r)}\right)
$$

- aggregation은 FedAvg 또는 FedDA 등의 방법으로 수행 가능
- updated global model $w^{(r)}$를 selected MEC server에 다시 배포
- target accuracy 또는 stopping condition을 만족할 때까지 Step 2와 Step 3 반복



### A. Learning Efficiency Analysis for FL Process

**Local Computation Latency**

- local training을 수행하기 위한 MEC server $k$의 computation latency

$$
L_k = I_l(\theta)\frac{c_kv_kD_k}{f_k}
$$

- $I_l(\theta)$ : target local accuracy $\theta$를 달성하기 위해 필요한 local iteration 수
- $c_k$ : 1-bit data를 처리하기 위해 필요한 CPU cycle 수
- $f_k$ : CPU frequency
- $v_kD_k$ : 실제 local training에 사용하는 data amount
- synchronous FL에서는 selected MEC server가 모두 update를 전송해야 aggregation을 수행할 수 있음
- 따라서 total service latency는 가장 느린 selected server에 의해 결정

$$
L = \max_{k\in\mathsf{K}} L_kx_k
$$

- MEC server와 CN이 over-provisioned wired network로 연결되어 있다고 가정
  - model upload / download transmission latency는 local computation latency보다 매우 작다고 보고 무시
  - global aggregation latency도 무시

**Learning Efficiency**

- 정확도와 latency를 하나의 metric으로 결합하기 위해 Learning Efficiency (LE)를 정의

$$
LE = \frac{A}{L}
$$

- $A$ : selected MEC server와 utilized data에 따른 estimated global accuracy tendency
- 동일한 accuracy라면 latency가 짧을수록 LE가 높음
- 동일한 latency라면 더 높은 accuracy를 얻을수록 LE가 높음

### B. Energy Consumption Analysis for FL Process

- MEC server $k$의 local training energy

$$
E_k = I_l(\theta)\frac{\alpha_k}{2}f_k^2c_kv_kD_k
$$

- $\alpha_k/2$ : MEC server chipset의 effective capacitance coefficient
- selected server 전체의 energy consumption

$$
E = \sum_{k\in\mathsf{K}} E_kx_k
$$

- transmission energy 역시 computation energy보다 작다고 가정하여 제외



## V. Proposed FL Framework

### A. Problem Formulation

**Utility and Cost Function**

- objective는 Learning Efficiency를 높이고 energy consumption을 줄이는 것
- weighted-sum 방식의 utility function

$$
U(v_k,x_k) = \frac{A}{L} - \beta_EE
$$

- $\beta_E \ge 0$ : energy reduction을 얼마나 중요하게 고려할지 나타내는 preference weight
- optimization을 minimization 형태로 표현하기 위해 cost function 정의

$$
C(v_k,x_k) = -\frac{A}{L} + \beta_EE
$$

- $\beta_E$가 작으면 LE 개선을 더 중요하게 고려
- $\beta_E$가 크면 energy reduction을 더 강하게 선호



**Simplified Case: Linear Accuracy Model**

- narrow data utilization range에서는 global accuracy tendency를 linear function으로 근사

$$
A_l = \xi\sum_{k\in\mathsf{K}}D_kv_kx_k
$$

- $\xi$ : ML model에 따른 accuracy coefficient
- notation을 간소화하기 위해

$$
G_k = I_l(\theta)\frac{c_kD_k}{f_k}
$$

$$
H_k = I_l(\theta)\frac{\alpha_k}{2}f_k^2c_kD_k
$$

**Problem 1**

$$
\begin{aligned}
\min_{x_k,v_k}\quad
&-\frac{\sum_{k\in\mathsf{K}}D_kv_kx_k}
{\max_{k\in\mathsf{K}}G_kv_kx_k}
+\beta_E\sum_{k\in\mathsf{K}}H_kv_kx_k \\
\text{s.t.}\quad
&\gamma_d \le v_k \le 1, \quad \forall k \\
&x_k\in\{0,1\}, \quad \forall k \\
&\sum_{k\in\mathsf{K}}x_k \ge \gamma_sK
\end{aligned}
$$

- $\gamma_d$ : selected MEC server가 최소한 사용해야 하는 data ratio
- $\gamma_s$ : 전체 MEC server 중 최소 participation ratio
- binary variable과 nonlinear ratio / max term이 함께 존재하는 MINLP 문제
- 일반적으로 NP-hard이므로 그대로는 효율적인 최적해 계산이 어려움

**Binary Relaxation**

- binary selection variable $x_k$를 continuous variable $\tilde{x}_k$로 relaxation

$$
0 \le \tilde{x}_k \le 1
$$

- 최종 solution을 얻은 뒤 threshold 0.5를 기준으로 binary decision으로 복원
  - $\tilde{x}_k\ge 0.5 \Rightarrow x_k=1$
  - $\tilde{x}_k<0.5 \Rightarrow x_k=0$

**Max Latency Transformation**

- objective의 `max` term은 differentiable하지 않으므로 auxiliary variable $t$ 도입

$$
t = \max_{k\in\mathsf{K}} G_kv_k\tilde{x}_k
$$

- 다음 constraint로 변환

$$
G_kv_k\tilde{x}_k \le t, \quad \forall k
$$

- transformed problem

$$
\begin{aligned}
\min_{\tilde{x}_k,v_k,t}\quad
&-\frac{\sum_{k\in\mathsf{K}}D_kv_k\tilde{x}_k}{t}
+\beta_E\sum_{k\in\mathsf{K}}H_kv_k\tilde{x}_k \\
\text{s.t.}\quad
&\gamma_d \le v_k \le 1 \\
&0 \le \tilde{x}_k \le 1 \\
&\sum_{k\in\mathsf{K}}\tilde{x}_k \ge \gamma_sK \\
&G_kv_k\tilde{x}_k \le t
\end{aligned}
$$

- 논문은 $t$를 고정하면 $(\tilde{x}_k,v_k)$에 대해 LP로 다룰 수 있음을 보임
- objective는 $t$에 대해 strictly increasing하므로 optimum에서는 latency constraint의 lower boundary가 활성화

**LP-Based Block Coordinate Descent Algorithm**

```pseudocode
Algorithm 1 Proposed LP Problem Algorithm

Input: βE, γd, γs, Ek, Gk, Dk, Hk, θc
Initialize: vk, x̃k, t within the constraints
Output: x*k, v*kDk

1| while True do
2|   t ← maxk Gkvkx̃k
3|   vk ← solve Problem 3 with fixed x̃k and t
4|   x̃k ← solve Problem 3 with fixed vk and t
5|   Ci ← C(vk, x̃k, t)
6|   if |Ci − Ci−1| < θc then
7|     break
8|   end if
9| end while
10| for each k do
11|   if x̃k ≥ 0.5 then xk ← 1
12|   else xk ← 0
13| end for
14| return x*k, v*k
```

- block coordinate descent 방식
  - 다른 variable을 고정한 채 $v_k$ optimization
  - $v_k$를 고정한 채 $\tilde{x}_k$ optimization
  - 현재 solution으로 $t$ update
  - cost difference가 threshold $\theta_c$보다 작아질 때까지 반복
- 논문은 block coordinate descent의 성질에 따라 **sublinear convergence rate**를 가짐을 설명
- 장점
  - original MINLP보다 훨씬 tractable
  - near-optimal solution을 얻으면서 computation complexity를 크게 완화



**General Case: Concave Accuracy Model**

- 일반적인 data-accuracy 관계를 반영하기 위해 root 형태의 concave function 사용

$$
A_c = \xi\sqrt{\sum_{k\in\mathsf{K}}D_kv_kx_k}
$$

- 이에 따른 optimization problem

$$
\begin{aligned}
\min_{x_k,v_k}\quad
&-\xi\frac{\sqrt{\sum_{k\in\mathsf{K}}D_kv_kx_k}}
{\max_{k\in\mathsf{K}}G_kv_k}
+\beta_E\sum_{k\in\mathsf{K}}H_kv_kx_k \\
\text{s.t.}\quad
&\gamma_d \le v_k \le 1 \\
&x_k\in\{0,1\} \\
&\sum_{k\in\mathsf{K}}x_k \ge \gamma_sK
\end{aligned}
$$

- square-root accuracy, binary selection, max latency가 함께 존재하는 nonconvex problem
- simple relaxation만으로 efficient closed-form 또는 LP solution을 얻기 어려움
- 여러 local optimum과 potential solution space를 탐색하기 위해 genetic algorithm 사용



### B. Preliminaries of Genetic Algorithm

- Genetic algorithm은 chromosome, gene, generation, fitness 개념을 기반으로 selection, crossover, mutation을 반복하여 해를 탐색한다.

### C. Proposed Genetic-Based Heuristic Algorithm

![figure3](./assets/figure3.png)

**Chromosome and Fitness Design**

- 하나의 chromosome은 모든 MEC server의 decision을 포함

$$
CH_j = [v_1,x_1,v_2,x_2,\cdots,v_K,x_K]
$$

- $x_k$ : binary gene
- $v_k$ : $[\gamma_d,1]$ 범위의 float gene
- objective는 cost minimization이므로 fitness는 negative cost 형태로 구성하여 큰 값이 좋은 chromosome이 되도록 설계
- initial population은 random initialization 후 minimum participation constraint를 만족하는지 validation

**Priority-Aware Selection**

- 일반적인 genetic algorithm은 높은 fitness의 chromosome이 parent로 선택될 가능성을 높이는 roulette-wheel selection을 사용
- 본 논문은 추가적으로 MEC server의 **traffic pattern similarity**를 priority로 반영
  - 각 MEC server의 augmented traffic pattern과 전체 centroid 사이의 distance 계산
  - centroid와 유사한 traffic pattern을 가진 MEC server에 높은 priority 부여
- 목적
  - 지나치게 다른 traffic distribution을 가진 MEC server가 함께 선택될 때 발생할 수 있는 convergence degradation 완화
  - non-IID effect를 줄이고 global model convergence speed 개선
- raw traffic data 자체를 전달하지 않고 statistical average 기반 augmented data를 활용

**Adaptive Crossover**

- 두 parent chromosome을 결합하기 위해 double-point crossover 사용
- population diversity를 fitness distribution으로 측정

$$
C'_{(i)} = \frac{C_{\max}^{(i)}-\bar{C}^{(i)}}{C_{\max}^{(i)}}
$$

- $C_{\max}^{(i)}$ : $i$번째 generation의 maximum fitness
- $\bar{C}^{(i)}$ : average fitness
- diversity가 낮으면 population이 빠르게 한 solution으로 몰리고 있다는 의미
  - local optimum을 피하기 위해 crossover probability를 증가
- diversity가 높으면 exploration이 충분하므로 stable convergence를 위해 crossover probability를 낮춤

$$
CP_i = \left(1-C'_{(i)}\right)CP_{init}
$$

**Adaptive Mutation**

- variable type에 따라 mutation 방식을 구분
  - $x_k$ : binary encoding mutation
  - $v_k$ : float encoding mutation
- $v_k$는 mutation 이후에도 $[\gamma_d,1]$ 범위를 유지
- mutation probability

$$
MP_i = \left(1-C'_{(i)}\right)MP_{init}
$$

- diversity가 낮을수록 mutation probability를 높여 새로운 solution space를 탐색

**Overall Procedure**

```pseudocode
Algorithm 2 Proposed Genetic-Based Heuristic Algorithm

1| CHlist ← randomly initialize chromosomes with xk and vk
2| CHlist ← ValidationCheck(CHlist, γs)
3| Flist ← evaluate fitness for every chromosome
4| while iteration ≤ Imax do
5|   CH*1, CH*2 ← RoulettePriority(CHlist, Flist, TPlist)
6|   if U(0,1) ≤ CPi then
7|     CHnew ← DoubleCrossover(CH*1, CH*2)
8|   end if
9|   if U(0,1) ≤ MPi then
10|    CHnew ← Mutation(CHnew)
11|  end if
12|  CHlist ← CHlist ∪ CHnew
13|  CHlist ← ValidationCheck(CHlist, γs)
14|  Flist ← reevaluate fitness
15|  CHopt ← chromosome with maximum fitness
16| end while
17| return CHopt
```

- root NWDAF의 Management Module에서 실행
- input
  - local resource information : $f_k$, $D_k$, $c_k$
  - traffic pattern priority
  - energy coefficient
- output
  - selected MEC server set $x_k^*$
  - selected server별 data utilization ratio $v_k^*$



## VI. Performance Evaluation

### A. Evaluation of Proposed Cost Model

- linear accuracy model을 사용한 proposed framework의 cost efficiency를 numerical simulation으로 평가
- benchmark
  - FedAvg / FedDA with $p=1.0$
  - FedAvg / FedDA with $p=0.8$
  - FedAvg / FedDA with $p=0.6$
  - Data set and Computation Management (DCM)
- $p$ : MEC server participation ratio
- FedAvg / FedDA는 random MEC server selection을 사용하고 data amount는 별도로 관리하지 않기 때문에 proposed cost model 관점에서는 같은 numerical performance를 가짐
- Monte Carlo simulation
  - 1000 random samples
  - $D_k \in [5\text{ MB},10\text{ MB}]$
  - $f_k \in [10^9,10^{10}]$
  - $c_k \in [50,100]$
  - $\beta_E = 4\times10^{-5}$
  - $\alpha_k = 10^{-28}$
  - $\gamma_s = 0.6$
  - $\gamma_d = 0.7$

![figure4](./assets/figure4.png)

- Fig. 4(a) Total Cost
  - proposed framework가 모든 MEC server 수 설정에서 가장 낮은 total cost 달성
  - MEC server 수가 증가할수록 benchmark 대비 성능 차이가 커짐
  - cost variance도 상대적으로 작음
- Fig. 4(b) Energy Consumption
  - proposed framework가 가장 낮은 energy consumption 달성
  - optimal MEC selection과 data utilization을 동시에 수행한 효과
- Fig. 4(c) Learning Efficiency
  - energy를 크게 줄이면서도 FedAvg/FedDA($p=1$), DCM과 유사한 LE 유지
  - $p=0.6$, $p=0.8$처럼 참여 server를 단순히 줄이기만 하면 LE가 크게 저하
- 결론
  - MEC server 수만 줄이는 것은 좋은 해결책이 아님
  - **server selection과 data set management를 공동으로 수행해야 accuracy-latency-energy tradeoff를 효과적으로 제어**할 수 있음

### B. Evaluation of Prediction Accuracy

**Mobile Traffic Data Set**

- Telecom Italia Big Data Challenge에서 공개된 실제 mobile traffic data 사용
- 대상 지역
  - Milano : 10,000 grid cells
  - Trento : 6,575 grid cells
- 기간
  - 2013년 11월부터 2014년 1월까지 약 2개월
  - 10분 간격으로 기록
- traffic type
  - call
  - SMS
  - Internet
- data split
  - 7주 training
  - 마지막 1주 testing
- sliding window scheme 사용
  - closeness dependency = 3
  - periodicity dependency = 3
- baseline model
  - LSTM
  - FedAvg
  - FedDA

**Prediction Result**

![figure5](./assets/figure5.png)

- Fig. 5는 Milano와 Trento에서 ground truth와 각 방식의 prediction curve를 비교
- proposed framework를 FedAvg와 FedDA 위에 적용해도 baseline과 거의 유사한 traffic trend를 예측
- 일부 point에서 작은 오차는 존재하지만 전체적인 periodic pattern과 burst variation을 안정적으로 추적

| Method | Milano MSE | Milano MAE | Trento MSE | Trento MAE |
| :--- | ---: | ---: | ---: | ---: |
| LSTM | 0.1697 | 0.2936 | 4.6976 | 1.1193 |
| FedAvg | 0.1096 | 0.2320 | 4.8379 | 1.0688 |
| FedDA | 0.1192 | 0.2468 | 4.7068 | 1.0478 |
| Proposed with FedAvg | 0.1180 | 0.2385 | 4.6357 | 1.0934 |
| Proposed with FedDA | 0.1123 | 0.2367 | 4.5111 | 0.9621 |

- proposed framework는 fewer MEC servers와 smaller data set을 사용하지만 prediction error가 baseline과 비슷한 수준
- Trento에서는 Proposed with FedDA가 MSE와 MAE 모두 가장 좋은 결과를 기록

**Convergence Performance**

![figure6](./assets/figure6.png)

- R-squared score를 communication round에 따라 비교
- Milano
  - 모든 방식이 약 0.9 수준으로 빠르게 수렴
  - proposed framework가 baseline과 거의 같은 최종 accuracy 유지
- Trento
  - data distribution과 pattern이 더 복잡하여 방식 간 차이가 크게 나타남
  - FedDA 계열은 traffic pattern과 geographical information을 사용하는 clustering 덕분에 FedAvg보다 더 적은 round로 높은 accuracy에 도달
- proposed framework는 server와 data를 줄였음에도 underlying aggregation method의 convergence tendency를 유지

**Cost Effectiveness**

![figure7](./assets/figure7.png)

- 전체 participation을 사용하는 FedAvg / FedDA 대비
  - MEC server participation ratio : 100% → 66%
  - energy consumption : 2814.99 J → 1266.7455 J
  - 약 45% energy reduction
  - Learning Efficiency : 0.589 → 0.512
- LE가 소폭 감소하지만 energy cost는 크게 줄어드는 결과
- mobile traffic prediction accuracy가 competitive한 수준으로 유지된다는 점까지 고려하면 cost-effective tradeoff를 달성

**Influence of Priority and Minimum Participation Ratio**

![figure8](./assets/figure8.png)

**Traffic Pattern Priority**

- genetic selection에서 centroid와 유사한 MEC server를 우선 선택하는 비율을 비교
  - 50%
  - 70%
  - without priority
- 70% priority 설정이 가장 높은 R-squared score를 달성
- priority 비율이 너무 낮으면 non-IID data의 영향이 충분히 완화되지 않음
- 반대로 특정 server만 계속 우선하면 diversity가 줄어 generalization에 악영향을 줄 수 있으므로 적절한 ratio 선택이 필요

**Minimum MEC Selection Ratio $\gamma_s$**

- $\gamma_s = 0.5, 0.65, 0.8$ 비교
- $\gamma_s$가 높을수록 더 많은 MEC server가 참여하므로 prediction accuracy 증가
- 하지만 energy reduction의 여지는 감소
- 따라서 $\gamma_s$는 service provider가 accuracy-energy tradeoff와 QoS policy를 고려하여 결정해야 하는 parameter

**Influence of Energy Preference $\beta_E$**

![figure9](./assets/figure9.png)

- $\beta_E$가 증가하면 objective에서 energy term의 weight가 커짐
- Fig. 9(a)
  - energy reduction을 더 중요하게 고려할수록 total cost 자체는 증가
  - 이는 objective의 positive energy penalty coefficient가 커지기 때문
- Fig. 9(b)
  - 실제 선택 결과의 energy consumption은 $\beta_E$가 증가할수록 감소
- 논문은 $\beta_E=4\times10^{-5}$를 knee point로 선택
  - 그 이후에는 추가 energy reduction이 작아지는 반면 total cost 증가가 커지는 지점



## VII. Conclusion

- FL 기반 mobile traffic prediction에서 모든 MEC server와 모든 local data를 사용하는 방식은 높은 training cost를 유발
- 본 논문은 다음 두 요소를 공동으로 최적화하는 framework를 제안
  - MEC server selection
  - local data set utilization ratio
- 정확도와 latency를 결합한 Learning Efficiency와 computation energy를 weighted cost로 구성
- linear accuracy model
  - MINLP formulation
  - binary relaxation 및 max-term transformation
  - LP + block coordinate descent 기반 near-optimal algorithm
- concave accuracy model
  - nonconvex formulation
  - traffic-pattern-aware adaptive genetic algorithm
- 실제 Milano / Trento traffic data를 사용한 결과
  - baseline과 competitive한 prediction accuracy 유지
  - 전체 MEC server 중 약 66%만 선택
  - energy consumption 약 45% 절감
  - 논문 전체 결론 기준 약 40% 수준의 training energy reduction 달성
- 핵심 메시지
  - FL의 cost를 줄이기 위해 단순히 participant 수를 줄이는 것만으로는 충분하지 않음
  - **어떤 MEC server를 선택할지와 각 server가 얼마만큼의 data를 사용할지를 함께 결정해야 accuracy, latency, energy 사이의 균형을 효과적으로 달성할 수 있음**
