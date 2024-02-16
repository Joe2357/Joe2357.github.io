---
title: "AI-based Resource Allocation: Reinforcement Learning for Adaptive Auto-scaling in Serverless Environments"
author: Joe2357
categories: [1. iMES, Lab Seminar]
tags: [Lab Seminar]
math: true
---

> 2022 / 12 / 30 iMES 세미나



## Abstract

- Serverless 컴퓨팅 : 클라우드 컴퓨팅의 새로운 패러다임으로 부상
  - 인프라 관리의 필요성 제거
  - 대규모 / 저비용으로 user service 만족시킴
  - Cloud 공급자 : <u>변동하는 수요</u>를 충족시키기 위해 **유연한 resource 관리 필요**
    - 활성화 방법 : resource의 <u>자동 provisioning, deprovisioning</u>
- Workload 기반의 autoscaling : 상용 / 오픈소스 serverless 컴퓨팅 플랫폼 간의 공통적인 접근 방법
  - 사용된 알고리즘 : 들어오는 request의 수에 따라 instance를 scaling하는 방법
  - Knative : request 기반 policy로 제안된 serverless 프레임워크
    - 알고리즘 : instance마다 병렬로 처리할 수 있는 configured된 최대 request 수에 따라 resource를 확장 ( **동시성** )
    - 동시성은 serverless app의 성능에 큰 영향을 미칠 수 있음 // 가능한 최고의 service 품질을 제공하는 concurrency configuration을 찾는 것은 어려움
      - 요인 : 다양한 workload, 복잡한 인프라 특징 => **throughput, latency에 영향을 미침**
- 가상머신 provisioning autoscaling은 연구가 진행된게 있었지만, serverless 환경에서는 논의되지 않았다고 함
  - **제안** : serverless 프레임워크에서 request 기반 autoscaling에 RL 접근법의 <u>적용 가능성</u> 조사
  - 제한된 수의 iteration 내에서 workload 당 효과적인 scaling 방법을 학습함을 보이고, 성능을 향상시킴을 보였음



## Introduction

- 가상머신, container 기술의 발전 -> serverless 컴퓨팅 환경의 채택 증가
  - user에게 2가지의 이점 제공 가능
    - 진실되고 세밀한 pay-as-you-go 가격 모델을 사용하여, idle인 상태를 제외한 **실제로 사용한 resource와 container**에 대한 비용만 발생
    - user는 인프라 유지관리에 대해 overhead가 위임되지 않음 ( cloud 공급자가 부담 )
      - 유연한 on-the-fly scalability 포함 : 들어오는 load에 따라 resource를 자동으로 추가 / 제거
  - 공급자 : autoscaling 기술을 통해 resource 활용률 최적화, 관리에 필요한 노력 감소
- 구현 관점 : serverless offering 내에서도 scaling 방법은 각기 다름
  - Kubernetes Horizontal Pod Autoscaler (HPA) : instance당 CPU / 메모리 사용 threshold 등을 scaling
  - Workload 기반 scaling : 들어오는 traffic에 맞춰 resource를 추가 제공
    - AWS Lambda : limit에 도달할 때까지 들어오는 request가 올 때마다 instance를 init
    - **cold start** : 새로운 instance를 실행할 때마다 시간적인 lag이 생김
      - Knative : instance당 미리 지정된 수만큼 **request를 병렬로 처리** ( cold start 회피 목적 )
      - 동시성에 도달한 경우 : load를 처리하기 위해 추가 pod를 배치
- workload에 따른 동시성 수준이 **성능에 영향을 미칠 수 있다**고 함 ( 최대 수 초의 latency 차이 )
  - serverless 컴퓨팅 환경에서는 치명적인 문제로 이어질 수 있음
  - 해결법 : RL 기반의 모델 제안 -> 개별 workload에 대한 최적의 동시성을 dynamic하게 결정하기 위함
  - RL : agent가 환경과의 trial-and-error 상호작용을 통해 효과적인 의사결정 policy를 학습
    - 들어오는 workload에 대해 사전시직이 필요하지 않음 + runtime 중에 변경사항에 적응할 수 있음
  - 기존 VM 연구들에서는 진행된게 있지만, **serverless 환경에서는 연구가 진행되지 않았음** -> 아이디어의 핵심
    - 목표 : 최적화된 성능을 갖도록 동시성 수준을 결정하기 위한 RL 방법과 Q-Learning 적용 가능성을 평가
