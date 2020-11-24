# SDN

필요 이유

1. 운영 자동화화 중앙관리의 어려움

   - 장비별 설정을 일일히 확인해야함

   - 변경하기위한 테스트를 미리 할수 없음

2. 효율과 비용 문제

   - 2계층의 STP, 3계층의 OSPF 모두 효율적으로 안좋음,

   - 따라서 네트워크 설계시 고비용 지출.

3. 개별 처리로 인한 네트워크 복잡성 증가

   - 장비 생산 벤더마다 RFC에 근거하지만 구현하는 방식에 특성이 있어 다른 기종 간 호환성 이슈가 발생하는 경우가 잦음.



구조

![What is SDN: Software Defined Networking » Electronics Notes](README%20assets/sdn-software-defined-networking-architecture-01.svg)



Control Plane Interface ----> Openflow로 예시를 둘 수 있음

## 가상화 종류 3가지



1. **호스트 가상화**

   호스트 가상화는 Base가 되는 Host OS위에 Guest OS가 구동되는 방식이다. 종류로는 VM Workstation, VMware Player, Virtual Box등이 있다.

   ![img](README%20assets/Image0.png)

   - 장점 : 가상의 하드웨어를 에뮬레이팅하기 때문에 호스트 운영체제에 크게 제약사항 없음

   - 단점 : OS위에서 OS가 얹히는 방식이기 때문에 오버헤드가 클 수 있음

     

2. **하이퍼바이저 가상화**

   하이퍼가상화는 Host OS 없이 하드웨어에 하이퍼바이저를 설치하여 사용하는 박식이다. 종류로는 Xen, MS hyper-V등이 있다.

   ![img](README%20assets/Image.png)

   - 장점 : 별도의 Host OS가 없기 때문에 오버헤드가 적고, 하드웨어를 직접 제어하기 때문에 효율적으로 리소스를 사용할 수 있음

   - 단점 : 자체적으로 머신에 대한 관리 기능이 없기 때문에 관리를 위한 컴퓨터나 콘솔이 필요함

     

     1. 전가상화 (Full-Virtualization)

        전가상화는 하드웨어를 완전히 가상화하는 방식으로 Hardware Virtual Machine이라고도 불린다. 하이퍼바이저를 구동하면 DOM0라고 하는 관리용 가상 머신이 실행되며, 모든 가상 머신들의 하드웨어 접근이 DOM0을 통해 이루어진다. 즉, 모든 명령에 대해 DOM0가 개입하기 때문에 OS가 뭐든지간에 각 OS들이 내리는 명령어를 알아들을 수 있다.

        - 장점 : 하드웨어를 완전히 가상화하기 때문에 Guest OS 운영체제의 별다른 수정이 필요 없음
        - 단점 : 하이퍼바이저가 모든 명령을 중재하기 때문에 성능이 비교적 느림

        

     2. 반가상화 (Para-Virtualization)

        반가상화는 하드웨어를 완전히 가상화하지 않는다. 전가상화의 가장 큰 단점인 성능저하의 문제를 해결하기 위해 하이퍼콜(Hyper Call)이라는 인터페이스를 통해 하이퍼바이저에게 직접 요청 가능. 각 OS들이 각각 다른 하이퍼콜을 가짐.

        - 장점 : 모든 명령을 DOM0를 통해 하이퍼바이저에게 요청하는 전가상화에 비해 성능이 빠름
        - 단점 : 하이퍼바이저에게 Hyper Call 요청을 할 수 있도록 각 OS의 커널을 수정해야하며 오픈소스 OS가 아니면 반가상화를 이용하기가 쉽지 않
        - 
        - 
        - 음

     

3. **컨테이너 가상화**

   호스트 OS위에 컨테이너 관리 스프트웨어를 설치하여, 논리적으로 컨테이너를 나누어 사용한다. 컨테이너는 애플리케이션 동작을 위한 라이브러리와 애플리케이션등으로 구성되기 때문에 이를 각각 개별 서버처럼 사용이 가능하다.

   ![img](README%20assets/Image2.png)

   - 장점 : 컨테이너 가상화는 오버헤드가 적어 가볍고 빠른 장점이 있음

## OpenFlow

#### OpenFlow Protocol Message

##### Type

