---
title: "Machine Learning-Based Scaling Management for Kubernetes Edge Clusters"
author: Joe2357
categories: [1. iMES, Lab Seminar]
tags: [Lab Seminar]
math: true
---

> 2024 / 6 / 19 iMES 세미나

## Abstract

- 쿠버네티스 : cloud 배포 app을 위한 container orchestrator
  - 계속 변화하는 processing 요구를 충족시키기 위해 app provider에 대한 autoscaling 제공
    - autoscaling : parameter set으로 커스터마이징 가능
    - 관리 parameter는 정적, but 들어오는 웹 request는 **동적**
    - scaling decision이 proactive (사전 예방적) 이 아닌  reactive (반응적) 함
- 클라우드 기반 app의 관리를 보다 쉽고 효과적으로 할 수 있도록 목표 설정
  - **제안** : autoscaling 결정을 incoming request의 실제 variability을 처리하기에 적합한 <u>쿠버네티스 scaling 엔진</u>
    - 다양한 기계학습 예측 방법들이 실제 request dynamic에 가장 적합한 방법으로 lead를 제공하기 위해 <u>short-term evaluation loop</u>를 통해 경쟁
    - cloud-tenant app 제공업체 : 'resource over-provisioning'과 'SLA 위반'의 tradeoff에 대해 sweet spot을 쉽게 설정할 수 있도록 compact한 parameter 도입
- 현재 쿠버네티스 동작에 대한 분석 모델링 및 평가를 통해 확장 솔루션에 대한 motivation 부여
  - 다중예측 scaling 엔진과 관리 파라미터 : 시뮬레이션과 수집된 웹 trace에 대한 측정을 통해 평가됨
    - 서비스의 demand에 provisioning된 resource를 맞추는 **향상된 품질**을 보임
  - 단지 몇가지 다른 경쟁 예측 방법으로, 쿠버네티스에서 구현된 autoscaling 엔진은 default baseline에 비해 더 많은 provisiong된 resource로 인해 더 적은 lost request를 달성함을 보임



## Introduction

- 웹서비스 & app : 일반적으로 cloud에 배치
  - 가상 리소스 : app 진입에 부딪히는 client 요청의 시간 변화량과 같은 수요에 맞게 지속적으로 적응함
  - web app : microservice 구조를 따름 // monolithic 소프트웨어가 더 작고 독립적으로 관리되는 component로 분해됨 (container로 알려진)
  
- container에서 동적으로 리소스를 할당하고 필요에 따라 리소스 확장하는 것은 어려운 작업이 될 수 있음
  - scaling 로직 : 다양한 서비스 관리 목표에 의해 구동될 수 있음
    - 주어진 서비스 품질 목표를 유지하면서 자원 사용을 최소화
    - provisioning된 자원에 대해 지불된 cost에 관계 없이 SLA 위반을 최소화
  - 의사결정 : 관리되는 app의 고유한 scaling 동작에 의해 더욱 악화됨
    - 주어진 SLA 조건을 존중하여 <u>제공될 request의 양을 provisioning에 필요한 양의 resource로 변환하는 기능</u>
  
- 쿠버네티스 : HPA, VPA라는 auto scaling 기능을 제공
  - VPA은 수년간 베타버전으로 사용 // **단점** : pod를 껏다 켜야지만 scaling을 할 수 있다
    - 껏다 켜지 않아도 scaling할 수 있는 방법이 이론적으로는 있지만, 쿠버네티스 core에 적용되지는 않음
    - VPA는 활발히 개발중이므로 미래에 기능이 추가될 수도 있음
  - pod : 소수의 단단히 결합된 container를 포함하는 **쿠버네티스의 가장 작은 배포장치**
    - user-define 또는 외부 metric도 지원함 // 기본적으로 HPA는 CPU 사용 metric을 고려하여 pod를 autoscaling하기 위해 **threshold 기반 decision**을 자동으로 수행
      - quasi-instance 정보를 사용하고 엄격한 규칙을 지시하지만, 대부분의 app에서 HPA 제공이 허용됨
    - <u>들어오는 demand가 많고 SLA 제약조건이 엄격하며, cloud resource 예산이 부족한 경우</u> custom metric을 활용하고 자체 autoscaling logic을 구현해야함
      - edge cloud : 제한된 인프라 용량 & 상대적으로 적은 수의 client => request가 안정적인 수요로 다중화되지 않지만, 일반적인 edge app들은 엄격한 SLA를 약속함 (low latency)
  
- 해결해야할 문제
  - scaling 결정은 "현재 scaling 기간의 관찰 결과에 따라 내려짐", 그러나 **더 큰 이력을 살펴본다면** 다음에 무엇이 나올지 더 좋은 idea를 얻을 수 있을 것
  - HPA parameter는 한번만 설정됨 -> 다양한 request 도착 패턴이 잘 알려져있지만 **scaling behavior는 demand의 현재 변동성에 적응되지 않음**
  - 기본 HPA를 적용할 때도 **설정해야할 parameter가 많음** -> scaling 행동을 정의하는 것이 번거로움
  
- 제안된 해결 방안

  - **적응형 자동 확장 엔진 설계** : 제한된 정보나 확장 로직의 경직성에 구애받지 않고 실제 수요에 맞춰 확장 결정을 조정하는 Kubernetes 자동 확장 엔진을 설계
  - **단순 관리 파라미터 도입** : 자원 과잉 공급과 부족 공급 간의 균형을 설정하기 위해 cloud tenant, 즉 애플리케이션 제공자가 설정할 수 있는 단순 관리 파라미터인 <u>excess</u> (초과)를 제안

- Contribution

  1. **HPA 평가** : 무손실 MMPP/M/c 큐잉 모델을 사용하여 HPA를 분석적으로 평가하고, 이산 시간 확장 방법을 시뮬레이션하여 excess 파라미터가 HPA의 작동을 어떻게 유도하는지 시뮬레이션을 통해 보여줌
  2. **ML 기반 확장 방법 제안** : 일일 및 주간 프로파일과 시간대별 요청 강도의 변동성과 같은 현상을 활용하기 위해 전력망부터 클라우드 서비스까지 다양한 시스템에서 시간 시계열 분석 및 확장 의사 결정에 대한 방대한 연구에서 영감을 받아 머신 러닝(ML) 기반의 확장 방법을 제안
     - 역사적 정보를 고려하여 요청 도착 동역학을 높은 정확도로 예측할 수 있으므로, 예측 기반 접근 방식을 사용한 <u>선제적 확장 정책</u>을 제안
  3. **예측 방법의 다양성 적용 및 평가** : 시간 시계열의 특성에 따라 ML 기반 예측 방법의 성능이 다르다는 연구를 바탕으로, 자가 회귀 (Auto Regressive, AR), 비지도 학습 (HTM) 및 지도 학습 (LSTM) 딥러닝, 강화 학습 (RL) 등 네 가지 방법을 적용하여 경쟁시킴
     - Kubernetes 커스텀 메트릭으로 이들의 예측을 구현하고, HPA+라 부르는 단기 백테스팅 플러그인을 통해 기본 HPA와 동적으로 전환
     - 대학 캠퍼스에서 수집한 네트워크 트레이스를 통해 에지 시나리오를 모방한 Kubernetes 웹 서버 배포에 대한 ML 기반 예측의 품질과 HPA+의 확장 성능을 평가

