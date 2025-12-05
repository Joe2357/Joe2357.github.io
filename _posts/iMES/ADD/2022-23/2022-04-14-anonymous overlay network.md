---
title: "익명 오버레이 네트워크"
author: Joe2357
categories: [iMES, Agency for Defense Development (ADD) / 2022-23]
tags: [Agency for Defense Development (ADD) / 2022-23]
uploaded_at: 2022-04-14 12:00:00 +0900
last_modified_at: 2022-04-14 12:00:00 +0900
description: "- Anonymous Overlay Networks"
math: true
---



### 개요

익명 오버레이 네트워크는 감시나 트래픽 추적을 피하기 위한 정보통신 보호를 목적으로 설계된 네트워크이다. 그러나 사이버 범죄자들은 익명 오버레이 네트워크가 의도적으로 일반적인 검색 엔진으로부터 숨겨져 있고, 가짜 IP 주소를 사용하며, 특정 소프트웨어나 네트워크 허가나 설정이 있어야 접근할 수 있는 **가상화 네트워크**라는 특징을 악용하여 사이버 범죄의 수단으로 활용하고 있다. 다크넷에 존재하는 사이트는 정체와 활동내용을 감출 수 있는 '토르 (Tor)'와 같은 익명 브라우저를 사용해 본래의 정체를 숨기고 사이버 범죄를 저지르는데 활용하고 있다. 이는 토르를 사용하면 위치를 속여 자신의 현재 위치가 아닌 다른 국가에 존재하는 것처럼 가장할 수 있다는 점을 이용한다. 토르에서는 납치, 신분증 위조, 아동 포르노, 마약 거래 등 중범죄와 이에 대한 거래가 연간 1,000억 원에 달한다는 조사 결과가 있을 정도로 그 규모가 상당하다.

다크넷 (Dark Net), 딥웹 (Deep Web), 그리고 다크웹 (Dark Web) 이라는 용어들은 서로 혼용되어 사용되고 있어 개념의 차이에 대해 정립할 필요가 있다. 다크넷 중 웹만을 따로 다크웹이라고 칭하고, 딥웹은 인터넷에서 접근이 불가능한 자원의 통칭으로, 구글이나 네이버 등 일반적인 검색엔진으로는 접근할 수 없는 사용자 이메일, 온라인 뱅킹, 인터넷 지불창 등을 모두 포함하는 개념이다. 다크웹은 토르 및 I2P (Invisible Internet Project) 와 같은 네트워크를 통해서만 접근할 수 있는 특정 웹사이트의 집합체를 의미한다. 사용자의 IP 주소를 여러 계층에서 암호화하고, 여러 개의 분산된 경로를 통해 접속하도록 하여 사용자를 추적할 수 없도록 하며 **익명성**을 특징으로 한다.



### 네트워크 종류



#### Tor 네트워크

> Tor, The Onion Routing

토르는 어니언 라우팅 (onion routing) 을 이용하여 사용자가 출발지와 목적지의 IP 주소를 노출시키지 않고 인터넷 서비스를 이용할 수 있도록 설계된 기술이다. 즉, 불특정 노드와 IP 주소의 암호화를 통해 사용자의 익명성을 확보하는 기술이다. 전 세계 참여자들이 운영하는 릴레이의 분산 네트워크를 통해 통신을 활용하여 사용자를 보호한다. 인터넷 연결을 감시하는 주체가 사용자가 방문하는 웹사이트를 학습하지 못하도록 하여 사용자가 방문하는 사이트가 사용자의 실제 위치를 학습하지 못하도록 하는 원리이다.