1. **Controller-to-Switch** : 스위치의 상태나 관리를 점검하기 위해 사용

   - Features : Openflow Channel을 형성할 때 사용, 스위치 용량 확인
     - Feature Request : Controller -> Switch , 응답요청
     - Feature Reply : Switch -> Controller, 응답보고
       - 수행가능한 Action 값, Port 연결속도, Duplex 정보등등을 담고 있음
   - Configuration 
   - Packet Out : Packet In Message 통해 스위치로 부터 수신한 패킷을 해당 스위치 상의 특정한 포트로 전송하기 위해 사용
   - Barrier

2. **Asynchronous** : 스위치에서 생성하는 메시지, 스위치의 상태 동기화를 위해 사용

   - Packet In : 스위치에 해당 Flow Table이 없으면 이를 Controller에 보내 제어를 받기 위해 사용

   - Flow-Removed : Flow Table에서 삭제할 Flow Entry 정보를 Controller에게 주기 위해 사용

     ​	A. Controller -> Switch : 해당 Flow에 대한 삭제 요청

     ​	B. Flow Timeout 으로 인한 Flow Expiry Process 

3. **Symmetric** : Controller&Switch 모두 생성, 상대방의 요청 없이도 전송

   - Hello : Controller -- Switch 간에 연결을 시작할때 사용
   - Echo :  Controller -- Switch 간 연결에 이상없음을 확인하기위해 사용
     - Echo에 대해 답변으로 Echo Reply를 전송
   - Experimenter



#### Controller -- OpenFlow Switch 간 Session 연결 순서

1. TCP Session 수립
2. Hello 교환
3. Feature Negotiation
4. Configuration Message 교환
5. Stats Message 교환
6. Vendor Message 교환
7. Flow Mod 전송
8. Stats Message 교환
9. Echo Message 교환



이때까지 Switch -- Controller 연결을 알아봤다면 이제는 Switch간의 연결 정보를 알아봄



Switch 포트의 status 정보를 Switch --> Controller 보내면서 Topology Discovery가 이루어짐

#### Topology Discovery 순서

1. Switch의 포트가 Down 에서 Up으로 변경된다.
2. 이 포트의 변경 사항을 Switch --> Controller
3. Controller-->Switch LLDP가 Switch의 변경된 포트를 통해 전송되도록 Packet Out Message로 생성하여 전송
4. Switch-->SwitchB Packet Out Message에 담긴 LLDP를 인접 SwitchB에게 전달
5. SwitchB --> Controller LLDP를 받은 인접한 SwitchB는 해당 LLDP를 어떻게 처리해야 할지 모르기 때문에 이 Packet을  Controller에게 Packet In Message로 보냄
6. Controller는 이를 통해 스위치 간 연결정보 확인



2번부터의 포트 변경 사항 전송 부분 Detail

1. Switch --> Controller , Interface Up/Down 이벤트를 알리기 위해 IF discovery 메시지(Port Status Message) 전송
   - Port Status Message 생성 : Source - Switch's IP, Destination - Controller's IP 으로 Switch에서 생성되어 전달
   - Port Status Message 정보 : Port 번호, MAC 주소 정보, 그외 다양한 기능정보
2. Controller --> Switch, Controller는 받은 Switch의 인터페이스를 통해 LLDP 패킷 전송되도록 Packet Out Message 전송
   - Packet Out Message 생성 : Source - Controller's IP, Destination - Switch's IP 으로 Controller에서 생성되어 Switch의 특정 Port로 패킷 전달 목적
   - Packet Out Message 에 Output Action 지정 : LLDP 패킷이 스위치를 통해 잘 전달되도록
3. Packet Out Message 수신한 Switch : 내부의 LLDP 패킷을 지정된 Output Action에 따라 처리
4. Switch -> SwitchB , LLDP를 받은 이웃한 SwitchB : unknown packet, Controller에게 알리자
5. SwitchB -> Controller, Packet In Message 전송
6. Controller  : 전체 네트워크 Topology 인식 완료



위의 파악된 Topology를 통해 

#### Controller는 SPF 알고리즘을 이용하여 최적 경로 계산 후 통신 안내 순서

0. Switch간의 Topology 파악 완료.

