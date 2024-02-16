---
title: "Multi-Agent Deep Reinforcement Learning for Slicing and Admission Control in 5G C-RAN"
author: Joe2357
categories: [1. iMES, Lab Seminar]
tags: [Lab Seminar]
math: true
---

> 2022 / 9 / 30 iMES 세미나



## Abstract

- 5G Cloud RAN : 동적 RAN 기능 분할 / 배치를 용이하게 함 "새로운 형태의 유연한 자원 관리"
  - 가상화 RAN function : resource 가용성, slice 제약조건에 따라 substrate network에 site들에 배치될 수 있음
  - substrate network에서는 제한된 resource 가용성에 의해 Infrastructure Provider (InP) 는 **전략적으로 network slicing을 수행**, <u>장기적인 이익을 위해</u> slice 요청을 **수락하거나 거부**해야 함
- Network slicing + Admission Control 문제를 공동으로 해결하는 Multi-agent Deep Reinforcement Learning을 사용
  - Multi-agent : 여러가지 별개의 작업을 최적으로 수행해야 하는 문제에 적합
  - DRL : slice 요청 traffic의 dynamic을 학습, 공동 문제를 효과적으로 해결
- 다른 접근 방식과 비교
  - 간단한 휴리스틱 기법 ( 대비 약 18% 이익 달성 )
  - DRL을 사용하는 slice, AC 접근방식 ( 대비 약 3.8% 이익 달성 )
- 추가사항
  - Multi-agent가 문제를 공동으로 해결하는 단일 agent DRL보다 더 성능이 좋음을 보임
  - 훈련된 model의 robustness를 "훈련에서 벗어난" 시나리오로 일반화하는 능력을 평가



## Introduction

- 5G RAN : New Radio protocol stack에 속하는 Network function의 chain을 포함
  - C-RAN이 5G mobile network에 채택 -> substrate network는 여러개의 상용 node로 구성된 site network
    - Network Function Virtualization (NFV)를 통해 InP는 리소스를 가상화, VNF를 다른 site에 유연히, 전략적으로 배치 가능
    - => network bottleneck 완화, infra 활용률, 수익 증가
- metro 5G C-RAN : 상호 연결된 site들은 tier로 구분
  - 하위 tier : Radio unit에 가까움, resource 적음
  - 상위 tier : Radio unit과 거리가 있음, resource 많음
    - resource가 부족한 VNF를 중앙 집중식으로 배치할 수 있음
    - 여러 VNF instance 간의 resource 공유 -> 더 높은 multiplexing 이익
  - 중앙 집중화 정도 : 개별 VNF들의 latency 허용 오차에 의해 제한될 것
  - 더 높은 수준의 중앙집중화 : 더 많은 data가 link를 통해야함 -> bandwidth 요구량이 커짐
  - 문제 제시
    - C-RAN에서 VNF를 위한 최적의 배치를 선택
    - bottleneck 상태 되지 않고 resource 활용을 극대화
    - delay 및 throughput 요구사항 충족
- 5G 모바일 네트워크 : QoS에 맞춰 광범위한 서비스를 지원할 준비
  - 종류
    - eMBB : 향상된 모바일 광대역
    - URLLC : 초신뢰성 저지연 통신
    - MMTC : 대규모 기계 통신
  - Network Slicing : 동일한 infra에서 서로 다른 서비스의 QoS를 만족시키기 위해 맞춤화된 **격리 end-to-end 가상네트워크** 제공 가능
    - VNF chain으로 구성되어 있음 -> 각각 서로 다른 site에 배치될 것
    - VNF를 배치할 때는 서비스 type과 SLA를 고려해야함
  - Slice request (SR) 는 InP의 수익에 기여
    - 자원이 한정되어있으므로 모든 SR를 수용할 수는 없음 -> **AC 결정의 필요성**
- DRL 방법은 환경과 상호작용하며 trial과 reward를 통해 환경에 대한 지식을 학습했음
  - AC를 해결하고 slicing을 실현하는데 적합한 기술임
  - 대부분의 연구에서는 기존 RL 또는 DRL 방식의 솔루션을 제시했었음
    - AC, slicing은 차별화된 QoS를 제공하는데 필수적이나, 대부분 <u>DRL 솔루션 단독</u>으로 제안했음
    - slicing과 AC 중 하나만을 DRL로 해결하고 그리디 등의 순진한 접근 방식을 사용하면, **잠재적으로 InP의 손실**을 만들 수 있다
