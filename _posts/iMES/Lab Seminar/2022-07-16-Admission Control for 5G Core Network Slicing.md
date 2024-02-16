---
title: "Admission Control for 5G Core Network Slicing Based on Deep Reinforcement Learning"
author: Joe2357
categories: [1. iMES, Lab Seminar]
tags: [Lab Seminar]
math: true
---

> 2022 / 7 / 25 iMES 세미나

## Abstract

- Network Slicing을 효율적으로 구현하기 위해서는 아래 메커니즘이 필요
  - Admission Control ( 접근 제어 )
  - Resource Allocation ( 자원 할당 )
- 기존 메커니즘 : Radio Access Network (RAN) 에만 초점이 맞춰져 있음
- <u>5G Core 네트워크</u>에서 Network Slicing을 위한 **지능적 / 효율적 메커니즘** 접근법 제안
  - AC 메커니즘 : SARA / DSARA의 2가지 강화학습 솔루션 도입
    - 5G 사용 사례에서 QoS 고려
    - Core / Edge node의 차별화
    - Time window 내에서 slice 요청 처리
      - 서비스 제공 업체의 수익 / resource 활용 목적



## Introduction

- 5G : 수많은 고객 / 장치에서 <u>주문형 (on-demand) 으로</u> 접근 가능한 수많은 서비스를 지원