- cloud 기반 프레임워크 구현 ( 2가지 consecutive한 실험을 수행할 곳 )
  - 서로 다른 autoscaling 구성에서 서로 다른 workload profile의 성능 변화를 분석
  - 동시성 수준에 대한 throughput과 latency의 의존성을 입증, 적응적 scaling 설정을 통한 개선가능성 보임
  - => 지능형 RL 기반 logic으로 프레임워크 개선 -> workload에 대한 사전지식 없어도 제한된 시간 내에 scaling 정책을 학습할 수 있음



## Background

### A. Knative Serverless Platform

- 주문형 resource를 자동으로 scaling할 수 있는 기능을 포함하여 <u>serverless 응용 프로그램 배포 및 서비스 지원</u>하는 쿠버네티스 기반 미들웨어 구성 요소 세트 제공
- autoscaling 기능은 다른 serving component에 의해 구현됨
  - service revision이 0으로 scaling되는 경우 : 서비스 배포가 null operating pod의 복제본으로 축소
    - ingress gateway : 들어오는 request를 activator로 바로 전달
  - activator : 정보를 autoscaler에 보고
  - autoscaler : revision의 배포를 적절하게 확장할 수 있도록 지시
- revision의 user pod가 사용 가능할 때까지 request를 buffer에 저장함 -> **cold start 비용이 발생할 수 있음** ( 단점 )
  - 최소 하나의 replica가 활성 상태로 유지되고있다면, activator가 bypass되고 traffic이 user pod로 직접 흐를 수 있음
- request가 pod에 도달한 경우 : queue proxy container에 의해 채널링됨 -> user container에서 처리됨
  - queue proxy : 특정 수의 request만 동시에 user container에 입력되도록 함 ( 필요한 경우 request는 queue에 넣어둠 )
  - 병렬 처리된 request의 양은 **동시성 매개변수**에 의해 지정됨 ( 0 ~ 1000, 기본값 100 )
    - 지정된 숫자만큼의 request만 병렬처리를 허용함
    - 0으로 설정하는 경우 : scaling 없이 무제한 병렬처리
  - 각 queue proxy는 들어오는 load를 측정하여 별도 port에 보고
    - autoscaler : 원하는 동시성 수준을 유지하기 위해 새로운 pod를 추가하거나 제거하는 결정을 내림

### B. Q-Learning

- RL : agent가 자신의 환경과 상호작용, 각각의 행동에 대한 reward 형태로 피드백을 받도록 하는 "시행착오" 방법

  - 일반적으로 model free Q-Learning 방법을 사용

- 최적의 action-value function $Q^*$의 근사치인 $Q_ \theta (s,a)$를 단계적으로 학습

  - 상태 $s$로부터 시작, 행동 $a$를 취한 다음, 영원히 <u>최적의 policy에 따라 행동하는 경우</u>에서의 reward

  - 각 iteration마다 실제 reward를 관찰하며, step $t$마다 $Q$함수를 최적화함
    $$
    \begin{equation*}
    Q(s_t, a_t) \leftarrow (1 - \alpha) Q(s_t, a_t) + \alpha \left[ r_t + \gamma \max \limits _a Q(s_{t+1}, a) \right]
  \end{equation*}
    $$
    
    - $\alpha$ : learning rate // 새로 관찰된 정보가 오래된 정보를 어느정도 재정의하는가?
    - $\gamma$ : discount factor // 현재와 미래 reward 간의 균형을 맞추는 요소

- RL : 시행착오를 겪는 방법

  - 훈련 중, agent는 2가지 방법 중 선택을 해야함

  - $\epsilon$-greedy 전략 : $\epsilon$ 확률만큼 exploration을 수행 ( 점차 감소하는 확률 )

    - exploration : 새로운 행동을 탐험 ( $\epsilon$ 확률 )

    - exploitation : 현재 상황에서의 최선의 행동을 취함 ( $1- \epsilon$ 확률 )
      $$
      \begin{equation*}{a^{\ast}}(s) = \arg \mathop {\max }\limits_a {Q^{\ast}}(s,a)\end{equation*}
      $$

  - 기본적으로 Q-value는 각 state-action마다 Q-table에 기록됨