- Find Point

  - **excess 파라미터** : 유연한 자원 overbooking과 SLA 위반 간의 균형을 명확하게 정립
  - **HPA+ 성능** : 비슷한 자원 제공량으로 요청 손실을 줄임

  - 이러한 예측 scaling 엔진은 edge cloud 환경에서 애플리케이션 관리에 유용
    - 이 경우 들어오는 수요가 급격하고 SLA 제약이 엄격하며 사용 가능한 클라우드 자원이 제한됨
    - traffic dynamics에 따라 확장하는 엔진이 이러한 기대를 충족시키고 기본 HPA보다 더 견고한 애플리케이션과 더 나은 자원 사용을 제공할 수 있음을 측정 및 시뮬레이션을 통해 확인
    - 협업 다중 사용자 AR(증강 현실) 게임과 같이 **초고신뢰 저지연** edge cloud backend에 엄격한 제약이 있는 웹 애플리케이션에 이러한 edge 배포가 필요할 수 있음



## Related Work on Auto-Scaling

- elasticity (탄력성) & dynamism (역동성) : cloud computing의 핵심 개념
  - 적절한 애플리케이션 scaling을 제공하는 것은 cloud 제공자의 가장 중요한 feature 중 하나!

- 참고 문헌 [[8]](https://link.springer.com/content/pdf/10.1007/s10723-014-9314-7.pdf), [[9]](https://ieeexplore.ieee.org/document/7937885)는 데이터 센터에서 적용되는 scaling 솔루션에 대한 포괄적인 survey를 제공
  - auto scaler들을 기반 이론 모델에 따라 분류
    - **Threshold-based** // 단점 : simplification (단순화) 가 scaling 성능의 제한요소가 된다고 생각
    - **Queuing model-based** // 단점 : exponential inter-arrival 시간에 대한 Markovian 가정이 제한요소라고 생각
      - 해결법 : HPA의 분석적 근사치에서 MMPP/M/c 모델을 사용 // Markov-modulated Poisson process (MMPP) 사용
    - **Control theory-based** : app 확장에서 높은 효율성을 보임
      - app의 요구에 맞게 다른 scaling method들과 결합되어짐
      - 장점 : controller가 <u>몇 초 단위로 동작할 수 있다</u> -> scaling 알고리즘이 "리소스 사용 변화에 빠르게 대응할 수 있다!"
    - **Reinforcement Learning-based**
    - **Performance prediction-based**
    - **Demand forecast-based**

- 많은 기존 접근 방식이 ML 기술을 사용하여 애플리케이션의 자원 요구 사항을 모델링
  - 과도한 프로비저닝과 SLA 위반을 방지하기 위해, 성능 모델을 사용하고 수요 예측 기반 방법의 아이디어를 따름
  - 이전 연구에서는 들어오는 트래픽을 예측하고 애플리케이션 자원 프로파일을 기반으로 확장하여 SLA 위반을 피하는 솔루션을 제시
- 이 연구에서는 들어오는 요청률에 대한 Markov 가정을 포함한 HPA 모델 연구를 추가하여 작업을 확장
  - 이를 위해 기본 HPA 작동을 가정하여 Pod 사용량을 계산할 수 있는 MMPP/M/c 대기열 모델을 제시
  - 모델은 광범위한 시뮬레이션을 통해 검증
- 이전에 사용된 실제 trace 외에도, 실제 trace에 맞춘 MMPP 모델을 사용하여 새로운 합성 클라우드 애플리케이션 요청 트레이스를 생성
  - MMPP 트레이스 모델 : 임의의 트래픽 양을 생성하고, MMPP/M/c 무손실 모델과 HPA의 이산 시간 손실 모델을 비교 분석할 수 있게 함
  - 생성된 synthetic trace는 예측 방법(AR, HTM, LSTM)이 예측 불가능한 트래픽 입력 상황에서 어떻게 동작하는지 시뮬레이션하는 데도 사용
- 이전 논문에서 제시한 예측 방법 외에도 강화 학습 기반 방법을 시뮬레이션 환경과 개념 증명 프로토타입에 적용하고 통합
  - 위 시스템은 scaling apparatus를 제어하기 위해 가장 성능이 좋은 방법을 선택하려고 노력
- scaling의 엔진들은 정량적 비교가 어려움 (**표준 test framework가 없기 때문**)
  - 대부분의 이전 연구들에는 검증을 위해 synthetic하거나 outdated된 traffic을 사용 -> dynamic하게 변화하는 app을 사용하는 <u>edge cloud 환경에 맞지 않음</u>
    - **Proactive Operation** : 사용자 수요가 동적으로 변하는 환경에서는 사전 대응 방식이 매우 중요
      - 이러한 방법은 SLA 위반과 애플리케이션 과부하를 피할 가능성이 더 높음
    - **Resource Level and Support** : 여러 방법이 사전 대응 방식으로 작동하지만, 확장이 VM 수준에서 이루어지는지, 컨테이너 수준에서 이루어지는지, 분산 애플리케이션의 확장을 지원하는지 여부에서 차이가 있음



## Cloud Application Resource Profiles

- 클라우드 자동 스케일링 문제를 세 단계로 나눔
  - 클라이언트 요청의 시간 시계열 기록과 적용 가능한 여러 ML 기반 예측 방법을 고려하여, 이러한 방법을 구현하고 최근 측정값에 대한 정확도를 평가하여 <u>가장 신뢰할 수 있는 예측을 항상 선택</u>하는 **Kubernetes 플러그인을 설계**
  - 선택된 예측 방법이 제공하는 잠재적인, 그러나 완전히 확실하지 않은 요청 비율을 가정하고, <u>excess 매개변수를 적용하여 SLA 위반을 최소화</u>하려는 애플리케이션 제공자의 의도에 맞추어 **애플리케이션 규모를 조정**
  - 임의의 request rates를 에뮬레이션하여 **특정 애플리케이션의 요구되는 규모를 실험적으로 결정**
    - 마지막 단계는 실제 배포 전에 수행됨
- 리소스 profiling : app의 연산 자원 requirements를 측정하는 것 // 자원 최적화를 위한 조사 주제
  - 예시 : 집중된 애플리케이션들은 모바일 장치에서 실행되거나 클라우드에서 실행될 수 있으며, 입력 데이터의 크기와 컴파일 기한을 기준으로 모바일 애플리케이션 프로파일을 작성
  - 본 연구 목표 : 주어진 애플리케이션 요청률을 처리할 때 애플리케이션의 자원 요구 사항을 가능한 한 정확하게 모델링하는 것
    - 이를 위해, Kubernetes 시스템에서 가장 작은 실행 및 확장 단위인 Pod에 투영된 애플리케이션 프로파일을 **Definition 1**과 같이 정의

##### Definition 1 : Application Profile

- 애플리케이션 프로파일 **AP** : $\mathbb{N}^+ \rightarrow \mathbb{R}^+$
  - 활성 Pod 수에 대한 Pod당 서비스 속도를 할당하는 함수

- Kubernetes에서 호스팅되는 애플리케이션의 프로파일을 결정하기 위해 측정 방법을 설계 (그림 1)
  - Kubernetes 클러스터에서 Kubernetes Service 객체의 표준 load balancer 기능을 ingress controller로 대체
  - Fortio 벤치마크 도구를 작업 노드에 배포했고, 테스트하려는 애플리케이션을 다른 작업 노드에 배포
  - 측정 동안 초당 request 수를 증가시키면서 벤치마크 테스트를 실행
    - 벤치마크는 하나의 Pod로 시작, 각 테스트가 완료되면 Pod 수를 증가시키고 다시 측정을 시작
    - 각 테스트는 1분 동안 지속, 각 테스트 후 Fortio에서 손실된 request의 비율을 수집하고 Kubernetes 모니터링 도구인 Prometheus에서 평균 CPU 사용량을 수집

- 실행 중인 Pod 수에 따른 각 애플리케이션의 초당 쿼리 수(QPS)를 보임 (그림 2)
  - 3개의 애플리케이션에 대한 결과 제시, 실험적 측정 결과를 사용하여 각 애플리케이션의 리소스 프로파일을 근사할 수 있음
    - 상단 다이어그램에서는 Node.js 애플리케이션과 Nginx 웹 서버의 실험 결과를 볼 수 있음
    - 하단 그림에서는 이미지 분류 애플리케이션의 프로파일이 표시
  - 예시 : 제공하는 Node.js 웹 애플리케이션의 측정 결과 (static HTML 파일 제공)
    - 실험 결과는 $125x + 209$의 선형 함수로 근사할 수 있으며, 여기서 $x$는 실행 중인 Pod 수
    - 이 함수의 $R^2$ 값을 계산한 결과 $0.997$
      - $R^2$ : 표본에서 예측 변수들이 결과 변수의 분산을 얼마나 설명하는지를 나타내는 통계적 척도
        - $0$과 $1$ 사이에 있으며, $1$에 가까울수록 예측 변수가 샘플의 분산을 완전히 설명함을 의미
        - 즉, 값이 $1$에 가까울수록 예측이 더 정확하다는 것을 의미
    - 근사 함수에 따르면, 시스템에서 활성 Pod가 많을수록 각 Pod가 제공할 수 있는 서비스 비율은 낮아짐
      - 이 선형 함수에서 Def. 1을 사용하여 Node.js 웹 애플리케이션의 프로파일을$AP_{\text{Node.js}}(x) = 125 + \frac{209}{x}$로 정의할 수 있음
      - 일부 애플리케이션의 프로파일은 선형 관계보다 더 복잡할 수 있음 (이 논문에서는 out of scope)
        - ex. 그림 2의 하단 부분에 있는 이미지 분류 애플리케이션의 측정값은 이차 함수로 근사할 수 있음
          - 그러나 Kubernetes는 주로 웹 애플리케이션에 사용되므로 대표적인 프로파일로 Node.js와 같은 인기 있는 웹 서버를 선택하는 것이 분명
    - 본 연구에서는 이 함수를 시뮬레이션 및 Kubernetes 배포에서 애플리케이션 프로파일로 사용할 것



## Analytical Models of HPA

- Kubernetes HPA의 동작을 설명하기 위한 분석 모델을 제안
  - 이러한 모델을 통해 제안된 ML 기반 예측 방법과 기준 HPA에 대한 스케일링 정확성을 평가하기 위한 광범위한 시뮬레이션을 수행할 수 있음

### A. HPA Operation in Details

- Kubernetes에서 HPA(자동 수평 확장)는 여러 매개변수에 의해 영향을 받음
  - 클러스터 수준 설정으로는 "down-scaling 안정화, Pod 동기화 주기, 스케일링 허용 오차" 등
  - HPA 수준으로는 "최소 및 최대 Pod 수, 임의의 메트릭에 따른 스케일링 임계값" 등
- 기본적으로 HPA는 CPU 사용률 측정을 기반
  - HPA : 주기적으로 시스템에서 모니터링 데이터를 가져와 <u>클러스터에 몇 개의 Pod가 있어야 하는지</u> 결정
    - 그 주기가 scaling interval
    - 스케일링 작업과 의사 결정 절차는 Definition 2에서 공식화

##### Definition 2 : Kubernetes HPA Parameters and Operation

- HPA를 구성하기 위해 필요한 parameter들

  - $c_{min}$, $c_{max}$ : 허용된 pod 수의 최소 / 최대

  - $\hat{u} \in [0, 1]$ : 실행 중인 모든 pod의 목표 평균 CPU 사용률

  - scaling interval : scaling 결정을 내리기 위해 HPA가 metric을 평가하는 time window (양의 실수 $\mathbb{R}^+$ 길이)

  - $s \in [0, 1]$ : scaling tolerance (허용 오차)

  - $d$ : downscale stabilization (현재 다운스케일 작업이 완료된 후 다른 다운스케일 작업을 수행하기 전에 HPA가 기다려야 하는 기간)

    - indicator $d_i$ : 스케일링 간격 $i \in \mathbb{N}$ 동안 다운스케일링이 활성화되었는지 여부를 나타냄
      $$
      \begin{align*} d_{i} = \begin{cases} 1 & \mathrm {if~stabilization~is~active~(downscale~is~disabled)}; \\ 0 & \mathrm {else}. \end{cases}\!\!\! \\ {}\tag{1}\end{align*}
      $$

    - $u_i \in [0, 1]$ : 스케일링 간격 $i \in \mathbb{N}$에서 측정된 CPU 사용률

- 추가 정보

  - 스케일링 결정은 아래의 조건을 만족할 때 스케일링 간격 $i$에서 이루어짐
    $$
    \begin{equation*} \left |{ \frac { u_{i}}{ \hat {u}} - 1 }\right | > s\tag{2}\end{equation*}
    $$

  - 그 후 $i+1$ 간격에서 pod의 수는 아래와 같이 재귀적으로 계산됨
    $$
    \begin{equation*} c_{i+1}^{*} = \left \lceil{ c_{i}\frac { u_{i}}{ \hat {u}}}\right \rceil \tag{3}\end{equation*}
    $$

  - 이후 limit을 적용하여 pod 수를 맞춤
    $$
    \begin{align*} c_{i+1}' = \begin{cases} c_{i+1}^{*} & \mathrm {if }~ c_{min}\leq c_{i+1}^{*} \leq c_{max} \\ c_{min}& \mathrm {if }~ c_{i+1}^{*} < c_{min}\\ c_{max}& \mathrm {if }~ c_{i+1}^{*} > c_{max}. \end{cases}\tag{4}\end{align*}
    $$

  - HPA : 이전 time window의 길이가 $d$인 동안 다른 다운스케일 작업이 있었으면 다운스케일 작업을 수행하지 않음
    $$
    \begin{align*} c_{i+1} = \begin{cases} c_{i} & \mathrm {if }~ c_{i+1}' < c_{i}~\mathrm { and }~ d_{i+1} = 1; \\ c_{i+1}' & \mathrm {else}. \end{cases}\tag{5}\end{align*}
    $$

  - $d$값이 클수록 더 많은 리소스 사용 -> scale시 안정화 없이 확장되지만, 축소는 더 느리게 이뤄짐

### B. An MMPP/M/c Model for Changing Arrival Regimes

- 모델링 목적을 위해 HPA의 작동과 애플리케이션 프로파일을 단순화
  - $d$ : 스케일링 간격과 같다고 가정
  - $d_i = 0 \text{ } \forall i$,  $c_\text{min} = 0$, $c_\text{max} = \infin$, $s = 0$
  - application profile은 일정하다고 가정. 즉, $AP(x) = \mu$
    - $\mu$ : 지수 분포의 service time 비율
  - request arrival 과정 : $m$개의 state를 갖는 **Markov-modulated Poisson process (MMPP)**로 설명될 수 있음
    - 관련 transition matrix는 $Q$; $m$개의 arrival rate는 $\lambda_1, \lambda_2 \cdots \lambda_{m}$

- HPA : 주기적으로 서버 수를 설정

  - 따라서 모델에서는 arrival rate 변화에 맞춰 동적으로 서버 수를 조정하는 경우를 고려

  - HPA는 **반응형** -> arrival rate 변화는 다음 스케일링 간격에서 서버 수에 영향을 미침

    - 서버 수는 MMPP arrival에 따라 달라지지만 한 스케일링 주기를 지연시킴

    - 시간 $t$에서의 arrival rate를 $\lambda(t)$라고 가정, 특정 state에서 MMPP가 머무는 시간이 요청 도착 간 시간보다 훨씬 길다고 가정하면 시간 $t$에서 서버 수는 아래 공식으로 계산
      $$
      \begin{equation*} C(t) = \left \lceil{ \frac {\lambda (t-1)}{\rho \mu }}\right \rceil,\tag{6}\end{equation*}
      $$

      - $\rho$ : M/M/c queue model의 서버 utilization, $\rho = \lambda / (c \mu)$
      - $\lambda$ : arrival rate
      - $c$ : 서버 수
      - $\mu$ : service rate
      - utilization은 management parameter와 같음 ( $\rho = \hat{u}$ )

  - 이 모델을 통해 Kubernetes HPA의 동작을 설명하기 위해 pod 사용량을 계산하고, MMPP 매개변수를 분석할 수 있음

    - MMPP의 arrival의 기본 마르코프 체인의 steady-state 확률 $\pi$는 아래와 같음
      $$
      \begin{equation*} \pi = \pi Q~\mathrm { and }~\pi \mathbb {I} = 1,\tag{7}\end{equation*}
      $$

      - $\mathbb{I}$ : all-ones 벡터

    - steady-state arrival rate : $\lambda _{MMPP} = \pi \lambda ^{T}\tag{8}$

      - $\lambda ^{T}$ : row vector $\lambda = (\lambda_1, \lambda_2 \cdots \lambda_{m})$의 transpose

- MMPP의 steady-state 분포를 사용하여 arrival rate를 추론할 수 있지만 시스템에 주어진 수의 pod가 있을 확률로부터 이 모델의 단점은 **job losses**를 설명하지 않는다는 것
  
  - operation 중에 timed-out request들이 가장 중요하기 때문에, <u>더 복잡한 HPA 모델을 제안</u>

### C. Our Discrete-Time Model for HPA

- HPA을 모방하기 위한 discrete-time queuing 모델을 제안
  - HPA의 다음 시뮬레이션에서 $c_{i+1}^{*}$를 계산하는 것을 모방
  - scaling decision은 대기 중인 고객의 비율과 서버 수를 기준으로 이루어짐
    - 그러나 대기 중인 요청 수 대신, 스케일링 결정을 위해 서비스된 요청 수를 사용
      - 이는 서비스된 요청 수와 CPU 사용률이 밀접하게 연결되어 있으며, 후자가 HPA의 스케일링 결정의 기초이기 때문

- 시스템에서 $i$ 기간 동안의 요청 수는 아래와 같이 정의
  $$
  \begin{equation*} L_{i} = L_{i-1} - M_{i-1} + \Lambda _{i} - T_{i}\tag{9}\end{equation*}
  $$
  - $\Lambda_i$ : 도착한 요청 수

  - $M_i$ : $i$ 기간 동안 서비스된 요청 수
    $$
    \begin{equation*} M_{i} = \min \left \{ { c_{i} AP\left ({c_{i}}\right), L_{i}}\right \}\tag{10}\end{equation*}
    $$

    - $c_i$ : pod 수
    - $AP(c_i)$ : service rate // definition 1에서 정의됨
    - 즉, $M_i$는 <u>시스템에서 서비스할 수 있는 요청 수</u>와 <u>시스템에 있는 요청 수</u> 중 **최솟값**

  - $T_i$ : $i$ 기간 동안 만료된 대기 요청 수

  - $L_i$ : 해당 시간 동안 도착한 요청은 포함. but '이전 기간에 서비스된 요청' 이나 '해당 기간에 손실된 요청'은 포함X

  - initial period에서 queue는 비어 있다고 가정 ($i=0$, $L_0 = 0$, $c_0 = 0$, $\Lambda_0 = 0$)

- 이 모델은 특정 스케일링 간격 동안 시스템의 under-provisioning을 반영, 만료된 요청의 수를 통해 요청 손실을 $T_i$로 공식화
  $$
  \begin{align*} T_{i} = \begin{cases} 0,\quad \mathrm {if }~(i-t) < 0 \\ \left ({L_{i-t} - \sum _{j=1}^{t}M_{i-j}}\right)^{+} \end{cases}\tag{11}\end{align*}
  $$

  - $t$ : scaling interval으로 표현된 <u>timeout 값</u> // 만약 $t=x$라면 ($i-x$) period에 도착하여 $(i-1$)까지 서비스되지 않은 모든 요청은 ($i$) period에 만료됨

- CPU usage : 서비스된 요청 수를 서비스할 수 있는 최대 요청 수로 나눈 비율로 근사
  $$
  \begin{equation*} u_{i} \simeq \frac { M_{i} }{ AP\left ({c_{i}}\right) c_{i}},\tag{12}\end{equation*}
  $$

- Kubernetes에서 필요한 Pod 수를 계산할 수 있음
  $$
  \begin{equation*} c_{i+1}^{*} = \left \lceil{ \frac { M_{i} }{ \hat {u} AP\left ({c_{i}}\right)}}\right \rceil.\tag{13}\end{equation*}
  $$

- 다운스케일 안정화 값을 나타낼 양의 정수 $d$ 도입

  - $d = 1$이면, 스케일링 작업이 하나의 스케일링 간격 내에서 수행될 수 없기 때문에 시스템에 영향을 미치지 않음

  - 스케일링 간격 $i$에서 다운스케일 안정화의 indicator 함수 (for $d > 1$)
    $$
    \begin{align*} d_{i} = \begin{cases} 1 & \mathrm {if }~\exists j~:~2 \leq j \leq d, c_{i-j+1} < c_{i-j} \\ 0 & \mathrm {else}. \end{cases}\tag{14}\end{align*}
    $$

    - 이후 측정하는 동안 다운스케일 안정화는 그 영향을 무시할 수 있도록 스케일링 간격의 길이로 설정됨

- 이 모델을 통해 <u>시스템의 pod 수</u>, <u>resource 사용량</u> 및 <u>손실된 request 수</u>를 시뮬레이션할 수 있음
  
  - 애플리케이션의 고유한 프로파일과 <u>excess</u> parameter를 고려할 수 있음

### D. Evaluation of the MMPP/M/c and Discrete Models

- NASA-HTTP, WorldCup98 : 웹 애플리케이션 테스트에 널리 사용되는 데이터 세트들
  - NASA-HTTP : 플로리다의 NASA 케네디 우주 센터 웹 서버에서 두 달 동안의 모든 HTTP 요청을 포함하고 있음
  - WorldCup98 : 1998년 월드컵 웹사이트에 대한 세 달 동안의 HTTP 요청을 포함
    - 그러나 이 로그는 각각 1995년과 1998년의 데이터로 다소 오래된 자료
- 제시된 HPA 모델을 검증하기 위해, 최근에 수집된 익명화된 웹 트래픽 데이터를 사용하여 시뮬레이션을 수행
  - 이 데이터는 가을 학기 중 일주일간 대학 캠퍼스 네트워크에서 Facebook으로의 초당 흐름 수를 포함하고 있음
  - Facebook 데이터는 실제 사용자에 의해 생성된 것이므로 클라우드 애플리케이션 트래픽을 잘 나타냄
    - 예를 들어, 일간 트렌드를 보여주고, 분당 평균 요청 수가 매우 높지는 않지만, 비교적 높은 변동성을 보임
    - 따라서 이는 **엣지 클라우드 애플리케이션의 특성에 적합**
    - 즉, 상대적으로 작은 사용자 기반과 변동이 심한 트래픽을 특징으로 함
    - 이러한 이유로 시뮬레이션 및 실험에 이 추적 데이터를 사용할 것

- 또한 시뮬레이션 목적으로 MMPP 기반의 트래픽 추적을 생성
  - Facebook 트래픽 추적에 MMPP를 적용
  - 이 알고리즘은 트래픽 정보를 바탕으로 MMPP의 매개변수를 계산할 수 있음
  - 시뮬레이션을 위해 이 맞춘 MMPP에서 생성된 도착 시퀀스를 사용
  - 이 합성 추적을 통해 MMPP/M/c HPA 모델과 손실이 있는 이산 모델을 비교할 수 있음
  - 또한, 합성 추적이 실제 추적만큼 명확하게 일일 패턴을 보여주지 않기 때문에, 예측 방법의 동작을 덜 예측 가능한 트래픽 상황에서 시뮬레이션하고 평가할 수 있으며, 이를 통해 시스템의 견고성을 분석할 수 있음
- 먼저 이산 모델이 HPA의 동작과 일치하는지 확인하기 위해, 실제 추적에 대응하여 시뮬레이터와 실제 Kubernetes 클러스터에서 시작된 Pod 수를 비교
  - 제안된 모델을 사용하여 시스템의 pod 수, 즉 리소스 사용량과 만료된 요청 수를 애플리케이션의 고유 프로파일을 고려하여 시뮬레이션
  - 스케일링 간격은 1분으로 설정되어 새 pod의 시작 시간을 무시할 수 있을 정도로 작게 만듬
  - 다운스케일 안정화도 1분으로 설정
  - 결과
    - x축은 시간을, 첫 번째 다이어그램의 y축은 Pod 수를, 두 번째 다이어그램의 y축은 Pod 분의 합계, 즉 시뮬레이션된 분 동안 누적된 Pod 수의 합계를 나타냄
    - 이산 모델은 실제 HPA 작동과 유사하게 동작했으며, Pod 수에 대한 좋은 근사치를 제공
    - 시뮬레이션된 모델의 분당 Pod 수와 실제 측정을 비교한 평균 제곱 오차(MSE)를 계산한 결과, MSE는 0.27로 나왔고, 이산 모델에 의한 평균 Pod 수는 2.38, HPA 측정에 의한 평균 Pod 수는 2.28

- 다음으로 이산 모델과 MMPP/M/c 기반 모델을 비교
  - 후자는 MMPP 입력을 필요로 하기 때문에, 합성 트래픽을 사용
  - 다음 공식을 적용하여 스케일링 간격 $i$에서 MMPP/M/c 기반 모델의 Pod 수를 계산 $c_{i+1} = C(i+1))$
    - $C$ : 주어진 pod 계산 방법
  - 일정한 서비스 속도와 스케일링 간격보다 짧은 다운스케일 안정화로 시뮬레이션을 수행
    - 두 분석 모델은 유사하게 동작
    - 입력 트래픽이 변동성이 클 때 약간의 차이가 관찰됨
      - MSE는 평균 pod 수에 비해 낮다는 것을 알 수 있음
