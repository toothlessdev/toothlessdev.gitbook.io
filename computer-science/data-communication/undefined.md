# 데이터 통신이란?

> 이 포스트는 경북대학교 백호기 교수님의 데이터통신 과목을 듣고 정리한 글입니다.

## 📖 데이터 통신 Data Communication 이란 ?

### ✏️ Data Communication?

> Data Communication is the exchange of data between two devices via some from of <mark style="background-color:orange;">transmission media.</mark> It depends on four characteristics.
>
> 데이터 통신은 <mark style="background-color:orange;">전송매체</mark>를 통한 두 기기 사이의 데이터 교환입니다.

데이터 통신은 4가지 특징들에 영향을 받습니다

### ✏️ 데이터 통신에 영향을 미치는  4가지

1. Delivery : 잘 전달되는가 ?
2. Accuracy : 정확하게 전달되는가 ?
3. Timeliness : 응답시간은 빠른가?
4. Jitter :  양방향 통신시 데이터가 일정하게 흐르는가 ? (안정적이고 혼선이 없는가?)

👉 X축이 패킷이 들어오는 시점일 때, 패킷 3개를 기준으로 하여 Jitter 값이 측정됍니다.\
👉 Jitter = T2 - T1 으로 계산이 되고, 값이 0에 가까울 수록 안정적입니다.

<figure><img src="../../.gitbook/assets/image (4) (1).png" alt="" width="563"><figcaption><p>Jitter = T2 - T1</p></figcaption></figure>

### ✏️ Data Communication Components 데이터통신을 이루는 구성요소

데이터 통신을 이루는 구성요소는 총 5가지로,\
Message, Sender, Receiver, Transmission Medium, Protocol 로 구성되어 있습니다.

1. Message : 통신을 하는 데이터, 정보로 주로 0과 1로 구성된 연속된 bits 를 말합니다.
2. Sender&#x20;
3. Receiver
4. Transmission Medium : 물리적 신호를 보내는 매체로, 유선 / 무선으로 생각하면 됍니다.
5. <mark style="background-color:orange;">Protocol</mark> : 전송 규약으로 쉽게 말해 뭐라고 물으면 뭐라고 대답해야할지 규칙이라고 생각하면 됍니다.

<figure><img src="../../.gitbook/assets/image (1) (1) (1).png" alt="" width="563"><figcaption></figcaption></figure>

### ✏️ Data Flow&#x20;

두 Device 사이의 통신은 Simplex, Half-Duplex, Full-Duplex 로 분류할 수 있습니다.

1. <mark style="background-color:orange;">Simplex</mark> : 단방향 통신입니다. 두 Device 는 보내거나 받는 둘중 하나의 역할을 합니다.
2. <mark style="background-color:orange;">Half-Duplex</mark> : 한 순간에는 단방향인 통신입니다. 두 Device 는 보내거나 받을 수 있지만, 동시에는 불가능합니다.
3. <mark style="background-color:orange;">Full-Duplex</mark> : 양 방향 동시 통신입니다.



## 📖 Network 네트워크란?

### ✏️ Network ?

> A network is the interconnection of a set of devices capable of communication
>
> 네트워크는 통신가능한 Device, 기기 사이의 연결입니다.

### ✏️ Physical Structures&#x20;

연결 타입에 따라...

1. Point-To-Point : Device 가 1:1로 연결된 구조입니다
2. MultiPoint : 1:1 을 제외한 연결 구조입니다

### ✏️ Topology&#x20;

물리적으로 네트워크가 어떻게 구성되어있는가...

Network 의 Topology 는 네트워크 Device (Node) 들의 Link, 연결관계를 Geometric Representaion 기하학적으로 표현한 것 입니다.

<figure><img src="../../.gitbook/assets/image (3) (1) (1).png" alt=""><figcaption></figcaption></figure>

### ✏️ LAN & WAN

LAN (Local Area Network) 는 어떤 사무실, 건물, 캠퍼스 등 소규모의 네트워크로 <mark style="background-color:orange;">크기가 제한</mark>되어 있습니다.\
WAN (Wide Area Network) 는 LAN 보다 더 <mark style="background-color:orange;">넓은 지역</mark>에 속하는 네트워크입니다.

LAN 들이 묶여 WAN 이 됍니다.

WAN 의 종류에는 Point-To-Point WAN 과 Switched WAN 이 있습니다.

1. Point-To-Point WAN : 두 Communicating Device 가 1:1로 연결되어 있습니다.
2. Switched WAN : 두개 이상의 종점이 있는 네트워크입니다.

