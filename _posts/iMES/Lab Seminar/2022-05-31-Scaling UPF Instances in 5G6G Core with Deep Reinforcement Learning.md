---
title: "Scaling UPF Instances in 5G/6G Core With Deep Reinforcement Learning"
author: Joe2357
categories: [1. iMES, Lab Seminar]
tags: [Lab Seminar]
math: true
---

> 2022 / 6 / 20 iMES 세미나

## Abstract

  - UPF ( User Plane Function ) : PDU ( Protocol Data Unit ) 세션에서 가입자와의 data 전송 담당
    - 소프트웨어로 구현됨
    - cluster에서 특정 resource 요구 사항이 있는 UPF instance로 시작될 수 있는 <u>가상 system / container</u> 등으로 압축될 수 있음
      - resource 소비를 줄이는 방법 : UPF instance의 수를 PDU 세션 수에 맞춰서 조절해야함 ( **scaling 알고리즘** )
  - DRL 방식 제안 : 쿠버네티스 container orchestration framework의 container에 패킹된 UPF instance을 scaling 목적
    - 역치 기반 보상함수 ( threshold-based reward function ) 공식화
    - 근위 정책 최적화 ( PPO 알고리즘 ) 적용
    - SVM ( Support Vector Machine ) 분류기 적용 : agent가 확률적 정책으로 인해 <u>원하지 않는 행동을 하는 경우에 대처하기 위함</u>
  - 이 접근 방법은 쿠버네티스 내장 HPA ( Horizontal Pod Autoscaler ) 보다 성능이 좋음
    - DRL은 평균 pod의 개수를 2.7% ~ 3.8% 절약
    - SVM은 0.7% ~ 4.5% 절약

  

## Introduction

  - 5G, 6G : 다양한 수직 산업에서 고객을 위한 서비스를 제공할 것
    - 네트워크 요소 및 기능의 변환 시작 ( 전용 특수 하드웨어 -> 소프트웨어 기반 container )
  - 3GPP : 서비스 기반 구조 (SBA) 를 기반으로 5G core framework 지정
    - 5G의 핵심 구조 요소는 VM / cloud의 container 등에서 실행되어짐
    - 6G가 5G core의 user plane function을 유지할 가능성이 높음
  - 네트워크 기능을 호스팅하는 가상 시스템 & container는 **cloud orchestration framework 내에서 실행됨** ( instance에 대한 cloud resource 관리 / 할당 )
  - 고객의 traffic 수요 변동에 따라 **network 기능의 instance를 조정해야함** ( 실행 / 종료 등 )
    - ex. 고객은 data 통신 전에 PDU 세션에 대한 요청을 시작해야함
    - 이 request는 peak time이 있고, 정규 기간이 있음 ( peak time에는 더 많은 instance가 필요 )
    - 따라서 <u>resource 사용을 제어하기 위한 적절한 알고리즘이 필요</u>
  - 논문 목적 : 5G / 6G core에서 UPF instance의 resource 관리에 **DRL** 적용방안 제안
    - UPF instance를 제어하는 쿠버네티스 내장 알고리즘인 Horizontal Pod Autoscaler (HPA) 와의 성능비교
    - DRL 방식에 의해 생성된 policy는 <u>때로 원치 않은 결정을 내리기도 했음</u> ( 랜덤성? )
      - 해결 방안 : **DRL agent에 SVM 적용** -> action을 classify하는 역할
    - 결과 : HPA보다 더 나은 성능을 낼 수 있는 **결정론적 SVM 기반 policy** 얻음 ( 약간의 성능저하가 있음 )
  - Contribution
    - 쿠버네티스 기반 cloud에서 UPF instance를 확장하는 문제 공식화
      - DRL 적용하여 HPA와의 성능비교 / 시뮬레이션
    - DRL agent가 원하지 않은 action을 취하는 경우에 대한 handling
      - 원인 : 학습된 policy의 확률적 특성
      - 해결방안
        - action을 label로 하는 state dataset 생성
        - SVM 모델을 훈련
      - traffic 변화가 느리다면 SVM agent는 DRL agent보다 성능이 좋지 않음
        - **갑작스런 traffic 변화** 환경에서 SVM agent는 원치 않은 action을 취하지 않음

  

## Notation

  - 환경에서의 변수 notation
    - $D_{min}$ : 시스템에서 pod의 최소 수
    - $D_{max}$ : 시스템에서 pod의 최대 수
    - $L_{sess}$ : pod에서의 PDU 세션 최대 수
    - $t_{pend}$ : pod의 첫 실행 시간 ( 부팅시간 )
    - $d_{busy}$ : busy pod의 개수
    - $B$ : 각 pod의 최대 pod 개수
    - $p_{b, th}$ : blocking rate threshold
    - $\lambda$ : PDU 세션의 arrival rate
    - $\mu$ : 세션의 service rate
    - $\Delta T$ : 두 decision 사이의 시간
  - 관측에서의 변수 notation
    - $d_{free}$ : idle pod의 개수
    - $d_{boot}$ : booting pod의 개수
    - $d_{on}$ : 실행 중인 pod의 개수 ( idle + busy )
    - $l_{sess}$ : PDU 세션의 개수
    - $l_{free}$ : 시스템이 더 수용 가능한 PDU 세션의 개수 ( 최대 - $l_{sess}$ )
    - $p_b$ : blocking 확률 ( request가 거절될 확률 )
    - $\hat{\lambda}$ : 이전 decision 이후의 근사 arrival rate
    - $\hat{p}_{b}$ : 이전 decision 이후의 측정된 blocking rate
  - MDP ( Markov Decision Problem ) 에서의 변수 notation
    - $\mathcal{S}$ : MDP에서의 state set
    - $\mathcal{A}$ : MDP에서의 action set
    - $p$ : MDP에서의 transition 확률
    - $T_{\text{i}}$ : i번째 decision time
    - $r, r_{\text{i}}$ : 중간 reward 함수, 시간 $T_{\text{i}}$에서의 i번째 reward
    - $\kappa$ : reward multiplier
    - $\gamma$ : discounting factor
    - $\mathcal{P}$ : set의 확률 분포
    - $s(t), s_{\text{i}}$ : 시간 $t$에서의 state, 시간 $T_{\text{i}}$에서의 i번째 state
    - $a(t), a_{\text{i}}$ : 시간 $t$에서의 action, 시간 $T_{\text{i}}$에서의 i번째 action
    - $\pi, \pi^{*}$ : policy function, optimal policy
    - $V$ : value function

  