- 작동 원리 : 어니언 라우팅은 인터넷 상의 여러 장소에 transaction을 분산시켜 트래픽 분석의 위험을 줄여주므로 어떤 단일 지점에서도 목적지에 연결할 수 없다는 점을 이용한다. 토르 네트워크의 데이터 패킷은 출발지에서 목적지로 직접 이동하는 대신, 트랙을 커버하는 여러 릴레이를 통해 랜덤한 경로로 전송되므로 <u>단일 지점에 존재하는 관찰자는 패킷이 어디에서 왔는지, 어디를 향하는지 알 수 없다</u>. 토르를 사용하여 개인 네트워크 경로를 만들기 위해 사용자의 소프트웨어는 네트워크의 릴레이를 통해 암호화된 연결 회로를 점진적으로 구축한다. 회로는 한 번에 한 홉씩 연장되며, 도중에 존재하는 각 릴레이는 어떤 릴레이가 패킷을 주고 어떤 릴레이에게 그것을 전달하고 있는지만을 알 수 있다. 사용자는 회로를 따라 각 홉에 대해 별도의 암호화 키 집합을 협상하여 각 홉을 통과할 때 연결을 추적할 수 없도록 한다. 암호화 및 복호화 구간이 많아짐에 따라 전체적인 통신 속도는 느려지지만, **정보 은닉**이 가능하다.
- Tor Hidden Service : 고유한 onion 호스트 이름을 가진 내부 토르 전용 서비스를 등록할 수 있다. 방문자가 토르 네트워크에 연결하면 토르는 해당 onion 주소를 확인하고 해당 이름 뒤에 있는 익명 서비스로 안내한다. 그러나 다른 서비스와는 다르게 양방향 익명성을 제공하여 서버가 사용자의 IP 주소를 알 수 없지만 <u>사용자도 서버의 IP 주소를 알 수 없다</u>. 양방향에서 모두 보호되기 때문에 궁극적인 개인정보 보호를 달성할 수 있다.



#### I2P 네트워크

> I2P, Invisible Internet Project

I2P는 오버레이 네트워크의 일종으로, 어플리케이션이 서로에게 가명으로 안전하게 메세지를 보낼 수 있도록 하는 **다크넷**이다. 사용자의 활동, 위치 및 신원을 보호하기 위해 설계상 개인정보 보호 및 보안으로 개발된 <u>완전히 암호화된 사설 네트워크 계층</u>을 의미한다. Tor 네트워크와 달리 외부 네트워크에 접속하는 기능보다는 흔히 **다크웹**이라는 내부 네트워크 기능에 초점이 맞춰져 있다. I2P의 모든 트래픽은 내부 네트워크에만 존재하며, 이것들은 인터넷과 직접 상호작용하지 않는다. 사용자와 peer 간에는 암호화된 단방향 터널을 제공하며, 트래픽이 어디서 오는지, 어디를 향하는지, 내용물이 무엇인지는 알 수 없다.

- 작동 원리 : 다른 오버레이 네트워크와는 다르게, I2P는 '**갈릭 라우팅**'이라는 기술을 사용한다. 네트워크에 연결된 사용자는 중앙 집중형 노드 데이터베이스에 의존하지 않고, 각각 고유한 암호화 라우팅 ID를 가지는 노드 또는 라우터로 전환된다. I2P 네트워크는 라우터(peer)와 단방향 inbound, outbound 가상 터널로 구성된다. 라우터는 메세지를 전달하면서 기존 전송 매커니즘 (TCP, UDP 등) 에 구축된 프로토콜을 이용하여 서로 통신한다. 사용자 측에는 어플리케이션에서 메시지를 보내고 받을 수 있는 자체 암호화 식별자 (destination) 가 존재한다. I2P는 암호화를 사용하여 구축하는 터널과 전송하는 통신에 대한 다양한 속성을 얻는다. 터널은 NTCP2, SSU를 사용하여 전송되는 트래픽의 특성을 숨긴다. 트래픽 연결은 라우터와 라우터, 사용자와 사용자 부분에서 암호화되며, 모든 연결에 대해서 보안이 작동한다. I2P는 암호화되어 처리되므로 IP주소는 자체적으로 인증될 수 있고, 생성한 사용자에 속하게 된다. 사용자가 보내는 메세지는 갈릭 라우팅에 의해 여러 개의 메세지가 하나의 clove로 묶여 bundle로써 전송된다. 이 암호화 주소 지정과 bundle 메세지 시스템을 통해 사용자는 익명으로 I2P 네트워크 상에서 통신이 가능하다.