- 3GPP : 5G usecase를 정의

  - eMBB (enhanced mobile broadband : 향상된 모바일 광대역
  - URLLC (ultrareliable low-latency communication) : 초신뢰성 저지연 통신
  - mMTC (massive machine-type communications) : 대규모 기계형 통신

- 네트워크 슬라이싱 : NSL (Network Slice, 맞춤형 / 격리된 논리 네트워크) 관리하기 위한 <u>유연성 / 모듈성 / 프로그래밍 가능성</u> 제공

  - SDN (Software Define Networking), NFV (네트워크 기능 가상화) 기술을 이용하여 구현
  - SLA (Service-level Agreement, 서비스 수준 계약) 으로 slice를 사용자 정의할 수 있음

- NSP (Network Slice Provider) : 5G usecase를 지원하기 위해 NSLR (NSL Request) 수신

  - NSL에 대한 지능적이고 효율적인 AC (Admission Control), RA (Resource Allocation) 수행
  - 제공자의 이익 증가 / core network 자원 활용도 향상
  - 기존 논문들의 AC 방식 : queuing theory (대기열 이론) / 휴리스틱 / RL / DRL 등이 있었음
    - 개별 NSLR을 고려하여 Admission decision을 제안하지만, 최근 할당된 resource들의 가용성으로 인해, 단기적으로 도착하는 <u>더 많은 수익성을 가진 요청이 거부될 수 있음</u> => **suboptimal**하다
    - QoS 요구사항이나 5G core network에서의 RA는 고려하지 않았음
  - RA만 고려한 논문들 : 휴리스틱 / queuing 이론 / complex network 이론 / 선형프로그래밍 / alternating direction method of multipliers / ANN / DRL 등이 있었음
    - AC에 대한 고려가 없었음 -> <u>NSP의 목표 달성 방해</u>
  - 결론 : 지능적 방식으로 **AC와 RA를 공동으로 수행**한다면 목표 달성을 향상시킬 수 있다

- 이 논문에서 5G network slicing에서 **지능적, 효율적인 AC, RA 제공**하여 <u>서비스 제공자의 이익과 네트워크 자원 활용도를 높이는</u> 새로운 접근법 제안

  - 2가지 솔루션 소개 - 수용 / 할당 결정을 내리기 위해 5G usecase 및 core network의 **QoS 요구사항 고려**
    - RL 기반의 NSL 요청 승인 \+ 자원 할당 (SARA)
    - DRL 기반의 심층 SARA (DSARA)
  - Q-Learning / Deep Q-Learning을 이용하여 주어진 시간대에 도달하는 NSLR 탐색 / 활용 / 학습 -> **이익 극대화**
    - DQL : 2개의 NN / replay memory / mini-batch로 개선시킴
  - RL / DRL을 이용하여 자원 활용을 선호할 수 있도록 descriptive parameter의 최신값을 보는 대신, <u>slice의 허용 결정 이력을 고려할 수 있음</u>
  - 수익 및 자원 활용 측면에서 2가지 알고리즘을 능가함을 보임
    - Always Admit Request (AAR)
    - Node Ranking (NR)

- Contribution

  - 5G Core network에서 NSL을 위한 효율적 AC / RA 달성을 위해 RL / DRL 사용하는 구조 제안
  - RL / DRL 기반의 알고리즘 개발 ( 5G usecase / QoS 요구사항 등을 고려 )
    - 최적의 수용 및 할당 결정 내림
    - core node와 edge node를 차별화
    - time window 내에서 NSLR를 처리
    - **수익 및 자원활용 극대화**
  - DRL 사용 이유 : RL 방식의 계산 장벽 극복 -> 더 큰 slicing 환경에서 더 효율적인 솔루션 만들 수 있음

- Acronyms of this paper

| Acronym |                      Term                       |               뜻               |
| :-----: | :---------------------------------------------: | :----------------------------: |
|   AAR   |              Always Admit Request               |        요청을 항상 수용        |
|   AC    |                Admission Control                |           수락 제어            |
|   ACM   |            Admission Control Module             |         수락 제어 모듈         |
|   AMF   |          Access and Mobility Function           |                                |
|   CP    |                  Control Plane                  |                                |
|   DQN   |                 Deep Q-Learning                 |          심층 Q 러닝           |
|   DRL   |           Deep Reinforcement Learning           |         심층 강화학습          |
|  DSARA  |                    Deep SARA                    |                                |
|  eMBB   |           enhanced Mobile Broad Band            |      향상된 모바일 광대역      |
|  mMTC   |       massive Machine Type Communications       |       대규모 기계형 통신       |
|   NFV   |         Network Function Virtualization         |      네트워크 기능 가상화      |
|   NN    |            artificial Neural Network            |           인공신경망           |
|   NR    |                  Node Ranking                   |          NR 알고리즘           |
|   NRF   |           Nerwork Repository Function           |                                |
|   NSL   |                  Network Slice                  |       네트워크 슬라이스        |
|  NSLR   |              Network Slice Request              |     네트워크 슬라이스 요청     |
|   NSP   |            Network Service Provider             |     네트워크 서비스 제공자     |
|   RA    |               Resource Allocation               |           자원 할당            |
|   RAM   |           Resource Allocation Module            |         자원 할당 모듈         |
|   RAN   |              Radio Access Network               |          무선 접속망           |
|   RL    |             Reinforcement Learning              |            강화학습            |
|  SARA   | Slice request Admission and Resource Allocation | 슬라이스 요청 수용 및 자원할당 |
|   SDN   |           Software-Defined Networking           |    소프트웨어 정의 네트워킹    |
|   SMF   |           Session Management Function           |                                |
|   UP    |                   User Plane                    |                                |
|   UPF   |               User Plane Function               |                                |
|  URLLC  |    Ultra Reliable Low Latency Communications    |      초신뢰성 저지연 통신      |
|   VNF   |            Virtual Network Function             |                                |



## Architecture for Admission Control based on (Deep) Reinforcement Learning

### A. 5G Core Network Substrate

- 유럽 통신 표준 협회 권고안의 구조를 따름
  - NFV 인프라 (NFVI) 는 존재 지점 (NFVI-POPs) 이 작동하는 여러 위치에 존재할 수 있음
  - 위치간 네트워크 또한 NFVI의 일부 -> 고려되어야함
  - NFVI-POPs : processing capacity를 제공하는 node ( 큰 data center / 작은 end user쪽 data center )
  - Core node : 높은 처리량 / 대역폭 요구하는 VNF 포함 -> **5G control plane에 적합**
  - Edge node : VNF가 end user쪽에 가깝게 존재해야함 -> **5G user plane에 적합**
  - NSL을 구성하는 VNF : edge / core 등에 배치할 수 있음
    - eMBB : 광범위한 VNF 활성화 / core, edge node에 걸친 distribution 필요
    - URLLC : latency 요구사항을 지원하기 위해 edge측에 배치된 VNF 필요
    - mMTC : latency 요구사항이 크게 없으므로 core측에 배치 가능
- <u>라벨링, 가중치를 부여한 **무방향 그래프**</u> $SN = \lbrace { N, L } \rbrace$로 모델링함
  - $N = \lbrace { n_1, n_2, ..., n_m } \rbrace$ : node들의 집합
    - 각 node들은 처리용량인 $CPU(n_i)$가 주어짐
  - $L = \lbrace { (n_1, n_2), (n_1, n_3), ..., (n_l, n_m) } \rbrace $ : link들의 집합
    - 각 link들은 bandwidth인 $BW(n_i, n_j)$가 주어짐

### B. 5G Network Slice Requests

- $nslr = \lbrace { s_{type}, T_o, G } \rbrace$로 표현
  - $s_{type}$ : 5G usecase를 나타냄 ( eMBB / URLLC / mMTC )
  - $T_o$ : 요청된 작동 시간 ( slice의 지속시간 )
  - $G = \lbrace { F, V } \rbrace$ : 라벨링되고 가중치가 부여된 무방향 그래프 ( NSL 표현 )
    - $F$ : VNF의 집합
    - $V$ : VNF를 잇는 가상 link의 집합
  - node의 label : VNF가 요구하는 resource의 양 / type 을 제공
    - 필요한 처리 용량 : $cpu(vnf_i)$
    - VNF가 요청한 node type : $type(vnf_i)$
  - edge의 weight : 가상 link에서 요청한 bandwidth 제공
    - 가상 link $(vnf_i, vnf_j)$가 요구하는 bandwidth : $bw(vnf_i, vnf_j)$
- 3가지 usecase ( eMBB, URLLC, mMTC ) 에 대한 slice 그래프를 제공
  - slice 그래프는 5G core plane (CP) / user plane (UP) 로 분리되어짐 ( 유연한 배포 / 독립적 진화 / 확장성 )
  - 각 그래프에는 필수 CP VNF (AMF, SMF) / UP VNF (UPF) 포함
    - AMF : 연결 및 이동성 작업 처리
    - SMF : data 단위 세션 설정 / 수정 / 해제
    - UPF : packet 라우팅 / 전달
  - URLLC 그래프 : AMF, SMF, UPF에 대한 **백업** 고려
    - 높은 신뢰성 / 낮은 latency를 제공해야함 -> <u>end user와 가까운 node에 instant화 되어야함</u>
    - eMBB와 mMTC는 초고신뢰성 요구사항이 없으므로 SMF와 UPF의 **백업 고려 X**
  - mMTC 그래프 : 다른 slice보다 **많은 AMF 소유**
    - 여러 type의 device가 제공되어야함
  - eMBB 그래프 : mMTC보다는 **적은 AMF 소유**
    - mMTC보다는 참여하는 device 개수가 적기 때문

### C. Modules

- 구조에 포함되는 모듈
  - 모니터링 모듈
    - 네트워크 substrate ( node / link ) 에서 "자원 가용성" 정보를 수집하고, 주기적으로 ACM과 RAM에게 정보 전달 ( state / reward 형식 )
  - 수락 제어 모듈 ( Admission Control Module, **ACM** )
    - agent와 우선순위 지정자 ( prioritizer ) 를 활용하여 NSLR의 승인을 수행
      - agent : 각 5G usecase에 대해 <u>정규화된 weight value</u>를 결정
      - prioritizer : 위 값을 이용하여 <u>NSLR 정렬</u> ( 자원 할당 순서 설정 )
        - 가중치 값은 최대 이익으로 이어짐 -> agent는 **최대 이익을 내는 쪽으로 action을 선택**해야함
    - agent : 환경과의 상호작용을 통해 state / reward의 정보를 고려하며 <u>이익을 증가 시키는 선택 학습</u>
      - state : agent가 action을 실행한 후 **사용 가능한 resource**
      - reward : action을 취함으로써 얻는 **profit**
    - prioritizer : NSLR의 [weight, 도착 시간] 에 의해 주어지는 비율에 따라 **최소 크기로** in batches queuing
      - ex. 각 usecase에 따라 가중치가 10, 5, 5의 NSLR이 들어온다면, 각각 2, 1, 1의 NSLR로 batch하여 queuing될 것
      - class당 NSLR은 시간 순서대로 queuing
    - ACM에는 2가지 agent 사용
      - RL에 기초한 agent ( SARA )
      - DRL에 기초한 agent (DSARA )
  - 자원 할당 모듈 ( Resource Allocation Module, **RAM** )
  - **lifecycle** 모듈
- 도착되는 slice request ( NSLR ) 는 time window 내에서 수집되면 일괄처리
- 특정 type의 모든 NSLR이 queuing되었다면, 다른 usecase의 weight를 이용하여 <u>후속 batch의 NSLR 수 결정</u>
  - ex. 모든 usecase마다 10개의 NSLR이 들어온 경우, 아래와 같은 상황이 가능할 것
    - 5개의 batch ( 2, 1, 1 NSLR ) + 5개의 batch ( 0, 1, 1 NSLR )
- 우선순위 큐를 만든 후, 각 NSLR은 dequeue되어짐 // ACM -> RAM으로 RA (자원 할당) 요청을 보냄
  - 자원 할당을 받았다면 NSLR 승인 / 받지 못했다면 NSLR 거부
  - 위 과정은 우선순위 큐가 빌 때까지 반복
- RAM : NSLR ( node mapping ) 을 구성하는 VNF를 가진 node에게 **자원 할당**
  - VNF는 네트워크 저장소 기능(NRF) 에서 사용할 수 있어야함
  - 이후, 할당된 node를 연결시키기 위해 선택한 link에 **bandwidth 할당** ( link mapping )
    - link 매핑 절차 : 가상 link의 요구 bandwidth를 만족하는 substrate 내의 **최단경로**의 가상 link를 매핑
  - 의사 결정을 위해, 2가지 사항 고려
    - node가 수요를 지원할 수 있는 **자원을 가지고 있는지**
    - NSLR type의 latency / 신뢰성 등의 **요구사항을 충족할 수 있는지**
  - CP VNF를 core node 측에 매핑
  - UP VNF는 대체로 core node쪽에 매핑하는 것이 좋음
    - URLLC의 UP VNF는 edge node에 매핑 ( 엄격한 latency 요구사항 충족 목적 )
  - 기본 VNF와 그것의 백업은 **동일한 node에 배치하지 않음** ( 신뢰성 요구사항 충족 목적 )

- node mapping, link mapping을 모두 성공하면 RAM -> ACM으로 성공알림을 보냄
  - 실패한 경우 nonmapped 알림을 보냄
- NSLR 수락 -> lifecycle 모듈이 VNF와 가상 link를 인스턴스화하여 NSL 구현
- NSL의 수명이 만료되면 자원할당 해제



## Admission Control based on RL

- SARA : ACM이 Q-Learning 기반 AC 알고리즘을 수행하는 agent

  - Q-Learning : off-policy (비정책적) / model-free (모델 없음) / temporal difference (시간적 차이) RL 알고리즘

    - agent : state ( $s_t$ ) 에 있을 때 action ( $a_t$ ) 를 수행 -> reward ( $R_t$ ) 획득, 다음 state ( $s_{t+1}$ ) 이동

    - Lookup table ( Q-table ) 을 사용하여 action의 quality value ( Q-value ) 를 아래와 같이 저장   
      $$
      Q_{t+1}(s_t, a_t) \leftarrow Q_t(s_t, a_t) + \alpha \cdot [R_t + \gamma \cdot \max Q(s_{t+1}, a) - Q_t(s_t, a_t)]. \tag{1}
      $$

      - $\alpha$ : learning rate [ 0 ~ 1 ] // RL agent에 의해 유지되는 Q값을 정의함
        - 0이라면 최근 Q값을 버림 ( agent가 새로운 것을 학습하지 않음 )
        - 1이라면 이전 Q값을 버림
      - $\gamma$ : 즉각적인 reward와 최대 예상 reward 사이의 균형을 맞추는 discount factor [ 0 ~ 1 ]
        - 1이라면 agent가 예상 미래 reward를 평가

### A. State Space

- state : 5G Core network substrate에서 **사용 가능한 resource**를 나타냄
- 상태공간 $\mathcal{S}$ : RL agent가 경험할 수 있는 모든 state 집합
  - $s \in \mathcal{S}$ 일 때, $s = \lbrace { cpu(E), cpu(C), bw(L) } \rbrace$로 나타낼 수 있음
    - $cpu(E)$ / $cpu(C)$ : edge set (E) / core node set(C) 에서의 가용 처리 용량
    - $bw(L)$ : link set (L) 에서의 가용 bandwidth

### B. Action Space

- 작업공간 $A$ : agent가 취할 수 있는 모든 action 집합
- 각 step마다 agent는 최대 이익을 줄 수 있는 action $a$를 선택
  - $a \in A$ 일 때, $a = \lbrace { w_{\text{embb}}, w_{\text{urllc}}, w_{\text{mmtc}} } \rbrace$로 나타낼 수 있음
    - $w_{x}$ : type $x$에서의 weight ( eMBB, URLLC, mMTC ) 를 각각 대표함
      - action space 크기를 줄이기 위해 범위가 [ 0 ~ 10 ] 사이로 되도록 제한
- 예시 : $a_i = \lbrace { 10, 5, 5 } \rbrace$
  - SARA가 eMBB type의 NSLR을 우선시해야함
  - eMBB를 2번 수용할 때마다 URLLC와 mMTC NSLR을 각각 하나씩 수용해야함

### C. Reward

- reward : 어떤 state에서 action을 취했을 때 얻을 수 있는 금전적 이익

  - action : 다양한 type의 NSLR 수용 / resource측도 cost를 사용하는 것

- 일반적으로 core node쪽의 resource가 edge보다 풍부하고 저렴함

- RL agent : 자원 활용도를 최적화하면서 **NSP 수익 극대화** 목적

  - reward 값은 agent가 action을 취하고 얻는 profit을 고려

  - NSL $nsl_i$를 수락하면서 얻은 이익 $p(nsl_i)$ : NSL을 판매하고 얻은 금액 - 처리 및 bandwidth 자원 운영 비용

    
    $$
    p(nsl_i) = (rev_i - cst_i) \times T_o \tag{2}
    $$

    $$
    cst_i = \sum_{j=0}^{m} cpu(vnf_j) \times f_{\text{cpu}j} + \sum_{j=0}^{n} bw(v_j) \times f_{bw} \times h \tag{3}
    $$

    |       용어        |                         설명                          |
    | :---------------: | :---------------------------------------------------: |
    |       $rev$       |        NSP가 $nsl_i$을 instant화하여 받는 수입        |
    |       $cst$       |                $nsl_i$를 구동하는 비용                |
    |       $T_o$       |                  $nsl_i$의 작동 시간                  |
    |        $m$        |                  $nsl_i$의 VNF 개수                   |
    |        $n$        |               $nsl_i$의 가상 link 개수                |
    |   $cpu(vnf_j)$    |                  $vnf_j$의 cpu 수요                   |
    |     $bw(v_j)$     |           가상 link $v_j$의 bandwidth 수요            |
    | $f_{\text{cpu}j}$ |     node 유형에 따라 달라지는 $vnf_j$의 처리 cost     |
    |     $f_{bw}$      |                    bandwidth cost                     |
    |        $h$        | 가상 link $v_j$가 할당되는 경로를 구성하는 hop의 개수 |

  - reward $R$은 아래와 같이 계산

    
    $$
    R = \frac{\sum_{i=0}^{k} p(nsl_i)}{\max P(SN, T)} \tag{4}
    $$

    - $\max P(SN, T)$ : 특정 시간 $T$에 substrate $SN$의 모든 자원을 사용할 때 NSP가 얻을 수 있는 **최대 이익**

### D. Exploration and Exploitation Method

- RL agent : 각 step마다 action을 explore(탐색) 하거나 exploit(이용) 하기 위해 <u>epsilon-greedy 방법</u> 사용

  - action을 고르기 위해서 난수 ( $rn$ ) 생성 ( $rn \in [0, 1]$ )
  - 만약 $rn > \epsilon$ 이라면 최대 Q값을 갖는 action을 취함 ( 과거의 최적의 action을 exploit )
  - 아니라면 random action 수행 ( 새로운 action을 explore )

  $$
  a = \begin{cases}
  \max\limits_{a} Q_t(s_t, a), & \text{if }rn > \epsilon \\
  \text{random action}, & \text{otherwise}
  \end{cases} \tag{5}
  $$

### E. RL-Based Admission Control Algorithm

- SARA의 RL agent로는 Q-Learning 알고리즘 사용

  ```algorithm
  Algorithm 1
  RL-based AC Algorithm
  ---
  Data : Learning parameters
  Input : Sets of NSLRs (RS) collected during the time window
  Result : Admission of NSLRs that generate the maximum profit
  ```
  
  ```procedure
  Initialize Q : Q(S, A) = 0, ∀s ∈ S, ∀a ∈ A
  for episode ← 1 to n do
      The agent observes the initial state si // when 100% of substrate resources are available
      while next state s(t+1) is not the final state do
          The agent choosed at using ε-greedy exploration method (Eq. 5) // selection of a random or an optimal action that increases the profit
          The agent invokes the Prioritizer to sort the NSLRs PR into a priority queue
          for each nslr ∈ PR do
              The agent invokes the RAM that runs Algorithm. 3 to map nslr;
              if nslr is mapped then
                  The agent admits nslr and send information to Lifecycle;
              end
              else
                  The agent rejects nslr;
              end
          end
          The agent observes the reward Rt that is calculated by using Eq. 4
          The agent observes the new state s(t+1) // The current resource availability
          The Q-table is updated according to Eq. 1
          The current state is updated st ← s(t+1)
      end
  end
  ```

- 알고리즘 목표 : 가장 수익성이 높은 NSLR set을 수용하는 것
  
  - 시간은 discretized (이산화) 되어진다고 가정
  
  - 입력 : $RS = \lbrace { nslr_1, ..., nslr_n } \rbrace$ (  time window 내에 들어온 NSLR들 )
  
  - 알고리즘이 사용하는 data
  
      - action 공간 $A$
      - state 공간 $\mathcal{S}$
      - learning rate $\alpha$
      - discount factor $\gamma$
      - exploration-exploitation factor $\epsilon$
      - step 수 $m$
        - 하나의 step : agent와 환경의 <u>한번의 상호작용</u> ( state, action, reward )
      - learning episode 수 $n$
      - 하나의 episode : $m$개의 step
  
  - $n$번의 episode가 끝나면 output으로 각 5G usecase에 대한 **가중치 값**을 제공
  
  - 절차
  
      - Line 1 : \|S\| x \|A\| 차원의 Q table 생성
        - 초기값은 0으로 초기화
        - 각 state-action 쌍마다의 Q값을 저장하는 lookup table임
      - Line 2 : $n$번의 episode를 반복하는 outer loop
        - Line 4 : $m$번의 step을 반복하는 inner loop
          - agent : $\epsilon$-greedy 방법을 이용하여 random action / 최적의 action $a_t$ 중에서 선택
          - Q값이 과거의 reward에 의존하므로, RL agent는 이익을 높이는 방향으로 action을 선택하도록 학습됨
          - Line 6 : 선택된 최적 action $a_t$는 prioritizer로 보냄
            - 모든 NSLR을 우선순위 큐에 정렬 ( 도착 시간, usecase마다의 가중치 )
            - NSLR을 dequeue되어 RAM으로 전송됨
          - Line 8 : resource가 할당되면 NSLR은 substrate 상에 매핑됨 ( 수용된 것으로 간주 )
          - Line 10 : NSLR 수용 정보는 lifecycle 모델에게 전송
          - Line 16 이후 : RL agent는 action에 대한 reward 받음, 새로운 상태 $s_{t+1}$ 관찰
            - $s_{t+1}$ : $a_t$를 수행한 이후 substrate에서 사용 가능한 새로운 processing / bandwidth 용량 )
          - Q-table 업데이트 진행, $s_{t+1}$을 현재 상태 $s_t$로 지정하고 iteration 반복
  
  - 시간복잡도 : state-action pair $s$, agent가 학습해야하는 각 pair에 유한한 visit 수 $r$, 알고리즘 3의 시간복잡도 $O(n + n \log n + fn + v(n+l))$에 의존함
  
      | 용어 |        설명         |
      | :--: | :-----------------: |
      | $v$  |   가상 link 개수    |
      | $n$  | substrate node 개수 |
      | $l$  | substrate link 개수 |
      | $f$  |      VNF 개수       |
      
      - 알고리즘 1은 알고리즘 3을 RA에 대해 $sr$번 반복함
      - 따라서 시간복잡도는 $O(sr(n + n \log n + fn + v(n+l)))$