## Approach

- 서로 다른 concurrency 구성을 조사하기 위해 RL 기반 flexible 쿠버네티스 기반 프레임워크 설계
  - cloud 구조, 활용한 workload의 사양, 표준 프로세스 flow를 포함한 실험 설정 제시
- 2개의 실험의 foundation을 제공
  - 첫 번째 실험 : concurrency 변화의 영향을 평가
  - 두 번째 실험 : RL 기반 autoscaling을 평가

### A. Cloud Architecture

- 2개의 별도 쿠버네티스 cluster를 설정
  - service 클러스터 : 실험에 사용된 sample service 포함
    - 9개의 노드, 16 vCPU, 64GB 메모리
  - client 클러스터 : service 클러스터로 요청 전송하여 load를 생성
    - 1개의 노드, 16 vCPU, 64GB 메모리
- agent : 수집된 metric 기반으로 sample service의 configuration 업데이트를 포함하여 <u>두 클러스터의 활동 관리</u>
  - IKS user 역할을 맡아 실험의 프로세스 flow를 조정
- Knative 리소스는 service 클러스터에 설치됨
  - 배포된 sample service의 상태를 제어, 추가적인 pod의 on-demand autoscaling 지원
  - RL의 시행착오 방법을 사용하여, 각 iteration에서 service의 concurrency 구성을 업데이트, 매번 새로운 revision을 만듬
  - incoming request는 기본적으로 최신 revision으로 routing됨
- autoscaling 기능을 테스트하기 위해 각 iteration에서 scale-to-zero 기능을 활성화
  - 성능 문제를 bypass하고 autoscaling 기능에만 집중하기 위해 load balancing을 처리하는 ingress gateway의 replica 수를 추가하였음

### B. Workload

- serverless 환경 : 다양한 resource requirement를 수반하는 다양한 app에서 사용됨
  - Ex. 비디오 처리, 병렬화된 분석 작업 load 등 -> 상당한 memory + computing resource 요구
  - Ex. chained API, 챗봇 등 -> computing 집약도는 낮음, 대신 실행 / 반응시간 요구사항이 길 수 있음
- workload의 concurrency 영향을 조사하기 위해, serverless app을 시뮬레이션하는 workload profile 생성
  - Knative의 Autoscale-go application을 사용
    - workload 특성의 incremental variation을 테스트하는 request와 다른 parameter를 전달 -> **다양한 CPU / 메모리 집약적 workload를 emulate** 가능
  - 3가지 application parameter
    - **bloat** : 할당될 MB 수 ( 메모리 )
    - **prime** : memory / computing intensive한 load를 생성하기 위해 <u>주어진 숫자까지의 소수를 탐색</u>하는 데 사용 ( 소수 계산 )
    - **sleep** : request를 pause하여 app에서 특정 latency가 있는 경우처럼 행동 ( miliseconds )

### C. Process Flow

- 각 iteration : agent는 service 클러스터에 concurrency update 전송, 그에 따라 **각 concurrency limit이 있는 new revision을 생성**
  - service update가 완료된 이후 : agent는 start signal을 client cluster로 전송
  - client cluster : service cluster에 대해 병렬 request 시작 ( Vegeta 사용 : HTTP request를 일정 속도로 전송할 수 있음 )
    - scaling에 대한 충분한 demand와 추가 instance를 제공하기 위한 시간을 보장하기 위해, <u>30초 동안 500개의 request를 동시 전송</u>
  - 마지막 responce를 수신한 이후 : Vegeta는 request의 latency 분포, 평균 throughput, 성공 ratio 등의 **test 결과를 보고서 출력** -> <u>성능 측정값이 agent에 의해 저장됨</u>
  - agent : 클러스터 내의 prometheus 기반 HTTP API를 통해 expose된 Knative 모니터링 구성요소로부터 metric을 크롤링
    - cluster, node, pod, container 수준의 resource 사용에 대한 추가정보를 얻을 수 있음
    - 위 data를 이용하여 concurrency update가 다음 iteration으로 proceed되도록 선택