#### VPN 네트워크

> VPN, Virtual Private Network

VPN 네트워크는 인터넷과 같은 공용 네트워크 위에 회사 또는 단체에서 비공개적인 데이터를 통신하기 위해 구축하는 사설 네트워크이다. 인터넷망과 같은 공중망에 터널링 기술을 이용하여 마치 전용 회선처럼 사용할 수 있는 암호화된 통신 서비스를 제공하는 사설망을 의미한다.

- 작동 원리 : 두 집단의 인터넷이 서로 통신할 때 각 인터넷의 게이트웨이 사이에 가상의 전용 회선 역할을 하는 VPN 터널을 생성하고, 이를 이용하여 통신한다. 이 때 터널의 생성은 규격화된 터널링 프로토콜을 활용하여 생성하며, 이 프로토콜이 전송할 패킷을 암호화하는 역할도 담당한다. 패킷을 암호화하는 해당 작업을 **캡슐화**라고 하는데, 이는 실제 전송할 패킷을 새로운 패킷 안에 넣어 실제 패킷의 도착지와 정보를 숨기고 통신하는 기술이다. VPN의 종류로는 크게 <u>통신 방식</u>, <u>터널링 프로토콜의 종류</u>, <u>통신 계층</u>에 따라 나뉜다. 통신 방식은 **site-to-site 방식** ( 2개의 게이트웨이 장비 사이에 터널을 개설하는 방법 ) 과 **client-to-site 방식** ( 각 사용자가 특정 장비에 접속하는 방법 ) 2가지가 존재한다. 프로토콜은 그 종류가 다양하며, 이 중에서 자주 사용되는 프로토콜은 **IPSec** ( IP Security ), **SSL** ( Secure Sockets Layer ) 이 있다.
  - IPSec VPN은 site-to-site 방식을 사용하며, 개설된 터널에서 네트워크 계층 (L3) 단계에서 패킷을 암호화한다. 먼저 IKE ( Internet Key Exchange ) 에서 2단계로 이루어진 과정을 통해 통신하는데, 1단계에서는 통신하려는 두 장비 사이에 인증서와 암호화 키를 주고받고, 이를 통해 터널을 생성한다, 2단계에서는 암호화 정보와 인증 정보를 가진 헤더를 전송할 패킷에 추가하고, 새로운 IP 주소를 부여하여 데이터를 전송하는 방식으로 통신이 이루어진다.
  - SSL VPN은 client-to-site 방식을 사용하며, 전송계층 (L4) 에서 터널 생성과 패킷 암호화를 진행한다. 최근에는 국제 인터넷 표준화 기구에서 표준으로 인정받은 **TLS** ( Transport Layer Security ) 기술로 패킷을 암호화하므로 사용자들이 별도의 소프트웨어 설치 및 기타 준비 없이 최신 웹 브라우저만 있다면 VPN을 사용할 수 있다. TLS 기반의 SSL 프로토콜은 메세지를 주고받으며 상대를 확인하는 handshake 과정을 통해 보안 통신을 구축하는데, 먼저 사용자와 서버가 서로 TLS 통신을 할 수 있는지 확인하고, 두번째 메세지를 통해 서로 인증서를 주고받아 신뢰할 수 있는지 확인한 뒤, 세번째 메세지로 암호를 주고받아 해당 암호 키를 통해 패킷을 암호화하는 3단계 과정을 통해 통신한다.



### 국내외의 연구 사례 및 동향

#### 토르 서버 추적 사례 및 분석기법에 관한 연구