## Admission Control based on DRL

- Q-Learning 알고리즘의 문제점 2가지 ( 큰 환경 )

  - agent가 state-action 쌍을 여러번 방문해보아야 하므로 **수렴시간이 너무 길어짐**
  - Q값을 모두 저장해두어야할 정도의 **높은 메모리 사용량 요구**

- DQN : 이미 방문한 일부 state에서 배운 지식을 다른 유사한 state로 일반화하기 위해 <u>함수 근사치</u>를 사용할 수 있음 ( 일반적으로 NN )

  - DQN agent는 환경과의 몇가지 상호작용을 통해 학습할 수 있도록 함 ( DSARA 제안 )
  - DQN agent : 가장 잘 추정된 Q값으로 action을 취함, reward $R_t$를 받음, 새로운 상태 $s_{t+1}$ 관찰

- DSARA : replay memory, Eval NN, Target NN으로 구성된 DQN으로 AC 알고리즘 실행

  - 각 학습 step에서 최적의 수락 action을 학습하고, 안정성 달성

- 이전의 DQN : 단일 NN을 사용하여 Q값 추정, Q값 타겟팅 -> **추정치에서 높은 상관관계 발생** 

  - DSARA : replay memory에 경험을 저장 $( s_t, a_t, s_{t+1}, R_t)$ // 평가 NN을 학습하기 위한 mini-batch을 사용하기 위함

    - 평가 NN : Q값 추정 ( $Q(s_t, a_t)$ )

    - 타겟 NN : 타겟 Q값 추정

      
      $$
      Q^{+}(s_t, a_t) = R_t + \gamma \cdot \max Q(s_{t+1}, a). \tag{6}
      $$

      - 타겟 Q값은 평가 NN을 훈련시킬 때 loss를 계산하기 위한 label로 쓰임

    - 평가 NN 가중치로 target NN 가중치를 주기적으로 update

