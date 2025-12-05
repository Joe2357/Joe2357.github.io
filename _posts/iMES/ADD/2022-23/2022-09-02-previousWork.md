---
title: "익명 오버레이 네트워크 공격 관련 선행연구"
author: Joe2357
categories: [iMES, Agency for Defense Development (ADD) / 2022-23]
tags: [Agency for Defense Development (ADD) / 2022-23]
uploaded_at: 2022-09-02 12:00:00 +0900
last_modified_at: 2022-09-02 12:00:00 +0900
description: "- 선행연구 분석 및 체계 정리"
math: true
---



## 서론

최근 사이버 환경은 다크웹 등을 통한 사이버 범죄 수단 및 불법적인 서비스를 제공하는 환경이 늘어나고 있다. 사이버 공격자는 공격 원점을 숨기고 다크웹 및 암호화폐 거래를 통해 사이버 범죄 도구를 확보하기 위해 Tor, I2P, VPN 네트워크 등의 익명 오버레이 네트워크를 다양하게 활용하고 있다. 이는 노드간 암호화 통신을 이용하고, 중간에 다양한 릴레이 노드를 활용하면서 통신 경로를 동적으로 운영하므로, 역추적 및 대응을 어렵게 만든다.

Tor, I2P 등 오버레이 네트워크를 통해 이루어지는 경로를 추적하여 공격 원점 식별 및 공격자 추적 정보를 획득할 필요성이 생겼다. 따라서 오버레이 네트워크를 역추적하기 위한 지표 및 데이터 수집 / 분석 기법 연구가 사이버공격에 대한 방어 또는 공격자 식별을 위한 필요 기술이 되었다.



## Tor 네트워크 공격 분류

### Intersection Attack

active한 source set이 제한적이라고 가정하고 목적지에 접속 패턴 (packet flow 수 등) 의 정보를 분석하여 source set과 대조하여 식별하는 공격 방법이다.

단점으로는 Tor 네트워크가 10분마다 circuit이 재구성되며, active한 source set이 굉장히 크기 때문에 식별 과정이 쉽지 않다.



### Multiplication Attack

entry와 exit 릴레이 노드를 확보했다고 가정하고, 패킷에 특정 변형을 가해 식별에 활용할 수 있도록 하는 공격 방법이다.

단점으로는 확보된 랜덤 노드들 중 entry와 exit 노드를 확보활 확률이 높지 않다는 점이다.



### Timing Attack

출발지와 목적지의 communication이 의심된다는 것을 안다고 가정하고, 두 노드의 경로 사이에 특정 마커를 주입하여 flow pattern을 통해 communication의 정황을 파악하는 공격 방법이다.

단점으로는 출발지와 목적지의 정보가 확보된 이후에 정황 파악을 시작할 수 있다는 것이다.



### Fingerprinting Attack

출발지의 entry를 점령하였다고 가정하고, 이를 통해 출발지 노드의 트래픽 패턴을 수집할 수 있다고 할 때, 트래픽 패턴을 이용하여 식별에 활용하는 공격 방법이다.

단점으로는 entry 노드를 확보해야한다는 점이다.




## 선행연구 조사

> 공격 분류들 중 Intersection Attack, Fingerprinting Attack만을 조사하여 정리하였음



### Fingerprinting Attack on Tor Anonymity using Deep Learning

> Abe, Kotaro and Shigeki Goto. “Fingerprinting Attack on Tor Anonymity using Deep Learning.” (2016).

#### 아이디어

- 클라이언트와 entry 노드를 볼 수 있다는 가정

- 불법 웹사이트에 접속하는 사용자를 탐지하기 위한 방법으로 SDAE(Stacked Denoising Autoencoder) 딥러닝 방법 사용

- 두 개의 SDAE 레이어와 출력 레이어는 softmax로 구현