1. Host A로부터 패킷 들어옴, Packet In Message를 이용하여 Controller에 전송
2. Controller는 MAC과 IP Table을 업데이트하고 최적 경로 연산
3. Controller는 최적 경로 상에 있는 Switch에 Flow Table이 생성되도록 Flow Modification Message 전송
   - Flow Modification Message 생성 : Controller에서 생성, Source - Controller's IP, Destination - Switch's IP
   - Flow Modification Message 정보 : flow를 match할 조건 값들과 flow entry에 대한 설명, Output Aciton 값등 포함

 

#### 장애시 우회 경로 확보 방법

기존 Legacy 환경에서는 STP 등의 원인으로 상당 시간동안 네트워크 단절됨. OpenFlow를 사용하면 매우 빠른 절제 시간.

1. Port가 down되면 Switch는 Controller에 Port Status Message를 생성하여 전송
2. Controller는 Topology 수정 후 다른 경로 계산
3. Switch에게 Flow Modification Message 를 전송
4. Flow Modification Message의 command field를 보면 flow entry의 Output Action을 수정하는것을 볼 수 있음

# Protocol

1. ARP
2. STP O
3. IGP
   1. RIP X
   2. OSPF O
   3. IGRP X
4. EGP
   1. BGP X



## ARP (Address Resloution Protocol)

주소 결정 프로토콜 == ARP는 네트워크 상에서 IP 주소를 물리적 네트워크 주소로 bind시키기 위해 사용되는 프로토콜이다. 즉, IP주소는 알겠는데 MAC주소를 모를때 호스트 혹은 라우터의 MAC주소를 알기 위해 해당 노드의 IP 주소를 타겟으로 Target IP address 필드에 타겟 주소를 넣어서 ARP Request를 보내고 해당 노드는 자신의 MAC을 포함하여 ARP Reply를 보냄.

호스트/라우터의 ARP Table(ARP cache)를 갱신



## STP (Spanning Tree Protocol)

L2 계층의 Ethernet Frame이 장비들에서 빙빙 도는 것을 Looping(루핑)이라 한다. 이 루핑을 방지시켜 주는 것이 STP이다.

(참고1 : L3 계층에서 사용하는 IP Packet은 Header에 *TTL Field*가 있어 Packet의 루핑을 막아준다.)

(참고2 : IEEE 802.1D-2004 standard 에서 STP -> RSTP(Rapid STP)로 대체되었지만 현재도 다수의 스위치들은 STP를 사용하고 있다.)



![img](README%20assets/99902F365E5358840A)



STP가 없을시에는 Broadcast Frame이 모든 포트로 Flooding 되면 앞 뒤로 Broadcast Frame의 Looping이 일어나게 되어 결과적으로 엄청난 브로드캐스팅 폭풍이 발생되어 이 Frame을 처리하느라 정상적인 동작이 어려워 속도가 저하되거나 시스템이 다운된다.



STP가 동작하면 물리적으로 Loop 구조인 Network에서 특정 Port를 차단 상태로 바꾸어 놓기 때문에 Loop가 발생하지 않는다. 그러다 다른 동작 중인 스위치의 Port가 Down되면, 차단 상태로 바꿔놓은 특정 Port를 다시 전송 상태로 바꾸어 통신이 끊기지 않도록 한다. 

그 Loop를 막는 경로를 구성하는 프레임을 BPDU(Bridge Protocol Data Unit)라고 한다.

BPDU는 Configuration BPDU(설정 BPDU)와 TCN BPDU(Topology Chance Notification BPDU) 2가지가 있다.

1. Configuration BPDU : Switch 및 Port의 역할을 결정하기 위하여 사용되는 Frame
2. TCN BPDU : Switch Network의 구조가 변경되었을 때 알리기 위하여 사용되는 Frame

Switch 는 Configuration BPDU를 2초마다 전송하여 Root Switch를 선출하고, Switch Port의 역할을 결정하고, 그 BPDU를 인접 Switch로 중계한다.

Configuration BPDU에는 Bridge ID, Root Bridge ID, 경로값, Port ID 등과 Timer등이 포함되어 있다.



단점

1. 장애 복구 시간이 오래 걸림

   ​	STP는 장애 복구 시간이 50초, RSTP 는 3초 but 이론적인 시간이며 대량 트래픽이 흐르는 경우 수분 이상, 	이로 인해 통신 끊어질 확률 높음

2. 링크 한쪽을 끊는 것이기 때문에 비효율적 회선 사용

   ​	구성된 자원의 50%만 사용