### A. State-Space, Action-Space, and Reward

- SARA와 같은 reward, action 구조 체계를 가지고 있음
- 상태공간 $\mathcal{S}$ : DRL agent가 경험할 수 있는 모든 state의 집합
  - $s \in \mathcal{S}$일 때, $s = \lbrace { cpu(E), cpu(C), bw(L), n_e, n_u, n_m } \rbrace$ 으로 표기
    - $n_e, n_u, n_m$ : 각각 eMBB, URLLC, mMTC를 실행하는 NSL의 개수

### B. Exploration and Exploitation Method

- epsilon-greedy 정책을 따르지만, **좀 더 decay한 $\epsilon$을 사용**

  - 탐구적인 방향에서 벗어나 <u>더 exploitative한 방향으로 $\epsilon$을 이동</u>

- DRL agent : 처음에는 아래 수식에 따라 decay 탐색  

  
  $$
  \epsilon_{t} = \epsilon_{\text{max}} - (n\text{steps}_t \times dr). \tag{7}
  $$

  |          용어           |                             설명                             |
  | :---------------------: | :----------------------------------------------------------: |
  | $\epsilon_{\text{max}}$ |           탐색 최대 확률 ( training 시작때 쓰임 )            |
  |    $n\text{steps}_t$    |                    agent가 경험한 step 수                    |
  |          $dr$           | decay rate ( $\epsilon$을 시간이 지남에 따라 선형적으로 줄이는 상수 ) |
  |          $rn$           |          agent가 생성하는 난수 ( $rn \in [0, 1]$ )           |

  - agent의 action 선택은 SARA와 같은 식 활용 ( Eq. 5 )