## Baseline Experiment

- 다양한 concurrency limit의 영향을 결정하기 위한 실험
- 상대적인 성능에 대한 다양한 workload를 비교하는 과정

### A. Design

- application parameter인 bloat, prime, sleep을 사용하여 <u>다양한 workload 특성을 시뮬레이션</u>
  - parameter가 없는 무작동 workload부터 시작하여, **점진적으로 메모리 할당 / CPU load를 증가**시켜가며 실험
  - 메모리 할당 parameter (bloat) 의 step size : serverless 플랫폼의 표준 price 모델의 memory bukkit을 일반적으로 사용
  - prime / sleep : computing 집약적이고 latency가 긴 request을 시뮬레이션하기 위해 다양한 parameter 값을 선택
- 각 profile마다 process flow에 따라 **다양한 concurrency level에 대한 성능 테스트** 진행
  - concurrency limit : $0$ ~ $1000$ 사이의 값임 ( 기본값 : $100$ )
  - 실험을 계산적으로 실현가능하도록 유지하기 위해, <u>concurrency limit을 10 ~ 310까지</u> 20단계로 진행
  - 주요 성능 측정 기준 : latency, throughput
    - 평균 throughput : 초당 request 수 ( Requests Per Second, RPS )
    - 평균 latency : request responce를 반환하는데 걸린 평균 시간(초)
    - tail latency : 95% 백분위수 latency를 추가 metric으로 포함
  - 각 test는 10회 반복 : 다음 limit으로 update되기 전에 **outlier, 다른 변동으로부터 결과를 유지하기 위함**

### B. Results

- 실험 결과를 3부분으로 구성
  - 서로 다른 concurrency 구성에서 개별 workload profile의 behavior 조사
  - target variable의 throughput, latency의 관계에 초점
  - container 및 pod 수준의 resource utilization에 대한 추가 metric 분석
- 가능한 사용 사례를 시뮬레이션하기 위해 <u>3가지 parameter의 서로 다른 combination에 대한 실험</u> 진행
  - 최적의 시험 결과로 이어지는 concurrency limit을 가진 outcome의 개요를 정리 ( table로 )
    - 표의 왼쪽 부분 : 각 workload의 구성
    - 표의 오른쪽 부분 : 최고의 성능을 낼 때의 concurrency limit ( 최소 동시성 10인 것이 가장 좋은 성능을 내는 일반적인 구성 )
      - KPA의 목표 concurrency 기본 설정값 = $100$, **KPA는 최적이 아니라고 이야기할 수 있을 것**
    - memory만 사용하는 workload : pod instance당 <u>병렬 request 개수가 적을수록</u> 성능 향상 ( 2, 9, 16, 17 )
    - CPU 사용량이 추가적으로 낮은 workload (낮은 prime 값) : 유사한 결과로 관찰됨 ( 3, 10 )
    - request가 특정 시간동안 일시중지되는 workload : <u>높은 concurrency를 선택</u>해야 throughput이 높음, latency 낮아짐 ( 7, 8, 14 )
- workload에 따라 최적인 구성과 차선의 concurrency 사이의 distance가 작을 수 있음
  - Ex. test #7
    - 개별 측정 지점이 변동하더라도 평균 value에서 명확한 추세 확인이 가능
    - concurrency limit을 70으로 올리면 처리량이 크게 증가할 수 있음
      - 이 때 평균 latency가 가장 낮아짐
      - concurrency limit을 50으로 조정할 때와 latency는 80ms밖에 차이나지 않음
    - 사실, 결과를 따지기 위해서는 740ms 정도 차이나는 95% tail latency의 distance를 보는 것이 더 중요
      - concurrency limit을 10으로 하면 95% latency가 3초에 달함
      - **설정이 다른 경우 발생하는 성능 변화**를 더욱 강조할 수 있음
  - concurrency limit이 210을 넘는 경우 <u>전체 성능이 저하</u>되는 경향이 있음
    - 원인 : 많은 request의 높은 동시처리 -> 단일 request에 대해서는 제한된 양의 resource를 사용하게 되기 때문
  - memory 활용도가 증가하는 경우 ( bloat 증가 ) : 더 낮은 concurrency limit에서 성능이 떨어지는 경향이 있음
  - 성공률이 크게 하락하는 경우도 있음
    - Ex. test #16 : 170 이상의 concurrency에서 request의 10% 이상이 성공하지 못함
    - Ex. test #17 : 90 이상의 concurrency에서 request의 10% 이상이 성공하지 못함
