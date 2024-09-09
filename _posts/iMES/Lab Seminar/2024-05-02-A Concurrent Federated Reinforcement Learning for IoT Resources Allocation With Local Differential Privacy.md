---
title: "A Concurrent Federated Reinforcement Learning for IoT Resources Allocation With Local Differential Privacy"
author: Joe2357
categories: [1. iMES, Lab Seminar]
tags: [Lab Seminar]
math: true
---

> 2024 / 5 / 2 Joint 세미나

## Abstract

- edge 기반 IoT 시스템에서의 자원 할당은 어려운 작업이 될 수 있음
  - 다양한 resource의 배분 전략을 설계하는 연구가 진행됨
  - 강화학습 : resource 할당 기법의 효율성 극대화하는 방법 중 하나
- 일반적인 강화학습 : host가 local parameter를 중앙서버에 업로드해야함
  - host가 처리하는 데이터 중 일부가 민감할 수 있음 -> 개인정보 보호 문제
- 문제 해결 :: 지역 차등 프라이버시에 기반한 <u>concurrent joint RL</u> (동시 공동 강화학습) 방법 개발
  - 개인정보 보호를 유지하기 위해 local train 중에 noise 생성
  - 최적의 resource 할당 전략을 고안하기 위해 중앙서버와 공동으로 decision 진행
- 결론 : 위 접근법이 edge host의 프라이버시를 유지하면서 높은 성능을 유지한다



## Introduction

- IoT 시스템에서 resource 사용을 극대화해야할 필요성이 대두됨
  - 그러나 이를 달성하기 위한 효율적인 방안을 만드는 것은 어려움
  - 자원 최적화 전략 : **다양한 운영 요구사항 및 SLA 등의 성능요구사항** 등을 고려해야 함 (동적이어야함 + 다양한 운영체제 핸들링 + 개인정보 보호 규정 등) 
- <u>강화학습을 이용한 최적의 할당 전략</u>을 학습하는 것은 효율성을 극대화할 전략 중 하나
  - 단점 : **개인정보 침해**를 방지하지 못할 수 있음
    - 기존의 강화학습에서 개인정보를 보호하면서도 전략을 얻는 연구도 있었지만, 오늘날 딥러닝 기반의 자원 할당 방법은 대부분 <u>개인정보 보호 문제를 고려하지 않음</u>
      - 더 좋은 성능을 산출 // 일반 데이터 보호 규정이 있는 유럽과 같은 곳에서는 사용할 수 없음
      - 모델 반전, 데이터 재구성 공격 등의 방법으로 시스템이나 데이터를 손상시키려는 악의적인 서버에 취약함
- 프라이버시 보호를 위해서는 FL 프레임워크가 필요
  - FL : 클라이언트가 업로드한 gradient parameter로 집계된 모델을 작성 -> 클라이언트는 raw data를 업로드할 필요가 없음
  - FRL : 데이터 프라이버시를 보존하면서 optimal decision making을 지원 (FL + RL)
    - 아직 부족함 :: 클라이언트가 model gradient를 서버에 업로드하면 이것으로부터 private data를 수집할 수도 있음
    - 추가로, 공격자가 model gradient만으로도 model input data를 재구성할 수 있음을 보인 연구가 있음
- 로컬 차등 개인 정보 보호는 신뢰할 수 없는 타사 서버의 문제를 해결하려고 함
  - 이러한 방식에서는 사용자 데이터가 제3자 서버에 업로드되기 전에 로컬로 방해되어 데이터의 개인정보가 보호됨
- edge host의 개인정보 보호에 대한 보장을 제공하기 위해 **local differential privacy** (LDP)를 사용할 가능성을 확인
  - 자원을 구성하는 방법으로는 FL 구조에 의존
  - LDP : 악의적인 서버가 edge host에 의해 업로드된 data에 대해 '모델 반전' 공격을 수행하는 것을 방지할 수 있음
    - 단점 : 시스템 성능에 영향을 미칠 수 있음
    - 해결법 : 강화학습 적용 (edge agent와의 상호작용을 통해 누적 reward의 일부를 최대화하려는 decision-making process 지원)
      - agent : 시행착오적인 방법으로 action을 배우고 개선 -> 최적의 결정을 내리도록 학습 가능
- 각 edge host의 local training gradient에 noise를 추가한 Concurrent FRL 방식 구축
  - LDP-CFRL : edge host가 noisy output만을 서버와 공유
    - 신뢰할 수 없는 서버는 모델 반전 또는 데이터 재구성 공격을 할 수 없도록 함
- Contribution
  - 프라이버시 보장을 포함하는 IoT 컴퓨팅을 위한 새로운 자원 할당 방법을 제안
  - 에이전트와 서버가 모델을 공유할 필요 없이 공동으로 결정을 내리는 연합 강화 학습에 동시성 개념을 도입



## Related Work

- 강화학습이 엣지컴퓨팅 자원 할당 문제에서 널리 사용되고 있음
- 기존의 자원 할당 방법은 **프라이버시 보호 정도**에 따라 2가지 카테고리로 분류할 수 있음
  - DRL 방법
  - Federated RL 방법

### A. Resource Allocation-Based Deep Reinforcement Learning

- DRL 기반의 알고리즘은 엣지컴퓨팅의 자원 할당 문제를 다루는 데 좋은 성능을 보임
  - 다만, 이 방법은 <u>유저 프라이버시를 지키지는 못함</u>