### C. Neural Networks

- 평가 NN, target NN의 구조는 아래와 같음

  - 입력층에 6개의 뉴런 ( state tuple에 있는 변수 하나당 뉴런 1개 )
  - 150개 뉴런이 있는 single hidden layer // target function을 근사하기에 충분하다
  - 30개 뉴런이 있는 출력층 ( action space에서 action 하나당 뉴런 1개 )
    - 출력층의 각 뉴런은 action $a_i \in A$과 연관된 Q값인 $Q_i$를 추정

- DSARA : 평가 NN의 각 계층에서 weight와 bias를 조정하여 Q값 추정 loss를 줄이기 위해 gradient descent, backpropagation 방식 사용

  - loss ( error cost ) : 평가 NN에 의해 추정된 Q값과 target NN에 의해 획득한 target Q값과의 차이

  
  $$
  \text{Loss} = (Q^{+}(s_t, a_t) - Q(s_t, a_t))^2. \tag{8}
  $$

  - 학습 안정성을 이유로 target NN을 일시적 수정, 평가 NN의 훈련된 parameter를 이용하여 주기적 update

### D. DRL-based Admission Control Algorithm

- DQN 알고리즘을 이용하여 DSARA의 DRL agent를 구현

  ```algorithm
  Algorithm 2
  DRL-based AC Algorithm
  ---
  Data : Learning parameters
  Input : sets of NSLRs (RS) collected during time windows
  Result : Admitted NSLRs that generates the maximum profit
  ```

  ```procedure
  Initialze: Evaluation NN Q with weights θ, Target NN Q^ with weights θ^, and Replay Memory D
  for episode ← 1 to n do
      The agent observes the initial state si
      while next state s(t+1) is not final state do
          Update exploration probability ε by using Eq. 7
          The agnet selects action a(t) according to Eq. 5
          The agent invokes the Prioritizer to sort the NSLRs into a priority queue PR
          for each nslr ∈ PR do
              The agent invokes RAM that runs Algorithm 3 to map nslr
              if nslr is mapped then
                  The agent admits nslr and sends it to Lifecycle
              end
              else
                  The agent rejects nslr
              end
          end
          The agent receives reward Rt (Eq. 4) and observes the new state s(t+1)
          Store experience et = (st, at, s(t+1), Rt) into D
          Sample random mini-batch of experiences from D
          Calculate Q+(st, at) = Rt + γ . max Q+(s(t+1), a), if state is final state, Q+(st, at) = Rt;
          Minimize loss function (Eq. 8) by gradient descent and updates the weights θ of the Evaluation NN Q;
          Update the current state st ← s(t+1);
      end
      Every C episodes, the agent copies weight and biases from Evaluation NN to Target NN
  end
  ```

  - 알고리즘 목적 : 가장 수익성이 높은 NSLR을 수용하는 것
    - 시간은 이산화되어있다고 가정
    - input : 주어진 시간 window 내에 도달한 NSLR 집합
    - output : 이익을 극대화할 수 있는 수용할 NSLRs
    - 알고리즘이 사용하는 data
      - hidden layer의 뉴런 수
      - discount factor ( $\gamma$ )
      - 감쇠율, decay rate ( $dr$ )
      - 최대 탐색, maximum exploration ( $\epsilon$ )
      - 훈련 시작, training start ( $k$ )
      - mini-batch 크기 ( $sz$ )
      - target NN update ( $C$ )
      - action 공간 ( $A$ )
      - state 공간 ( $S$ )
      - episode 수 ( $n$ )
    - 절차
      - Line 1 : 평가 NN, target NN, replay memory를 초기화
      - Line 2 : $n$번의 episode를 반복하는 outer loop
        - Line 4 : $m$번의 step을 반복하는 inner loop
          - agent : decay (Line 5, Eq. 7) 와 함께 탐사 확률 $\epsilon$을 update
          - agent : 현재 상태 $s_t$에서 2개중 선택 ( random action, 현재의 최적 동작 $a_t$ )
            - $a_t$ : 우선순위 지정자가 NSLR을 정렬할 때 사용하는 각 usecase마다의 weight를 포함 (Line 7)
          - Line 8 ~ 16 : NSLR이 dequeue되면서 RAM으로 요청 전송
            - Line 11 : NSLR이 성공적으로 매핑되면 lifecycle 모듈로 정보 전송
          - Line 17 : agent가 보상 $R_t$ 수신, 새로운 상태 $s_{t+1}$ 관찰
          - Line 18 : 경험 $e_t = (s_t, a_t, s_{t+1}, R_t)$ 가 replay memory D에 저장됨
          - Line 19 ~ 21 : $k$개의 경험이 쌓인 이후에 실행됨
            - agent가 평가 NN을 훈련시키기 위해 replay memory로부터 경험 몇개 ( mini-batch ) 를 무작위로 가져옴
              - agent와 환경과의 상호작용 횟수를 줄이는 효과
            - 경험으로부터 $s_t, R_t, s_{t+1}$을 취함으로써, 평가 NN이 $Q(s_t, a_t)$을 추정하고, target NN이 $Q+(s_t, a_t)$를 계산
            - loss를 줄이기 위해 평가 NN의 weight / bias를 조정해야함 -> gradient descent / backpropagation 사용
              - loss가 낮다 = agent의 **추정이 정확하다**
          - 새로운 state가 현재 state가 되고, agent가 새로운 반복을 시작함
        - target NN의 parameter는 고정된 상태로 유지하며, 매 episode $C$마다 평가 NN의 parameter로 변경 ( 학습의 안정성 )

