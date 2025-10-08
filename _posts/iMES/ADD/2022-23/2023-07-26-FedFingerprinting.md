---
title: "FedFingerprinting: A Federated Learning Approach to Website Fingerprinting Attacks in Tor Networks"
author: [j.Bang, Joe2357, j17.lee]
categories: [1. iMES, Agency for Defense Development (ADD) / 2022-23]
tags: [Agency for Defense Development (ADD) / 2022-23, Publication]
date: 2023-07-26 12:00:00 +0900
last_modified_at: 2023-07-26 12:00:00 +0900
description: "- Published in IEEE Access (Vol.11)"
math: true
---



## Abstract

- Website Fingerprinting (WF) 공격 : Tor 네트워크에서 트래픽을 분석하여 불법 웹사이트에 접근하는 익명 사용자를 탐지하기 위한 네트워크 공격
  - 패킷 길이, 수, 시간 등과 같은 여러가지 트래픽 기능을 고려하여 <u>금지된 컨텐츠에 접근하는 사용자를 식별</u>하는 것이 목적
  - AI의 발전으로 **기계학습 또는 심층 학습** 기술을 통해 불법 사용자의 프라이버시를 깨기 위한 모델 생성 방식이 널리 채택되기 시작하고, 여러 문제점이 발생
    - WF에 대한 이전 접근방식은 다양한 Tor 노드에서의 전체 데이터가 수집되어 모델을 생성하는 **중앙집중식 학습**이 필요함
      - 그러나 훈련 데이터셋에는 Tor 노드들이 공유하기를 원하지 않는 중요정보가 포함될 수 있음
    - 중앙집중식 학습은 중앙서버의 연산 및 네트워크 병목현상을 유발하게 됨
- Tor 노드가 본인의 데이터셋을 노출하지 않으면서 global model을 생성할 수 있는 **WF의 새로운 프레임워크** 제안
  - Federated Learning (FL) 과정에서 선택된 Tor 노드들의 local training의 부담을 덜어주기 위해 머신러닝의 앙상블 기법을 활용하여 feature들의 중요성을 분석하고 평가
  - 정확도와 학습시간의 균형을 맞추기 위해, raw data 말고 중요도 순으로 나열된 최상위 feature들의 조합을 활용할 수 있는 FL 기법을 통해 학습
  - 각 Tor 노드의 local model 정확도를 고려하여 FL process를 위한 효과적인 Tor 노드 선택 알고리즘 설계
  - 실제 Tor 데이터셋을 사용하는 closed world에서 제안된 방법이 raw data 및 feature selection dataset 모두에게서 벤치마크보다 좋다는 것을 입증
  - Tor 노드를 selection하는 기법의 성능을 수렴 속도 측면에서 평가



## Introduction

- Onion Router (Tor) : 익명으로 인터넷을 탐색할 수 있는 사실상 가장 유망한 도구 중 하나
  - Onion 프로토콜 : 무작위로 선택된 3개의 **중간 Tor 노드**를 통해 사용자로부터 서버로의 <u>암호화된 회로</u>를 생성 // 중간 노드 (relay) 가 통신의 시작점 (source) 와 도착점 (destination) 을 모두 알 수 없도록 하는 개인정보 보호 방법
  - 다양한 연구에서 다양한 접근 방식을 통해 Tor 네트워크가 제공하는 프라이버스를 공격하려고 시도
    - Website Fingerprinting 공격 (WF)
    - Multiplication 공격
    - Intersection 공격
    - Timing 공격
  - Tor 네트워크의 프라이버시를 공격하는 이유 : 네트워크의 불법 웹사이트에 접근하려는 익명 사용자를 탐지하기 위함
- 최근 인공지능 (AI) 기술의 확산으로 인해 머신러닝 또는 딥러닝을 통해 트래픽 분석 공격을 진행할 수 있게 되었고, WF 공격에 적용되어 연구되고 있음
  - Website Fingerprinting (WF) : 각 웹사이트의 트래픽이 고유한 sequence를 가지고 있으며, 분류기가 이러한 sequence를 학습하여 트래픽을 분류하는 공격 기법
  - Tor 사용자 트래픽은 암호화되어있지만, WF 공격은 공격자가 **Tor 사용자가 접근한 웹사이트를 식별**할 수 있도록 활용할 수 있음
    - 공격자가 다양한 웹사이트나 Tor 노드의 트래픽 정보를 수집하여 패킷 통계 / 방향 / 트래픽 burst 패턴 등의 feature를 바탕으로 <u>각 웹사이트에 고유한 지문 (fingerprinting) 을 생성하는 방법</u>
    - 생성된 지문은 Tor 사용자가 접근한 웹사이트를 식별하려는 목표를 가진 머신러닝 분류기 (classifier) 모델의 입력으로 주어짐
    - 딥러닝 분류기의 경우, raw data 또한 활용할 수 있다는 장점이 있음
      - 기본적인 feature들에 대한 사전지식은 부족하지면 raw data를 활용하여 자동으로 feature를 발견할 수 있음
- 이전까지의 WF 공격은 머신러닝을 활용하기 위해 feature engineering을 사용 (공격에 유용한 feature들을 추출하는 기법)
  - 여러 연구에서 CNN 등의 딥러닝 모델을 활용하여 feature engineering을 사용한 WF 공격기법이 주로 연구됨
    - Deep Fingerprinting (DF) : 더 복잡한 CNN 모델을 활용하여 확장된 딥러닝을 활용한 WF 공격기법
    - Tik-Tok : raw data로부터 방향 이외에 **timing 정보**를 활용한 WF 공격기법
  - 요약 : 최근 연구들은 다른 머신러닝/딥러닝 모델을 사용하거나, 새로운 기능을 도입하여 <u>WF 공격에서 높은 정확도를 달성하려는 목적</u>을 가지고 연구가 진행됨
    - 문제점 : **중앙 집중식 프레임워크** // '데이터 저장용량 + 연산 리소스' 측면에서 한 곳에 부하가 몰리는 상황이 발생
      - 실제 상황이라면, 프라이버시 문제도 존재 // Tor 노드들은 익명성을 깨기 위한 공격에 데이터를 제공하려 하지 않을 것
    - 기존 연구들은 Tor 네트워크에서 WF의 정확도를 높이는 데에만 집중됨 -> 위에서 말한 문제들을 해결하려는 노력이 부족