- Wang et al. : 엣지컴퓨팅 시스템에서 적응적으로 컴퓨팅 자원을 할당하기 위한 DRL 기반 자원 할당 방법을 제안
  - DQN 프레임워크 기반, '자원 활용' 과 '평균 서비스 시간' 사이의 균형을 맞추려 함
- Xiong et al. : 각각의 host가 독립적으로 자원을 할당하는 셀룰러 네트워크에서 deploy된 모바일 엣지컴퓨팅 시스템을 고려
  - task processing 문제를 MDP로 다루고 여러 개의 replay memory를 가진 DRL 방법을 사용
  - '자원 활용'의 효율성 향상과 '모든 task가 가능한 한 빠르게' 끝낼 수 있도록 2가지 측면을 생각

### B. Resource Allocation-Based Federated Reinforcement Learning

- edge host가 서버로 raw data를 전송하지 않아도 된다
  - 대신, 서버는 edge host로부터 parameter를 전송받아 aggregation model을 생성
  - 이 방법은 edge host의 프라이버시를 어느 정도 보호할 수 있음
  - 다만 **여전히 프라이버시 문제가 존재** ( 모델 inference 공격, 데이터 reconstruction 공격 )
- Zhan et al. : 최소 system cost로 작동하는 FRL 기반 경험중심 computing resource 할당 방법 제안
  - cost : '학습 시간'과 'latency'의 weighted sum으로 정의
  - 주요 아이디어 : 훈련 그룹 내의 더 빠른 mobile device들의 CPU cycle 빈도수를 줄이는 것
    - 에너지 소비와 latency 사이의 균형을 이루기 위함
- Yu et al. : 지능형 초고밀도 엣지컴퓨팅 프레임워크 제안 -> '연산 오프로딩', '자원 할당', '서비스 캐싱' 에 관련된 문제를 joint하게 연구
  - 서로 다른 속도로 작동하는 2개의 timescale로 구성된 DRL 방법 설계
  - edge device의 프라이버시를 보호하기 위해 FL 구조 사용
- Tianqing et al. : CFRL이라는 새로운 자원 할당 방법 제안
  - edge host와 server가 자원을 할당하기 위해 협력하는 구조
  - server : edge host로부터 자원 할당 정책에 대한 피드백에 따라 서로 다른 edge host들의 자원 조정
    - 이점 : edge host는 서버와 **제한된 정보만 공유**하면 됨
    - 기존의 FL 방법보다 더 강한 프라이버시 보장

### C. Local Differential Privacy in FL and RL

- 모델 공격 (모델 반전 공격 / 데이터 재구성 공격) 에 대한 방어 방법 중 한가지 : **인공적인 noise**를 추가하는 것
  - FL 프레임워크에 Differential Privacy를 결합 -> 높은 수준의 프라이버시 보호를 제공할 수 있음
  - 제 3자가 신뢰되지 않은 시나리오에서, Local Differential Privacy (LDP) 는 더 높은 프라이버시 보호를 제공할 수 있음
- Hu et al. : 악성 서버가 수신된 메세지로부터 개인 사용자 정보에 접근할 수 없도록 LDP를 기반으로 한 Personalized FL 접근 방식 제안
  - user가 local update를 upload할 때, neural network의 gradient에 Gaussian noise를 추가
  - 보조 정보가 제공되지 않는다면 정보 유출을 효과적으로 막을 수 있다
- Truex et al. : condensed local differential privacy ($\alpha$ - CLDP) 을 FL에 통합 -> 참가자들이 local deep neural network instance를 훈련시킬 때 gradient를 교란시킬 수 있도록 함
  - 복잡한 모델에 대한 inference 공격에 효과적
  - 참가자들은 개인화된 LDP를 제공받을 수 있음
    - $\alpha$ - CLDP는 상대적으로 큰 프라이버시 budget (예산) 이 필요하므로, 개인정보 보호 보장이 충분하지는 않음

- Ono and Takahashi : LDP를 강화학습에 적용
  - agent가 noisy gradient를 중앙 aggregator에 업로드하고, 중앙 aggregator가 global parameter를 업데이트하는 private gradient collection 프레임워크 제안
  - LDP를 만족하면서도 robust한 policy를 학습시킬 수 있음

### D. Discussion of Related Work

- DRL : 엣지컴퓨팅 시스템에서 자원 할당 문제에서 좋은 성능을 보이지만, edge host의 <u>데이터 보안에 대해서는 간과</u>
  - Peng et al. : IoT 환경은 보안 문제에 매우 취약함 ( 본질적인 취약성, 프로그래밍 취약성, 공격 등의 보안 문제 )
  - FL : 보안 프레임워크로써 계속 등장하는 성공적인 사례. But IoT 분야에서 데이터, 모델, device의 이질성 문제가 있음
    - Lu et al. : PFL 프레임워크 제안, IoT 분야에서 FL을 적용하여 우수한 결과
- FRL : edge host의 raw data를 어느 정도 보호할 수 있는 방법 (방어의 한 방법으로도 사용)
  - Yang et al. : raw device data를 서버에 전송하지 않고 로컬 모델 학습이 가능한 비동기 FL 프레임워크 제안
  - Zhang et al. : 효율성을 향상시키고 개인정보 보호를 위해 IoT 장치에서 생성된 데이터를 관리하는 방법으로 FL과 DRL 적용
  - Chen and Liu : DDPG 프레임워크에 기반한 FL 알고리즘 (FL-DDPG) 제안 -> task offloading과 자원 할당을 공동 최적화 -> IoT 장치의 에너지 소비를 최소화하며 프라이버시를 보호, 훈련 성능 향상