- 알고리즘 2와 1은 유사한 구조를 가지고 있음

  - 시간 복잡도 : $O(sr(n + n \log n + fn + v(n + l)))$
    - agent가 훈련을 위해 state-pair 쌍을 방문해야하는 횟수 $r$과 알고리즘 3의 복잡도에 의존하기 때문
    - 알고리즘 2가 RA 영역에서 알고리즘 3을 사용함



## Resource Allocation

- RAM : NSLR에게 **자원을 할당하는 것**이 목표 ( mapping, embedding )
  
  ​    $M : G = \lbrace { F, V } \rbrace \rightarrow SN' = \lbrace { N', L' } \rbrace $
  
  - $G = \lbrace { F, V } \rbrace $ : 라벨링되고 가중치가 부여된 무방향 그래프 ( NSL 표현 )
  
    - $F$ : VNF의 집합
    - $V$ : VNF를 잇는 가상 link의 집합
  
    - $N' \in N$ : node들의 집합
    - $L' \in L$ : link들의 집합
  
  - latency, bandwidth, 처리 및 신뢰성 요구사항을 충족해야함
  
- 알고리즘 3이 RA 알고리즘을 묘사함

  ```algorithm
  Algorithm 3
  RA Algorithm
  ---
  Input : An NSLR, substrate resource availability
  Result : Mapped NSLR
  ```

  ```procedure
  for each sustrate node n ∈ N do
      Calculate embedding potential (Eq. 9);
  end
  for each vnf ∈ F do
      for each node n in the ranked list do
          if match(type(n), type(vnf), s_type) == True and isAllowed(n, vnf) == True and CPU(n) / cpu(vnf) ≥ 1 then
              Map vnf onto n;
              Break;
          end
      end
      if vnf is not mapped then
          Return non-mapped notification;
      end
  end
  for each virtual link v ∈ V do
      Obtain source src and destination dst from v;
      Compare candidate paths from src to dst;
      Sort the paths into the list CandidatePaths regarding the number of hops // The first path in the list has the lowest number of hops
      for each path CP in CandidatePaths do
          if each link l ∈ CP satisfies BW(l) / bw(v) ≥ 1 then
              Map v onto CP;
              Break;
          end
      end
      if v is not mapped then
          Return non-mapped notification;
      end
  end
  Return mapped NSLR
  ```

  - 알고리즘 목적 : SARA와 DSARA에서 사용하는 RA 알고리즘 구현
    - 2가지 단계로 이루어져있음
      - node mapping ( Line 1 ~ 15 )
      - link mapping ( Line 16 ~ 29 )
    - 입력 : ACM에서 제공하는 $nslr = \lbrace { s_{\text{type}}, T_o, G } \rbrace$
    - 출력 : 자원 할당 성공 / 실패의 notification
    - 허용된 nslr에 대한 자원 할당 정보는 lifecycle 모듈로 전달됨