- 결론 : 특정 조건에서는 MMPP/M/c 모델을 이산 모델 대신 사용할 수 있으며, 이를 통해 HPA 동작을 분석적으로 평가할 수 있음
  - MMPP/M/c 모델은 무손실 운영을 가정하기 때문에, 직관적으로 excess 매개변수 값이 낮아 손실된 요청이 적은 설정에서 HPA의 근사치가 더 좋음
  - 따라서 MSE 값이 0.85로 상대적으로 낮음



## ML for Scaling: The Proactive HPA+

- 리소스 사용을 보다 효과적으로 하고 서비스 품질을 높이기 위해 미래의 요청을 예측하고 자원을 미리 할당하거나 해제하는 스케일링 방법을 설계한 내용을 다룸
- HPA+라고 명명된 솔루션에서는 <u>들어오는 요청을 예측</u>하고, 이에 따른 <u>스케일링 결정을 내리는</u> **두 개의 모듈**로 구성
  - 많은 솔루션이 서버 스케일링에 Q-learning만 사용하는 것과 달리, 제안에서는 Q-learning 방법 외에도 auto-regression (AR)과 신경망(NN) 같은 인기 있는 머신러닝(ML) 방법을 예측에 사용
    - 작동 방식이 상당히 다른 모델을 활용하는 것이 중요
  - 각 모델의 단점
    - AR은 간단한 모델로 리소스 요구가 적은 반면, LSTM과 HTM은 더 많은 컴퓨팅 파워가 필요
    - 두 모델은 본질적으로 다름
      - LSTM : 지도 학습을 요구
      - HTM : 비지도 학습 접근 방식을 사용
    - Q-learning 기반의 강화 학습(RL) 방법은 좋은 행동(또는 예측)을 내놓기까지 state space를 탐색하는 데 비교적 오랜 시간이 걸림