## The Operation and Scaling Issue of 5G UPF Instances

### A. 5G UPF

  - User Equipment (UE) 와 Data Network (DN) 연결을 위해서는 PDU session을 설정해야함
    - UE -> Radio Access Network (RAN)의 gNB -> transport network -> 5G Core -> DN
    - transport network는 무선 / 유선 / 광 접속 등이 있음
    - 5G Core는 서비스 기반 구조 (SBA) 를 구현하는 다양한 기능들의 집합
      - Access and Mobility Management Function (AMF) : UE의 access 제어
      - Session Management Function (SMF) : PDU 세션의 설정 및 종료 지원, 세션 상태 추적관리
      - User Plane Function (UPF)
    - AMF & SMF : control plane / UPF : user plane
      - UPF : PDU session Anchor (PSA) 역할 수행 // 5G core에 접속 network를 위한 연결지점 제공
      - 추가로, 패킷 검사, 라우팅, 전달 처리 -> QoS 처리, 특정 traffic rule 적용 가능
    - Control and User Plane Separation (CUPS) : 개별 요소가 독립적으로 확장될 수 있고, 데이터 처리가 네트워크 edge에 더 가깝게 배치됨을 보장

### B. 5G UPF Instances within the K8S Framework

  - 쿠버네티스 : cloud 기반 인프라에서 container화된 app을 관리하는 오픈소스 container orchestration 플랫폼
    - pod : 쿠버네티스의 특정 app을 위한 **가장 작은 deploy 가능한 computing 장치** ( 더 많은 container를 포함 가능 )
    - node : 물리적 / 가상적 cloud 상의 machine ( 특정 resource set ( CPU, 메모리, disk 등 ) 보유 )
    - controller : node에서 주어진 resource 요구사항으로 pod 구성
      - pod 내에서 하나 이상의 container를 실행
  - UPF 설정 방법 : UPF instance를 pod에 mapping하는 것
  - 5G network에는 다양한 유형의 requirement를 가진 사용자에게 서비스 제공 가능
    - PDU 세션에는 서로 다른 requirement가 있을 수 있음
      - PDU 세션 유형의 수를 제한해야할 필요성 생김
    - 각 PDU 세션 유형에 대해 동일한 resource requirement으로 UPF instance 유형이 생김

### C. The Problem of Scaling UPF Pods

  - UPF pod scaling 목적 : 시스템의 resource 소비를 줄이는 것
    - scaling 기능 : UE가 필요로 하는 PDU 세션 수에 따라 UPF pod 수를 변경하는 작업 ( 새 pod 시작 / 기존 pod 종료 등 )
      - pod 수가 너무 적다면, 새로운 PDU를 처리할 UPF가 없기 때문에 QoS 하락
      - pod 수가 너무 많다면, 예약된 resource가 많아 운영비용이 높아짐
    - QoS와 운영비용 간의 균형을 이루는 방법을 찾아야함
  - 예시
    - $d_{on}(t)$ : 시간 $t$에서 실행 중인 pod의 개수라고 가정하면 아래 식이 성립
      - $D_{min} \leq d_{on}(t) \leq D_{max}$
      - 시스템에서의 최대 세션 개수 : $L_{sess} \times D_{max}$
    - $l_{sess}(t)$ : 시간 $t$에서 실행 중인 PDU 세션의 개수라고 가정하면 아래 식이 성립
      - $0 \leq l_{sess}(t) \leq D_{max} \times L_{sess}$
    - $l_{free}(t)$ : 시간 $t$에서 추가적으로 사용 가능한 PDU 세션의 개수라고 가정하면 아래 식이 성립
      - $l_{free}(t) + l_{sess}(t) = D_{max} \times L_{sess}$
      - load balancer : 새 PDU 세션을 적절한 UPF pod에 할당함
        - PDU 세션은 추가 가능할 때, $l_{free}(t) > 0$ 인 경우에만 추가생성 가능
        - 만약 추가 용량이 없는 경우, $l_{free}(t) = 0$이라면 세션과 UE의 request를 차단
      - $p_b$ : request가 거절되는 rate를 기록

### D. K8S HPA

  - autoscaler : pod 확장 기능 담당
    - metric server : pod의 resource 사용량을 모니터링, autoscaling entity 제공
    - autoscaler : 필요한 pod replica 수를 계산, scaling action 결정할 수 있음
    - control interface : replica 수를 조정함
  - HPA : 쿠버네티스의 autoscaling 알고리즘
    - notation
      - $\bar{\rho}$ : 평균 CPU utilization
      - $d_{\text{desired}}(t)$ : 시간 $t$에서 필요한 pod 수
      - 2개의 config 가능한 parameter
        - $\rho_{target}$ : 목표 CPU utilzation
        - $v$ : tolerance ( 수용가능범위 )
    - 알고리즘 방정식
      - $d_{\text {desired}}(t)=\left \lceil{ d_{\text {on}}(t) \frac { \bar {\rho }(t)}{ \rho _{\text {target}}} }\right \rceil$
    - 이후 $d_{\text{desired}}(t) / d_{\text{on}}(t) \in [1-v, 1+v]$ 인지 검사
      - 만약 아니라면 scale issue 발생시켜 값을 조정하는 시도를 함
      - 검사 주기 : $\Delta T$ ( 설정에서 조정 가능 )

  