### A. Node Mapping

- 목표 : 처리, latency, 신뢰성 요구사항을 존중하면서 **NSLR의 VNF를 substrate network의 node에 매핑** ( $F \rightarrow N'$ ) 하는 것

  - $F$의 각 node는 각각 VNF를 나타냄, 각각의 label은 처리 요구량 $cpu(vnf)$를 제공 ( 충족시켜야함 )
  - usecase type $s_{\text{type}}$은 SN에서 node type을 선택하는 데 사용

- NSL 그래프에 대한 고려가 있음

  - VNF는 $\text{type}(vnf)$( CP ,UP ) 와 $s_{\text{type}}$( eMBB, URLLC, mMTC )에 따라 매핑됨
    - CP에 속한 VNF는 항상 core node에 매핑됨
    - UP에 속한 VNF는 usecase에 따라 매핑됨
      - URLLC : edge node에 매핑 ( 엄격한 latency 요구사항 )
      - mMTC : core node에 매핑 ( latency에 민감하지 않아서 )
      - eMBB : core node에 우선적으로 매핑, 불가능하다면 edge node에 매핑
    - 백업 VNF는 본 VNF와 다른 node에 매핑 ( 신뢰성 향상 )

- substrate node인 $n \in SN$은 potential embedding value $EP$에 따라 정렬됨 ( Line 1 ~ 3 )  

  
  $$
  EP(n_i) = cpu(n_i) \times \sum_{l \in adj(n_i)} bw(l_j) \tag{9}
  $$

  - 이용 가능한 처리용량 $CPU(n_i)$와 이용 가능한 bandwidth 용량 $BW(n_j)$에 기초하여, VNF을 embed하기 위한 **substrate node의 capaicty를 반영**
  - RAM : EP값의 내림차순으로 기판 node 정렬

- Line 5 ~ 15 : 각 VNF $vnf \in F$를 기판 node $n$에 매핑하려고 시도

  - 3가지 조건 ( Line 7 ) 을 모두 만족해야 성공적인 매핑
    - node $n$의 type $\text{type}(n)$이 usecase의 type $s_\text{type}$과 VNF type $\text{type}(vnf)$와 맞아야함
    - 백업 VNF는 primary VNF와 같은 node에 설치되면 안됨 ( isAllowed 함수 )
    - 가용 처리 용량이 요구 처리 용량보다 커야함 ( $\frac{CPU(n)}{cpu(vnf)} \ge 1$ )
    - 모든 조건이 충족되면 $vnf$는 $n$에 매핑, node 자원 할당
      - 아니라면 다음 node를 방문하여 매핑할 수 있는지 판단
      - RAM이 <u>적어도 하나의 VNF를 매핑할 수 없다면</u> nslr에 대해 "매핑되지 않음" 알림을 통지
        - NSLR에 할당된 모든 node를 할당해제
        - 결국 **모든 VNF를 매핑할 수 있어야** RAM이 link mapping 단계로 넘어감

### B. Link Mapping

- 목표 : 가능한 한 적은 bandwidth를 소비하여 가상 link를 기판 link에 매핑하는 것 ( $V \rightarrow L'$ )
  - 추구되는 path : <u>bandwidth 요구사항을 충족하면서 가장 적은 수의 hop을 지나는 경로</u> ( 네트워크 상의 bandwidth 이용률 최소화 목적 )
- Line 16 ~ 29 : RAM이 link mapping 절차 수행
  - 각각의 가상 link $v \in V$에 대해, $v$를 substrate path에 매핑하는 시도를 함
    - Line 18 : 시작점 $src$와 도착점 $dst$ 사이의 후보 path를 계산
    - Line 19 : path들을 hop의 개수에 대해 내림차순 정렬, CandidatePaths 대기열에 넣음
      - 이후 하나씩 빼서 $CP$로써 사용 ( hop 수의 관점에서 **가장 짧은 경로**임 )
    - RAM : CP에 있는 모든 link에 대한 bandwidth 가용성이 요구량을 충족하는지 판단 ( $\frac{BW(l)}{bw(v)} \ge 1$ )
      - 모든 link가 충족한다면 $v$를 CP의 link에 매핑
        - 아니라면 다음 최단경로를 고려함
      - RAM이 <u>적어도 하나의 가상 link를 매핑할 수 없다면</u> ACM에게 "매핑되지 않음" 알림을 통지
        - node가 할당해제됨
        - **모든 link를 매핑해야** RAM이 ACM에게 매핑알림을 보냄 // 모두 성공했을 경우

### C. Algorithm Complexity

- Line 1의 첫 loop는 시간복잡도가 $O(n)$ ( $n$ : substrate node 개수 )
- Line 4의 NR은 timsort 사용, 최악 시간복잡도 $O(n \log n)$
- Line 5의 2번째 loop는 시간복잡도가 $O(fn)$ ( f : VNF 개수 )
- Line 16의 3번째 loop는 시간복잡도 $O(v(n + l))$
  - 가상 link 개수 $v$번 반복, 1번당 시작점 -> 도착점까지 bfs 계산 복잡도가 $O(n + l)$ ( $l$ : substrate link 개수 )
- 최종 Algorithm 3의 복잡도 : $O(n + n \log n + fn + v(n + l))$



## Performance Evaluation

### A. Metrics

- 평가에 사용할 metric : 이익 / 자원 활용도 / 수용 비율

  - 이익 $P$는 아래 식으로 계산 가능  

    
    $$
    p(nsl_i) = (rev_i - cst_i) \times T_o \tag{2}
    $$

  - 자원 활용도는 아래 식으로 계산 가능

    
    $$
    U = \frac{\frac{\sum_j cpu(nsl_j)}{CPU(SN)} + \frac{\sum_j bw(nsl_j)}{BW(SN)}}{2} \tag{10}
    $$

    - $CPU(SN)$ : SN의 전체 처리 용량
    - $\sum_j cpu(nsl_j)$ : SN에 instant된 모든 NSLR에 의해 활용되고 있는 처리자원
    - $BW(SN)$ : 전체 network bandwidth
    - $\sum_j bw(nsl_j)$ : 모든 NSLR에 의해 활용되고 있는 bandwidth

  - 수용 비율은 아래 식으로 계산 가능

    
    $$
    AR = \frac{req_a}{req_t} \tag{11}
    $$

    - 수용된 NSL ( $req_a$ )
    - 전체 NSLR 수 ( $req_t$ )

### B. Experimental Setup

- 구조 모듈은 Python 3을 이용하여 구현
- 추후 정리

### C. Result and Analysis

- 추후 정리



## Conclusion

- 5G Core network에서 eMBB, URLLC, mMTC를 위한 NSLR를 위한 AC / RA를 효율적으로 구현하기 위한 방법으로 SARA, DSARA 제안
  - 주어진 time window 내에서 NSLR를 가장 수익성 높게 뽑을 수 있도록 Q-Learning, DQL을 사용하여 구현
  - model-free이므로 substrate network에 대한 가정을 하지 않음
- DSARA가 다른 알고리즘들에 비해 더 큰 이익 / 더 높은 자원 활용 가치를 달성함을 보임
  - NSP가 SARA / DSARA를 채택할 가치가 있음을 입증
- 향후 연구
  - AC 매커니즘 풍부하게 : DRL의 최신 향상된 매커니즘 활용
  - RA 매커니즘 풍부하게 : 서비스당 도착 요청 수를 예측하고, VNF 체인에 자원 할당을 수행하기 위한 예상 네트워크 사용률과 관련하여 **정교한 전략을 갖추도록**
  - 5G의 RAN, Core network와 관련된 end-to-end slice에 대해 AC, 적응형 RA 기술을 제공할 계획