- 기본 FL 프레임워크만 사용한다고 프라이버시는 안전하지 않다!!
  - 신뢰할 수 없는 서버의 경우, 모델 반전 공격에 취약하며 개인정보 유출의 문제를 여전히 가지고 있음
  - Wang et al. : LDP 매커니즘을 FL과 결합 -> 강력한 데이터 개인정보 보호 제공, 높은 가용성 보장
    - 문제점 : 위의 개념이 자원 할당 문제에 적용되지 않았다
- 본 논문에서는 모델 반전 공격과 멤버십 추론 공격으로부터 학습 데이터를 보호할 수 있도록 자원 할당 방법에 LDP 매커니즘을 도입
  - LDP-CFRL 기법 : FRL과 LDP를 결합 -> 우수한 성능을 보이면서도 수평적인 프라이버시 보호를 제공



## Preliminaries

### A. Deep Reinforcement Learning

- 강화학습 : 복잡한 sequential 의사결정 문제를 해결하는 효과적인 self-training 과정

  - agent : 동적인 환경에 적응할 때 decision에 대해 시뮬레이션을 할 수 있음
  - 4-tuple : <S, A, T, R> 로 구성된 Markov Decision Process (MDP) 로 볼 수 있음
    - State set / Action set / Transition model / Reward function
  - $\pi$ : 한 state에서 action들의 확률 분포
    - agent : $\pi(a \vert s)$에 따라 수행할 다음 동작을 선택
  - 최종 목표 : 반환된 reward $\mathbb{E}_{\pi ^ *} [R]$를 최대화하는 최적의 policy $\pi ^ *$를 찾는 것

- 논문에서 사용된 강화학습 알고리즘은 **DQN 알고리즘**

  - Q-Learning : action-value function $Q(s, a)$ 기반의 value-based 방법

    - $Q_{\pi}(s,a) = \mathbb{E}_{\pi} [R(t) \vert s_t = s, a_t = a]$ : state $s$에서 action $a$를 취했을 때 얻을 수 있는 expected reward

  - policy $\pi ^ *$가 수행된다면 최적의 action-value function $Q^*$을 얻을 수 있음

    - 반대로 $Q^*$를 계산할 수 있다면 $\pi ^ *$을 얻을 수 있음  

      
      $$
      \pi ^ * = \arg \underset{\pi}{\max} Q^* (s,a)
      $$

    - 즉, 하나를 찾을 수 있다면 다른 것도 찾아낼 수 있다는 의미이다. 이에 따라 $Q^*$를 Bellman optimal equation으로 재작성하면 아래와 같다  

      
      $$
      Q^{*}\left ({s, a}\right )=\underset {s^{\prime } \sim T}{\mathbb {E}}\left [{r\left ({s, a}\right )+\gamma \max _{a^{\prime }} Q^{*}\left ({s^{\prime }, a^{\prime }}\right )}\right ]
      $$

      - $\gamma$ : discount rate (0 ~ 1)

      - action-value function $Q(s,a)$는 temporal difference update로 반복하여 계산할 수 있음

        
        $$
        Q\left ({s,a}\right )\leftarrow Q\left ({s,a}\right )+\alpha \left [{r\left ({s,a}\right )+\gamma \max _{a^{\prime }}Q\left ({s^{\prime },a^{\prime }}\right )-Q\left ({s,a}\right )}\right ]
        $$

- DQN : Q-Learning에서의 Q-table을 neural network $Q(s,a, \theta)$로 대체 -> 다양한 복잡한 시나리오에 적응 가능

  - $Q(s,a,\theta)$ : action-value function $Q(s,a)$를 근사시키기 위해 weight $\theta$를 학습하는데 사용

  - neural network의 loss function ::
    $$
    \mathcal {L}(\theta )=\mathbb {E}\left [{\left ({\underbrace {r\left ({s, a}\right )+\gamma \cdot \max _{a}, Q\left ({s^{\prime }, a^{\prime }, \theta }\right )}_{\text {Target }}-Q\left ({s, a, \theta }\right )^{2}}\right )}\right ]
    $$
    
    - $s', a'$ : 다음 state, 다음 action
    - loss function $\mathcal {L}(\theta )$를 최소화한다 = mean square error를 최소화한다
    - weight $\theta$를 update하는 흔한 방법 : stochastic gradient descent

### B. Federated Learning

- Google : client-server 구조에서 언어 모델을 교육하는 방법으로 처음 제안
- FL : 여러 client가 중앙 server와 함께 작업할 수 있도록 하는 분산 학습 형태 중 하나
  - client : 개인의 data를 서버와 공유할 필요 X // 대신 aggregation을 위해 training parameter를 업로드
  - 분산 시나리오에서, 머신러닝을 향상시키기 위해 보다 안전하고 안정적인 학습 서비스 제공 가능
- FL은 **개인정보 누출을 완전히 방지할 수는 없다**
  - client의 개인 data에 접근하는 것은 막아주지만, <u>서버가 신뢰되지 않는다면</u> 개인 data는 중간 parameter를 통해 도출될 수 있음
  - => 훈련 model에 필요한 parameter를 안전하게 공유하기 위해서는 **추가적인 개인정보 보호 매커니즘**이 통합되어야함
    - `secure multiparty computation`, `homomorphic encryption` : 엄청난 계산 overhead가 필요
    - `differential privacy` : cost가 낮음 // data가 noise에 의해 교란되어도 모델의 정확도가 크게 감소되지 않을 때 활용 가능