## RIP (Routing Information Protocol)



## OSPF (Open Shortest Path First)

![Understanding OSPF External Route Path Selection](README%20assets/ospf_external_routing-1.png)



RIP 라우팅 프로토콜이 망간 라우팅이 비효율적이여서 나옴.

OSPF는 L3계층의 IP 네트워킹에서 사용하며 최적 경로를 위해 Dijkstra의 SPF(Shortest Path First) 알고리즘을 이용하여 계산하는 Link-state 계열의 라우팅 프로토콜이다.

IGP(Interior Gateway Protocol)에 속하며 동일 자치시스템(AS, Autonomout System) 내에 있는 라우터끼리만 라우팅

(참고1. AS 내에서의 라우팅 프로토콜인 IGP, Interior Gateway Protocol 와 AS 간의 라우팅 프로토콜인 EGP, External Gateway Protocol 로 분류됨.)



장점

1. AS 내부에 Area 개념을 두어 계층적 라우팅을 구현함으로써, 라우팅의 트래픽을 줄여줌. 
   Backbon Area가 핵심
2. 대규모 네트워크에 적합함.
3. RIP에 비해 Hop Count 제한이 없고 Convergence(수렴) 시간이 빠름
4. 네트워크 변화가 있을때만 정보가 Multicast로 날아가므로 경제적. RIP는 30초마다 Broadcast 트래픽 발생



단점

1. Router의 내부 Resource 소모가 많다.
2. CPU - 잦은 SPF 알고리즘 수행한다.
3. Memory - Network Topology 정보 관리가 필요하다.
4. 반드시 계층적 Design Rule을 따라야 한다.



## IGRP

## BGP





## REST

HTTP URI를 통해 자원을 명시, HTTP Method를 통해 자원에 대한 CRUD Operation을 적용을 의미. 

**상태/정보 전달** : 일반적으로 XML, HTTP, JSON 등의 형태로 전달



 **CRUD Operation**

| **Create** |      **생성 (POST)**       |
| :--------: | :------------------------: |
|  **Read**  |       **조회 (GET)**       |
| **Update** |       **수정 (PUT)**       |
| **Delete** |     **삭제 (DELETE)**      |
|  **Head**  | **header 정보 조회(HEAD)** |

 

**REST에 적용되어야하는 조건**

- Server/Client 구조
- 무상태(Stateless)
- 캐시처리가능(Cacheable)이 필요
- 계층화(Layered System)
- Code-On-Demand도 있지만 무조건은 아님.
- 인터페이스의 일관성(Uniform Interface) 요구

 

**REST의 구성 요소**

- 자원(Resource) : URI
- 행위(Verb) : HTTP Method
- 자원의 표현(Representation of Resource) : 요청에 대한 적절한 응답(JSON, XML 등)

 

#### REST API

: REST 기반의 API

- REST 기반으로 시스템을 분산하면 확장성과 재사용성을 높여 유지보수 및 운용을 편리하게 할 수 있음
- HTTP 표준을 기반으로 구현하니 HTTP를 지원하는 모든 언어들로 Server, Client를 구현 가능

 

1. 슬래시(/)는 계층 관계를 나타내는 곳에 사용

```
http://blog.naver.com/keunsoo
```

2. URI 마지막 문자로 슬래시(/)를 포함하지 않는다.

```
ex) http://blog.naver.com/	(x)
ex) http://blog.naver.com	(o)
```

3. 하이픈(-)은 URI의 가독성을 높이는데 사용한다.

4. 언더바(_)는 URI에 사용하지 않는다.
5. URI 경로에는 소문자가 적합하다. (대소문자에 따라 다른 리소스로 인식하기 때문)

```
RFC 3986 is the URI (Unified Resource Identifier) Syntax document
```

6. 파일확장자는 URI에 포함시키지 않는다.

```
ex) http://blog.naver.com/test/example.jpg  (x)
ex) GET / test/example HTTP/1.1 Host:blog.naver.com Accept: image/jpg  (o)
```

7. 리소스 간의 관계를 표현하는 방법이 있어요.

```
/리소스명/리소스ID/관계있는 다른 리소스명
ex) GET : /users/{userid}/devices
```



**RESTful**

: REST 아키텍처를 구현한 웹 서비스를 나타내는 용어