> 이경빈, 토르 서버 추적 사례 및 분석기법에 관한 연구, 2018 고려대학교 정보보호대학원

토르 히든서비스에서 운영된 서버를 분석하여 그 특징을 확인해보고, 기존의 연구결과를 실제 적용할 수 있는 가능성에 대해 제안함. 실제 토르서버 추적에서 사용된 기법으로 알려진 몇몇 응용 프로그램 상에서의 취약점 사례에 대해서 살펴보고, 더불어 토르 히든서비스 서버에 대한 간접적 추적기법에 해당하는 비트코인 지갑 주소를 통한 추적 기법을 제시함.



#### 포렌식 수사를 위한 다크웹 데이터 수집 및 분석 방안 연구

> 진필근, 포렌식 수사를 위한 다크웹 데이터 수집 및 분석 방안 연구, 2021 고려대학교 정보보호대학원

기존 암호 화폐 거래를 추적하는 방법과 함께 본 연구에서 새롭게 제시한 Tracking code와 Status mode를 사용하여 다크웹 사이트 분석 사례 연구를 제시함으로써 제안한 데이터 분석 방법론이 다크웹과 표면웹 사이트를 연결하여 불법 행위가 이뤄지는 다크웹 사이트의 운영자를 식별할 수 있음을 입증함.



#### Website fingerprinting: attacking popular privacy enhancing technologies with the multinomial naïve-bayes classifier

> Herrmann, D., Wendolsky, R., & Federrath, H. (2009, November). Website fingerprinting: attacking popular privacy enhancing technologies with the multinomial naïve-bayes classifier. In Proceedings of the 2009 ACM workshop on Cloud computing security (pp. 31-42).

토르에 대한 웹사이트 지문 채취 공격을 최초로 실행한 사람임. 그들은 유사성을 측정하기 위해 패킷 크기 분포와 다항식 Naimbayes 분류기를 유일한 특징으로 사용했음. 하지만, 그들은 단지 약 3%의 인식률을 얻었을 뿐으로 알려짐.



#### Website fingerprinting in onion routing based anonymization networks

> Panchenko, A., Niessen, L., Zinnen, A., & Engel, T. (2011, October). Website fingerprinting in onion routing based anonymization networks. In Proceedings of the 10th annual ACM workshop on Privacy in the electronic society (pp. 103-114).

트래픽의 볼륨, 시간 및 방향을 기준으로 웹사이트 지문을 정의하고 지원 벡터 머신(SVM)을 분류 방법으로 활용했음. 토르의 인식률은 3%에서 54%로 크게 향상됨.



#### Enhancing Tor's performance using real-time traffic classification

> AlSabah, M., Bauer, K., & Goldberg, I. (2012, October). Enhancing Tor's performance using real-time traffic classification. In Proceedings of the 2012 ACM conference on Computer and communications security (pp. 73-84).

실시간으로 토르의 암호화된 회로를 어플리케이션 유형별로 분류하는 기계학습 기반 접근 방식인 DiffTor를 제안했음. 각 어플리케이션 유형을 적절한 QoS 요구사항에 맞게 매핑하고 토르의 성능을 향상시키는 것이 주 목적이었음. 어니언 라우터는 네트워크 패킷으로부터 셀을 추출할 수 있기 때문에, 그들의 분류 기능에는 회로 수명, 셀 사이의 도착 시간 및 최근에 전송된 셀의 수가 포함되었음.



#### A Distributed Flow Correlation Attack to Anonymizing Overlay Networks Based on Wavelet Multi-Resolution Analysis

> F. Palmieri, "A Distributed Flow Correlation Attack to Anonymizing Overlay Networks Based on Wavelet Multi-Resolution Analysis," in IEEE Transactions on Dependable and Secure Computing, vol. 18, no. 5, pp. 2271-2284, 1 Sept.-Oct. 2021, doi: 10.1109/TDSC.2019.2947666.