- 이 논문에서는 위 문제점들을 해결하기 위해 **분산 학습 프레임워크**를 도입하여 제한 사항을 극복할 수 있는 접근방식을 탐구
  - Federated Learning (FL) : 원래의 데이터를 공유하지 않고 node에서 머신러닝 모델을 공동으로 훈련하기 위해 제안된 분산 학습 프레임워크
    - 클라이언트는 raw data를 서버와 공유하지 않고, 머신러닝 또는 딥러닝 모델의 parameter만을 서버로 업로드
    - 서버는 global 모델을 업데이트하기 위해 모든 노드들로부터 local 모델의 parameter를 집계 (aggregation)
      - 다음 train round를 위해 업데이트된 global 모델을 노드들에게 분배
      - train round는 특정 정확도가 달성될 때까지 반복하는 것이 일반적
  - FedFingerprinting : 웹사이트를 식별하기 위한 새로운 **Federated Learning 기반 Website Fingerprinting 공격 기법**
    - 공격자 : FL process에 참여하는 Tor 노드의 raw data 수집 없이 WF 공격을 위한 global 모델을 공동으로 생성할 수 있음
    - training 시간을 줄이고 수렴 속도를 향상시키기 위한 추가 알고리즘 제안
      - 가장 유효한 feature를 선택할 수 있는 효율적인 매커니즘 (feature selection)
      - FL process에 참여할 효율적인 노드를 선택할 수 있는 매커니즘 (node selection)
      - 목적 : 더 많은 노드들이 FL process에 참여하도록 동기부여 -> 더 빠른 모델 업그레이드
- Contribution
  - FL 기반 WF 프레임워크 (FedFingerprinting) 제안
    - 프라이버시 문제 + 연산 및 네트워크 병목 문제 해결이 목적
    - FedAvg 방식을 통해 local dataset을 노출하지 않고 local model을 훈련시키고 global 모델을 생성
  - feature selection : 정확도와 손실을 최소화하면서 학습 시간을 줄이기 위한 방법
    - FL 과정에서 선택된 Tor 노드의 local training에 대한 부담을 줄여줄 수 있음
    - 효율적으로 feature space를 줄이기 위해 앙상블 기법 (tree ML의 한 종류) 을 채용하여 feature들의 정확도를 분석
      - WF에 대한 다양한 접근에 사용되는 수작업 feature들의 중요성을 평가
    - 선택된 feature 조합을 활용하여, 각 노드는 raw data가 아닌 **선택된 feature들로 전처리된 새로운 데이터셋**으로 훈련을 진행
  - node selection : FL에서 global 모델의 빠른 수렴을 가능하도록 하는 방법
    - 각 Tor 노드로부터 정확도 정보를 수집하고 정렬
    - 기존의 무작위 선택을 채용하지 않고, local 모델의 성능을 기반으로 더 높은 정확도의 Tor 노드를 선택하는 greedy 정책을 채용
  - 실제 Tor 네트워크의 dataset을 사용하여 closed world에서의 시나리오 실험을 진행
    - FedFingerprinting의 훈련 시간과 수렴 속도 측면에서의 효과에 대한 평가
      - 기존의 중앙집중형 접근 방식과 비교할 수 있는 정도의 정확도 수준을 보임
      - feature selection을 활용한 FedFingerprinting은 정확도 손실을 최소화하면서 FL 참가자의 부담을 덜어줄 수 있음을 보임



## Related Work

### A. Fingerprinting Attack