## Scaling UPF Pods with DRL

  - HPA를 적용하기 위해서는 2개의 파라미터 ( $\rho_{\text{target}}, v$ ) 의 적절한 값이 필요
    - 많은 시행착오를 통해 QoS를 유지하면서 pod를 최소로 하는 값을 찾을 것
  - DRL 적용 : 운영자의 도움 없이도 동적으로 pod 개수를 설정할 수 있도록 함
    - agent : 시스템 관찰, policy의 지속적 개선 -> 올바른 action 출력 결정

### A. Formulation of the Markov Decision Problem

  - 강화학습을 적용하기 전에 MDP 문제로 변환할 필요성이 있음

    - 각 파라미터를 정의해야함
      - 상태공간 $\mathcal{S}$, 동작공간 $\mathcal{A}$
      - 보상함수 $r$ : $\mathcal{S} \times \mathcal{A} \times \mathcal{S} \rightarrow \mathbb{R}$
      - 상태 전이 확률 $p$ : $\mathcal{S} \times \mathcal{A} \times \mathcal{S} \rightarrow [0, 1]$
        - 현재 상태에서 행동이 일어날 때, **시스템이 다음 상태로 들어갈 확률**
        - 이러한 전이는 실제 가치 보상 $r$를 초래함
      - discounting factor $\gamma$ : $\gamma \in [0, 1]$

  - 모델 기반 공식과 같이, $p$ 전이 함수의 specification을 피하기 위해 model-free RL 방식을 사용

  - MDP framework에서는 agent가 환경과 상호작용을 함

    - decision time $t$에서 상태 $s(t) \in \mathcal{S}$ 를 관찰
    - 정책 $\pi$ : $\mathcal{S} \rightarrow \mathcal{A}$를 따름 // 행동 $a(t) \in \mathcal{A}$ 수행
    - 결과로 reward $r(t)$를 받고, 다음 decision time에 다음 state를 관찰함

  - $s_{\text{i}}$는 optimal scaling 결정을 위해서는 **필요한 모든 정보를 포함해야함**

    - $s_{i} = \lbrace {d_{\text {on}}(T_{i}), d_{\text {boot}}(T_{i}), l_{\text {sess}}(T_{i}), l_{\text {free}}(T_{i}), \hat {\lambda }(T_{i})} \rbrace $
    - $\hat {\lambda }$ : 이전 결정에서부터의 측정 arrival rate

  - action은 3가지로 구성

    - 새로운 pod를 `start`
      - cluster에 용량이 있는 경우, $d_{\text{on}}(t) + d_{\text{boot}}(t) < D_{\text{max}}$인 경우에만 추가 pod 실행 가능
      - $d_{\text{boot}}$의 존재 이유 : 새로운 pod를 실행하면 필요한 container를 실행하는데 $t_{\text{pend}}$만큼의 시간이 필요하기 때문
    - 기존 pod를 `terminate`
      - $d_{\text{on}}(t) > D_{\text{min}}$ 인 경우에만 기존 pod 제거 가능
      - PDU 세션이 종료되기를 기다린 후 종료된다고 가정 -> 추가 PDU 세션 요청을 받지 않음

    - 아무런 행동을 취하지 않음 `no action`

  - 보상함수 
    $$
    r_{i} = \begin{cases}
    -\kappa \hat{p}_{b, i} & \text{if } \hat{p}_{b, i} > p_{b, th} \\
    -d_{\text{on}}(T_i) & \text{if } \hat{p}_{b, i} \leq p_{b, th}
    \end{cases}
    $$

    - $ \hat{p} _ {b, i} $ : 이전 결정 $ \[ T _ {i-1}, T ) $ 이후 측정된 차단율
    - $p_{b, th}$ : 초과하면 안되는 QoS level의 차단율 임계값
    - $\kappa$ : blocking 속도를 scaling하는 coefficient scalar 값
    - 목적 : 측정된 차단 속도가 임계값을 <u>초과하면 차단 속도를 최소화해야한다</u>
      - 임계값을 <u>초과하지 않는다면 pod 수를 최소화한다</u>