### C. Local Differential Privacy

- 기존 DP 방법 : 신뢰할 수 있는 서버는 device로부터 업로드한 raw data에 noise 추가
  
  - 단점 : 대부분의 경우, <u>신뢰할 수 있는 서버를 찾기 힘듬</u>
  
- LDP : 각 device의 data는 먼저 local에서 교란된 다음 서버로 전송됨

  - 서버 : 원래 data가 아닌 교란된 data를 처리
  - => 서버가 많은 양의 device data를 가지고 있더라도, 민감한 data를 추론해낼 수 없음 (교란되어있으므로)
  - LDP : device가 스스로 private data를 처리할 수 있도록 함
    - device의 input을 바꿈으로써 device의 absolute output을 서버가 추론할 수 없도록 하는 것

- 공식으로는 아래와 같이 표현  
  $$
  \begin{equation*} {\mathrm{ Pr}}\left [{\mathcal {M}(D) \in Y}\right ] \leq \exp (\epsilon ) \cdot {\mathrm{ Pr}}\left [{\mathcal {M}\left ({D^{\prime }}\right ) \in Y}\right ] + \delta.\tag{6}\end{equation*}
  $$

  - pure DP의 기본적인 정의에서는 추가 parameter $\delta$가 포함되지는 않음
  - $\delta$ : approximate DP에서 처음 제안된 파라미터, pure DP가 break될 확률을 의미



## Concurrent Federated Reinforcement Learning Methodology

|      Notation       |                  Description                   |
| :-----------------: | :--------------------------------------------: |
|         $S$         |                  state space                   |
|         $A$         |                  action space                  |
|         $T$         |              transition function               |
|         $R$         |                reward function                 |
|         $D$         |              replay memory space               |
|     $\tilde{D}$     |          a batch of samples from $D$           |
|        $n_e$        |                 edge host의 수                 |
|         $B$         |              gradient norm bound               |
|    $\mathbf{C}$     |            edge host의 state matrix            |
| $\boldsymbol{\tau}$ |       처리되기를 기다리는 task들의 queue       |
|         $m$         |     edge host가 소유한 computing resource      |
|         $n$         |    edge host가 소유한 sliding window의 길이    |
|    $c^{\tau_i}$     |   task $\tau_i$에 할당된 computing resource    |
|         $t$         |                   time step                    |
|    $u^{\tau_i}$     |      utility of handling tasks $\tau_{i}$      |
|        $r_e$        |               edge host의 reward               |
|        $r_s$        |            서버로부터 얻어진 reward            |
|         $I$         |               maximal iteration                |
|    $\mathcal{L}$    |                 loss function                  |
|     $\epsilon$      |                 privacy budget                 |
|    $\mathcal{M}$    |           DP의 randomized mechanism            |
|    $\varepsilon$    | RL에서의 $\varepsilon$-greedy 기법의 parameter |
|      $\alpha$       |                 learning rate                  |
|      $\gamma$       |                discount factor                 |
|      $\lambda$      |        Poisson distribution의 mean rate        |
|         $z$         |                 Gaussian noise                 |

### A. Overview of the System

- edge computing 시스템은 3가지 유형의 entity로 구성됨
  1. IoT devices
     - 일반적으로, 연산 성능과 저장용량이 제한되어있음
     - 정기적으로 task를 edge host로 업로드
  2. edge hosts
     - 수신된 task들을 queue에 저장, FIFO 규칙에 따라 처리
     - server와 함께 처리할 task들을 기반으로 resource allocation 전략을 공동으로 공식화
       - edge host가 처리할 수 없는 task의 경우 서버로 업로드
     - edge host : 서버를 신뢰하지 않음 -> device의 data privacy를 위해 학습 과정에 noise 추가
  3. server
     - 신뢰할 수 없는 제 3자 entity
     - 자원 할당 전략에 따라 각 edge host에 할당되는 resource의 수를 조정
     - 시스템의 목표 : 가능한 한 빠르고 효율적으로 **queue 안의 task들을 완료하는 것**
- 목표 달성을 위해 resource 할당 전략은 MDP로 공식화 (state / action / reward)
  - edge host측
    - **env** : 본인 자체 (itself)
    - **state** : task를 어떻게 처리하는가에 따라 달라짐
    - **action** : task에 할당된 resource
    - **reward** : task 크기와 처리 시간에 따라 달라짐
  - server측
    - **env** : 그 자체 (itself) + 모든 edge host
    - **state** : edge host가 제출한 자원 할당 전략에 달라짐
    - **action** : edge host의 자원을 조정하는 방법
    - **reward** : edge host의 피드백에 따라 달라짐

### B. Edge Hosts' Learning Process

