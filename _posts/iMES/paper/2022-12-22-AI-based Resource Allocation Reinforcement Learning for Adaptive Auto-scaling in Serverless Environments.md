---
title: "AI-based Resource Allocation: Reinforcement Learning for Adaptive Auto-scaling in Serverless Environments"
author: Joe2357
categories: [iMES, Review/paper]
tags: [iMES, review, paper]
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
- revision의 user pod가 사용 가능할 때까지 request를 buffer에 저장함 -> **clod start 비용이 발생할 수 있음** ( 단점 )
  - 최소 하나의 replica가 활성 상태로 유지되고있다면, activator가 bypass되고 traffic이 user pod로 직접 흐를 수 있음
- request가 pod에 도달한 경우 : queue proxy container에 의해 채널링됨 -> user container에서 처리됨
  - queue proxy : 특정 수의 request만 동시에 user container에 입력되도록 함 ( 필요한 경우 request는 queue에 넣어둠 )
  - 병렬 처리된 request의 양은 **동시성 매개변수**에 의해 지정됨 ( 0 ~ 1000, 기본값 100 )
    - 지정된 숫자만큼의 request만 병렬처리를 허용함
    - 0으로 설정하는 경우 : scaling 없이 무제한 병렬처리
  - 각 queue proxy는 들어오는 load를 측정하여 별도 port에 보고
    - autoscaler : 원하는 동시성 수준을 유지하기 위해 새로운 pod를 추가하거나 제거하는 결정을 내림

### B. Q-Learning