### A. ML-Based Prediction of Web Requests

- ML 모델은 익명화된 웹 트래픽 데이터를 기반으로 훈련 및 테스트
  - Facebook을 대상으로 하는 요청의 시계열 데이터를 사용
  - 이 데이터는 하루와 주간 패턴을 보여주며, 분당 방문 횟수의 표준 편차가 높아 중간 규모의 고객 기반을 가진 평균 웹 서비스의 request dynamics를 잘 설명
- 예측 모델을 구축하고 평가하기 위해 데이터셋을 Z-점수 정규화를 사용하여 표준화
  - 웹 요청 시간 초과와 Pod 시작 시간이 초 단위이기 때문에 시간 단위를 분 단위로 설정
    - grid search를 통해 시간 단위와 다른 시스템 매개변수를 최적화한 결과, 분 단위로 설정하는 것이 트래픽 추세를 가장 잘 포착한다는 것을 발견

- 예측 모델
  - **Auto-Regressive (AR)**
    - 이전 time series 값에 따라 현재 traffic load를 선형적으로 예측하는 간단한 모델
    - 최상의 정확도를 위해 이전 32분의 관측값을 사용해야 한다는 것을 train을 통해 알아냄
    - 각 이전 관측값을 어느 정도 사용할지를 최적화하여 coefficients를 계산
  - **Long Short-Term Memory (LSTM)**
    - 지도 학습 방법 중 하나로, 인기 있는 딥러닝 모델
    - 다양한 look-back window 크기를 검사한 결과, 15분이 가장 유익하다는 것을 발견
      - $t-15$에서 $t$까지의 모든 load 값을 사용하여 $t+1$의 load를 예측
    - network의 input으로 각 $t$에 대해 'look-back window에서의 load'와 '자정으로부터 지난 시간' 사이의 **load difference**를 사용
  - **Hierarchical Temporal Memory (HTM)**
    - 비지도 학습에 속하는 생물학적 신경망 모델로, 시퀀스의 시간 패턴을 학습
    - Numenta.org의 구현을 사용하여 입력 부하와 타임스탬프를 인코딩하여 네트워크가 15분의 역사를 고려하도록 함
    - HTM 아키텍처의 여러 매개변수를 최적화하여 최고의 성능을 달성
  - **SARSA 및 Q-learning**
    - 강화 학습(RL) 기반 접근법으로, 시스템의 현재 부하와 실행 중인 Pod 수를 상태로 모델링
    - 주어진 상태에서 최적의 동작(scale out, scale in, idle)을 찾음
    - RL 솔루션의 출력은 필요한 Pod 수이지만, 이를 주어진 Pod 수로 처리되는 QPS로 변환할 수 있음