- Tor 사용자의 온라인 활동을 학습하기 위해 다양한 머신러닝 / 딥러닝 기술이 WF에 사용됨

  - Tor : 트래픽 분석에 취약 -> 사용자의 활동에 대한 정보를 나타낼 수 있기 때문

  - 많은 연구자들은 트래픽 분석을 통해 Tor 네트워크의 **프라이버시를 깨기 위해** 정확한 모델 생성 방법을 연구

    ![table1](https://github.com/Joe2357/Joe2357.github.io/blob/main/assets/img/post/add/2023-07-26-FedFingerprinting/table1.png?raw=true)

##### Use Machine Learning in Fingerprinting

- 이전의 WF 공격 : 수동으로 추출된 feature를 가지는 머신러닝을 활용
  - 공격자 : 분류기 (classifier) 를 교육하기 위한 효과적인 기능을 설계하는 feature engineering 과정을 수행해야 함
    - K-NN 분류기 : 패킷 길이, 패킷 순서, in/out 패킷 수, burst 개수 등의 **광범위한 feature set**를 활용
      - 100개의 웹사이트로 구성된 closed world 환경에서 90% 이상의 정확도 달성
    - SVM 분류기 : 패킷 크기, 패킷 순서 및 패킷의 방향과 같은 간단한 feature를 사용
      - closed world 실험에서 92%의 정확도를 달성
    - K-FP : random forest + K-NN 분류기를 활용하여 <u>feature의 중요성을 분석하고 그에 따른 순위를 매김</u>
      - 가장 중요한 상의 20개의 feature는 패킷 sequence의 패킷 수를 계산하는 것
      - 패킷 순서, arrival time에 기반한 feature들 같은 복잡한 feature들보다 더 효과가 좋았다고 함

##### Use Deep Learning in Fingerprinting

- 딥러닝 : 이미지 인식 및 분류와 같은 많은 연구 분야에서 널리 보급 중인 기술 // 분류 작업에 특히 상당한 성능을 보이고 있음
- WF 분야에서도 많은 연구를 통해 WF 공격에 딥러닝 기반 분류기를 사용하려는 시도가 진행됨
  - SDAE : WF 공격에 사용할 트래픽 분석에 DL을 사용하는 방법 연구
    - 각 발신 패킷을 $1$, 수신 패킷에 $-1$의 시퀀스로 구성된 **패킷의 방향**만을 입력 데이터로 사용
    - 각 시퀀스에 $0$을 추가하거나 패킷을 제거하여 시퀀스의 길이를 5000으로 고정 (입력 벡터의 길이 조정)
    - closed world 실험에서는 88%의 정확도를 달성
  - Rimmer et al. : 자동화된 feature 추출을 위해 다른 딥러닝 모델을 적용하는 것을 제안
    - 제안한 딥러닝 모델을 CNN, LSTM, SDAE과 성능 비교
    - 패킷의 방향만을 사용한 데이터로, 900개의 웹사이트와 2500개의 example이 있는 dataset으로 훈련
    - closed world 실험에서 95 ~ 97%의 정확도를 달성
  - Deep Fingerprinting (DF) : AWF보다 더 복잡한 CNN 구조를 활용한 딥러닝 WF 공격기법
    - 역시 패킷의 방향만을 이용한 데이터셋을 활용하여 학습 진행
    - closed world에서 98%의 정확도를 달성 -> WF 공격들 중 정확도가 높은 수치
    - DF는 Tor, WTF-PAD 등과 같은 <u>WF 공격 방어에 대한 공격 성능</u> 또한 평가
      - WTF-PAD 방어 기술에 대한 공격 정확도는 90%를 달성하였다고 함
- 딥러닝 기반 WF 공격은 머신러닝 기반 WF 공격보다 높은 정확도를 달성할 수 있지만, **정확한 모델 생성을 위해서는 더 많은 양의 데이터가 필요** -> raw data에 대한 교육을 받을 때는 계산이 불필요할 수 있다
  - 최근 DL 모델을 학습하는 부담을 줄이기 위해 DL과 추출된 feature를 결합하려는 시도가 있음
    - Var-CNN : DF 접근법보다 더 복잡한 CNN 구조를 제안
      - raw timestamp를 사용하면 WF 공격이 더 효과적임을 발견
      - closed world 실험에서 92%의 정확도를 달성
    - Tik-Tok : burst 기능 사용 // burst : 연속된 패킷 방향으로 이루어진 sequence
      - 8개의 새로운 burst-level timing feature를 제안 -> fingerprinting 채취에 얼마나 효과적인지를 보임
      - 추출된 feature를 평가 -> closed world에서 84%의 정확도를 달성
      - timing 정보 + 방향 정보를 결합한 **directional timing** 제안 -> 어느 하나를 사용하는 것보다 성능을 향상시킴
- 기존의 딥러닝 기반 WF 방법의 문제점
  - 많은 양의 raw data를 학습하기 때문에 **많은 연산량을 필요**로 함
  - 중앙집중형 방식은 feature가 추출된 경우에도 데이터를 수집하고 전처리 수행하므로 **상당한 양의 데이터 저장이 필요**로 함
  - 공격자에 의한 데이터 수집 과정에서 raw data가 Tor 노드로부터 공격자로 전송되므로 **개인정보 보호 및 네트워크 대역폭의 문제 해결이 필요**로 함
  - => 위 문제를 해결하기 위한 FL 기반 프레임워크가 필요 // 데이터 저장에 대한 부담 완화 + 프라이버시 우려를 완화할 수 있음

### B. Distributed Learning and FL

- 높은 정확도로 모델을 교육하는데 필요한 데이터 양이 많아 <u>높은 계산 효율성 + 분산 학습 구현</u>이 필요
  - 분산학습의 경우, 크고 복잡한 데이터에 대한 학습 오류를 줄이는 데 효과가 있음이 입증됨
  - 다만, 모든 사용자 데이터가 서버에 노출되어 **개인정보 보호 및 보안에 대한 심각한 우려**가 있음

- 구글에서 연합학습 (Federated Learning) 프레임워크 개발
  - 중앙서버는 각 클라이언트의 <u>raw data에 접근하지 않고</u> 학습에 참여하는 클라이언트의 parameter만을 관리하고 업데이트
  - 개별 장치에서 사용할 수 있는 '계산 리소스'를 활용하여 분산된 장치 네트워크 간에 복잡한 교육 작업을 수행할 수 있음
  - => 개인화된 추천 서비스, 전환금융, 의료 데이터, 개인 키보드의 다음 단어 예측 등의 시나리오에 활용될 수 있음
    - 데이터의 특성상 수집, 저장, 분석에 대한 신중한 고려가 필요한 경우에 프라이버시를 해치지 않고 유용하게 사용할 수 있음
- Tor 노드의 경우, raw traffic data에는 시작점, 도착점의 주소와 같은 중요한 정보들이 포함되어 있음
  - FL은 의료 데이터 분석과 같은 응용 프로그램 외에도 **Tor의 WF 공격에도 유용할 수 있음**
  - 다양한 Tor 회로에서 노드를 사용하는 학습 분류 모델은 통신 네트워크 변화에도 강하게 대응할 수 있음
  - 그리고, 이전까지 Tor의 익명성을 공격하는 연구에 FL이 적용되었던 사례는 없었음
    - FL의 적용은 WF 공격을 개선할 수 있는 상당한 잠재력이 있다고 판단됨



## Proposed Fedfingerprinting Framework

### A. Background of WF in Tor Networks

- Tor : 네트워크의 **기밀성**을 보장 (대상 서버의 주소를 숨김)
  - 트래픽 분석을 방지하기 위해 Tor 사용자 대신 <u>암호화된 데이터를 중계하는 분산된 자원봉사 서버</u>로 구성
    - 사용자가 Tor를 사용하여 웹사이트를 탐색하면, Tor는 3개의 중간 노드를 통과하는 회로를 설정 (entry / middle / exit relay node)
    - 네트워크의 패킷은 회로의 각 릴레이를 지날 때마다 암호화
  - 10년 동안 다양한 연구를 통해 암호화된 트래픽 시퀀스를 분석하여 Tor 네트워크 사용자가 접근하는 웹사이트를 식별할 수 있음은 보였음
    - WF : 사용자의 트래픽 흔적을 관찰 -> 웹브라우저 사용자의 행동을 식별하려는 의도
      - 최근 공격 연구는 사용자의 트래픽 흔적을 아무 수정 없이 관측하여 수행되며, 이것들은 인터넷 제공 업체 (ISP) 또는 국가 정부에 의해 수행될 수 있음
      - 공격자가 분석하려는 웹사이트의 하위 집합 : 모니터링된 사이트 // 그 이외의 웹사이트 : 모니터링되지 않은 사이트
        - closed world : 사용자가 모니터링된 사이트들만을 접근한다고 가정된 시나리오 세계
        - open world : 사용자가 모니터링되지 않은 사이트들을 방문할 수도 있는 시나리오 세계

![figure1](https://github.com/Joe2357/Joe2357.github.io/blob/main/assets/img/post/add/2023-07-26-FedFingerprinting/figure1.png?raw=true)

- WF 공격을 수행하기 위해서는 머신러닝 분류 모델을 훈련시킴
  - Tor를 통해 모니터링된 웹사이트를 방문하고, 각 웹사이트의 트래픽 패턴을 수집 (패킷 수, 방향, 시간 정보 등)
  - 이후 모니터링된 웹사이트의 트래픽 정보로 구성된 데이터셋을 사용하여 분류 모델을 훈련
  - 훈련된 모델을 네트워크에 배포하여 사용자가 방문한 웹사이트와 훈련된 모델이 예측한 웹사이트와 비교
    - 훈련된 모델을 평가하기 위해, 공격자는 Tor 사용자가 생성한 트래픽 시퀀스를 수집해야 함

### B. Federated Learning Process for Website Fingerprinting

![figure2](https://github.com/Joe2357/Joe2357.github.io/blob/main/assets/img/post/add/2023-07-26-FedFingerprinting/figure2.png?raw=true)

- FedFingerprinting 프레임워크는 2단계로 구성됨

  - 가정 사항
    - 각 Tor 사용자에 대해 생성된 N개의 Tor 네트워크가 있다고 가정 (TNs)
    - 각 Tor 네트워크는 랜덤 회로를 만들기 위해 Tor 릴레이 노드로 구성됨

  1. **Local Training 요청 단계**
     - 공격자 집계 서버 : Tor 종료 노드 (exit node) 또는 Tor 사용자에게 연합학습에 참여할 것을 요청
       - 위 지점들은 분석하고자 하는 대상 (웹사이트) 에 대한 정보를 알고 있음
     - Tor 종료 노드와 사용자는 연합학습에 참여할 것인지에 대한 승인 여부 결정
       - 이 연구에서는, <u>Tor 네트워크 내에서 악의적인 목적으로 Tor를 사용하는 불법 사용자를 탐지하는 목표</u>를 가진 Tor 종료 노드 또는 사용자가 있다고 가정
  2. **Local Training 수행 단계**
     - 연합학습 요청을 수락하면 FL의 참가자가 되며, 수락된 노드들 (ANs) 들은 local traffic data로 local 분류 모델을 훈련
       - AN : 암호화된 트래픽 데이터만 관찰할 수 있음 // 그 정보들로 **패킷의 시간 정보와 방향 정보를 추론할 수 있음**
       - 공격자 : 집계 서버를 통해 AN에서 업데이트된 local 모델을 수신하고, global 모델을 업데이트
     - 가정
       - WF 공격은 사용자가 파일 다운로드, 여러 브라우저 탭 열기 등 <u>고유한 트래픽 시퀀스와 혼동될만한 다른 활동을 수행하지 않는다</u>

- 이후 사용될 Notation과 정의

  |    Notation    |                             정의                             |
  | :------------: | :----------------------------------------------------------: |
  |  $\mathsf{N}$  |                      Tor 네트워크의 Set                      |
  |       AN       |                   연합학습에 참여하는 노드                   |
  |  $\mathsf{K}$  |                          AN의 집합                           |
  |      $C$       |                $t$번째 라운드의 AN 선택 비율                 |
  |      $w$       |                   global 모델의 parameter                    |
  |     $w_k$      |                AN $k$의 local 모델 parameter                 |
  | $\text{acc}_k$ |                  AN $k$의 local 모델 정확도                  |
  | $\mathsf{D}_k$ |                   AN $k$의 local data set                    |
  |      $r$       | $C$로부터 정확도 순으로 노드를 선택할 top 비율, $0 \leq r \leq 1$ |
  |    $\alpha$    |      기능을 조절하기 위한 상수, $0 \leq \alpha \leq 1$       |

  - $\lvert \mathsf{K} \rvert = K$ : AN의 총 수를 나타냄
    - $k \in \mathsf{K}$일 때, 각각의 AN $k$는 그들의 local traffic data $\mathsf{D}_k = ((x_k^1,y_k^1),(x_k^2,y_k^2),(x_k^3,y_k^3), \cdots , (x_k^i,y_k^i) , \cdots , (x_k^{D_k},y_k^{D_k}))$를 가지고 있고, $\lvert \mathsf{D}_k \rvert = D_k$는 트래픽 시퀀스의 총 개수임
    - $x_k^i, y_k^i$는 각각 사용자가 접근한 $i$번째 트래픽 시퀀스와 그에 대응하는 ground-truth 값임

- 훈련 단계는 아래 단계로 이루어짐

  1. Initialization

     - FL 작업이 주어지면 공격자의 집계 서버는 hyperparameter (학습 속도, 배치 크기 등) 를 초기화
     - global model을 초기화하고 그 parameter $w$를 ANs들에게 전달

  2. Local model training and update

     - 각 라운드에서 각 AN $k$는 local traffic data로 모델을 교육하며 본인의 파라미터 $w_k$를 이용하여 loss function인 $F_k(w_k)$를 최소화하려 한다  

       
       $$
       \begin{equation*} F_{k}(w_{k}) = \frac {1}{D_{k}}\sum _{i=1}^{D_{k}}f(w_{k};x_{k}^{i}, y_{k}^{i}). \tag{1}\end{equation*}
       $$

     - $F_k$는 일반적인 FL에 대한 다양한 loss function으로 고려될 수 있음

  3. Aggregation

     - 집계 서버는 AN의 local model parameter를 집계하고 global model을 업데이트

     - 이 논문에서는 FedAvg 방식을 채택하여 집계 단계에서 활용  

       
       $$
       \begin{equation*} \underset {w}{min} \left \{ F(w) = \frac {D_{k}}{D}\sum _{k=1}^{K}F_{k}(w_{k}) \right \} \tag{2} \end{equation*}
       $$

       - $D$ : AN을 통한 총 트래픽 데이터 양

     - 서버는 업데이트된 global 모델 파라미터 $w$를 다시 AN으로 전달

  4. Iteration

     - 각 AN에 대한 분산 로컬 AN 훈련 및 모델 파라미터 집계 process는 global model이 원하는 정확도로 수렴하거나 일정 횟수의 반복에 도달할 때까지 반복

- FedFingerprinting : 집계 서버는 다양한 Tor 네트워크에서 다양한 조건으로, 서로 다른 AN에 의해 생성된 특정 웹사이트 트래픽 시퀀스에 대해 AN의 모델들을 집계하여 global 모델을 훈련시킬 수 있음

  - 회로 변경으로 인해 문제될 수 있는 네트워크 변동성에 강인함
  - 트래픽 데이터를 공유하는 대신 AN들이 hyperparameter 공유 -> 프라이버시가 노출되지 않음

- `알고리즘 1` : FedFingerprinting의 절차

  ```pseudocode
  Algorithm 1 FedFingerprinting
  
   1| Initialize the global model and parameter w
   2| Initialize K ← ∅ & accuracy buffer A
   3| The attacker aggregation server request FL participating to the Tor exit nodes or user
   4| if Nodes accept a FL participating request then
   5|   Add nodes to K
   6| end if
   7| if achieving lightweight = True then
   8|   Fpre ← preprocess to extract features
   9|   Set required threshold θ , α
  10|   Fselect ← Feature selection(Fpre , θ , α )
  11| end if
  12| for each round t = 1, 2, …, T do
  13|   if t = 1 then
  14|     Add nodes to K
  15|   else
  16|     K ← AN selection(C , r , A)
  17|   end if
  18|   AN ∈ K download parameter w
  19|   AN train local model in parallel
  20|     Fk(w) ← 1/Dk * ∑Dk,i=1 f(w;xik,yik)
  21|     Local update wk ← wk − η▽Fk(w)
  22|   ANs upload local parameter wk and acck to the attacker server
  23|   The attacker server aggregates all local parameters and updates the global model
  24|     w ← 1/D * ∑K,k=1 Dkwk
  25|   The attacker server stores acck into A
  26| end for
  27| return w
  ```

  - 초기화 단계 (Line 1 ~ 2) : 서버는 global model, AN set 및 정확도 buffer A를 초기화
  - 공격자 : 악의적인 목적으로 Tor를 사용하는 사용자를 탐지하는 데 도움이 되는 Tor exit node 또는 Tor user에게 FedFingerprinting에 참여하라는 요청을 전송
    - 요청을 받은 노드들은 승인 여부를 결정하고 승인한 노드들은 AN이 됨 (Line 4 ~ 6)
  - 각 AN은 자신의 트래픽 데이터셋으로 local model을 훈련시키고 local 파라미터 $w_k$및 정확도 $\text{acc}_k$를 집계 서버로 업로드 (Line 18 ~ 22)
    - 집계 서버 : 각 AN의 모든 local parameter를 집계하여 FedAvg 집계 방식을 통해 새로운 global model 업데이트 (Line 23 ~ 25)
    - AN의 훈련을 경량화시키기 위해 **feature selection**이 추가적으로 적용될 수 있음 (Line 7 ~ 11)
    - FL process의 수렴을 신속하게 하기 위해 **AN selection**을 활용 (Line 13 ~ 17)

### C. Feature Engineering

- 목적 : AN이 local training을 하는데 부담을 느낄 수 있음

  - FedFingerprinting : raw data를 사용하여 훈련하지 않고 입력 데이터의 dimension을 줄여 학습 부담을 줄이기 위해 WF 공격에 사용되는 **수작업 feature를 얻기 위한** <u>새로운 feature engineering 방안</u>을 제안

  - 암호화된 Tor 트래픽에서 raw data를 받고, 먼저 추출된 '패킷의 방향 / 시간 정보'를 얻고, 그것을 기반으로 다양한 feature 유형에서 **총 89개의 feature**를 추가로 추출

    |      Feature Type      | 개수 |
    | :--------------------: | :--: |
    |    패킷의 기본 정보    |  6   |
    | 패킷 시간간격 통계정보 |  49  |
    |      버스트 정보       |  16  |
    |  window 알고리즘 사용  |  18  |
    |          합계          |  89  |

  - 패킷 기본 정보 (packet general information) : 일반적인 트래픽으로부터 추출할 수 있는 정보

    - 패킷 수, 송수신 패킷 비율, 로딩 시간 등

  - 패킷 시간간격 통계정보 (packet time interval statistics) : 패킷 간의 시간 간격에 대한 통계 정보

    - 최댓값, 최솟값, 평균, 표준편차, 백분위 수 (0.25, 0.5, 0.75)
    - 처음 5초 또는 마지막 5초에 수집된 패킷에 대한 시간 지연 정보를 추가로 사용

  - 버스트 정보 (burst statistics) : 같은 방향으로 나가는 연속된 패킷 그룹을 의미함

    - $+1$ : 사용자 -> Tor 네트워크로 나가는 방향
    - $-1$ : Tor 네트워크 -> 사용자로 들어오는 방향
    - 집중된 트래픽의 패턴을 포착하고, burst 내의 패킷 수를 활용하여 통계량을 계산

  - window 알고리즘 사용 (using window) : 특정 기간에 발생한 트래픽 패턴을 관찰

    - 시간 정보 window : 1초 window / 5초 window 등
    - 길이 window : 초기 30개 패킷 window 등
    - 트래픽 시퀀스의 시작 부분에 window를 두는 것은 **각 웹사이트에 대한 트래픽 패턴을 식별**하기 위해 트래픽 시작 부분을 주의깊게 보겠다는 의미

- 논문에서 제안한 feature들의 정밀도는 **다양한 머신러닝 앙상블 알고리즘을 활용해 평가**하고, 전체 raw data를 사용하는 대신 **학습 시간과 정확도의 균형을 맞추기 위해** feature의 하위 집합을 효율적으로 선택하도록 feature들의 중요도를 검토

  - Wang 데이터셋 활용 : Tor 네트워크 웹사이트 트래픽

    - 총 100개의 label이 있고, 각 레이블당 90개의 instance로 구성됨

  - 추출된 feature를 이용하여 4개의 앙상블 머신러닝 알고리즘을 사용하여 웹사이트를 분류하는 실험 진행

    - 각 머신러닝 모델의 하이퍼파라미터는 GridSearch 방식을 통해 찾고, 정확도 평가는 10-way cross-validation을 사용

      |               | n_estimation | max_depth | random_state | max_features |
      | :-----------: | :----------: | :-------: | :----------: | :----------: |
      | Decision Tree |      -       |    15     |      42      |     0.5      |
      | Random Forest |     110      |    18     |      42      |     0.5      |
      |  Extra Tree   |     350      |    18     |      42      |     0.5      |
      |    XGBoost    |     500      |     3     |      -       |      -       |

    - 결과

      |    ML 모델    | 정확도 |
      | :-----------: | :----: |
      | Decision Tree | 0.754  |
      | Random Forest | 0.864  |
      |  Extra Tree   | 0.892  |
      |    XGBoost    | 0.888  |

      - 평가에 사용한 앙상블 모델들 중에서는 Extra Tree 방법이 정확도가 가장 높음 // 이전 연구들에서 달성한 정확도에 필적하는 정도

        - 가장 높은 정확도를 보인 Extra Tree 모델을 기준으로 `feature_importances_`를 측정할 것

          - purity에 따라 분할되는 tree 기반 모델에서, **특정 feature가 트리 분할에 얼마나 기여하는가?**

          | rank |                 feature 정보                 | rank |                feature 정보                 |
          | :--: | :------------------------------------------: | :--: | :-----------------------------------------: |
          |  1   |                 수신 패킷 수                 |  13  |        송신 버스트의 패킷 수의 평균         |
          |  2   |                 전체 패킷 수                 |  14  |              송신 버스트 개수               |
          |  3   |                 송신 패킷 수                 |  15  |              수신 버스트 개수               |
          |  4   |         전체 패킷 중 송신 패킷 비율          |  16  | 초기 5초 동안의 수신 버스트에서의 패킷 개수 |
          |  5   |         전체 패킷 중 수신 패킷 비율          |  17  | 초기 5초 동안의 송신 버스트에서의 패킷 개수 |
          |  6   | 전체 수신 패킷 중 초기 30개의 수신 패킷 비율 |  18  |        수신 패킷 1초마다의 평균 개수        |
          |  7   |   전체 패킷 중 초기 30개의 수신 패킷 비율    |  19  |   전체 패킷 중 초기 5개의 수신 패킷 비율    |
          |  8   |        송신 버스트 중 최대 패킷 개수         |  20  |   전체 패킷 중 초기 5개의 송신 패킷 비율    |
          |  9   |   전체 패킷 중 초기 30개의 송신 패킷 비율    |  21  |       웹사이트 로딩 시간 (전체 시간)        |
          |  10  | 전체 송신 패킷 중 초기 30개의 송신 패킷 비율 |  22  | 전체 수신 패킷 중 초기 5개의 수신 패킷 비율 |
          |  11  |         마지막 5초 동안의 패킷 개수          |  23  |   마지막 5초 동안의 패킷의 시간 간격 평균   |
          |  12  |     송신 버스트 중 패킷 개수의 표준편차      |  24  | 전체 송신 패킷 중 초기 5개의 송신 패킷 비울 |

      - feature들의 순위에 따라 상위 6개, 12개, 24개를 사용하였을 때의 정확도 및 학습시간에 대한 실험 진행

        | 입력 데이터 | 훈련 시간(초)  |     정확도     |
        | :---------: | :------------: | :------------: |
        |  상위 6개   | 0.583 +- 0.006 | 0.745 +- 0.031 |
        |  상위 12개  | 0.641 +- 0.019 | 0.841 +- 0.024 |
        |  상위 24개  | 0.753 +- 0.003 | 0.874 +- 0.024 |
        |  전체 89개  | 1.646 +- 0.013 | 0.884 +- 0.023 |

        - **상위 24개의 feature**를 사용하는 경우 <u>높은 정확도를 유지하면서 훈련 시간을 줄일 수 있다</u>
          - 추출된 feature들 중 핵심 feature들을 식별하는 것이 정확도를 크게 희생시키지 않고 효율성을 향상시킬 수 있다!
          - 주요 feature들을 효율적으로 선택하면 기존의 입력 크기가 5000패킷이었던 이전의 연구들보다 **훨씬 더 작은 입력 데이터 크기**를 사용하면서도 정확도를 유지할 수 있다

### D. Proposed Feature Selection for FedFingerprinting

- 훈련 시간과 정확도의 균형을 맞추기 위해, 효율적인 feature 선택 알고리즘을 제안

  - `알고리즘 2` : FedFingerprinting에 적용할 수 있는 효율적인 feature 선택 방법

    ```pseudocode
    Algorithm 2 Feature selection(Fpre, θ, α)
    Require: Preprocessed features Fpre, Required threshold θ, Ratio of training time and accuracy α
    
     1| Fimp ← sort Fpre by rank using feature_importances_
     2| Initialize Fselect ← ∅
     3| Fselect ← top 1 feature in Fimp
     4| while do
     5|   Calculate accuracy and training time using Fselect:
     6|     θselect ← α(1 / training_time) + (1 − α)accuracy
     7|   if θ > θselect then
     8|     Add next rank feature from Fimp to Fselect
     9|   else
    10|     return Fselect
    11|   end if
    12| end while
    ```

    - 모델 교육 시간 및 웹사이트 식별 정확성을 고려하며, 출력으로 **선택할 상위 feature의 수**를 반환 (feature의 집합)
      - 모델 훈련에 사용될 전처리된 feature $F_{pre}$를 중요도순으로 정렬하여 $F_{imp}$로 저장
      - 아래 과정 반복
        - 선택되지 않은 feature들 중 첫 번째 feature를 $F_{select}$에 추가 (Line 1 ~ 3)
        - $F_{select}$에 있는 feature들로 정확도 및 학습 시간을 계산하여 $\theta_{select}$ 계산 (Line 5 ~ 6)
        - $\theta_{select}$가 임계값인 $\theta$의 기준을 넘기는지 확인
          - 넘긴다면 알고리즘 종료. 선택한 feature들 $F_{select}$를 사용하는 것이 최적일 것이라고 판단 후 반환
          - 넘기지 않는다면 원하는 결과를 얻지 못함. $F_{select}$에 feature를 하나 더 추가 (Line 7 ~ 8)

### E. Proposed AN Selection for FedFingerprinting

- FL 프레임워크에서 수렴 속도를 향상시키기 위해 추가적인 AN 선택 방안을 제안

  - 특정 비율의 AN이 무작위로 선택되는 FedAvg 대신 **local 모델 정확도에 따라 AN을 선택**하는 것이 목표
    - raw data를 활용하여 AN을 선택하지 않으므로 FL에 적합한 AN 선택 방법일 수 있음
  - AN 선택 과정은 3단계로 구성
    - FedFingerprinting : 제안된 AN 선택 단계에서 공격자가 각 AN의 파라미터를 수신할 때, **모델의 정확도도 함께 수신**
    - 공격자 : 정확도 순으로 정렬한 후, $t$ 라운드에 참여할 AN들 중 $r$ 비율만큼은 <u>정확도가 높은 AN</u>을 선택
    - 이후 남은 비율 $1-r$만큼은 랜덤 선택
      - 정확도가 낮은 AN을 선택하는 이유 : global 모델에 중요한 정보를 제공할 수도 있는 노드들이 있기 때문에, 제외하지 않고 학습 프로세스에 참여할 수 있도록 하기 위함
  - 목적 : FL 프로세스의 수렴 속도를 향상시키고, 모든 AN이 학습 프로세스에 참여하도록 하는 것

- `알고리즘 3` : 제안된 AN 선택 알고리즘

  ```pseudocode
  Algorithm 3 AN selection(C, r, A)
  Require: selection ratio C, top selection ratio r
  
   1| Sort A by accuracy
   2| Select ANs up to C × r
   3| Randomly select C × (1 − r) ANs from after C × r rank
   4| return set of selected ANs
  ```

  ![figure5](https://github.com/Joe2357/Joe2357.github.io/blob/main/assets/img/post/add/2023-07-26-FedFingerprinting/figure5.png?raw=true)



## Performance Evaluation

### A. Data Sets and Experiment Settings

- 실제 Tor 데이터셋 (DF 데이터셋) 을 사용하여 실험을 수행할 것

  - DF 데이터셋 : 2016 Alexa Top 사이트에서 수집한 모니터링된 사이트들로 구성
    - 95개의 label, 각 label당 1000개의 instance // 총 95000개의 샘플

- 실험을 위해 FL에서 사용하는 기본 딥러닝 모델은 DF 모델을 사용

  - DF 모델 : 사용하는 데이터 집합에 따라 hyperparameter 값이 다름

    |    입력 데이터     |                     하이퍼파라미터 : 값                      |
    | :----------------: | :----------------------------------------------------------: |
    |      raw data      | 입력 차원 : 5000<br />[Filter, Pool, Stride] 크기 : [8, 8, 4] |
    | feature extraction | 입력 차원 : 89<br />[Filter, Pool, Stride] 크기 : [4, 3, 2]  |
    | feature selection  | 입력 차원 : 24<br />[Filter, Pool, Stride] 크기 : [3, 2, 2]  |

    - DF 모델은 일반적으로 8개의 convolution Layer, 3개의 dense Layer로 구성됨
      - dense Layer는 softmax regression을 통해 각 class의 확률을 제공하는 분류 layer의 역할을 수행
      - convolution 및 dense layer를 정규화하는 것은 batch normalization 방식 사용
      - 2개의 convolution layer를 지날 때마다 max pooling과 0.1의 dropout이 진행됨
      - 처음 2개의 dense layer는 각각 0.7과 0.5의 dropout이 진행됨
      - ReLU 활성화 함수가 대부분의 layer에 사용되지만, 수많은 음수 값이 포함된 **방향 정보** 때문에 처음 2개의 convolution layer에는 ELU가 대신 사용됨

### B. Performance of Feature Engineering in DL

- feature extraction 과정의 효율성을 평가하기 위해 raw data와의 비교실험 진행

  - 서로 다른 feature 선택 설정 하에서 정확도를 계산 -> 훈련 시간과 정확도 사이의 tradeoff를 측정
  - raw data / 추출된 feature들 (89) / 알고리즘 2로부터 얻은 feature들 (24) 을 모델 학습을 위한 입력 데이터로 사용하였을 때, **각 epoch당 평균 학습 시간과 정확도**를 기록
    - 각 입력 데이터로 100개의 epoch 훈련 진행 

- 알고리즘 2에 명시된 feature selection 방법을 진행하기 위해서는 알고리즘 수행 전에 raw data에 대한 **데이터셋 전처리**가 필요

  - 필요한 시간 : 약 120초 (2min) // sunk cost : 모델 교육 전에 1번만 발생하는 침몰 비용

  - 데이터 전처리가 완료된다면 feature extraction과 selection을 진행하는 시간은 무시할 수 있는 수준으로 작아짐

    |       입력 데이터       | 정확도 (%) | epoch당 평균 시간 (s) |
    | :---------------------: | :--------: | :-------------------: |
    |        raw data         |   96.500   |        44.850         |
    | feature extraction (89) |   82.784   | 7.295 + 1.191 = 8.486 |
    | feature selection (24)  |   75.237   | 7.023 + 1.191 = 8.214 |

    - raw data를 사용한다면 가장 높은 정확도를 달성할 수 있지만, **너무 많은 시간을 소요** -> FL에 참여하는 AN에게 부담을 줄 수 있음
    - feature extraction / selection을 사용하면 epoch당 시간을 최대 18.3%까지 줄일 수 있음 -> AN의 분류 모델 학습 부담을 덜어줌

### C. Comparison of Centralized Learning and Fedfingerprinting

- 테스트 정확도를 유지하면서 cost를 절감하는 효과를 입증하기 위해 IID data를 가진 AN들을 이용한 연합학습 실험 진행

  - FedFingerprinting을 중앙집중 방식과 비교하여 각 라운드에서의 테스트 정확도를 측정하고 수렴에 도달할 때 최대 테스트 정확도를 측정 -> **제안된 프레임워크의 효율성 평가**

  - 실험 환경 세팅

    - $K = 1, 10, 30, 50$
      - $K=1$이라면 중앙 집중식 실험 (FL이 아님)
      - $K=10$이라면 각 AN은 class당 100개의 instance를 가짐
    - $C=1$, $\text{local epoch} = 1$, FedAvg 집계방식 사용

  - 결과 : FedFingerprinting은 중앙집중식 접근방식과 비교할만한 수준의 테스트 정확도를 달성하며 **정확도를 보장**하면서 **FL의 장점을 가진** 알고리즘이다

    ![figure6](https://github.com/Joe2357/Joe2357.github.io/blob/main/assets/img/post/add/2023-07-26-FedFingerprinting/figure6.png?raw=true)

    - 위 그래프 (b)에서, $K$가 커질수록 정확도가 크게 감소하는 것을 볼 수 있음
      - 이유 : 각 AN이 훈련할 수 있는 데이터 샘플 개수가 적기 때문에, feature engineering의 효율성이 제한됨
      - 해결법 : 훈련 데이터 샘플을 더 늘리면 해결할 수 있음
    - pkl 형식 모델과 DF dataset에 필요한 저장공간은 각각 11.72MB, 4.04GB임
      - 공격자의 입장에서, 필요한 저장 공간에 대한 **공격 비용 부담을 크게 줄일 수 있음**을 보임
      - 또한, DF dataset의 경우 훈련에 필요한 모든 데이터를 수집하는데 시간이 많이 걸리고 비용이 비쌈을 의미
    
  - 수렴을 달성할 때의 정확도와 시간
  
    |          입력 데이터           | 라운드 수 | 정확도 (%) | 시간 (s)  |
    | :----------------------------: | :-------: | :--------: | :-------: |
    | without feature selection (89) |    230    |  73.7689   | 1090.9598 |
    |  with feature selection (24)   |    230    |  64.5075   | 871.6447  |
  
    ![figure7](https://github.com/Joe2357/Joe2357.github.io/blob/main/assets/img/post/add/2023-07-26-FedFingerprinting/figure7.png?raw=true)
  
    - feature selection 알고리즘 실행 유무에 관계 없이 230 라운드에 수렴이 진행됨
    - feature selection 알고리즘을 수행하면 정확도가 9.2% 감소 / 훈련 시간은 20.1% 단축
    - => feature selection을 사용하면 **overhead가 적은 모델을 만들 수 있음**
  
- AN selection 알고리즘에 대한 성능평가 진행

  - 수렴을 달성하기 위해 필요한 round 수와 round에 대한 테스트 정확도 비교

  - $K = 100, C = 0.3$으로 위와 같은 FL 실험을 진행

    - 각 AN은 class당 10개의 instance 보유
    - global model의 목표 수렴 : 85%

  - $r$의 값을 다르게 하며 실험한 결과를 아래와 같이 기록

    ![figure8](https://github.com/Joe2357/Joe2357.github.io/blob/main/assets/img/post/add/2023-07-26-FedFingerprinting/figure8.png?raw=true)

    - 결과 : AN selection 알고리즘에서 요구되는 round 수가 랜덤 선택 알고리즘에서 요구되는 양보다 적음
      - $r=0.5$일 때 랜덤 선택($r=0$) 일 때보다 19% 빠르게 수렴함을 보임 -> **목표 정확도를 빠르게 달성하며 일반화를 향상시킬 수 있음**
      - AN selection 알고리즘의 테스트 정확도 또한 상당한 이점을 보임
      - $r$이 너무 높으면 선택받지 못한 다른 AN이 학습에 참여하지 않게 되기 때문에 모델의 generalization에 부정적인 영향을 미칠 수 있음



## Future Work and Discussion

### A. Feature Selection

- 기존 연구에서 공통적으로 사용되던 feature를 기반으로 핵심 feature 조합을 추출하고 **FL에 참여하는 노드들의 연산 부하를 줄이는 것**이 초점
  - feature set 전체에 걸쳐 훈련 시간과 정확도 사이의 균형을 이루는 상위 feature들을 선택, <u>경량 feature 선택</u>에 집중
    - 주요 feature 수를 5000개 -> 89개로 제한 => 무차별 대입 검색 방식으로 feature selection을 수행하기 위한 계산 load를 무시할 수 있도록 함
  - 추출된 feature들 중 적응적으로 select하는 방법들은 더 정확한 feature selection을 가능하게 하지만, 더 많은 계산 부하를 요구함
    - 계산 부하가 높다면 FL에 참여하는 노드들에게 추가적인 training load를 부과하게 됨
    - 계산 자원 측면에서 tradeoff 고려사항으로 인해, 처음부터 경량 feature 선택을 유지하며 **FL을 Tor에 적용하는 접근 방식**을 위한 <u>무차별 대입 검색 방식</u> 탐구
      - 향후 연구에서, 계산 overhead를 최소화하며 적응적인 feature selection을 효과적으로 도입하는 방향으로 확장시킬 수 있을 것

### B. ANS With Non-IID Data Distribution

- global model의 품질은 local model의 데이터 분포에 따라 달라짐
  - ex. 하나의 AN에 하나의 도메인에 대한 트래픽만 있다면, local model이 만족스럽게 학습할 수 없으므로 global model의 성능 또한 저하
  - 향후 연구에서, Tor 시나리오에서도 Non-IID 문제를 해결하려는 노력을 할 수도 있을 것

### C. Malicious ANs

- FL에 참여하는 노드들은 **익명성을 유지하기 위해** 공격자의 aggregation을 방해할 수도 있음
  - 악의적인 노드 : global model의 정확도를 낮추기 위해 열악한 local model을 제공할 수도 있음
  - 향후 연구에서, 모델 성향이 너무 낮은 노드를 배제하기 위해 다양한 연구를 수행하거나, 악의적인 AN의 영향을 완화시키는 기법을 찾을 수도 있을 것



## Conclusion

- Tor 네트워크에서 Website Fingerprinting을 위한 Federated Learning을 사용한 "FedFingerprinting"이라는 새로운 프레임워크 제안
  - Tor 노드에 대한 local train 부담을 줄이고, 경량화를 위해 WF에 사용되는 다양한 feature 수작업 과정의 중요성을 평가 / feature selection 방법 도입
  - FL의 공정성을 위해 효과적인 Tor node selection 방법 설계
  - 실제 Tor 데이터셋을 사용하여 FedFingerprinting의 성능 평가
- 제안한 프레임워크는 Tor 노드의 **프라이버시를 보호**하면서 **목표 정확도에 도달**하기 위한 **훈련 시간과 수렴에 필요한 라운드 수를 줄임**