- edge host의 학습과정 : DRL 알고리즘 기반으로 할당 전략을 구축하는 과정

  - edge host : queue에서 task를 예약 ('local 처리를 위해 리소스 할당' or '서버에서 처리하도록 task를 보내는 작업' 을 포함)

  - 서버를 신뢰할 수 없음 :: 훈련 단계에서 gradient에 충분한 noise를 추가해야 함 (LDP 충족 목적)

    - noise gradient를 만드는 좋은 방법 : gradient에 **Gaussian Noise**를 추가하는 것
    - 다만, 확률적 구배에 대한 민감도 문제를 고려할 필요가 있음

  - edge host들이 gaussian noise를 추가할 때, gradient clipping을 먼저 적용한 후 noise를 적용함  
    $$
    \begin{equation*} \overline {\mathbf {g}_{t}}\left ({x_{i}}\right )=\frac {\mathbf {g}_{t}\left ({x_{i}}\right )}{\max \left \{ {1, \frac {\|\mathbf {g}_{t}\left ({x_{i}}\right )\|_{2}}{B}}\right \}}\tag{7}\end{equation*}
    $$

    - $\mathbf {g}_{t}\left ({x_{i}}\right )$ : $\tilde{D}$의 $i$번째 샘플에 대한 gradient vector
    - $B$ : clipping threshold
      - clipping된 두 gradient에 대해, ${\| \bar{\mathbf {g}_{1}} - \bar{\mathbf {g}_{2}} \|_{2}} \leq B$
    - gradient는 $\ell_2$ norm으로 초기화

  - Gaussian noise $z$는 아래와 같이 생성  
    $$
    \begin{equation*} \mathbb {P}\left ({z_{i}}\right )=\frac {1}{\sqrt {2\pi }\sigma } \exp \left ({\frac {z^{2}}{2\sigma ^{2}}}\right ).\tag{8}\end{equation*}
    $$

  - Gaussian noise가 추가되면, edge host는 noise gradient를 생성  
    $$
    \begin{equation*} \tilde {\mathbf {g}}_{t} \leftarrow \frac {1}{\tilde {D}}\left ({\sum _{i} \overline {\mathbf {g}}_{t}\left ({x_{i}}\right )+z}\right )\mathrm {\mathrm {\mathrm {}}}.\tag{9}\end{equation*}
    $$

    - 추가된 noise $z$는 아래 방정식이 성립한다면 $(\epsilon, \delta)$ 분포가 됨

  - DP는 아래 방정식이 성립되어야함
    $$
    \begin{equation*} \sigma \le \frac {\sqrt {2 \ln \left ({1.25 / \delta }\right )} \Delta f}{\epsilon }.\tag{10}\end{equation*}
    $$

  - 생성된 "교란된 전략"은 server로 넘겨짐

    - edge host : task 완료에 대한 reward를 얻고 DQN 모델을 업데이트

### C. Server's Learning Process

- server의 학습과정 : 역시 DRL 알고리즘 기반, 서로 다른 edge host의 자원 조정하는 것이 목적
  - server : edge host에서 업로드한 resource **allocation** 전략에 따라 학습, 대가로 resource **adjustment** 전략을 host에게 리턴
  - edge host로부터 피드백 수신 -> 자원 할당 전략의 합리성 결정
    - 합리적인 자원 할당 전략이라면 edge host가 더 높은 reward를 제공
  - 마지막으로 DQN 모델 업데이트

### D. State

- edge host의 state $s$는 MDP의 상태정보를 보여줌
  $$
  \begin{equation*} S=\left \{ {s|s=\left \langle{ \mathbf {C},\boldsymbol {\tau }}\right \rangle }\right \}\tag{11}\end{equation*}
  $$

  - $\mathbf {C}$ : edge host에서 task의 처리 상태
  - $\boldsymbol {\tau }$ : 처리할 task의 queue
    - queue에 있는 task의 크기는 평균이 $\bar{X}$인 이산균일분포를 따를 것
  - $\mathbf{C}$ 행렬은 computing cell로 구성
    - 행의 수 $m$ : host의 computing resource
      - 연산 자원 : `allocation`, `available`, `unavailable` 3가지 상태를 가지고, 각각 $0$, $1$, $-1$로 나타냄
    - 열의 수 $n$ : host의 sliding window 길이
    - 행렬에서, 첫번째 열의 자원 요청은 각 timestep에 처리되고, sliding window가 한칸 이동
  - 적절한 자원 조절 전략은 자원 활용도를 향상시키고 작업 완료 속도를 높일 수 있다!
    - 그림 4에서, edge host의 computing resource를 4->5로 조정
    - => edge host가 1개의 timestep에 task $\tau_{1}$을 완료할 수 있게 됨 -> 전체적인 활용률 향상
  - $\boldsymbol {\tau }$ : edge host에 의해 처리되기를 기다리는 task들의 queue
    - queue 안의 값 = task의 크기 = 각 task에 필요한 computing resource 수

- server는 edge host와는 다른 state space를 가지며, $S'$로 표기  
  $$
  \begin{equation*} S^{\prime }=\left \{ {s^{\prime }|s^{\prime }=\left \langle{ \mathbf {C^{\prime }},\boldsymbol {\Omega }}\right \rangle }\right \}.\tag{12}\end{equation*}
  $$

  - 서버의 상태공간 $\mathbf{C}'$ : edge host의 할당 정책으로 구성됨
    - 행의 수 : 서버에 연결된 host의 수
    - 열의 수 : edge host가 처리할 task의 수
  - queue $\boldsymbol {\Omega }$ : server가 이 때 edge host에게 할당한 resource의 개수
    - 만약 edge host가 5개의 resource를 가진다면, 처리될 필요가 있는 task에는 5개 이하의 resource를 할당할 수 있음

### E. Action