- 모델 훈련
  - train 데이터셋 : week의 첫 3일동안 // test 데이터셋 : 4일째
  - grid search과 walk-forward cross validation을 사용하여 각 모델의 하이퍼파라미터를 최적화
  - 최적의 하이퍼파라미터를 사용하여 최종 모델을 생성하고 테스트 데이터셋에서 평가

- 평가 결과
  - **RMSE 및 R² 값**으로 예측 방법의 결과를 평가했습니다.
    - $R^2$ : 모델이 미래의 outcome을 얼마나 잘 설명하고 예측하는지 나타내는 점수 (1에 가까워야 좋음)
  - 각 모델의 결과
    - **AR** : RMSE = 0.092, R² = 0.879, 훈련 시간 = 14.7ms ± 0.6ms
      - 빠른 훈련 시간과 적은 리소스 요구로 가장 효율적
      - 이상치에 민감
    - **LSTM** : RMSE = 0.107, R² = 0.859, 훈련 시간 = 283s ± 12s.
      - 이상치에 강함
      - 훈련 시간이 길고 많은 리소스를 필요
    - **HTM** : RMSE = 0.138, R² = 0.819, 훈련 시간 = 13s ± 2s.
      - AR과 LSTM의 중간 정도의 리소스 요구
      - 노이즈에 강함
      - 새로운 입력 패턴에서 지속적으로 학습할 수 있음
      - 사전 훈련이 필요 없음
    - **RL** : RMSE = 0.737, R² = 0.456, 훈련 시간 = 212s ± 24s.
      - 사전 지식 없이 학습할 수 있음
      - 실제 관찰을 통해 환경 지식을 온라인으로 업데이트할 수 있음
      - 학습 시간이 김