- 입력벡터가 1,-1 (셀의 방향) 만 사용클라이언트에서 토르 노드로 보내지면 1로 표현, 토르 노드에서 클라이언트로 셀이 보내지면 -1로 표현

  ![fig 1](https://github.com/Joe2357/Joe2357.github.io/blob/main/assets/img/post/add/2022-09-02-previousWork/fig1.png?raw=true)

  ![fig 2](https://github.com/Joe2357/Joe2357.github.io/blob/main/assets/img/post/add/2022-09-02-previousWork/fig2.png?raw=true)

- AE (autoencoder) : 입력과 출력을 같게 weight를 조정테스트 시 클라이언트 트래픽을 입력으로 넣으면 비슷한 웹사이트 트래픽이 출력됨

#### 실험방법
- 해당 사이트의 데이터셋을 사용 (http://home.cse.ust.hk/~taow/wf/)

- 데이터셋

  ![fig 3](https://github.com/Joe2357/Joe2357.github.io/blob/main/assets/img/post/add/2022-09-02-previousWork/fig3.png?raw=true)

  - **인풋으로 셀의 방향만을 사용**

#### 성능사항

![fig 4](https://github.com/Joe2357/Joe2357.github.io/blob/main/assets/img/post/add/2022-09-02-previousWork/fig4.png?raw=true)

- closed world : 모니터링하는 웹사이트들만 접속한 경우
- open world : 모니터링하지 않는 웹사이트들 포함하여 접속한 경우
- TPR (모니터링하는 웹사이트를 정확하게 분류) 과 FPR(모니터링되지 않는 사이트를 모니터링되는 사이트로 분류류) 을 지표로 사용
- feature를 수동으로 선택하지 않고도 이전 방법과 정확도 차이가 거의 없다는 장점

#### 한계점

- 다른 딥러닝 방법을 사용할 수 있음. (CNN, RNN, LSTM 등)
- SDAE 의 output 자체를 분석에 사용할 수 있는 feature 로 사용 가능



### Automated Website Fingerprinting through Deep Learning

> Rimmer, Vera & Preuveneers, Davy & Juarez, Marc & Van Goethem, Tom & Joosen, Wouter. (2018). Automated Website Fingerprinting through Deep Learning. 10.14722/ndss.2018.23115.

#### 아이디어

- 공격자가 feature engineering process를 자동화하고 Tor 트래픽을 자동으로 반익명화(deanonymize) 시키는 방식으로 딥러닝 방법 적용
  - SDAE, CNN, LSTM의 3가지 모델을 설계, 튜닝, 평가 과정을 거침
  - keras를 이용하여 구현 (https://github.com/DistriNet/DLWF)
- Website Fingerprinting(WF) 문제를 classification 문제로 공식화hyperparameter 선택 방식으로 semi-automatic hyperparameter tuning을 사용 ( 매개변수 영향을 고려하는 방식? )![fig5](https://github.com/Joe2357/Joe2357.github.io/blob/main/assets/img/post/add/2022-09-02-previousWork/fig5.png?raw=true) 

#### 실험방법

- 900개의 website에서 2500 페이지 방문 트래픽 추적 데이터셋을 사용
  - 자체 제작한 것으로 추정, 공개는 되지 않았지만 요청시 사용할 수 있도록 한다고 되어있음 (https://distrinet.cs.kuleuven.be/software/tor-wf-dl/)
- closed world, open world에 대한 실험 진행

#### 성능사항

![fig 6](https://github.com/Joe2357/Joe2357.github.io/blob/main/assets/img/post/add/2022-09-02-previousWork/fig6.png?raw=true)![fig 7](https://github.com/Joe2357/Joe2357.github.io/blob/main/assets/img/post/add/2022-09-02-previousWork/fig7.png?raw=true)![fig 8](https://github.com/Joe2357/Joe2357.github.io/blob/main/assets/img/post/add/2022-09-02-previousWork/fig8.png?raw=true)

- 정확도와 model loss, 그리고 모델 런타임 시간을 지표로 사용
- concept drift ( 시간이 지남에 따라 모델링 대상의 통계적 특성이 바뀌는 현상 ) 에 대한 추가 성능평가 존재

#### 한계점

- 홈페이지 방문에 대한 공격만 분석 / 고려된 웹사이트의 다른 페이지에 대한 내용은 생략 (비현실적인 가정)
- open world와 closed world에서 사이트 방문 확률을 근사시키려 하지 않았음 (동일한 확률을 가진다고 가정하므로, 현실을 반영하지 못함)



### DeepCorr: Strong Flow Correlation Attacks on Tor Using Deep Learning

> Milad Nasr, Alireza Bahramali, and Amir Houmansadr. 2018. DeepCorr: Strong Flow Correlation Attacks on Tor Using Deep Learning. In Proceedings of the 2018 ACM SIGSAC Conference on Computer and Communications Security (CCS '18). Association for Computing Machinery, New York, NY, USA, 1962–1976.

#### 아이디어

![fig 9](https://github.com/Joe2357/Joe2357.github.io/blob/main/assets/img/post/add/2022-09-02-previousWork/fig9.png?raw=true)

- 위 사진은 correlation attack 의 대표도

- 딥러닝 (CNN)을 사용

  ![fig 10](https://github.com/Joe2357/Joe2357.github.io/blob/main/assets/img/post/add/2022-09-02-previousWork/fig10.png?raw=true)![fig 11](https://github.com/Joe2357/Joe2357.github.io/blob/main/assets/img/post/add/2022-09-02-previousWork/fig11.png?raw=true)

  - T는 흐름 i의 패킷 간 지연(inter-packet delays)(IPD) 벡터, S는 i번째 패킷 크기의 벡터, u 및 d 위첨자는 양방향 흐름 i의 "업스트림" 및 "다운스트림"
    - ( 예를 들어, Tu i는 i의 업스트림 IPD의 벡터.)(Fi , Fj) 한 쌍의 연결된 흐름 (associated flows)

- 목표는 모든 수신 및 송신 흐름에서 패킷 타이밍 및 크기와 같은 트래픽 특성을 비교하여 연결된 흐름 쌍(예: (Fi , Fj))을 식별

  ![fig 12](https://github.com/Joe2357/Joe2357.github.io/blob/main/assets/img/post/add/2022-09-02-previousWork/fig12.png?raw=true)

- output은 i 와 j 가 연관될 확률을 나타냄

  - e.g., being the entry and exit segments of the same Tor connection.
  - DeepCorr declares the flows i and j to be correlated if pi,j > η, where η is our detection threshold discussed during the experiments. 

#### 실험방법 및 성능사항

- 각각의 Tor 클라이언트를 사용하여 Tor를 통해 상위 50,000개의 Alexa 웹사이트를 탐색하고, Tor 네트워크에 들어오고 나가는 흐름을 캡처

- 연결의 절반은 training, 나머지 절반은 테스트에 사용

- 이전 방법보다 적은 데이터셋으로도 높은 성능을 냄

  ![fig 13](https://github.com/Joe2357/Joe2357.github.io/blob/main/assets/img/post/add/2022-09-02-previousWork/fig13.png?raw=true)

- True Positive, False Positive을 성능지표로 사용

  - TP : 상관관계가 있다고 예측했고, 실제로 상관관계가 있는 경우
  - FP : 상관관계가 있다고 예측했지만, 실제로 상관관계가 없는 경우

#### 한계점

- 들어오고 나가는 패킷을 모두 볼 수 있다는 강한 가정



### Critical Traffic Analysis on the Tor Network

> Florian Platzer, Marcel Schäfer, and Martin Steinebach. 2020. Critical traffic analysis on the tor network. In Proceedings of the 15th International Conference on Availability, Reliability and Security (ARES '20). Association for Computing Machinery, New York, NY, USA, Article 77, 1–10.

#### 아이디어

- 토르 히든 서비스 식별하는 방법 제안

- 토르 히든 서비스를 식별하기 위해 필요한 전제조건

  - 우리가 제어하는 Tor 노드가 introduction point 회로의 일부인지 탐지하는 방법을 제시

  - Tor 노드가 회로의 어느 위치에 있는지 확인할 수 있는 방법을 제시

  - 전송된 패턴을 통해 우리가 관리하는 노드에서 트래픽을 인식

    ![fig 14](https://github.com/Joe2357/Joe2357.github.io/blob/main/assets/img/post/add/2022-09-02-previousWork/fig14.png?raw=true)

    - 가정 : Hidden server의 guard 노드를 제어

1. introduction 확인

   ![fig 15](https://github.com/Joe2357/Joe2357.github.io/blob/main/assets/img/post/add/2022-09-02-previousWork/fig15.png?raw=true)

2. 회로에서 어떤 위치에 위치하고 있는지 파악하는 방법

   ![fig 16](https://github.com/Joe2357/Joe2357.github.io/blob/main/assets/img/post/add/2022-09-02-previousWork/fig16.png?raw=true)![fig 17](https://github.com/Joe2357/Joe2357.github.io/blob/main/assets/img/post/add/2022-09-02-previousWork/fig17.png?raw=true)

   - 회로를 처음 구성할 때 발생하는 트래픽 지문을 이용하여 어느 위치에 있는지 확인
   - 토르의 평균 lifetime은 10분, guard 노드는 잘 바뀌지 않는다고 함
   - 시간을 이용해서 노드 위치 확인

#### 실험방법 및 성능사항
- 히든 서비스에 트래픽을 보내고 가드노드에서 확인되는 트래픽 패턴과 동일한지 확인

#### 한계점

- 토르 최신 버전에 padding cell 이라는 보안장치가 새로 패치되어서 제안한 방법들 사용 불가능
- guard 노드를 장악할 확률 작다



### Deep Fingerprinting: Undermining Website Fingerprinting Defenses with Deep Learning

> Payap Sirinam, Mohsen Imani, Marc Juarez, and Matthew Wright. 2018. Deep Fingerprinting: Undermining Website Fingerprinting Defenses with Deep Learning. In Proceedings of the 2018 ACM SIGSAC Conference on Computer and Communications Security (CCS '18). Association for Computing Machinery, New York, NY, USA, 1928–1943.

#### 아이디어

![fig 18](https://github.com/Joe2357/Joe2357.github.io/blob/main/assets/img/post/add/2022-09-02-previousWork/fig18.png?raw=true)![fig 19](https://github.com/Joe2357/Joe2357.github.io/blob/main/assets/img/post/add/2022-09-02-previousWork/fig19.png?raw=true)

- 더 좋은 모델의 CNN 네트워크를 사용하여 성능향상
- 인풋으로 방향을 나타내는 +1, -1 만을 사용 (packet length는 성능 향상에 큰 도움이 안됨)
- 하나의 인스턴스의 길이를 5000으로 고정, 인풋으로 사용

#### 실험방법 및 성능사항

- 데이터셋 : https://github.com/deep-fingerprinting/df

  - closed world / open world : 모니터링 웹사이트

  - non-defended / WTF-PAD / Walkie-Talkie : 패킷 데이터셋

    ![fig 20](https://github.com/Joe2357/Joe2357.github.io/blob/main/assets/img/post/add/2022-09-02-previousWork/fig20.png?raw=true)

    ![fig 21](https://github.com/Joe2357/Joe2357.github.io/blob/main/assets/img/post/add/2022-09-02-previousWork/fig21.png?raw=true)

#### 한계점

- 좋은 딥러닝 모델이 결국 좋은 결과로 이어지는 듯 함



### A Method for Identifying Tor Users Visiting Websites Based on Frequency Domain Fingerprinting of Network Traffic

> Yuchen Sun, Xiangyang Luo, Han Wang, Zhaorui Ma, "A Method for Identifying Tor Users Visiting Websites Based on Frequency Domain Fingerprinting of Network Traffic", *Security and Communication Networks*, vol. 2022, Article ID 3306098, 12 pages, 2022.

#### 아이디어

- DWT기반 WF 공격 방법인 FDF 제안

  ![fig 22](https://github.com/Joe2357/Joe2357.github.io/blob/main/assets/img/post/add/2022-09-02-previousWork/fig22.png?raw=true)

  1. 웹사이트를 방문하는 사용자의 트래픽 패킷을 캡처

  2. 네트워크 트래픽 패킷에서 회로 시퀀스의 방향 및 길이 정보를 추출

     ![fig 23](https://github.com/Joe2357/Joe2357.github.io/blob/main/assets/img/post/add/2022-09-02-previousWork/fig23.png?raw=true)

     - 시퀀스 방향, 길이(length)

  3. DWT 변환으로 회로의 frequency domain feature 시퀀스 생성.(저주파 시퀀스 사용)

  4. 주파수 영역 feature 시퀀스와 해당 사이트 레이블을 데이터베이스에 저장

  5. 딥러닝 방식으로 학습 및 테스트

  6. 아웃풋 : 웹사이트 레이블

- DWT (Discrete Wavelet Transformation)

  ![fig 24](https://github.com/Joe2357/Joe2357.github.io/blob/main/assets/img/post/add/2022-09-02-previousWork/fig24.png?raw=true)

- 필터를 사용하여 회로 시퀀스를 여러 주파수 도메인 구성 요소로 분해, 노이즈 간섭을 크게 줄임

  - 저주파 : 시퀀스의 기본 프레임이며 시퀀스의 대략적인 정보에 속함
  - 고주파 : 시퀀스의 빠르게 변화하는 부분이며 노이즈가 포함된 시퀀스의 상세 정보에 속함저주파 부분만을 이용하여 노이즈가 제거된 부분 사용 가능

#### 실험방법 및 성능사항

- 데이터 수집

  ![fig 25](https://github.com/Joe2357/Joe2357.github.io/blob/main/assets/img/post/add/2022-09-02-previousWork/fig25.png?raw=true)

  ![fig 26](https://github.com/Joe2357/Joe2357.github.io/blob/main/assets/img/post/add/2022-09-02-previousWork/fig26.png?raw=true)

#### 한계점

- 학습에 많은 데이터셋과 시간 필요



### Tik-Tok: The Utility of Packet Timing in Website Fingerprinting Attacks

> Rahman, Mohammad Saidur & Sirinam, Payap & Mathews, Nate & Gangadhara, Kantha & Wright, Matthew. (2020). Tik-Tok: The Utility of Packet Timing in Website Fingerprinting Attacks. Proceedings on Privacy Enhancing Technologies. 2020. 5-24. 10.2478/popets-2020-0043. 

#### 아이디어

- burst-level 을 기반으로 새로운 타이밍 feature 제안

- 타이밍과 방향의 조합을 이용 (directional timing)

- burst : 클라이언트에서 웹서버로 나가거나 서버에서 클라이언트로 들어오는 동일한 방향의 연속적인 패킷 그룹

  ![fig 27](https://github.com/Joe2357/Joe2357.github.io/blob/main/assets/img/post/add/2022-09-02-previousWork/fig27.png?raw=true)

- 타이밍에서 추출할 8개의 feature

  ![fig 28](https://github.com/Joe2357/Joe2357.github.io/blob/main/assets/img/post/add/2022-09-02-previousWork/fig28.png?raw=true)

  ![fig 31](https://github.com/Joe2357/Joe2357.github.io/blob/main/assets/img/post/add/2022-09-02-previousWork/fig31.png?raw=true)

  - **Median Packet Time (MED)** : 각 버스트의 타임스탬프 중앙값, B1 : 0.10
  - **Variance** : 버스트 내의 시간 분산, B1의 경우 0.0067
  - **Burst Length** : 버스트 내의 첫번째와 마지막 타임스탬프의 차이, B1 : 0.2
  - **Inter Median Delay (IMD)** : 연속된 버스트에서 각 중앙값들의 차이, 0.4
  - **Inter-Burst Delay-FF** : 연속된 버스트에서의 첫번째 패킷 타임 차이, 0.4
  - **IBD-LF** : 연속된 버스트에서 첫번째 버스트는 마지막 패킷, 두번째 버스트에서는 첫번째 패킷의 시간 차이, 0.2
  - **IBD-IFF** : incoming 버스트에서 첫번째 패킷 타임 차이, B2, B4 : 0.35
  - **IBD-OFF** : outgoing 버스트끼리 첫번째 패킷 타임 차이, B1, B3 : 0.65
  - 각 feature를 히스토그램으로 하고, 각 인스턴스에 대한 최종 feature set을 생성할 때 b 빈의 범위를 사용 (feature에 대해 더 잘 캡쳐할 수 있다고 함)

#### 실험방법 및 성능사항

- 데이터셋 : https://github.com/deep-fingerprinting/df

- Timing feature < Raw Timing < Directional Time

  ![fig 29](https://github.com/Joe2357/Joe2357.github.io/blob/main/assets/img/post/add/2022-09-02-previousWork/fig29.png?raw=true)

  ![fig 30](https://github.com/Joe2357/Joe2357.github.io/blob/main/assets/img/post/add/2022-09-02-previousWork/fig30.png?raw=true)

#### 한계점



### Triplet Fingerprinting: More Practical and Portable Website Fingerprinting with N-shot Learning

> Payap Sirinam, Nate Mathews, Mohammad Saidur Rahman, and Matthew Wright. 2019. Triplet Fingerprinting: More Practical and Portable Website Fingerprinting with N-shot Learning. In Proceedings of the 2019 ACM SIGSAC Conference on Computer and Communications Security (CCS '19). Association for Computing Machinery, New York, NY, USA, 1131–1148.

#### 아이디어

- 기존 연구는 데이터셋의 변화하게 되면 다시 학습해야 하는 단점

  - 데이터셋을 모으는데 시간이 오래 걸리는 단점
  - 학습 데이터셋과 테스트 데이터셋의 분포가 비슷하다는 가정

- 개선 목표

  - 학습 데이터의 부실, 학습 및 테스트 데이터의 이질적인 분포에 견고해야 함

  - 이미 훈련된 모델은 나중에 추가된 데이터에 대해서 유효해야 함. 훈련해야 하는 경우 필요한 훈련 데이터의 양이 작아야 함

  - N-shot learning (특정 클래스를 식별하기 위해 약간의 훈련 샘플만 필요로 함)

  - Triplet Network 사용

    ![fig 32](https://github.com/Joe2357/Joe2357.github.io/blob/main/assets/img/post/add/2022-09-02-previousWork/fig32.png?raw=true)

    ![fig 33](https://github.com/Joe2357/Joe2357.github.io/blob/main/assets/img/post/add/2022-09-02-previousWork/fig33.png?raw=true)

    ![fig 34](https://github.com/Joe2357/Joe2357.github.io/blob/main/assets/img/post/add/2022-09-02-previousWork/fig34.png?raw=true)

    - Pre-Training Phase
    - Attack Phase
    - Classification

#### 실험방법 및 성능사항

- 데이터셋

  - Wang dataset / AWF dataset / DF dataset 사용 (공개 데이터셋)

- pre-training 에서 학습한 데이터셋과 attack 에서 학습한 데이터셋을 어느정도 겹치게 한 경우 ( similar but mutually exclusive )

  ![fig 35](https://github.com/Joe2357/Joe2357.github.io/blob/main/assets/img/post/add/2022-09-02-previousWork/fig35.png?raw=true)

- 학습시 사용된 데이터셋과 테스트시 사용된 데이터셋의 분포 차이 (오래된 데이터셋과 비교적 최근 데이터셋으로 비교)

  ![fig 36](https://github.com/Joe2357/Joe2357.github.io/blob/main/assets/img/post/add/2022-09-02-previousWork/fig36.png?raw=true)

- https://github.com/triplet-fingerprinting/tf

#### 한계점

![fig 37](https://github.com/Joe2357/Joe2357.github.io/blob/main/assets/img/post/add/2022-09-02-previousWork/fig37.png?raw=true)



### Deep Nearest Neighbor Website Fingerprinting Attack Technology

> Maohua Guo, Jinlong Fei, Yitong Meng, "Deep Nearest Neighbor Website Fingerprinting Attack Technology", *Security and Communication Networks*, vol. 2021, Article ID 5399816, 14 pages, 2021.

#### 아이디어

- Deep Nearest Neighbor Website Fingerprinting (DNNF) 공격기술 제안

  - CNN으로 deep local fingerprinting 특징 추출
  - KNN으로 예측 분류

  ![fig 38](https://github.com/Joe2357/Joe2357.github.io/blob/main/assets/img/post/add/2022-09-02-previousWork/fig38.png?raw=true)

- 기존 방법들보다 정확도가 높고, 전이학습 / concept drift 문제 등에 더욱 효과적

- pretraining 단계 / attack 구성 단계로 나누어 모델 학습

  ![fig 39](https://github.com/Joe2357/Joe2357.github.io/blob/main/assets/img/post/add/2022-09-02-previousWork/fig39.png?raw=true)

  ![fig 40](https://github.com/Joe2357/Joe2357.github.io/blob/main/assets/img/post/add/2022-09-02-previousWork/fig40.png?raw=true)

  ![fig 41](https://github.com/Joe2357/Joe2357.github.io/blob/main/assets/img/post/add/2022-09-02-previousWork/fig41.png?raw=true)

#### 실험방법

- AWF 데이터셋, Wang 데이터셋 사용
- closed world, open world에 대한 실험 진행
- 데이터 패킷 길이가 공격에 의미가 없다는 사실을 발견하고, 딥러닝과 동일하게 데이터 표현
  - 데이터 패킷 및 timestamp 크기 무시
  - traffic을 sequence로 변환각 패킷의 inbound / outbound 방향 저장 ( -1 / +1 )
  - closed world / open world 에서 테스트 진행

#### 결과

- model training loss

![fig 42](https://github.com/Joe2357/Joe2357.github.io/blob/main/assets/img/post/add/2022-09-02-previousWork/fig42.png?raw=true)

- closed world 성능

  ![fig 43](https://github.com/Joe2357/Joe2357.github.io/blob/main/assets/img/post/add/2022-09-02-previousWork/fig43.png?raw=true)

  - 샘플 개수가 15개 이상부터는 96%에서 안정적으로 지나가는 것을 관찰 가능
  - concept drift 상황에서도 분류 정확도를 높게 유지할 수 있었음

  ![fig 44](https://github.com/Joe2357/Joe2357.github.io/blob/main/assets/img/post/add/2022-09-02-previousWork/fig44.png?raw=true)

#### 한계점

- 네트워크 모델 복잡성이 높음 ( 파라미터가 수백만개 )
  - 복잡도가 낮은 네트워크 모델을 만들어야할 것
- 데이터 수집 과정에서, 순수성을 확보하기 위해 매번 웹사이트를 방문해야하는데, 이것은 실제 상황에서는 불가능함
- 트래픽 분석이라기보다, 특정 웹사이트 홈페이지의 fingerprint 인식만을 중점적으로 조사



### Detection of Tor Traffic using Deep Learning

> Sarkar, Debmalya & Vinod, P. & Yerima, Suleiman. (2020). Detection of Tor Traffic using Deep Learning. 10.1109/AICCSA50499.2020.9316533.

#### 아이디어

- Tor 트래픽을 탐지 / 분류하는 DNN 시스템 제안
  - Tor 트래픽인지 Non-Tor 트래픽인지 구분하는 방법을 제시한 것
  - 암호화된 flow로부터 트래픽 type이 검출 가능하다는 것을 보임
    - ISP가 Tor 네트워크를 통해 전송되는 특정 콘텐츠를 차단할 수 있음

![fig 45](https://github.com/Joe2357/Joe2357.github.io/blob/main/assets/img/post/add/2022-09-02-previousWork/fig45.png?raw=true)

- ML에 사용하기 위한 데이터셋 전처리 과정을 거침
  - feature 선택 알고리즘 ( filter / wrapper / hybrid 접근방식 사용 )
  - feature 선택 방법으로 상관 기반 필터 선택 (CFS) / 대칭 불확실성 (SU) 사용
- 대칭 불확실성 ( SU ) : 확률변수의 불확실성을 측정하는 entropy 기반
  - entropy를 이용하여 information gain을 계산하며, 이를 이용하여 계산
  - information gain에 의해 발생하는 bias, 비정규화 문제 등을 해결하는데 SU 활용
  - 총 11개의 feature가 선택
- Correlation Based Filter Selection (CFS)
  - feature가 중요하다 = feature가 class와 관련이 있고, 동일한 class의 다른 feature와 중복
  - linear correlation 또는 entropy를 추정하여 결정
  - 4개의 속성이 높은 정확도를 얻음
- 선택한 feature들을 이용하여 DNN 모델 구현 / 훈련
  - 입력으로 주어진 feature <-> 출력으로 주어진 label을 mapping하는 것을 훈련시킴
  - 역전파 / Gradient Descent 알고리즘 사용하여 뉴런 가중치 조정

#### 실험 방법

- UNB-CIC Tor network Dataset + Non-Tor real-world Dataset 에서 DNN 모델 평가
- GAN ( generated adversarial network ) 이용하여 트래픽 생성기 구현
  - 합법적인 것과 동일한 통계 분포를 가진 악의적인 트래픽을 추가하여 학습

![fig 46](https://github.com/Joe2357/Joe2357.github.io/blob/main/assets/img/post/add/2022-09-02-previousWork/fig46.png?raw=true)

- 평가는 Precision / Recall 등의 기본 metric 사용
- 4가지 시나리오 고려
  - DNN 모델을 이용하여 트래픽 식별 ( Tor / Non-Tor )
  - 트래픽을 특정 범주로 분류 ( 브라우징, 이메일 등 )
  - shallow network / DNN의 정확도 비교
  - 적대적 공격을 사용하여 모델 견고성 평가 ( GAN 활용 )

#### 결과

- 시나리오 1 : 98 ~ 99%의 정확도를 보임
- 시나리오 2 : 95.6%의 정확도를 보임
- 시나리오 3 : 97 ~ 99%의 정확도를 보임
  - 네트워크가 깊어질수록 구별되는 class의 관찰을 효과적으로 구별하기 위해 차별적인 패턴을 학습
- 시나리오 4 : 새로운 DNN 모델이 적대적 공격에 더 탄력적임을 보임

#### 한계점

- source / destination을 찾아볼 수 있는 모델은 아니었음



### AttCorr: A Novel Deep Learning Model for Flow Correlation Attacks on Tor

> J. Li, C. Gu, X. Zhang, X. Chen and W. Liu, "AttCorr: A Novel Deep Learning Model for Flow Correlation Attacks on Tor," *2021 IEEE International Conference on Consumer Electronics and Computer Engineering (ICCECE)*, Guangzhou, China, 2021, pp. 427-430

#### 아이디어

- Tor에 대한 flow correlation 공격을 위한 새로운 딥러닝 모델 설계
  - 패킷 크기, flow 방향, 패킷 간 delay를 포함한 flow pair의 raw traffic feature를 입력으로 사용
- flow correlation 문제를 이진 분류 문제로 다룸
  - multi-head self-attention 메커니즘을 기반으로 함
- 목적 : Tor의 feature, noise의 복잡한 특성에 관련된 정보를 포착하는 것

​	![fig 47](https://github.com/Joe2357/Joe2357.github.io/blob/main/assets/img/post/add/2022-09-02-previousWork/fig47.png?raw=true)

- flow correlation 표현 ( input )
  - T : inter-packet delay / S : size of packet / u : upstream / d : downstream
  - flow마다 패킷 개수가 다르기 때문에 앞의 개만 feature를 추출하여 모델 input으로 사용 ( 없다면 0으로 padding )
  - n은 hyperparameter ( 작으면 계산 부담 감소 / 크면 정확도 향상 )
- 모델
  - 다중 head attention 메커니즘 사용 ( transformer의 encoder 기반 )
  - 위치 인코딩 추가 -> 시퀀스에서 token의 상대적 / 절대적 위치정보가 주입됨 -> sine, cosine 함수 사용 가능
  - 3개의 동일한 layer는 multi-head attention 메커니즘 layer + position-wise fully connected feed-forward network로 구성되어있음

#### 실험 방법

- 여러개의 관련 / 비관련 flow pair의 dataset을 준비
  - pair가 Tor 연결의 두 segment로 구성되어있다면 양수샘플, y(i, j) = 1
  - 아니라면 음수샘플, y(i, j) = 0
  - loss function : output ( psi(F\_{i, j}) ) 과 label 사이의 cross entropy // Adam을 이용해서 loss값 최소화 목적
- 모델이 올바른 판단을 했는지 확인하는 방법 : 해당 flow pair가 서로 연관되어있는지 여부를 확인하기 위해 가장 큰 sample을 선택하는 경우
- 데이터셋은 DeepCorr에서 사용했던 실제 네트워크 환경 캡처 데이터셋을 사용 ( 샘플의 비율은 다름 )
  - 연관 flow의 개수 : 7324개

#### 결과

|   모델   | 정확도 |   각 layer의 시간복잡도   |
| :------: | :----: | :-----------------------: |
| DeepCorr | 71.4%  | O($ndkl$ $C_{in}C_{out}$) |
| AttCorr  | 69.7%  |      O($n^2d+nd^2$)       |

- 복잡도 비교
  - AttCorr : transformer encoder의 시간복잡도임
  - DeepCorr : CNN 기반의 시간복잡도임
    - $k \times l$ : convolution 커널 size / $C_{in}$ : 입력 채널 수 / $C_{out}$ : 출력 채널 수
    - $n$ : 시퀀스 길이 / $d$ : 차원 (dimension)
  - AttCorr이 제곱항이 있지만 n과 d의 크기가 작기 때문에 괜찮음 ( n <= 500, d < 20 )
    - DeepCorr은 $C_{in}, C_{out}$이 한참 크기 때문에 AttCorr의 시간복잡도가 더 낮다
- 3가지 이점이 있음
  - feature 통합 단계가 필요없음
  - 거리에 때른 두 위치간 연관성을 계산하는 작업 수가 증가하지 않음 -> **길고 짧은 데이터 시퀀스 모두 처리 가능**
  - 자기주의 구조가 transformer 모델로부터 상속받은 해석가능성 향상시킴

#### 한계점

- 웹사이트 접속하여 오고 나가는 데이터셋 사용했음 ( 연구 주제와 100% 맞는지 )
- 해석가능성에 대한 설명이 없음



### 2ch-TCN: A Website Fingerprinting Attack over Tor Using 2-channel Temporal Convolutional Networks

> M. Wang, Y. Li, X. Wang, T. Liu, J. Shi and M. Chen, "2ch-TCN: A Website Fingerprinting Attack over Tor Using 2-channel Temporal Convolutional Networks," 2020 IEEE Symposium on Computers and Communications (ISCC), Rennes, France, 2020, pp. 1-7

#### 아이디어

- 패킷 시퀀스, 타이밍 정보로부터 feature를 추출하는 2채널 temporal convolutional network 모델 제안

  ![fig 48](https://github.com/Joe2357/Joe2357.github.io/blob/main/assets/img/post/add/2022-09-02-previousWork/fig48.png?raw=true)

  - CNN 기반, WaveNet, Fully-convolution network, residual block 구조를 결합한 TCN 구조임
  - 가중치 layer 정규화 / spatial dropout / ReLU 등을 잔차블록으로 사용
  - TRB를 수직 / 수평으로 쌓은 구조로 최종 output 출력하는 구조
    - $d$ : 확장계수 목록

- 전체 모델 구조

  ![fig 49](https://github.com/Joe2357/Joe2357.github.io/blob/main/assets/img/post/add/2022-09-02-previousWork/fig49.png?raw=true)

  - 2종류의 시퀀스 ( 시퀀스, 타이밍 방향 / 크기 ) 를 입력으로 사용함
    - 위 정보를 이용하여 추후 사용할 특정 작업에 대한 표현벡터를 얻을 수 있음
  - FCL로 표현모델 출력 수집 / SoftMax로 시퀀스 표현과 class 레이블에 매핑
  - overfitting을 피하기 위해 마지막 층에 dropout 매커니즘 도입

- 장점

  - 병렬 처리 / 수용 필드 크기가 유연 / 안정적인 gradient / 낮은 메모리 요구
  - CNN에 비해 시퀀스 정보를 더 잘 추출 가능
  - RNN에 비해 더 긴 시퀀스 특성을 배울 수 있음

- 공격 위치 / 방식

  ![fig 50](https://github.com/Joe2357/Joe2357.github.io/blob/main/assets/img/post/add/2022-09-02-previousWork/fig50.png?raw=true)

  - 현 상황으로는, 공격자는 ISP, AS 운영자가 주로 있을 것
  - fingerprinting : 감시할 웹사이트 선택 / 여러번 방문하여 트래픽 추적 / 웹사이트마다의 **지문 생성**
    - 공격자 : 지문을 이용하여 희생자가 시작한 통신에서 수집한 트래픽을 분류기에 넣어, 방문하려하는 웹사이트를 식별한다

- 현재의 최첨단 WF 공격 : Deep Fingerprinting (DF)

#### 실험방법

- 데이터셋을 자체 수집 / 트래픽 데이터를 TCP, TLS, Tor cell 계층의 3가지 추출계층으로 변환
  - 데이터셋을 다른 공개된 데이터셋도 사용하여 결과 비교
- 방문 중에 client가 선택한 **entry node에 Tor cell log**를 기록
  - 목적 : 어디에 있는 공격자가 더 효율이 좋은지 판단
  - 생성된 데이터셋은 사용 가능하다고 적혀있음
- 회로 구성 프로세스를 추가하여 랜덤성 높임

#### 결과

- 기존 attack들과의 비교에서 높은 정확도를 보임

  ![fig 51](https://github.com/Joe2357/Joe2357.github.io/blob/main/assets/img/post/add/2022-09-02-previousWork/fig51.png?raw=true)

- VarCNN과 비교하여 정확도, loss 모두 이득임을 보임

  ![fig 52](https://github.com/Joe2357/Joe2357.github.io/blob/main/assets/img/post/add/2022-09-02-previousWork/fig52.png?raw=true)

- entry node에 있는 공격자가, client와 entry node 사이의 traffic을 엿듣는 것보다 유리하다는 결론 도달

#### 한계점

- 문제를 단순화시키기 위해 파일 다운로드 등의 활동 없이 <u>한번에 하나의 웹사이트만 load한다고 가정</u>
- 트래픽 추출 데이터가 실제 cell과 맞지 않는다 ( 더 가깝게 만들어야할 것 )



### snWF: Website Fingerprinting Attack by Ensembling the Snapshot of Deep Learning

> Y. Wang, H. Xu, Z. Guo, Z. Qin and K. Ren, "snWF: Website Fingerprinting Attack by Ensembling the Snapshot of Deep Learning," in *IEEE Transactions on Information Forensics and Security*, vol. 17, pp. 1214-1226, 2022

#### 아이디어

- 비정상 앙상블을 이용하여 새로운 WF 기술 제안 ( 신경망의 분산을 줄이고 공격을 더 강인하게 할 수 있음 )

  ![fig 53](https://github.com/Joe2357/Joe2357.github.io/blob/main/assets/img/post/add/2022-09-02-previousWork/fig53.png?raw=true)

  - 스냅샷 앙상블 기법으로 경제적인 앙상블 전략 수행 -> 추가 리소스 없이 여러 모델을 교육할 수 있다는 이점

    ![fig 54](https://github.com/Joe2357/Joe2357.github.io/blob/main/assets/img/post/add/2022-09-02-previousWork/fig54.png?raw=true)

    - 12개의 CN 사용 / 6개의 모듈로 패키지화
      - GAP : 전역 평균 pooling 계층 ( 컨볼루션 출력을 vector로 변경 )
    - warm restart : 순환 cosine annealing을 이용하여 어널링 ( 전략 2개 )
      - SDGWR : 확률론적 gradient descent를 최적화로 사용 / 특정 문제의 매개변수 튜닝으로 사용
      - AWR : SDGWR의 업그레이드 버전 / Adam, Nadam 등의 적응형 최적화론과 함께 작동하며 여러 local min을 얻기 위해 warm restart 수행
      - 장점 : 앙상블 방법으로 적응형 gradient descent 업데이트를 통해 최신 network에 적용될 수 있다고 함
    - 딥러닝의 non-convex 특성

  - 학습 알고리즘 : 서로 다른 local min에 $M$번 수렴하게 하고, 수렴할 때마다 한번씩 모델에 저장

- 불균형 데이터에서도 견고함 ( 소량의 데이터에서도 작동 가능 )

- 이 모델은 공격자가 user <-> entry 사이의 트래픽을 보고 있다고 가정


#### 실험방법

- 40만개의 웹사이트가 있는 open world 환경에서, 모니터링된 웹사이트를 방문하는지 여부를 결정
- 이전에 모니터링하지 않은 웹사이트가 있는 "넓은 세상"에서 추가 평가
- 모델의 hyperparameter, 앙상블 크기 등을 추가 조정 과정을 거쳐 정함

#### 결과

![fig 55](https://github.com/Joe2357/Joe2357.github.io/blob/main/assets/img/post/add/2022-09-02-previousWork/fig55.png?raw=true)

- snWF가 DF, CNN-R보다 모든 metric에서 **더 좋은 성능을 보임**

  - 모니터되지 않은 dataset을 사용할 때에는 그 차이가 분명히 보일 정도로 증가한다고 함
  - 데이터 불균형 상태에서도 snWF는 성능변화가 크게 없다고 함

- 앙상블 기법이 효과가 있음 ( 스냅샷 개수가 늘수록 error rate가 작아짐 )

  ![fig 56](https://github.com/Joe2357/Joe2357.github.io/blob/main/assets/img/post/add/2022-09-02-previousWork/fig56.png?raw=true)

- concept drift 상황에서도 더 탄력적임을 보임


#### 한계점

- 정확도를 높이기 위해 웹사이트를 계속해서 모니터링해야함
- Tor 사용자의 보안문제가 심하게 존재함