이 연구는 익명화 네트워크의 내부 및 출구 내에서 수집된 트래픽 흐름의 익명성을 제거하기 위한 새로운 전략을 제안하였음. 이는 웨이브렛 기반 다중 해상도 분석에 의해 구동되는 분산 흐름 캡처, 특성화, 상관 공격에 의존함.



#### Darknet Security: A Categorization of Attacks to the Tor Network

> Cambiaso, E., Vaccari, I., Patti, L., & Aiello, M. (2019, February). Darknet Security: A Categorization of Attacks to the Tor Network. In ITASEC (pp. 1-12).

다크넷 환경과 관련된 공격과 관련된 다크넷 보안 주제를 조사하여 분류위협에 기초한 공격대상에 기초한 포괄적인 분류법을 제안했음. 이러한 분류법은 다크넷 환경과 관련된 사이버 공격을 식별하고 그 기능을 더 잘 이해하기 위한 특성화 체계를 나타냈음.



#### Improving the defense of routing attack in TOR anonymization system

> Rezaei, S. and Araghizadeh, M. A., 1398, Improving the defense of routing attack in TOR anonymization system, 6th National Congress of New Electrical and Computer Engineering of Iran with a practical view on new energies, Tehran, undefined, undefined, https: // civilica .com / doc / 923961

토르의 라우팅 공격을 공격한 후 토르 네트워크 구성을 분석하여 그래프 내의 저항을 평가하고 개선된 방법을 제시했음.



#### The sniper attack: Anonymously deanonymizing and disabling the Tor network

> Jansen, R., Tschorsch, F., Johnson, A., & Scheuermann, B. (2014). The sniper attack: Anonymously deanonymizing and disabling the Tor network. Office of Naval Research Arlington VA.

노드 선택, 성능 및 확장성과 같은 중요한 측면을 모두 비교했음. 특히, 네트워크 흐름 정보를 지수가중평균기반으로 토르 트래픽을 다양한 범주로 분류하는 연구를 진행함.



### 국내외의 산업계 동향

#### 다크웹 이용 암호화폐 트랜잭션 분석 및 프로파일링

> 고려대학교, 이르테크, 유니스소프트 (2019.2). 다크웹 스캐닝 기술 기반의 정보 수집·분석 시스템 개발 최종보고서.

Tor, I2P, Freenet의 기술 활용 및 Ahmia Project 등에서 사용된 수집/저장 방법을 준용하는 익명화 서비스(Tor의 Hidden Service 등)를 통해 제공되는 다크웹 콘텐츠 크롤링용 웹로봇 개발하여 다크웹 내 범죄 사이트 콘텐츠를 수집하고, 범죄에 사용된 암호화폐 등의 분석을 통한 Feature 정보, Tor 릴레이서버 라우팅 알고리즘에 따른 범죄 노드 선택 확률 연구를 통해 다크웹 이용 범죄자 프로파일링 기술 및 시스템을 개발함.



#### 익명 네트워크에서 이루어지는 저작권 침해 식별 및 검증

> ㈜에스씨테크원, 엘에스웨어㈜, 단국대학교 (2019.12). 저작권 보호를 위한 익명 네트워크 실명화 기술 개발 최종(연차)보고서.

사용자 및 서비스 제공자의 신원을 공개하지 않는 익명 네트워크(토르) 상에서 불법적으로 콘텐츠를 공유하는 행위의 저작권 침해를 탐지하기 위하여 트래픽 핑거프린팅(Traffic Fingerprinting) (트래픽의 암호화를 풀지 않고 패킷의 흐름 패턴을 분석하여 토르 네트워크의 사용자를 알아내는 데 사용되는 기술; 클라이언트와 서버 사이의 패스 중 구성요소의 하나로 작동하여 트래픽을 분석하는 능동형 핑거프린팅 기술과, 순수하게 네트워크의 트래픽만을 분석하여 서버와 클라이언트의 토르 접속 활동을 탐지하는 수동형 핑거프린팅 기술로 나뉨) 등을 통해 사용자 및 서비스 제공자의 사이트 정보, IP 등과 같은 접속 정보를 실명화 하는 기술을 연구 개발하고 있음.