### B. The More ML, the Better

- HPA의 손실이 있는 이산 시간 대기열 모델과 ML 예측 모델의 즉각적인 성능을 수치 시뮬레이션에서 비교
  - '오라클'과 비교
    - 각 스케일링 간격에서 도착하는 요청 수를 완벽하게 예측하고 그에 따라 Pod 수를 설정하는 이상적인 모델
    - 리소스 사용의 벤치마크로 사용
    - 각 모델의 Pod 사용량 초과분을 오라클과 비교하여 평가
    - 프로비저닝 결정 문제의 다른 측면에서, 전체 웹 요청 대비 손실된 요청 비율(Loss)을 살펴봄
  - 네 가지 ML 기반 모델 (AR, LSTM, HTM, RL)
    - 트래픽 추적에서 facebook.com으로 향하는 3일간의 플로우 카운트 시리즈로 훈련
    - 4일째 트래픽에서 평가

- 다섯 가지 스케일링 방법에 대한 시뮬레이션 결과 (그림 7)
  - 상단 다이어그램: 누적 요청 손실 비율 (Loss)
    - 시뮬레이션 초반: HTM이 손실 측면에서 가장 좋은 성능
    - 하루 중반: 모델들의 누적 손실 비율이 동적으로 변화
    - 하루가 끝날 때: AR이 가장 낮은 값 기록
    - RL은 하루 종일 가장 나쁜 손실 성능
  - 하단 그래프: Pod 사용량 초과분 (오라클 대비) 누적치
    - HTM은 최적의 리소스 사용과 거리가 멀며, 오라클보다 약 8% 더 많은 Pod 분 사용
    - RL 방법도 비슷하며, 하루 중 몇 가지 경우를 제외하고 나머지 모델보다 더 나은 예측 제공 못함
    - 다른 세 가지 방법 (AR, LSTM, HTM)은 유사한 성능
      - 초반: AR이 최적에 가장 가까움
      - 시뮬레이션 후반: LSTM이 최적에 더 가까움
    - 결과: LSTM이 Pod 사용량 초과 측면에서 오라클에 더 가깝지만, 더 높은 손실 초래
    - 다른 방법들은 약간 더 많은 Pod 사용

- 합성 MMPP 추적을 사용하여 비교 시뮬레이션 (그림 8)
  - AR과 LSTM 모델이 손실 측면에서 선두
  - 복잡한 MMPP 트래픽: HTM 모델로 잘 추적되지 않아 과도한 프로비저닝 초래
  - RL의 스케일링 결정: 지연되거나 불충분하여 많은 요청 손실 초래
  - 손실이 있는 이산 HPA 모델: 필요한 것보다 훨씬 적은 리소스 할당, 많은 손실 초래, 첫 번째 두 모델보다 많은 요청 손실

- 결과
  - 최적의 스케일링을 항상 잘 근사하는 단일 방법이 없음
  - 입력 추적은 하루 동안 다양한 역학을 보이며, 각 방법은 때때로 잘 수행되거나 잘못 수행
  - 모델이 확장 가능하고 시스템 매개변수를 언제든지 변경할 수 있는 프레임워크 설계
    - 임의의 수의 모델 적용 가능
    - 더 많은 모델은 더 많은 리소스를 필요로 하지만, 다양한 모델을 통해 입력 트래픽 변동의 다양한 경우 커버
  - 여러 방법이 서로 경쟁하고, 최근 입력에서의 성능에 따라 활성화되고 의사 결정을 내리는 스케일링 엔진 설계

### C. The Proactive Scaling Engine: HPA+

- HPA+ 스케일링 엔진의 고수준 운영 설계
  1. **예측 모델**: 네 가지 머신러닝(ML) 기반 예측 모델을 포함하여 들어오는 웹 요청의 비율을 예측
  2. **모델 선택**: 매 분마다 최근 과거에 대한 예측 모델의 정확도를 계산하고, 가장 정확한 모델을 선택
  3. **Pod 수 계산**: 선택된 모델의 예측을 기반으로 다음 분에 필요한 Pod 수를 애플리케이션의 프로파일을 사용하여 계산
  4. **기본 HPA로 전환**: 모든 ML 기반 모델의 정확도가 낮을 경우, 일시적으로 기본 HPA 운영으로 전환.