- 미래 SR에 대해서는 알 방법이 없다
  - 지능적인 slicing과 AC를 위해서는 미래 SR을 예측할 필요가 있다
  - 기존 연구들은 이를 하지 못해 솔루션의 적용 가능성을 제한하고 있다
  - DRL agent는 누적 reward를 극대화하여 미래 결과를 예측할 수 있다
    - 특정 slice 결정으로 인해 SR이 이후에 bottleneck에 의해 거부되는 경우, agent는 이를 예상하여 결정을 회피할 수 있음
- 단일 DRL 방법은 slicing과 AC 문제를 효과적으로 join시킬 수 없음
  - ex. agent가 최적이 아닌 AC action에 negative reward를 주는 경우
    - 전체 policy에 영향을 줄 수 있음 ( 주어진 환경 state에서 서로 다른 action의 optimality를 비교하는 NN 등 )
    - 그렇다면 동시에 진행되는 slicing 작업이 최적의 action이더라도 동기화가 해제될 수 있음
  - Multi-agent DRL을 사용하면 AC와 slicing에 서로 다른 분리된 policy를 가질 수 있음
    - 두 reward function은 서로에게 부정적인 영향을 끼치지 않고 시너지를 낼 수 있음
- Multi-agent DRL 기반 솔루션을 제안하여 5G C-RAN에서 slicing과 AC를 동시에 해결하는 방안 제안
  - 장기적인 InP의 이익을 향상시키는 것이 목적
  - 단일 agent 방식과의 비교를 위해 slicing과 AC framework를 구현
- Contribution
  - Multi-agent DRL 기반 솔루션을 그리디 방법이나 node-ranking 방법과 비교하여 InP의 이익을 더 극대화시킴을 보임
  - Multi-agent DRL 기반 솔루션을 DRL 방식과 비교하여 slicing과 AC를 묶어서 최적화하는 것이 InP의 이익을 더 극대화시킴을 보임
  - Multi-agent DRL 기반 솔루션을 단일 agent DRL 방식과 비교하여 multi-agent 방법이 더 높은 이익과 더 빠른 수렴 속도를 나타냈음을 보임
  - 훈련된 model의 강인성을 practical network 상태에서 평가하여 다양한 SR 도착비율에도 시나리오를 일반화시킬수 있음을 보임



## System Design

- 5G C-RAN에서 VNF의 closed loop, 자율관리, orchestration을 용이하게 할 수 있는 **MAPE** control loop 기반
  - monitor 모듈 : substrate network로부터 데이터를 수집
  - analyze 모듈 : raw data로부터 유용한 지식 추출, visualization 및 plan에 필요한 모든 metric 계산
    - 처리된 data와 incoming SR은 plan 모듈로 전송
  - plan 모듈 : AI / ML 기술을 활용하여 지능형 slice orchestration, 성능 및 오류 관리 수행
    - AC와 slicing sub-component를 활용하여 문제 해결 ( 동시에 해결 or 순차적으로 해결 )
  - execute 모듈 : SR이 승인되면 orchestration에 필요한 action을 전달받음
    - substrate network와 실제로 상호작용
  - 이 논문에서는 subsrate network를 시뮬레이션하므로, plan 모듈을 제외하고는 연구제시가 없음
    - <u>slicing, AC component 설계</u>에 중점을 둘 것

### A. Substrate Network