#### 익명 네트워크를 통해 유포된 악성코드 분석

> 한국인터넷진흥원 (2014.5). Tor 네트워크의 원리와 관련 악성코드 사례 분석.

Tor를 이용해 유포된 악성코드의 경우 명령제어 서버나 악성코드 경유지, 유포지에대한 차단 및 조치가 필요하지만 정확한 서버의 위치를 알 수 없어 어렵기 때문에 모니터링을 통해 악성코드 유포에 대한 사례를 통해 기술을 분석하고 있음.



#### 원자력 발전소

> Vinothkumar, C., Rajasekar, B., Balasankar, Karavadi., & Premalatha, J,. (2021, February). A secured tor local network for nuclear power plant industry. Journal of Physics: Conference Series, Volume 1913, International Conference on Research Frontiers in Sciences (ICRFS 2021). *Conf. Ser.* 1913 012111.

중요 산업, 특히 핵심 전력 인프라에 대한 사이버 보안 위험이 악화되고 있는 것으로 보임. 공격자가 원자로를 제어하는 시스템을 조작할 수 있도록 하는 공격은 매우 어려운 공격이지만, 굉장히 심각한 결과를 초래할 수 있음. 이 공격의 주요 원인은 발전소 직원에 대한 피싱 공격 및 바이러스 등과 같은 사이버 공격에 대해 주요 보호 장치 없이 인터넷에 직접 액세스할 수 있는 발전소의 로컬 컴퓨터 네트워크임. 이러한 공격에 대한 주요 방어 수단은 발전소 네트워크와 직원을 인터넷에서 익명으로 만들어, 해커가 발전소의 정보가 손실될 수 있는 직원에 대한 정보를 얻지 못하게 하는 것임. 기존 시스템의 단점은 아래와 같음.

1. 각 로컬 네트워크마다 보안 시스템을 별도로 설치해야 함
2. 사이버 공격에 취약함
3. 해커가 발전소 네트워크를 가로챌 수 있음
4. 보안 조치가 오래됨

토르 라우터를 사용하면 사용 중인 네트워크와 관계없이 트래픽을 암호화하고 개인 정보를 보호할 수 있음. 발전소의 네트워크를 보호하기 위해 이러한 시스템의 장점은 아래와 같음.

1. 네트워크 해킹 방지

2. 최신 보안 기술로 업데이트할 수 있는 유연성

3. 추적 불가능

4. 군용 등급 보호

   

#### IoT 주소 지정 및 연결을 위한 활용

> Baumann, Felix W. and Odefey, Ulrich and Hudert, Sebastian and Falkenthal, Michael and Breitenb. (2018, March). Utilising the Tor Network for IoT Addressing and Connectivity. SciTePress, isbn: 978-989-758-295-0.

사물인터넷(IoT) 기기 및 사이버 물리 시스템(CPS)의 경우 제어, 관리 및 활용을 위해 어떤 형태의 클라우드 환경이나 컴퓨팅 개체에도 안전하고 안정적인 연결이 필수로 요구됨. 클라이언트-서버 구성에서 IoT 장치를 연결하는 방법으로 IoT 및 CPS 장치의 주소 지정 및 보안 통신을 위해 토르 네트워크를 사용함. 이 접근 방식의 이점은 방화벽으로 보호되며 잠재적으로 알려지지 않고 변화하는 네트워크 환경에서, IoT 장치의 주소 지정 및 연결을 가능하게 하여 IoT 장치를 방화벽 뒤에서 안정적으로 사용할 수 있도록 함. 