- 모델 정확도 계산
  - 각 모델의 정확도는 짧은 히스토리 윈도우에서의 과거 예측을 기반으로 계산
  - 가장 성능이 좋은 모델의 예측이 스케일링 결정에 사용됨
  - 정확도 평가는 최근 n분 동안의 상대 퍼센트 차이(Relative Percentage Differences, **RPD**)의 평균을 기반으로 수행
    $$
    \frac {1}{n}\sum _{i=t-n}^{t-1}\left ({2\frac {|\hat {q_{i}}-q_{i}|}{\hat {q_{i}}+q_{i}}}\right), \tag{15}
    $$
    - $q_i$ : $i$번째 분의 평균 QPS
    - $\hat{q}_i$: $i$번째 분의 예측된 평균 QPS
    - 모델의 정확도가 0에 가까울수록 예측이 더 좋음
      - 네 가지 ML 모델 중에서 가장 낮은 RPD 값을 가진 모델이 사용됨
    - 기본 HPA로 되돌아가는 매개변수로서 fallback threshold 사용
      - 모델의 정확도 값이 임계값보다 크면 엔진은 기본 HPA 운영으로 전환됨
  
- 실험 전 HPA+ 운영 시뮬레이션
  - 기본 Kubernetes 자동 확장기인 HPA보다 더 나은 스케일링 결정을 내릴 수 있는지 확인
  - 다양한 매개변수 세트 (히스토리 윈도우와 fallback threshold)에 대해 ML 모델과 HPA 이산 모델을 3일간의 데이터셋으로 훈련하고, 4일째 데이터로 평가
  - 활성 모델은 매 분마다 (15) 방정식에 의해 결정
  - 사용된 Pod 수와 분당 손실된 요청 수는 선택된 활성 모델의 스케일링 결정을 고려하여 계산
  - 비결정적 동작을 고려하여 각 매개변수 세트에 대해 시뮬레이션 25회 수행

- 최적 매개변수 탐색 결과
  - 히스토리 윈도우: 5분
  - fallback threshold: 0.3

- 입력 부하 증가 시 HPA+ 효율성 변화 조사
  - 입력 요청 비율을 원래 추적의 30배까지 확대한 시뮬레이션 실행
  - HPA+와 HPA에 의해 발생하는 비용 비교
    - 비용: Pod 분의 가격과 손실된 요청에 대한 페널티 포함
    - 비용 항목을 무역 오프 매개변수로 요약
      - SLA 위반의 비용 (손실된 요청 수)을 클라우드 리소스 비용 (Pod 분)으로 변환
  - 결과 (그림 9)
    - y축: HPA+의 총 비용 대비 HPA의 총 비용의 평균 비율
    - x축: 원래 추적과 비교한 입력 요청 비율 강도
    - 5개의 다른 무역 오프 매개변수 (0.01, 0.1, 1, 10, 100)에 대한 비용 비율 표시
    - 클라우드 리소스가 SLA 위반보다 훨씬 더 비싼 경우 (무역 오프 매개변수 100) HPA+가 약간 더 높은 비용 초래
    - 다른 경우: HPA+가 상당한 비용 절감, 주로 HPA+로 인해 크게 줄어든 요청 손실 덕분
    - 입력 요청 비율이 높아질 때 (5배 이상 증가할 때) HPA+는 운영 비용 크게 절감
      - 이유: Pods의 용량이 전체 수요에 비해 상대적으로 작아지기 때문에 요청 비율에 더 잘 맞출 수 있음

## Evaluation of HPA+ and the Excess Parameter

- Kubernetes에서 HPA+ 구현에 대한 세부 사항을 설명
- 애플리케이션 제공자가 자신들의 trade-off parameter에 따라 조정할 수 있는 사용하기 쉬운 **management parameter**를 소개
- HPA+와 함께한 실험의 유망한 측정 결과를 제시

### A. HPA+ Implementation

- 실험을 위해 Amazon Web Services(AWS)에서 Kubernetes 배포를 생성
- Kubernetes 클러스터를 AWS에 배포하고 유지 관리하는 오픈 소스 도구인 Kubernetes Operations(kops) 사용
  - 클러스터 : 하나의 마스터 노드와 다섯 개의 작업자 노드로 구성
  - 모든 노드는 EC2 m5.xlarge 인스턴스로, 각 인스턴스는 4개의 vCPU와 16GB RAM을 갖춤

- HPA+ : '애플리케이션 리소스 프로파일' + 'AR, LSTM, HTM, RL 모델' 을 사용하여 구현된 ML 기반 시스템
  - 스케일링 엔진 : Python으로 구현
  - Kubernetes 마스터 노드에서 demon 서비스로 실행

- system 구조와 workflow
  - **테스트 애플리케이션**: HAProxy 인그레스 컨트롤러를 통해 접근할 수 있는 Node.js 웹 애플리케이션 사용
  - **메트릭 수집**: Prometheus 모니터링 시스템 사용
    - Node.js 애플리케이션이 로드될 때 HAProxy Prometheus 어댑터가 관련 메트릭(예: 요청 수, 응답 수, 오류) 캡처
    - Prometheus가 메트릭을 수집
  - **HPA+ 시스템 주요 부분**: 커넥터 모듈
    - Prometheus에서 QPS 메트릭 수집
    - AR, LSTM, HTM, RL 모델을 사용하여 미래의 QPS 값 예측
    - Node.js 애플리케이션 프로파일을 기반으로 요청을 처리할 수 있는 Pod 수 계산
    - 시스템의 Pod을 사전에 scale out 또는 scale in
- 시스템 매개변수 최적화가 사전 수행되지 않은 경우의 HPA+ 운영
  - **초기 훈련**
    - ML 모델이 QPS 데이터를 기반으로 훈련 시작
    - HTM과 같은 온라인 학습 모델 및 기본 시스템 매개변수를 활용하여 작동
    - 특정 애플리케이션 입력 속도 역학에 맞게 폴백 임계값, 히스토리 윈도우, 결정의 시간 세분성을 최적화 필요

  - **역사적 데이터 수집 후 최적화**
    - 충분한 양의 역사적 데이터 수집 후 과거 트래픽의 시뮬레이션 생성
    - 시스템 전체 매개변수 최적화

  - **기본 HPA 메커니즘으로 폴백**
    - CPU 사용률을 스케일링 메트릭으로 사용하는 기본 HPA 메커니즘으로 되돌아갈 수 있음

  - **ML 기반 스케일링 전환**
    - 모델이 예측 준비가 되면 정확도가 충분히 좋은 경우 ML 기반 스케일링으로 전환
    - 지난 1분간의 평균 QPS가 모델의 히스토리에 추가
    - 역사적 값을 기반으로 다음 1분의 QPS 예측

  - **경쟁 환경에서의 모델 운영**
    - 예측 모델의 정확도를 지속적으로 평가
    - 다음 1분의 QPS 예측 전에 각 모델의 정확도를 (15) 공식을 사용하여 계산
    - 가장 성능이 좋은 모델 사용

  - **메트릭 보고**
    - 예측된 메트릭과 폴백 시 CPU 메트릭을 커넥터가 HPA에 사용자 정의 메트릭으로 보고
    - 기본 HPA 모듈이 이를 기반으로 스케일링 결정