- target metric에 맞춰 **concurrency 제한을 적절한 설정으로 조정하면** <u>throughput과 latency를 크게 향상할 수 있음</u>
  - throughtput과 latency는 서로 음의 상관관계를 가짐 -> **metric을 throughput 1종류로 줄일 수 있음**
    - 일반적으로, throughput이 향상되면 latency가 줄어듦
    - 결론 : concurrency를 조정할 때 <u>서로 다른 target metric 간에 균형을 맞출 이유가 없음</u> ( 하나의 metric으로 축소 가능 )



## Reinforcement Learning Experiment

- 2번째 실험 개시 : runtime 동안 concurrency limit을 조정하여 **효과적인 scaling 정책을 학습**하는 것이 목적
  - model-free RL algorithm Q-Learning의 적용 가능성 평가

### A. Design

- process flow는 기존과 동일, agent에 의해 더 정교한 logic으로 확장

  - agent : concurrency를 점진적으로 증가시키지 않고 시스템 환경 지식을 이용하여 <u>서로 다른 concurrency update를 테스트 + reward score를 받아가며 평가</u>

- 각 iteration에서 환경 : 최적의 의사결정을 위한 모든 관련정보를 포함하는 현재 state에 의해 정의됨

  - 성능에 영향을 주는 요인이 많이 때문에, **Q-Learning이 추적할 수도 없고 실현 가능하지도 않음**

    - 숨겨진 cluster 활동, 네트워크 활용량 등

  - => 상태 공간 $\mathcal{S}$를 3가지 주요 기능으로 분해함
    $$
    s_i = (\text{conc}_i, \text{cpu}_i, \text{mem}_i)
    $$

    - $\text{conc}_i$ : concurrency limit

    - $\text{cpu}_i$ : user container당 평균 CPU 사용률

    - $\text{mem}_i$ : user container당 평균 메모리 사용률

    - CPU와 메모리 활용률은 continuous한 값임 -> 이산화 과정을 거침

      - agent : 3가지 action 중 하나를 선택 ( concurrency limit을 감소 / 유지 / 증가 )
        $$
        \mathcal{A} = \lbrace -20, 0, 20 \rbrace
        $$

  - agent : 각 iteration 이후 즉각적으로 reward를 받음

    - reward : 성능 측정과 특정 Service Level Agreement (SLA) 사이의 distance나 ratio에 기초함
    - 사전정보가 없어서, 인공 기준값 $\text{ref_value}$을 **현재까지의 최상의 값**으로 정의
    - throughput이 높으면 latency 또한 좋아지기 때문에, <u>1개의 목표만으로 집중할 수 있음</u> ( $\text{thrghpt}$ )

  - reward 계산
    $$
    \begin{equation*}{r_i} = \begin{cases} 
    \frac{\text{thrghpt}_i}{\text{ref_value}} & { \text{if }\text{thrghpt}_i \leq \text{req_value} \cdot 0.95 \\ \text{or }\text{thrghpt}_i \geq \text{req_value} \cdot 1.05} \\
    1 & \text{else}
    \end{cases}\end{equation*}
    $$

    - $\alpha$ : learning rate ( 0.5 )
    - $\gamma$ : discount factor ( 0.9 )
    - exploration을 장려하기 위해 iter #50에서 $\epsilon = 1$로 지정, 감소 계수는 $0.995$, 최솟값은 $0.1$

  - agent가 획득한 지식은 Q-table에 저장

- 모델이 높은 throughput 구성으로 concurrency를 효과적으로 학습할 수 있는가?

  - Ex. test #7 : concurrency 70에서 높은 성능을 보임
  - Ex. test #10 : concurrency 10에서 높은 성능을 보임

### B. Results

- TODO