### B. Reinforcement Learning

  - 강화학습 : 환경과 상호작용하기 위해 agent를 적용하는 방법

    - 시스템 상태 관찰 / 후속 작업의 결과에 reward 적용
    - 적용하기 위해 state 정보가 필요 ( 활성 / 부팅중인 UPF pod의 수, 시스템 내의 PDU 세션 수, arrival rate의 근사값 등 )
      - 얻는 방법 : 5G core의 SMF & AMF 기능 모니터링 or SMF & AMF의 기능 자체

  - agent : 2개의 scaling action 사이에 관측치를 이용하여 policy 업데이트 / 개선

    - cluster를 운영하는 동안 online에서 학습 ( 사전교육도 가능 )

  - RL의 목적 : value function $V^{\pi}(s)$를 최대화하는 정책 $\pi$를 찾는 것

    - 즉, 상태 $s$에서 시작하는 long-term 예상 누적보상을 최대화하는 것이 목적 ( 시작 state에 의존하지 않음 )

    $$
    \begin{equation*} V^{\pi }(s) = \mathbb {E}_{\pi } \left [{ \sum _{k=0}^{\infty }{\gamma ^{k} r_{i+k} \bigg \vert s_{i} = s } }\right] \end{equation*}
    $$

  - Proximal Policy Optimization (PPO) 사용하여 풀어냄

    - PPO : actor-critic 알고리즘 -> 매개변수화된 policy $\pi(s, \theta)$ 를 actor로 사용하여 action 선택

      - 또, $\omega$ 벡터로 매개변수화된 $V(s, \omega)$로 함수값 근사시킴

        - Generalized Advantage Estimator (GAE) 이용하여 advantage $\hat{A}_{\text{GAE}}$를 계산하는 데 사용됨

      - $\theta$ : 매개 변수 vector

      - 크기 $N_{\text{batch}} + 1$의 벡터 $A_{\text{AGE}}$에서 a batch of advantages를 고려하면, 벡터의 j번째 요소는 아래와 같이 계산 가능
	  
        $$
        \begin{equation*} \hat {A}_{\text {GAE}, j} = \sum _{l=0}^{N_{\text {batch}-j-1}}\lambda _{\text {GAE}}^{l}\delta _{j+l} \quad j=0,1,\ldots, N_{\text {batch}}\end{equation*}
        $$

      - $A_{\text{AGE}}$ : 암시적으로 포함하는 가중치 hyperparameter $\gamma$를 포함

      - $\delta_{j}$ : batch $\delta$의 시간차 오차의 j번째 요소
	  
          $$
		  \begin{equation*} \delta_{j} = \text{TD}_{\text{target}, j} - V(s_{j}, \boldsymbol \omega) \end{equation*}
		  $$
		  
		  - j번째 시간차 목표 $\text{TD}_{\text{target}, j}$
          
			$$
            \text {TD}_{\text {target}, j} = r_{j} - \tilde {r} + V(s_{j}', \boldsymbol \omega)
		  $$
		  
		    - 이 식에서는 평균 보상 체계 $\tilde r$를 사용했음 ( soft update를 통해 추적하는 평균 reward )
			
              $$
      	      \tilde {r}\leftarrow (1-\alpha _{R})\tilde {r} + \alpha _{R}\sum _{j=0}^{ N_{\text {batch}}- 1} r_{j} / N_{\text {batch}}
			  $$
			
			  - $\alpha_{\text{R}}$ : update rate hyperparameter
    
- 벡터 값을 굵은 기호로 표시하고, 그 요소를 index를 부여하여 표기하였음
  
- advantage와 $\text{TD}_{\text{target}}$ 사용 목적 : policy를 평가하는 것
      
  - 알고리즘은 작동 중에 반복적으로 $\theta, \omega$를 업데이트 -> 정책 개선 시도
    
	  $$
      \begin{align*} \boldsymbol \omega\leftarrow&\boldsymbol \omega + \alpha _{\omega } \textbf {T}\textbf {D}_{\text {target}}\odot \nabla _{\omega }V(\mathbf {s}, \boldsymbol \omega) \\ \boldsymbol \theta\leftarrow&\boldsymbol \theta + \alpha _{\theta }\nabla _\theta \min \{\mathbf {r_{t}}(\boldsymbol \theta)\odot \hat {\textbf {A}}_{\text {GAE}}, \\&\text {clip}(\mathbf {r_{t}}(\boldsymbol \theta), 1-\varepsilon, 1+\varepsilon)\odot \hat {\textbf {A}}_{\text {GAE}}\} \\&+\, \xi H(\pi (\cdot |\mathbf {s}, \boldsymbol \theta)).\end{align*}
    $$
    
    - $\alpha_{\omega}, \alpha_{\theta}$ : gradient descent step에서의 learning rate
      - $\epsilon$ : PPO의 cliffing ratio
      - $r_t(\theta)$ : 확률 ratio

        $$
        r_{t}(\boldsymbol \theta)_{j} = \pi (a_{j} | s_{j}, \boldsymbol \theta) / \pi _{\text {old}, j}
        $$
			
          - $\pi(a_{j} \| s_{j}, \boldsymbol \theta), \pi _{\text {old}, j}$ : 상태 $s_j$에서의 행동 $a_j$의 확률
          - $\pi (a_{j} \| s_{j}, \boldsymbol \theta)$ : epoch 진행 중에 바뀔 수 있는 $\theta$에 의존함
          - $\pi _{\text {old}, j}$ : batch에 저장되어 agent에 의해 실행되었을 때 $a_j$의 확률을 나타냄
          - 즉 epoch 시작 전에는 둘의 값이 같지만, epoch를 진행하면서 전자의 값이 바뀌기 시작함
          - $\odot$ : 원소적 곱 기호 ( 아다마르 곱 )
  
- entropy 정규화 추가

  $$
  \begin{equation*} H(\pi (\cdot |\mathbf {s}, \boldsymbol \theta)) = -\sum _{a'\in \mathcal {A}}\pi (a' | \mathbf {s}, \boldsymbol \theta)\log {\pi (a' | \mathbf {s}, \boldsymbol \theta)}\end{equation*}
  $$
  
  - weight : $\xi$
  
- 알고리즘 1
  
  ```algorithm
  Algorithm 1
  Proximal Policy Optimization Update in the i-th Decision Epoch
  ```
  
  ```pseudocode
  procedure Store(si, ai, π(⋅|si, θ), ri, si+1)
      Append (si, ai, π(⋅|si, θ), ri, si+1) to batch.
  end procedure
  procedure Update()
      if size of the batch < Nbatch then return
      end if
      Get batch s, a, πold, r, s′ of size Nbatch .
      Update r~ using (8).
      for k epochs do
          Compute A^GAE and TDtarget using (5–7).
          Compute rt(θ) using (10).
          Update ω and θ using (9).
      end for
      Clear batch storage.
  end procedure
  ```
  
  - 절차가 2개 존재
    - Store : 궤적 샘플 $(s_i, a_i, π(\cdot \| s_i, θ), r_i, s_i+1)$ 을 일괄적으로 update하기 위해 일괄적으로 저장하는 것
      - $π(\cdot \| s_i, θ)$ : 상태 $s_i$에서 모든 action에 대한 확률 벡터
    - Update : policy를 개선하는 데 사용되는 method를 보임
      - 샘플이 $N_{\text{batch}}$에 도달한 경우에만 실행됨
      - 평균 reward $\tilde r$를 근사시킨 다음, gradient descent를 k번 수행
      - 각 단계마다 $\theta, \omega$를 업데이트하여 정책 $\pi$ 변경
  
- 알고리즘 2
  
  ```algorithm
  Algorithm 2
  RL Training Loop
  ```
  
  ```pseudocode
  Initialize system, and get initial state s0.
  Initialize learning parameters of Agent.
  i ← 0
  for Ntrain steps do
      Get action from agent: ai, ← Sample π(si, θ).
      Execute action ai to scale the cluster.
      Observe the new state si+1 and performance measures after ΔT time.
      Compute reward ri from the measurements using (3).
      Agent.Store(si, ai, π(⋅|si, θ), ri, si + 1)
      Agent.Update()
      i ← i + 1
  end for
  ```
  
  - RL agent를 훈련시키는 알고리즘임
    - 시스템이 아직 초기화되지 않았다면 초기화 진행
      - hyperparameter를 설정
      - $\theta, \omega$ 벡터를 랜덤한 값으로 설정
      - $\tilde r$를 0으로 설정
    - $N_{\text{train}}$ 단계를 실행
      - 각 단계마다 관찰한 상태 $s_i$를 기반으로 agent로부터 수신한 scaling action $a_i$를 실행
      - 새로운 상태를 관찰하고, store하고 update하여 agent의 정책을 개선시킴

### C. Neural Network Approximation

- 시스템 상태 $s_i$는 5-tuple임 -> 이건 5차원 상태 공간을 만들어냄
  - 상태값은 $D_{\text{max}}$에 의해 제한되고 도착 속도는 최대 도착속도 $\hat{\lambda}_{\text{max}}$에 의해 제한됨
  - 상태공간이 매우 커서 메모리에 들어가지 않음
  
- 50개의 hidden node로 이루어진 하나의 hidden layer를 가진 NN을 사용함
  - 목적 : 정책 $\pi$와 value 함수 $V$를 극대화
  - $\theta, \omega$가 신경망의 매개변수 집합이 됨 ( Xavier 초기화로 시작 )
    - update에서는 gradient descent 방식 사용
  - 구성
    - 5개의 input ( $d_{\text {on}}, d_{\text {boot}}, l_{\text {sess}}, l_{\text {free}}, \hat {\lambda }$ )
    - hidden layer
      - ReLU 함수 적용
      - 출력계층에 softmax 함수 적용
    - 가능한 행동 확률 출력 ( $\pi_{\text{NoOp}}, \pi_{\text{ScaleOut}}, \pi_{\text{ScaleIn}}$ )
  
- 수치 안정화를 위한 정규화 진행
  - $d_{\text{on}}, d_{\text{boot}}$를 $D_{\text{max}}$ 값으로 나눔
  - $l_{\text{sess}}, l_{\text{free}}$ 를 $L_{\text{sess}}$로 나눔
  - $\hat{\lambda}$ 의 경우 최댓값을 알지 못하는데, 정확한 숫자를 알 필요는 없고 **정규화된 값의 순서만 중요**했기 때문에, 500으로 나누는 것으로 가정했음
  
- DRL agent를 훈련하기 위한 hyperparameter 검색을 수행

  - 가장 민감한 hyperparameter : reward multiplier $\kappa$, entropy regularization factor $\xi$

  - 수행 방법 : gridsearch 방식

    - $\kappa \in \lbrace 3, 5, 10, 13, 15, 20 \rbrace, \xi \in \lbrace 0.01, 0.05 \rbrace $
    - 다른 hyperparameter들은 성능에 영향이 덜 미친다는 결과를 얻었음 -> 자주 사용되는 대중적인 값으로 다른 파라미터를 설정했음

  - 결과

    |                    parameter                     |        value         |
    | :----------------------------------------------: | :------------------: |
    |           neural network hidden layer            |          1           |
    |             hidden layer node count              |          50          |
    | learning rate $\alpha_{\theta}, \alpha_{\omega}$ |        0.0001        |
    |                    epochs $k$                    |          5           |
    |          batch size $N_{\text{batch}}$           |          32          |
    |   reward averaging factor $\alpha_{\text{R}}$    |         0.1          |
    |        PPO clipping parameter $\epsilon$         |         0.1          |
    |       GAE parameter $\lambda_{\text{GAE}}$       |         0.9          |
    |            reward multiplier $\kappa$            | 3, 5, 10, 13, 15, 20 |
    |       entropy regularization factor $\xi$        |      0.01, 0.05      |
    |              random initialization               |    Xavier uniform    |
    |               activation function                |         ReLU         |

    - $\xi = 0.01, \kappa = 13$으로 설정했음

- DRL 모델은 Pytorch 1.5.1 사용

- training은 NVIDIA GeForce RTX 2070 (8GB) GPU 사용

### D. DRL with Classification

- 특정 관측치에 대해 1과 같은 확률로 action을 강제하는 non-stochastic 정책에 RL을 사용할 수 있음

  - DQN : 그리디한 정책을 초래 -> 과도한 provisioning 야기 -> **문제점**

- PPO 방법 : 행동 공간이 주어진 상태에 대한 **확률 분포를 가짐** -> 확률적인 정책을 학습 / 발견함

  - DRL agent는 학습된 확률 분포를 기반으로 작업 수행
    - $d_{\text{on}}$이 낮고 $\hat{\lambda}$가 높다면 새로운 pod를 실행해야할 것
    - 반대의 경우라면 pod를 제거해야할 것
  - 확률론적 정책에 의해, 원하지 않는 행동을 <u>0이 아닌 확률로 수행할 가능성이 있음</u>
    - traffic이 많아지면 agent는 pod 수를 늘릴 것임 ( blocking rate를 줄이기 위해 )
    - blocking rate가 threshold 이하로 떨어질 때가지 pod를 계속해서 늘려야하지만, **때때로 pod를 종료하기도 함**
      - 원인 : 확률론적 정책 ( bad action의 확률을 0으로, good action의 확률을 1로 올리는 <u>결정론적 정책이 아니기 때문</u> )
      - entropy 정규화를 했기 때문에 불가능하다고 함 ( agent가 전체 상태 공간을 탐색할 수 있도록 보장하는데 필요한 조치였기 때문 )

- dataset의 outlier에 대한 주의가 필요함

  - ex. 도착률이 매우 낮고 pod 수가 매우 높은 곳

  - label이 올바르면 분기선에 영향을 주지 않지만, 잘못 표시된 경우 **결정 경계를 원하지 않은 방향으로 이동시킴**

    - outlier를 제거하여 dataset을 정리하는 <u>classifier가 필요함</u>
	
      $$
      \begin{equation*} | d_{\text {on}}/ D_{\text {max}}- \hat {\lambda }/ \hat {\lambda }_{\mathrm {max}}| > 0.4\end{equation*}
      $$

      - $\hat{\lambda}_{\text{max}}$ : 실험에서 측정된 도착률 값 중 최댓값
      - 이것을 적용하여 outlier를 최대한 제거 ( 대각선 strip에 대부분의 data가 뭉치게 됨 )

- DRL agent를 이용하여 state와 그에 대응하는 action에 대한 label 작업을 거침

  - 만들어진 $N_{\text{data}}$ 크기의 dataset을 이용하여 선형 SVM 분류기를 생성함
  - Linear SVM : 두 class 사이의 dataset에서 분리 평면을 찾는 기계학습 방법
    - 다만 이 상황에서는 3개의 action ( `start`, `terminate`, `no action` ) 이 있으므로, multiclass 분류기가 필요함
    - One-versus-Rest 전략 : 각 행동 유형에 대해 별도의 모델을 만들 수 있음
      - 현재 행동 유형에 속한다면 +1 / 아니라면 -1로 표시하는 방법
    - 이렇게 수정한 action label을 $\tilde a$라고 표기함

- 상태집합 $\mathcal{S}$ 를 feature set으로 사용하여, $s_{i} \in \mathcal{S}$인 경우에 분리 초평면 $f(s_{i}) = \mathbf w^T s_{i} + w_0 = 0$ 을 계산할 수 있음

  - $\mathbf w, w_0$ : SVM의 hyperparameter

  - 위 식은 우리에게 분류 규칙을 줄 수 있음
  
    $$
    \begin{equation*} \tilde {a}_{i} = {\mathrm {sign}}(\mathbf {w}^{T} s_{i} + w_{0})\end{equation*}
    $$

    - 초평면은 여러개 존재할 수 있음 -> 가장 큰 마진 $M$을 갖는 평면을 선택할 것

      - 마진 : 평면과 가장 가까운 data 사이의 거리

      - 2가지 경우가 가능함

        - data가 분리 가능한 경우 : +1과 -1로 표시된 동작을 분리할 수 있는 평면이 존재한다면

          - 최적화 문제를 아래와 같이 표현 가능
		  
            $$
            \begin{align*}&\min _{\mathbf {w}, w_{0}} \quad \lVert \mathbf {w} \rVert \\&{\mathrm {subject~ to}} \quad \tilde {a}_{i}(\mathbf {w}^{T} s_{i} + w_{0}) \geq 1, i = 1, 2, \ldots, N_{\mathrm {data}} \\\end{align*}
            $$

        - dataset에 중복이 포함되어있고 분리할 수 없는 경우 : 훈련 set에서 **가장 적은 양의 point가 잘못 분류되도록 하는 초평면을 찾아야함**

          - slack 변수 $\zeta_{i}$를 추가하여 최적화 문제 수정
		  
            $$
            \begin{align*}&\min _{\mathbf {w}, w_{0}} \quad \lVert \mathbf {w} \rVert \\&{\mathrm {subject ~to}} \quad \tilde {a}_{i}(\mathbf {w}^{T} s_{i} + w_{0}) \geq 1- \zeta _{i} \\&\hphantom {subject to \quad } \zeta _{i} \geq 0, \sum \zeta _{i} \leq C, i \!=\! 1, 2, \ldots, N_{\mathrm {data}} \\\end{align*}
            $$

          - $C$ : SVM의 hyperparameter

            - $C$가 작을수록 더 많은 point가 잘못 분류될 수 있음 -> margin이 높아짐

- 알고리즘 3 : SVM 모델을 훈련시키는 절차를 나타냄

  ```algorithm
  Algorithm 3
  Training an SVM Classifier
  ```

  ```pseudocode
  Input : Environment and DRL parameters (see Table 1 and 3).
  Output : SVM model (w, w0) and its accuracy
  
  Initialize Pod count to Dmin.
  Initialize θ and ω of the AGENT with random values.
  Lstates ← {∅}, Lacts ← {∅}
  // Training agent and collecting states.
  i ← 0
  for Ntrain steps do
      ai ← Get action from the agent in state si (based on the neural network).
      ri, si+1 ←  Execute action ai and get reward and the next state.
      Store history and Update agent using the Agent.Store and Agent.Update procedures in Algorithm 1.
      if i > Ntrain − Ndata then
          Append state to Lstates.
      end if
      i ← i + 1
  end for
  i ← 0
  for Ndata steps do
      Evaluate DRL agent on state Lstates[i] to get action.
      Append action to Lacts.
      i ← i + 1
  end for
  Remove outlier points according to (12).
  Separate lists into train and test sets: Lstates → Ltrainstates, Lteststates; Lacts → Ltrainacts, Ltestacts.
  w, w0 ← Train SVM using Ltrainstates as features and Ltrainacts as labels and run a grid search on hyperparameter C.
  Get accuracy of the model using Lteststates and Ltestacts.
  return w, w0, accuracy
  ```

  - input으로 시뮬레이션 환경과 DRL의 parameter / hyperparameter를 필요로 함
  - output으로 SVM 모델 매개변수인 $\mathbf w, w_0$을 반환하고, 모델 정확도 (SVM의 성능 측정값) 을 반환함
  - agent와 환경을 초기화한 후, 이 알고리즘은 $N_{\text{train}}$ 단계에서 agent 교육을 시작
    - loop는 알고리즘 2와 거의 동일함
    - 차이점 : 마지막 $N_{\text{data}}$ 단계에서 나중에 dataset에 대한 $\mathcal{L}_{\text{states}}$ 목록에 state를 저장
  - agent의 교육이 끝나면 policy는 $\mathcal{L}_{\text{states}}$의 state를 평가가는데 사용됨
    - 결과 action은 $\mathcal{L}_{\text{acts}}$에 저장됨
    - $\mathcal{L} _ \text{states}, \mathcal{L} _ \text{acts}$ 는 SVM을 훈련시키는데 사용되어짐 ( dataset으로 활용 )
      - outlier를 제거하여 정리
      - train / test set으로 분리
        - train set을 이용하여 SVM 모델 학습
        - test set을 이용하여 모델의 정확도와 파라미터를 얻음
  - 이 알고리즘을 여러번 돌려 grid search 방식을 통해 최적의 C값을 얻었음

- SVM classifier를 평가하기 위해 logistic regression을 사용하여 실험했음

- SVM과 logistic을 위해 scikit-learn 0.24.2 라이브러리를 사용

- notation of SVM

  - $N_{\text{data}}$ : SVM을 훈련시키는 dataset의 크기
  - $ \mathcal{L} ^ \text{train / test} _ \text{states / actions}$ : state / action의 train / test set을 포함하는 list
  - $\mathbf {w}, w_0$ : SVM 파라미터
  - $\zeta_i$ : SVM slack 변수
  - $C$ : SVM의 margin 파라미터

### E. System Modeling

- 환경 : multinode cloud 환경 시뮬레이터 + pytorch를 이용한 DRL agent 구현

  - 시뮬레이션 : 시간 t에서 도착속도 $\lambda(t)$를 갖는 푸아송 process로서 UE의 도착을 생성하는 절차가 있음
  - 도착하자마자 pod 사이에 capacity 여유가 있을 경우 PDU 세션 시작
    - 만약 아니라면 UE의 요청 차단
  - PDU 세션 / traffic을 처리하는 UPF는 랜덤선택
  - 세션의 길이는 랜덤, rate $\mu$로 exponential하게 분포한다고 가정

- 실제 상황에서 우리는 arrival rate를 미리 알 수 없음

  - DRL 실험 단계를 2가지로 나누었음 ( training 단계 / evaluation 단계 )
  - 각 상황에서 다른 값으로 사용 ( $\lambda_{\text{train}}, \lambda_{\text{eval}}$ )
  - 훈련 단계에서 DRL agent를 초기화하고 미리 정의된 도착률 함수로 훈련하는 <u>사전 훈련 단계</u>로 생각할 수 있음
  - 평가 단계에서는 사전에 교육한 agent를 새로운 traffic model이 있는 환경에 적용하는 것임
    - training은 필요하겠지만, cold start까지는 할 이유가 없음

- 각 시뮬레이션 단계에서 도착률을 아래와 같이 계산하고 DRL agent를 훈련시킴

  $$
  \begin{aligned}
  \lambda_{\text{train}}(t) = & \text{ } 250 + 250 \sin(\frac{\pi}{6}t) \\
  \lambda_{\text{eval}}(t) = & \text{ } 330.07620 + 171.10476 \sin( \frac{\pi}{12}t + 3.08) \\
  & + 100.19048 \sin( \frac{\pi}{6}t + 2.08) \\
  & + 31.77143 \sin( \frac{\pi}{4}t + 1.14)
  \end{aligned}
  $$

  - 이 때 peak traffic이 500 PDU requests/s 까지 올라갔음
  
- blocking threshold $p_{\text{b, th}} = 0.01$ 로 설정하고 다양한 $t_{\text{pend}}$ 값에 대해 DRL 알고리즘 시행

  - 총 8개의 시뮬레이션 실시하여 <u>평가 단계에서 $d_{\text{on}}, \hat{p}_{b}$ 평균을 구함</u>



## Numeric Results

### A. Scenarios

- 수치적 평가를 위한 가정

  - UPF instance는 Intel Xeon 6238R 28 core 2.2GHZ 프로세서와 4x64 GB RAM을 갖춘 physical server에서 실행됨
  - 각 UPF 세션은 video 스트리밍 data를 전달하는 것임
  - 각 서버의 8개 코어가 OS / container 관리 시스템에 할당됨
  - 각 UPF instance는 하나의 core와 2GB RAM을 차지함 / 최대 8개의 동시 video 스트림을 제공할 수 있음
  - booting 시간은 무시하지 않음. 각 pod에 대해 고정되고 동일한 값을 가짐 ( 0이 아님 )

- 파라미터 값은 아래 표에서 알 수 있음

  |                 parameter                 |    value     |
  | :---------------------------------------: | :----------: |
  | the max. number of sessions per Pod ($B$) |      8       |
  |           service rate ($\mu$)            |     1/s      |
  | minimum number of Pods ($D_{\text{min}}$) |      2       |
  | maximum number of Pods ($D_{\text{max}}$) |     100      |
  |  initialization time ($t_{\text{pend}}$)  | 0.25, 5, 10s |
  |    time between decisions ($\Delta T$)    |      1s      |
  |   blocking rate threshold ($p_{b, th}$)   |     0.01     |

- PPO 알고리즘과 비교하기 위해 DQN과의 비교를 진행 ( greedy한 결정론적 policy )

  - DQN이 최적의 정책을 학습하는 것이 어려움을 알 수 있었다고 함
  - 결과
    - PPO : 다양한 도착속도에 잘 적응하여 pod 수와 blocking rate 간의 균형을 잘 맞춤
      - 매 실행마다 좋은 policy를 찾을 수 있었다고 함
    - DQN : pod 수를 낮게 유지하고, pod를 과도하게 공급할 수 없음을 볼 수 있었음
      - traffic 변화에 적응할 수 있는 policy를 배우지 못하는 경향을 보였다고 함
  - DQN의 사용 배제 => PPO에 중점을 두고 진행

- PPO를 적용한 결과가 아래 표와 같음

  | $t_{\text{pend}}$ | $d_{\text{on}}$ (DRL) | $\hat{p}_b$ (DRL) | $d_{\text{on}}$ (HPA) | $\hat{p}_b$ (HPA) |
  | :---------------: | :-------------------: | :---------------: | :-------------------: | :---------------: |
  |       0.25s       |     46.598 (3.8%)     |     0.009338      |        48.446         |     0.006300      |
  |        5s         |     46.908 (3.2%)     |     0.009125      |        48.452         |     0.007300      |
  |        10s        |     47.129 (2.7%)     |     0.008800      |        48.456         |     0.007425      |

  - 결과값은 8번의 결과의 평균치임
  - %는 HPA와 비교하였을 때의 성능 증가량
  - **차단 속도 임계값 0.01 이하로 유지중인 것을 알 수 있음**
  - 낮은 $t_{\text{pend}}$ 값에서 DRL agent가 더 적은 수의 pod를 유지하는 것을 알 수 있음
    - pod 시작하는 시간이 오래 걸리면 agent가 idle pod를 많이 종료시킬 수가 없기 때문

### B. Improving Performance with Action Classification

- 주어진 상태에서, 낮은 확률로 DRL agent가 잘못된 결정을 내릴 확률이 있음
  - 차단 임계값보다 높은 blocking rate인 상황에서도 pod를 종료시키는 경우가 있음
  - 이 상황은 train 상황에서는 유용함 : agent가 state 공간을 탐색해야하기 때문
- **원치 않은 action을 최소화하는 방안을 탐구**하는 접근법 제안
  - state 샘플을 가져와서 action을 label로 사용하는 dataset 생성
  - linear kernel을 사용하는 SVM에 넣고 학습 ( 다양한 커널을 시도했으나 선형 커널이 가장 성능이 좋았다고 함 )
  - 데이터셋 크기 $N_{\text{data}}$ : 450000개 ( 80% train, 20% test )
  - hyperparameter $C \in \lbrace 0.1, 1, 10, 40, 100 \rbrace $
- SVM을 이용한 agent가 PPO보다 더 일관된 방식으로 동작함
  
  - 새로운 pod를 계속해서 시작시키지만 blocking 기준을 넘어가지는 않는다는 것을 알 수 있음
- SVM을 적용한 결과가 아래 표와 같음
  
    | $t_{\text{pend}}$ | $d_{\text{on}}$  | $\hat{p}_b$ |
  | :---------------: | :--------------: | :---------: |
  |       0.25s       | 46.442775 (4.1%) |  0.010400   |
  |        5s         | 48.425250 (0.1%) |  0.003250   |
  |        10s        | 47.959200 (1.0%) |  0.004013   |
  
  - blocking rate는 0.01 임계값 이하로 유지할 수 있음
      - 다만 <u>DRL agent보다 성능이 좋지 않음</u>
  - 비교군으로 logistic regression 모델을 사용함
      - 결과
          - 두개의 모델 모두 HPA보다는 성능이 좋았음
          - SVM이 logistic보다는 성능이 좋았음
          - SVM에 비해 높은 $t_{\text{pend}}$에서 $d_{\text{on}}$ 감소량이 더 낮아졌음



## Discussions and Conclusion

- 쿠버네티스 컨테이너 조정 환경에서 실행되는 5G core에서 UPF pod의 자동 scaling에 대한 조사를 진행
  - DQN을 적용하는 것은 그리디한 정책을 불러온다는 사실을 인지
  - 해결 방안으로 **PPO 방법을 기반으로 하는 DRL 방식**을 제안
  - DRL 방법이 HPA 알고리즘을 능가할 수 있음을 보임
- DRL agent는 확률론적 정책 특성에 의해 작은 확률로 **원하지 않은 action을 취할 수 있음**
  - 원인 : 훈련 중에 수집된 dataset의 특이점 ( outlier )
  - 특이점을 제거하여 dataset을 정리할 <u>분류기의 필요성</u>
  - **SVM classifier를 이용**하여 pod 수를 절약할 수 있음을 보임
- 이 제안이 쿠버네티스 내장 HPA 알고리즘을 능가함을 보임
  - DRL : 평균 pod의 개수를 2.7% ~ 3.8% 절약
  - SVM : 0.7% ~ 4.5% 절약
- classifier를 훈련시킨다 = traffic의 갑작스런 변화에 더 잘 반응하도록 **결정론적인 정책**을 만들 수 있다
  - 단점 : 오히려 traffic 변화가 느리다면 DRL보다 낮은 성능을 보임 ( 미미함 )
  - HPA보다는 거의 항상 더 좋은 성능을 보임
- 최대 단점 : SVM 훈련 방식을 <u>온라인 환경에서는 실행할 수 없다고 함</u>
  - SVM 적용 기준 : DRL 정책이 사용 가능한 dataset으로 **충분히 안정적일때** 사용 가능
  - 이외에는 PPO를 포함하는 DRL 방법을 제안