- HPA+의 main control loop 알고리즘 수도코드

  ```pseudocode
  Algorithm 1 : Main HPA+ Control Loop
  Require : all ML models are trained
  Require : Profile(x) : an invertible Application Profile function (Definition 1)
  
   1| excess <- value of excess parameter from Section 6-B
   2| MLModels <- list of all ML models
   3| while TRUE do
   4|     compute accuracy for all models in MLModels
   5|     if at least one model is accurate, see (15) then
   6|         for all model in accurate MLModels do
   7|             get prediction from model
   8|         end for
   9|         pred <- select best prediction according to (15)
  10|         pod <- get # of pods (Prometheus)
  11|         newPods <- Profile^(-1) (pred / pod) / excess
  12|         send newPods to Prometheus as custom metric
  13|     else
  14|         switch back to default HPA
  15|     end if
  16|     sleep 1 minute
  17| end while
  ```

### B. Adaptive Operation Under Excess Policies

> 2024-06-24 여기부터 추가 정리할 것

애플리케이션 제공자는 Kubernetes 또는 유사한 관리 시스템을 사용하여 애플리케이션을 운영하며, 예약된 클라우드 리소스에 대한 비용을 지불하고 SLA에 따라 QoS를 제공합니다. 운영 비용과 SLA 위반 페널티에 따라 다양한 전략을 취할 수 있습니다.

- **전략**
  - **비용 무시, 최대한 SLA 준수**: 자원 과잉 할당 발생
  - **비용 최소화**: 자주 SLA 위반 발생
  - **중간 지점**: 적절한 자원 할당으로 드물게 SLA 위반

- **excess 매개변수**
  - 애플리케이션 제공자의 리소스 할당 전략 설명
  - 0과 1 사이의 숫자
  - 실험에서 사용된 세 가지 예시 값:
    - **보수적 전략 (excess = 0.85)**
      - 리소스 활용도 평균 85% 유지
      - 리소스 낭비 많음
      - SLA 위반 거의 없음
    - **일반 전략 (excess = 0.9)**
      - 활용도와 SLA 위반 사이의 균형 유지
      - 리소스 활용도 약 90% 유지
    - **최선 노력 전략 (excess = 0.95)**
      - 높은 부하 상태에서만 스케일링
      - 활용도 평균 95% 유지
      - SLA 위반 확실
      - 할당된 리소스 측면에서 운영 비용 최소화

- **실험적 비교**
  - HPA와 HPA+의 운영 비교
  - 초과 매개변수의 영향을 평가
  - 초과 매개변수와 리소스 활용률 일치
    - 애플리케이션 제공자는 시스템의 모든 Pod의 활용률이 평균적으로 이 매개변수 값에 있도록 스케일링 원함

### C. Results of HPA+ Experiments

HPA+를 실제 시나리오에서 다양한 스케일링 전략을 사용하여 테스트하고, 이를 기본 HPA와 비교했습니다. Fig. 3에 표시된 Facebook 트래픽을 사용하였으며, 각 스케일링 정책마다 1분마다 애플리케이션 Pods 수, 도착 요청 수, 손실 요청 수를 수집했습니다. 각 정책에 대해 5회 반복하여 평균 결과를 계산했습니다.

- **비용 분석**
  - 클라우드 제공업체에 지불하는 비용은 실행 중인 Pods 수에 비례하고, SLA 위반으로 인한 손실 요청에 대한 보상 비용도 포함됩니다.
  - Fig. 11: 왼쪽 y축은 누적 손실 요청 비율, 오른쪽 y축은 시간에 따른 Pod 분의 합을 나타냅니다.
  - Pod 사용량에서는 HPA와 HPA+ 간에 큰 차이가 없었으나, lower excess 매개변수 값이 낮을수록 시스템에서 손실되는 요청 수가 줄어듭니다.
  - 최선 노력 스케일링 정책 하에서도 HPA+는 HPA에 비해 손실 요청 수가 현저히 적습니다.

- **정책별 성능 비교 (Table III)**
  - **보수적 전략 (excess = 0.85)**
    - 손실 요청 수 감소: 44%
    - Pod 분 사용량 증가: 3%
  - **일반 전략 (excess = 0.9)**
    - 손실 요청 수 감소: 72%
    - Pod 분 사용량 증가: 9%

- **실험 결과**
  - 손실 요청 수 감소는 HPA+의 다양한 예측 방법의 경쟁적인 운영 덕분입니다.
  - Fig. 12: 다섯 가지 스케일링 방법이 활성화된 시간대를 보여줍니다. AR은 44%, LSTM은 32%, HTM은 16%, 기본 HPA는 8%의 시간을 사용했습니다.

- **추가 분석**
  - HPA+는 약간의 Pod 사용량 증가를 대가로 손실 요청 수를 줄일 수 있습니다.
  - 동일한 스케일링 정책이 HPA와 HPA+에서 다른 자원 사용량을 초래할 수 있습니다.
  - Fig. 13: 모든 스케일링 정책에 대해 HPA와 HPA+의 Pod 분 사용량과 요청 손실을 비교했습니다.

- **다른 최첨단 스케일링 엔진과의 비교 (Fig. 14)**
  - Elascale [12] 및 강화 학습 기반 방법 [21]과 비교한 결과, HPA+가 더 나은 성능을 제공했습니다.
  - Elascale의 경우, 스케일링 중에 하나의 인스턴스만 시작 또는 종료되므로, 급변하는 사용자 수요를 충분히 처리할 수 없었습니다.
  - 강화 학습 기반 알고리즘은 HPA+의 성능에 근접했지만, 다양한 모델을 선택하여 단일 RL 알고리즘을 약간 능가했습니다.



## Conclusion

마이크로서비스 개념과 관련 기술은 애플리케이션 개발 및 서비스 운영의 새로운 방식을 제공합니다. 리소스 스케일링과 같은 중요한 관리 작업은 기본 클라우드 플랫폼에 위임되지만, 클라우드 플랫폼에서 새로운 서비스를 제공하는 애플리케이션 제공자는 운영을 관리하기 위한 적절한 구성 인터페이스가 필요합니다. 우리는 가장 널리 사용되는 컨테이너 오케스트레이션 시스템인 Kubernetes를 다루고 여러 자동 스케일링 방법을 평가했습니다.

- **첫째**: 현재 사용 가능한 수평 자동 스케일러를 조사
  - 두 가지 분석 모델을 제안하여 널리 사용되는 방법의 특성을 포착
  - 주어진 고객 기반과 SLA 수준에 맞추어 기본 리소스 풀을 설계할 때 계획 단계에서 중요한 역할 수행

- **둘째**: 새로운 프로액티브 스케일링 엔진 설계 및 구현
  - 여러 ML 기반 예측 방법 포함
  - 다양한 상황에서 운영 최적화
  - 캠퍼스 네트워크에서 수집된 추적 데이터를 기반으로 한 시뮬레이션 및 실제 측정 평가

- **셋째**: 간단한 관리 구성 도입
  - 쉬운 구성 가능성에 중점
  - 리소스 과잉 할당과 SLA 위반 사이의 타협을 지정하여 리소스 할당 정책을 제어하는 초과 매개변수 도입

- **결과**: 자동 스케일링 엔진의 성능
  - 약간 더 많은 리소스 사용 비용으로 거부된 요청 수를 크게 줄임
  - 추가 리소스 소비는 일반적인 운영과 비교할 때 무시할 만한 수준
  - HPA와 HPA+가 유사한 양의 리소스를 소비하도록 초과 매개변수를 설정하면 더 낮은 손실 요청 비율 제공