- 그래프는 $G = (N, L)$로 나타낼 수 있음
  - $\lbrace {n_1, n_2, \cdots, n_k \rbrace } \in N$, $l_{n, n'} \in L$
  - $c_n^t$ : 특정 시간 $t$에서의 node $n$에서의 **computing resource**
  - $b_n^t$ : 특정 시간 $t$에서의 node $n$에서의 **bandwidth**
- Tier-1 site는 Radio Unit과 가까움 -> latency가 짧고 bandwidth가 높은 link로 연결
  - 매우 낮은 latency를 요구하는 VNF가 제격
- site가 중앙화될수록 RU로부터의 latency가 커짐
  - 사용 가능한 computing resource는 많지만 더 많은 수의 low-tier site를 지원해야함

### B. Functional Split Requirements

- NR protocol stack은 여러 필수 기능으로 구성됨

  - RF
  - Low/High PHY
  - Low/High MAC
  - Low/High RLC
  - PDCP
  - RRC

- 3GPP에서 위 기능들에 대해 가능한 분할을 열거하였음

  - Distributed Unit (DU) 와 Central Unit (CU) 에서 각각 분할 / 중앙집중 되어야하는 기능에 대해 설명
  - dynamic function splitting 에서는, 각 SR들에 대한 분할이 고정되지 않음
    - 다른 site에서 VNF의 배치에 따른 각 SR에 따라 vary해짐
  - Option 8 : RF 신호 생성을 제외한 모든 기능이 중앙집중
    - 가장 높은 multiplexing 이득을 가져옴
    - latency 요구사항과 bandwidth 요구사항이 매우 엄격
  - Option 2 : PDCU만 중앙집중
    - DU에 높은 computing resource 요구사항 초래
    - 덜 엄격한 latency와 bandwidth 요구사항
  - ITU : 높은 bandwidth와 엄격한 latency 요구사항에 의해 RU와 DU 사이의 split point로 **Option 7**을 권장
    - 이 논문에서는 Low PHY와 RF는 <u>항상 RU에 배치</u>됨을 고려
    - RLC와 MAC에 대한 split은 모델링이 되지 않아, 이 논문에서는 <u>Option 2, 4, 6, 7</u>만이 가능한 split point라고 가정
      - 각각의 기능 ( High PHY, MAC, RLC, PDCP ) 을 $f_1 - f_4$라고 표기

- 각 기능은 가상화 (VNF) 되고, substrate network 내에서 상이한 node들에 배치됨

  - 각각의 VNF는 Giga Oprations Per Second (GOPS) 에서 측정된 일정량의 computing resource를 필요로 함

  - resource 요구사항은 각 VNF의 operation type에 따라 다름
    $$
    \begin{align*}
	& G_1 = G_1^{ref} \cdot \frac{B}{B^{ref}} \cdot  \left( {\frac{A} {A^{ref}} } \right)^2 \cdot \frac{L} {L^{ref}}, \tag{1} \\
	& G_2 = G_2^{ref} \cdot \frac{B}{B^{ref}} \cdot \frac{A}{A^{ref}} \cdot \frac{L}{L^{ref}} \cdot \frac{M}{M^{ref}}, \tag{2} \\
	& G_3 = G_3^{ref} \cdot \frac{B}{B^{ref}} \cdot \frac{A}{A^{ref}}, \tag{3} \\
	& G_4 = G_4^{ref} \cdot \frac{B}{B^{ref}} \cdot \frac{A}{A^{ref}}, \tag{4} \\
	& G_5 = G_5^{ref} \cdot \frac{B}{B^{ref}} \cdot \frac{A}{A^{ref}},\tag{5}
	\end{align*}
	$$
  - $G_1, G_2$ : $f_1$ (High-PHY) 에서의 하위 기능에 대한 computing resource 요구사항
    - $G_3, G_4, G_5$ : 각각 $f_2, f_3, f_4$에 대한 computing resource 요구사항
    - $B, A, L, M$ : 각각 bandwidth, MIMO 안테나 개수, load (부하), modulation (변조)
  
- 다른 split에서의 bandwidth 요구사항은 아래와 같이 모델링 가능
  $$
    R(\lambda) = k_1 \lambda + k_2 \tag{6}
  $$
  
  - $R$ : 요구되는 link bandwidth
    - $\lambda$ : slice throughput (Mbps)
    - $k_1, k_2$ : 다른 split에 대한 특정 값을 가지는 상수 ( 다른 논문에서의 reference를 가짐 )
  
- latency 요구사항은 VNF 및 SR의 end-to-end latency 요구사항에 따라 다름

### C. Slice Requests

- slice request는 서비스 유형 중 하나에 전용으로 간주함 ( eMBB, URLLC, mMTC )
- notation ( 시간 $t$에 도달하는 SR $s_t$ )
  - slice의 service type
  - $\lambda ^ {s_t}$ : 지원받아야할 throughput 요구사항
  - end-to-end latency 요구사항
  - $\tau ^{s_t}$ : operation 시간
  - $r^{s_t}$ : 단위시간당 revenue
- 여러 SR은 같은 시간에 도달하지 않음
  - SR이 임베디드되면 $\tau ^{s_t}$ 시간동안 substrate network의 computing / bandwidth resource를 사용
  - depart된 이후에는 예약된 resource를 free
- type 종류
  - eMBB : 가장 높은 bandwidth 필요 / latency는 적당히 허용 가능
  - URLLC : bandwidth는 적당히 허용 가능 / latency는 매우 엄격
  - mMTC : bandwidth와 latency 모두 적당히 허용 가능
- 각 slice에는 우선순위가 있음 ( High/Low Priority )
  - revenue는 우선순위에 비례함

### D. Joint Slicing and AC

- SR $s_t$에 대한 slicing 결정은 $ M \times \|N\| $ 크기의 embedding-relationship matrix $\eta ^{s_t}$의 형태로 이루어짐

  - $M$ : substrate network에 임베딩되어야하는 VNF 수

- notation

  - $\eta_{ f_m, n} ^{s_t}$ : substrate의 node $n$에 $f_m$이 임베딩되었는지의 여부 (binary)
  - node 간의 경로 선택은 최단경로 알고리즘 사용
    - $d_{ru,n}, d_{n, n'}$ : 각각 RU와 $n$, $n$과 $n'$ 사이의 최단경로 delay
  - $\phi_{l, ru, n}, \phi_{l,n,n'}$ : link $l$이 각각 RU와 $n$, $n$과 $n'$ 사이의 최단경로에 존재하는지의 여부 (binary)

- $\eta ^{s_t}$에서, 모든 $s_t$에 대해 matrix가 <u>아래 constraints를 모두 만족해야</u> **valid**하다고 함

  $$
  \begin{align*}
  & \sum\limits_{n \in N} \eta _{f_m, n}^{s_t} = 1\quad \forall m \in \lbrace { 1, \ldots ,M } \rbrace ,\tag{C1} \\
  & \sum\limits_{m = 1}^M \eta _{f_m, n}^{s_t} \cdot c_{f_m}^{s_t} \leq c_n^t\quad \forall n \in {\mathbf{N}},\tag{C2} \\
  & d_{ru,n} \cdot \eta _{f_1, n}^{s_t} + \sum\limits_{f_i = f_1}^{f_m} \eta _{f_i, n}^{s_t} \cdot\eta _{f_{i+1}, {n^\prime}}^{s_t} \cdot {d_{n, {n^\prime }}} \leq d_{f_m}^{s_t} \\
  & \forall n, {n^\prime } \in \mathbf{N},\forall m \in \lbrace { 1, \ldots ,M } \rbrace , \tag{C3} \\
  & \eta _{f_1, n}^{s_t} \cdot R_{f_1}^{s_t} \left( {\lambda ^{s_t}} \right) \cdot \phi _{l,ru,n} + \sum\limits_{m = 1}^{M - 1} {\eta _{f_m, n}^{s_t}} \cdot \eta _{f_{m+1}, {n^\prime }}^{s_t} \cdot \\
  & R_{f_m}^{s_t} \left( \lambda ^{s_t} \right) \cdot \phi _{l, n, {n^\prime }} \leq b_l^t \quad \forall n, {n^\prime } \in \mathbf{N}, \forall l \in \mathbf{L}, \tag{C4}
  \end{align*}
  $$

  - notation
    - $c_{f_m}^{s_t}$ : computing resource 요구사항
    - $R_{f_m}^{s_t}( \lambda ^{s_t})$ : VNF $f_m$의 output data-rate
  - Constraints 설명
    - C1 : VNF가 하나의 node에 매핑됨을 보장
    - C2 : computing resource 한계조건
    - C3 : latency 한계조건
    - C4 : link bandwidth 한계조건
  - $s_t$는 valid한 matrix $\eta ^{s_t}$가 존재하는 경우 feasible함

- Slice와 AC의 목적 : **누적 InP 수익 최대화**
  $$
  \begin{gather*}
  \text{ maximize } r_{total} = \sum\limits_{t \in \mathbf{T}} {r^{s_t}} * \tau ^{s_t} \\
  \text{ s.t. } \quad (\text{C} 1) - (\text{C} 4), \tag{7}
  \end{gather*}
  $$

  - notation
    - $T$ : 다른 service type 전용 SR 도착시간 set
  - IF. slice 전용 문제로 공식화한다면?
    - SR에 대한 충분한 resource가 있을 때, SR을 거부하는 방법은 <u>실행 불가능한 embedding-relationship matrix를 만드는 것</u> ( slice idea와 **의도적으로 맞지 않음** )
  - Joint AC + slicing의 경우 : slice 문제 공식화에 AC를 포함하여 <u>AC 작업 중에 SR을 의도적으로 거부할 수 있음</u>
    - 목적 : resource bottleneck 방지 / 향후 있을 더 수익이 좋은 SR를 예측하여 resource를 보존



## Proposed Solution

- slicing과 AC 문제를 공동으로 해결하는 M-AC-VNE 방안 ( multi-agent DRL ) 제안
  - slicing agent, AC agent 해서 2개의 개별 agent가 사용됨
    - slicing agent : $s_t$가 실행가능한 경우 호출됨 ( valid한 $\eta ^{s_t}$ 가 존재하는 경우 )
    - AC agent : slicing agent가 valid한 $\eta ^{s_t}$를 생성하는 경우 호출됨
  
- agent들의 관측공간은 아래를 포함
  - 들어오는 SR의 service type ( URLLC, eMBB, mMTC )
  - SR의 동작 시간 ( $\tau^{s_t}$ )
  - 제공된 수익 ( $r^{s_t}$ )
  - 현재 substrate network 상태 ( $C^t, B^t$ )
  - $s_t$가 수용될 경우 결과의 substrate network 상태 // AC agent 한정
  
- DRL agent의 policy network weight를훈련시키기 위해 PPO 알고리즘 사용
  - on-policy, model-free, policy 기반 알고리즘으로, 다른 DRL보다 성능이 좋다고 함

  - 최적화식이 train episode가 끝난 후에만 계산될 수 있으므로, DRL agent의 reward로 사용하기에는 sparse함
    - -> Algorithm 1, 2를 사용하여 slicing과 AC에 대한 agent에 각각의 reward를 생성함
  
- 알고리즘 1

  ```Algorithm
  Algorithm 1
  Slicing agent's reward in M-AC-VNE
  ---
  Input : st, nst, (Ct, Bt), subsequent SRs (Ssub)
  Output : slicing agent's reward
  ```

  ```procedure
  reward = 0
  foreach constraint (C2 ~ C4) violated by nst do
      reward = reward - 1
  end
  if (nst is valid) and (st admitted by AC) then
      reward = reward + 1.5
  end
  for ssub in Ssub do
      if ssub not feasible then
          reward = reward - 1.5
      else
          return reward
      end
  end
  ```

  - **slicing agent의 경우**

    - reward : $\eta^{s_t}$에 의해 위배된 constraint 수, AC agent가 $s_t$를 승인하였으나 실행불가능한 SR ($S_{\text{sub}}$) 에 의해 달라짐
    - 목적 : bottleneck 현상 최소화하면서 AC agent가 허용할 수 있도록 <u>유효한 embedding-relationship 행렬</u>을 생성

- 알고리즘 2

  ```algorithm
  Algorithm 2
  AC agent's reward in M-AC-VNE
  ---
  Input : st, nst, (Ct, Bt), subsequent SRs (Ssub)
  Output : AC agent's reward
  ```

  ```procedure
  reward = 0
  if st is admitted then
      reward = reward + (rst * tau_st)
  end
  for ssub in Ssub do
      if ssub not feasible then
          reward = reward - (r^ssub * tau^ssub)
      else
          return reward
      end
  end
  ```

  - **AC agent의 경우**
    - reward : $r^{s_t}$에 의해 달라짐
      - $s_t$를 인정하면 그에 의해 제공되는 수익이 곧 reward
      - SR이 실행불가능일 경우, 잠재적인 수익 loss와 동일한 negative reward를 받음
        - Low Priority이며 bottleneck을 일으킬 가능성이 있는 <u>SR을 거부하는 policy 강화</u>
  - $1$, $1.5$ 수치는 negative reward와 positive reward 사이의 균형을 맞추기 위해 사용되는 상수 ( 시행착오 근거 )

- 제안된 접근법과 비교대상으로 S-AC-VNE 설계 ( single agent 방식 )

  - embedding-relationship 행렬과 AC 결정을 agent 하나가 동시결정

  ```algorithm
  Algorithm 3
  RL agent's reward in S-AC-VNE
  ---
  Input : st, nst, (Ct, Bt), max offered revenue by any SR (rmax)
  Output : RL agent's reward
  ```

  ```procedure
  reward = 0
  foreach constraint (C2 ~ C4) violated by nst do
      reward = reward - (rmax * tau^st)
  end
  if (nst is valid) and (st admitted by AC) then
      reward = reward + (rmax * tau^st)
  end
  return reward
  ```

  - **single agent의 경우**
    - reward : $s_t$가 성공적으로 임베디드되면 $r^{s_t}$에 비례
      - 아니라면 SR constraint 위반의 수와 비례하는 negative reward를 받음
      - embedding-relationship 행렬에 의한 constraint 위반의 경우, negative reward는 최대 제공 수익에 비례함
        - agent가 SR에 대해 실행불가능한 matrix를 생성하는 대신, SR을 거부하기 위한 AC action을 사용하기 위함
    - **slice와 AC에 대해 별도의 reward를 제공할 수 없음**
      - 한 측면의 학습은 다른 측면의 학습을 방해할 수 있음 ( 전체 policy에 영향을 미칠 수 있음 )
      - 하나의 work가 최적으로 돌아갈지라도 동기화가 해제됨



## Evaluation

- 추후 정리
- 5가지 벤치마크 존재
  - Greedy : 모든 SR을 받고 각 VNF를 RU에서 가까운 순의 node들에게 할당하는 것
    - 
  - Node Ranking : 