- edge host와 server의 상태공간이 다름 => **action 공간도 달라야함!**

  - edge host : 각 action 하나 이상의 resource를 task에 할당

    - => action space는 task에 할당된 resource의 수 + 해당 할당의 timing으로 구성
      $$
      \begin{equation*} A_{e}=\left \{ {a|a\in \left \{ {1,\ldots , m}\right \}}\right \}\tag{13}\end{equation*}
      $$

      - $a$ : edge host가 현재 task에 할당한 resource 수
        - edge host에서의 task 처리시간은 $a$에 의해 결정됨 (얼마나 할당했는지에 따라 task 처리에 걸리는 시간이 달라짐)
        - task는 가능하면 현재 time slide에 가깝게 처리하기 위해 $\mathbf{C}$에 배치됨
        - 다만, edge host가 <u>local에서 task를 수행할 수 없다고 판단</u>하면 서버로 업로드함 ($a=0$)

  - server : action space는 edge host의 일련번호 및 edge host에 할당된 resource의 수를 조정하는 등의 정보를 포함
    $$
    \begin{equation*} A^{\prime }=\left \{ {a^{\prime }|a^{\prime }=\left \langle{ d,n_{e_{i}}}\right \rangle }\right \}\tag{14}\end{equation*}
    $$

    - $d \in \{-1, 0, 1 \}$, $1 < n_{e_i} < n_e$
    - 각 action $a'$는 원소가 2개인 튜플로 제공됨
      - 의미 : $n_e$번째 edge host에서 $d$ resource를 조정하는 것 (더하거나 빼거나)

### F. Reward

- 설계 : "IoT device가 업로드하는 task를 **가능한 한 빨리 완료**하는 것" 이라는 system의 궁극적인 목표와 일치

  - 이 시스템에서는, 처리를 위해 task를 서버에 위임하면 <u>reward가 낮아지거나 부정적인 결과</u> (서버를 완전히 신뢰할 수 없으므로)

  - 그래서, 처리를 위해 task를 서버로 위임하는 것은 위험한 행동 // edge host는 task를 업로드하지 않으려고 함

  - => edge host와 server의 reward 체계는 차이가 발생

    - edge host : task를 완료하는데 걸린 time step 수, task 크기에 따라 결정
      $$
      \begin{align*} r_{e}= \begin{cases} {\mu }u^{\tau _{i}}-\nu \frac {\tau _{i}}{c^{\tau _{i}}} & ~\text {when edge host processing $\tau _{i}$} \\ \mu \left ({u^{\tau _{i}}-u^{\tau _{\mathrm {max}}}}\right ) - 1 &~ \text {when server processing $\tau _{i}$} \end{cases} \tag{15}\end{align*}
      $$

      - 추가 notation
        - $u^{\tau _{i}}$ : task를 완료한 후의 system utility (더 크고 복잡한 task를 완료하면 높아짐)
        - $\frac {\tau _{i}}{c^{\tau _{i}}}$ : task ${\tau _{i}}$를 수행하기 위해 필요한 시간
        - $\mu$, $\nu$ : 소요된 처리 시간과 task 크기 사이의 weight coefficients
      - task가 edge host에 의해 local로 처리되는 경우, reward는 "task 크기 - 처리하는데 걸린 시간"
      - task가 edge host에 의해 처리될 수 없는 경우 : edge host의 state matrix $\mathbf{C}$의 각 cell은 연산 자원을 나타내므로, task의 크기가 cell의 수를 초과하면 short term에 local에서 task를 완료할 수 없음 -> 서버로 task 업로드
        - 이때의 reward는 'task의 크기'와 'task의 최대 크기 $2 \times \bar{X}$' 의 차이에 의존
      - weight coefficient $\mu$와 $\nu$를 조정하여 reward signal은 **음수**로 설정됨
        - edge host : 과도한 패널티를 피하기 위해 자원 할당 전략을 지속적으로 최적화할 것

    - server : 2가지 시나리오가 존재

      - edge host로부터 task를 수신

      - edge host로부터 task를 수신받지 않음

      - action이 주어지면 server의 reward 식은 아래와 같이 정의
        $$
        \begin{align*} r_{s}= \begin{cases} -\sum _{i=1}^{n_{e}}\left ({d_{i}+c^{\tau _{i}}}\right ) & ~\text {when no tasks are received}\\ \mu u^{\tau _{i}}-v\left [{\frac {\tau _{i}}{c^{\tau _{i}}}}\right \rceil -\sum _{i=1}^{n_{e}}\left ({d_{i}+c^{\tau _{i}}}\right ) & ~\text {when received task from $n_{e_{i}}$}. \end{cases}\tag{16}\end{align*}
        $$

        - $\langle d_i, n_{e_i} \rangle$ 기반으로 edge host에 resource를 할당
        - 서버가 task를 받지 못하면 resource 낭비에 따른 패널티를 받음
        - 서버가 task를 받는다면 '처리 시간'과 '작업의 difficulty' 2가지 요소에 따른 reward를 받음

      - => 합리적인 resource 사용을 유지하면서 자원 할당에 도움을 줌

### G. Algorithm

- edge host 알고리즘

  ```pseudocode
  Algorithm 1
  Noising Edge Host's Algorithm
  
  Input : resource allocated by the server
  Output : 
  
  Initialize a ramdom weight θ for DQN;
  Initialize the state S by local computing resource;
  Initialize the replay memory D = ø;
  Initialize action-value function Q with θ;
  for t = 1 to maximul iteration number I do
      Observe the current state s^t;
      if task queue = ø then
          break;
      Generate random number φ;
      if φ > ε then
          select aτ = argMax Q(s^t, aτ; θ);
      else
          Randomly select an action aτ from the action space A;
      Perform action aτ and transform state s^t to s^(t+1);
      Receive the reward re;
      Store transition <s^t, aτ, re, s^(t+1)> into D;
      Randomly selected transition samples from replay memory D;
      Using Equation 5 to calculate the loss function L(θ) and updating θ' with differentially private stochastic gradient descent based on Algorithm 2;
      Send the action-value function Q vector Q(s^t) to the server;
  ```

  - edge host가 env와 상호작용할 때 가장 먼저 관측되는 것 : state $s^t$

  - $\varepsilon$-greedy 원리에 기초하여 action 선택 (랜덤값 $\phi$를 생성하고 threshold와 비교)

    - $\phi$가 threshold보다 크면 action으로는 greedy한 것을 선택 // 그렇지 않으면 임의의 action 선택
    - iteration 횟수가 증가함에 따라 threshold 값은 점차 감소

  - 선택된 action이 수행되면, edge host는 task 완료에 대한 reward를 받고, $s^t \rightarrow s^{t+1}$로 전이

    - transition sample <$s^t, a^{\tau}, r_e, s^{t+1}$>는 D에 저장됨

  - edge host는 메모리 D에서 sample batch를 가져와 loss function $\mathcal{L}$을 계산

  - DQN : gradient를 업데이트하기 위해 differentially private stochastic gradient descent 알고리즘 사용

  - Gaussian noise를 더하는 방법

    ```pseudocode
    Algorithm 2
    Gaussian Mechanism for Gradient Updating
    
    Input : gradient vector g, privacy budget ϵ, gradient norm bound B, noise scale σ;
    Output : Noisy gradient ~(g(x_i)) = (1/~D) * (Σi bar(g(x_i)) + z) 
    
    Clipping Gradient
    bar(g(x_i)) <- clipping gradient g sing Eq. 7
    
    Generating Noise
    z <- generating Gaussian noise z using Eq. 8
    ```

  - 마지막으로 edge host는 local model의 output을 서버로 upload함

- server 알고리즘

  ```pseudocode
  Algorithm 3
  Server's Algorithm
  
  Input : the edge host's Q-value vector Q(s^t);
  Output : Distribute allocation policies to edge hosts;
  
  Initialize a random weight θ' for DQN;
  Initialize the replay memory D' = ø;
  Initialize action-value function Q with θ';
  for t = 2 to maximal iteration number I do
      Combine all vectors Q(s^t) and assemble into a state matrix o^t;
      Observe the current state o^t;
      Generate random number φ;
      if φ > ε then
          select as = argMax Q(o^t, as ; θ');
      else
          randomly select an action as;
      Send the resource increase or decrease signal to the edge host according to action as;
      transform state to o^(t+1) and receive the reward rs;
      Store transition <o^t, a^s, rs, o^(t+1)> into D';
      Randomly selected transition samples from replay memory D';
      Using Equation 5 to calculate the loss function L(θ) and updating θ' with stochastic gradient descent;
  ```

  - DRL을 사용하여 edge host에 리소스 할당
  - env와의 각 iteration에서 server는 edge host가 업로드한 Q vector $\mathbf{Q}(s^t)$를 기반으로 state space $o^t$ 형성
  - edge host와 동일한 $\varepsilon$-greedy 원리에 기초하여 action 선택
    - action : resource 조정 명령을 edge host에게 전송, 이후 host로부터의 피드백을 통해 reward $r_s$를 획득
  - transition sample을 $D$에 저장하고, Q-network를 훈련시키기 위한 sample batch를 무작위 선택
  - server : 할당 정책을 edge host에게 배포 

### H. Case Study

- 제시된 scheme이 프라이버시를 보호하는 정확한 방법은 사례 연구를 통해 가장 잘 설명될 수 있음
  - 경로 계획 문제를 처리하도록 설계된 IoT + edge 시스템이 있다고 가정
    - 이 시스템은 수십 개의 IoT 장치와 에지 컴퓨터로 구성
    - 각 IoT 장치는 현재 위치에서 목표 위치까지의 최적 경로를 계획해야하며 각 최적 경로 계산은 task로 간주
    - edge computer가 각각의 IoT 장치에 대한 시작 위치를 알고 있고, 신뢰되지 않는다고 가정
      - 다른 말로 하면, 역방향 공격을 사용하여 IoT 장치의 출력을 기반으로 로컬 계산 모델 또는 경로 세부 정보를 유도할 수 있음
    - 일반적으로 위치 정보는 부동 소수점 위도 및 경도 값의 collection
- 자원 할당 방법이 프라이버시 보호를 고려하지 않는 경우, 각 IoT 장치는 로컬 모델의 그라디언트 정보를 직접 업로드
  - 그런 다음 에지 컴퓨터는 얻은 그래디언트를 음수 값으로 반복 변환하여 미분 가능한 함수의 로컬 최소값을 찾음
  - 공격의 필요성에 따라 위도 및 경도 정보는 그라디언트 반전을 사용하여 계산된 다음 경로 정보를 복구하는 위도 및 경도 오프셋 보정과 같은 위치 최적화를 위해 처리
- 그러나 본 논문에서 제안하는 기법에서는 그라디언트 하강을 수행할 때 노이즈가 추가되고 노이즈에 영향을 받는 그라디언트가 서버에 업로드되므로 로컬 계산 모델 데이터에 누출이 발생하지 않음
  - 따라서, 엣지 컴퓨터가 공격을 개시하더라도 경로 정보는 보호됨



## Theoretical Analysis



### A. Privacy Analysis



### B. Utility Analysis



### C. Convergence Analysis



## Experimental Results

### A. Environment Setup

- 제안한 방법의 성능을 정량적으로 평가하기 위해 OpenAI Gym 0.18.0과 PyTorch 1.9.1을 이용하여 파이썬 시뮬레이터를 개발

  |           Notation           |                     Description                      | Value |
  | :--------------------------: | :--------------------------------------------------: | :---: |
  |           $\alpha$           |                    learning rate                     | 0.01  |
  |           $\gamma$           |                    discount rate                     | 0.99  |
  |             $D$              |                  memory space size                   |  512  |
  |             $I$              |                  maximum iteration                   | 2000  |
  |     $d_{\text{initial}}$     |          the initial resource of edge host           |  10   |
  | $\varepsilon_{\text{start}}$ | the initial parameter of $\varepsilon$-greedy method |  0.9  |
  |  $\varepsilon_{\text{end}}$  |  the final parameter of $\varepsilon$-greedy method  | 0.05  |
  |          $\epsilon$          |                    privacy budget                    |  10   |

- 모든 실험에서 서버는 $n_e$개의 에지 호스트에 연결되어 있다고 가정하고 각 에지 호스트에는 초기 리소스가 $m$개 존재

- 각 에지 호스트가 처리해야하는 작업 대기열은 포아송 분포에 따라 처음 5 time step에서 동적으로 생성

  - 즉, 처음 5 개의 time step에서 각 시간 단계는 $\bar{X}$의 평균을 갖는 이산 균일 분포에 따라 생성된 작업 크기로 큐에 $\lambda$개의 task을 생성

- 서버가 다른 수의 에지 호스트에 연결된 세 가지 시나리오에서 LDP-CFRL의 성능을 평가

  1. edge host의 수를 변경
     - $n_e$은 $5, 10, 15$로 설정
     - 총 과제수는 $10$ 고정, $\bar{X}$는 $10$으로 고정

  2. edge host가 처리하는 task 수를 변경
     - task $\lambda$를 생성하기 위한 푸아송 분포의 평균 비율을 $5, 10, 15$로 설정
     - edge host $n_e$의 수는 $5$ 고정, $\bar{X}$는 $10$으로 고정

  3. edge host의 task size를 변경
     - task size 의 이산균일분포의 평균값 $\bar{X}$를 $5, 10, 15$로 설정
     - edge host $n_e$의 수는 $5$ 고정, $\lambda$는 $10$으로 고정
  4. **모든 시나리오**에서 $d_{\text{initial}}$은 $10$으로 고정, 초기 sliding window 크기를 (edge host, server) 순으로 $(6, 10)$으로 고정

- LDP-CFRL을 2개의 method와의 평가 진행

  - CFRL
    - edge host와 server가 협력하여 작업 처리 및 자원 할당을 관리
    - 작업 완료 속도 및 자원 활용 측면에서 잘 수행됨
      - 그러나 edge host에 대한 **데이터 개인 정보 보호에 대한 이론적 보장은 제공되지 않음**
  - DRL
    - 자원 할당을 위해 널리 사용되는 방법
    - edge host의 동작에서 CFRL과 유사하지만, server는 edge host에 **리소스를 동적으로 할당하지 않음**

- 3가지 측정기준 사용

  - **평균 utility** : task 완료에 대한 reward 반영
  - **평균 완료 time step** : edge host가 task series를 완료하는데 걸린 시간
  - **평균 resource utilization** : 각 task가 실제로 사용한 resource의 비율 (평균 리소스 사용률)

### B. Results



## Conclusion

- 서버가 부분적으로 신뢰할 수없는 시나리오에서 자원 할당 문제를 해결
  - LDP-CFRL이라는 새로운 FRL 알고리즘을 제안했는데, 이 알고리즘은 이전에 제안된 CFRL 알고리즘에 local differential privacy 매커니즘 통합
  - 알고리즘은 edge host의 training gradient에 LDP를 만족하는 Gaussian noise를 도입
    - 신뢰할 수 없는 서버가 있는 경우에도 데이터 프라이버시를 효과적으로 보호
    - 알고리즘의 프라이버시 보장은 이론적으로 분석되고 입증
- Contribution으로 효율적인 자원 할당과 강력한 개인 정보 보호 보장을 보장하는 실용적인 솔루션을 제공
  - LDP를 FRL 프레임워크에 통합 => edge host들이 그들의 local data의 프라이버시를 보존하면서 자원 할당 모델을 협력적으로 훈련할 수 있도록 함
  - 위 접근법은 전통적인 할당 알고리즘에 비해 높은 수준의 개인 정보 보호를 제공
- 실험 결과는 LDP-CFRL 알고리즘이 효율성 및 작업 처리 속도 측면에서 주목할만한 성능을 달성한다는 것을 보임
  - 광범위한 평가를 통해, 위 접근방식이 프라이버시 보호와 자원 활용 측면에서 기존의 할당 알고리즘을 능가한다는 것을 보여줌
- 요약하면, 위 연구는 부분적으로 신뢰할 수 없는 서버 시나리오에서 자원 할당 문제를 해결하는 새로운 FRL 알고리즘인 LDP-CFRL을 제시함으로써 상당한 기여를 함
  - 알고리즘은 LDP 메커니즘을 통합하여 edge host에 대한 강력한 프라이버시 보장을 제공하고 전체 시스템의 프라이버시를 보장





