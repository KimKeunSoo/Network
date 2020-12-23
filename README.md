# SDN

Control Plane과 Data Plane이 물리적으로 분리된 아키택쳐이다.

Control Plane : SW적인 요소들은 Controller Application으로 동작하는 개념

Data Plane : HW적인 관점으로 Control Plane에 의해 통제를 받아서 동작하는 개념



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
        - 단점 : 하이퍼바이저에게 Hyper Call 요청을 할 수 있도록 각 OS의 커널을 수정해야하며 오픈소스 OS가 아니면 반가상화를 이용하기가 쉽지 않음
     
     
   
3. **컨테이너 가상화**

   호스트 OS위에 컨테이너 관리 스프트웨어를 설치하여, 논리적으로 컨테이너를 나누어 사용한다. 컨테이너는 애플리케이션 동작을 위한 라이브러리와 애플리케이션등으로 구성되기 때문에 이를 각각 개별 서버처럼 사용이 가능하다.

   ![img](README%20assets/Image2.png)

   - 장점 : 컨테이너 가상화는 오버헤드가 적어 가볍고 빠른 장점이 있음



## OpenFlow

: Controller(Control Plane)과 Switch(Data Plane)간 서로 통신할 수 있는 표준 규격.

#### OpenFlow Protocol Message

##### Type

1. **Controller-to-Switch** : 스위치의 상태나 관리를 점검하기 위해 사용

   - Features : Openflow Channel을 형성할 때 사용, 스위치 용량 확인
     - Feature Request : Controller -> Switch , 응답요청 / 어느 포트가 사용가능한지 물어봄
     - Feature Reply : Switch -> Controller, 응답보고 
       - 수행가능한 Action 값, Port 연결속도, Duplex 정보등등을 담고 있음
   - Configuration (Set Config) : Controller -> Switch, flow expirations 를 보내기 위함.
   - Packet Out : Packet In Message 통해 스위치로 부터 수신한 패킷을 해당 스위치 상의 특정한 포트로 전송하기 위해 사용
   - Barrier

2. **Asynchronous** : 스위치에서 생성하는 메시지, 스위치의 상태 동기화를 위해 사용

   - Packet In : 스위치에 해당 Flow Table이 없으면 이를 Controller에 보내 제어를 받기 위해 사용

   - Flow-Removed : Flow Table에서 삭제할 Flow Entry 정보를 Controller에게 주기 위해 사용

     ​	A. Controller -> Switch : 해당 Flow에 대한 삭제 요청

     ​	B. Flow Timeout 으로 인한 Flow Expiry Process 

3. **Symmetric** : Controller&Switch 모두 생성, 상대방의 요청 없이도 전송

   - Hello : Controller -- Switch 간에 연결을 시작할때 사용, TCP handshake에 따라서 Controller는 Switch에게 버전 번호를 보내고 Switch는 가능 버전 번호를 응답합니다.
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

### OpenFlow의 Flow Table

Flow Table의 entry는 Header fields, Counters, Actions 필드로 구성되어 있음

Controller는 관리를 하기 전에 패킷 정책에 대한 Flow Table을 내림



1. Header Fields

   ![클라우드 네트워크 관리 기술, OpenFlow](README%20assets/helloworld-387756-4.png)

   - Ingress Port : 수신 포트, 포트번호 1부터 시작하여 표현함
   - Ethernet source address
   - Ethernet destination address
   - Ethernet type : 802.2의 SNAP 헤더와 OUI / 802.3의 SNAP 헤더가 없는 0x05FF 이 일치해야함
   - VLAN id
   - VLAN priority : VLAN PCP field
   - IP source address
   - IP destination address
   - IP protocol(8) : ARP 옵션코드의 경우 하위 8비트만이 사용됨
   - IP ToS bits(6) : 8비트 값으로 지정하고 ToS를 상위 6비트에 배치
   - TCP/UDP같은 전송 sorce port(16) : ICMP 타입일 경우 하위 8비트만 사용
   - TCP/UDP같은 전송 destination port(16) : ICMP 타입일 경우 하위 8비트만 사용



2. Counters

   Counter는 table, flow, port, queue 별로 관리함

   상세 설명 생략

3. Actions

   상세 설명 생략

   

# Protocol

1. ARP
2. STP O
3. IGP
   1. RIP X
   2. OSPF O
   3. IGRP X
4. EGP
   1. BGP X
5. SIP X

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



## SIP

SIP(Session Initiation Protocol)는 '세션 설정 프로토콜'이다. RFC 3261로 권고되어있으며, 멀티미디어 세션의 생성, 변경, 종료에 대한 응용 계층의 프로토콜이다. 멀티미디어 데이터 전송 자체보다는 Signaling을 통한 멀티미디어 통신 관리인 세션 제어에 중점을 두고 있음. URL 및 E-mail 형식의 텍스트 기반 어드레싱 방법을 사용하므로 메시지 파싱이나 확장이 용이하다. 보통은 UDP를 주로 사용하지만 현재는 TCP를 더 많이 사용함. 5060과 5061 포트 이용

![img](README%20assets/R1280x0)

SIP 메시지는 가변 길이의 텍스트로 만들어지며 SIP Header와 Message Body로 구성됨.

- SIP Header :  뒤에 올 Message Body의 종류를 표시
- Message Body : 옵션 필드로 있을 수도 있고 없을 수도 있음.



![img](README%20assets/133BC5144B40801513)

SIP 프로토콜 스택은 위와 같다. TCP/UDP 상위에 SIP, SDP, RTP등이 올라와있음. 

- SDP(Session Description Protocol) : 멀티미디어 세션 파라미터 설정
- Audio / Video Codec : 음성/영상 코딩 담당, 다양한 시스템과 호환을 위해 여러 규격 존재
- RTP/RTCP(Realtime Transport (Control) Protocol ) : 실시간 통신



![img](README%20assets/203D0F1D4B4084ADDD)

SIP 구성요소간 상호 관계는 다음과 같다. 구성요소는 두가지로 나뉘어진다.

- SIP 클라이언트
  - UAC(User Agent Client) : 세션 종단에 위치하며 Call을 생성하고 설정 요청
  - UAS(User Agent Server) : UAC로부터 Call을 수락하거나 거절 또는 Redirect
- SIP 서버 : UA간 직접 호츨이 가능하지만 SIP 서버를 둠으로써 확장성을 제공
  - Proxy Server : UAC로부터 SIP Call을 받아 자신이 Call을 대신 만들어줌
  - Register Server : User Agent로 부터 레지스터 요청을 수신하여 사용자의 위치 정보 유지
  - Redirect Server : 사용자가 직접 요청을 할 수 있는 상대방의 URL을 알려줌
  - Location Server : Proxy Server나 Redirect Server로부터 SIP Call의 목적지 노드의 주소가 요청되면 이를 Resolution 해주는 역할을 함

*세무 설명 SIP 메시지나 세션 흐름 생략



## SDP

: Session Description Protocol, 멀티미디어 세션 파라미터를 협상하는 프로토콜.



SIP 프로토콜이 시그널링에 대해 정의, SDP 프로토콜이 Capability Exchange에 대해 정의함.

SIP는 요청&응답 ( Request & Response ) 모델로 정의하였음.

SDP는 제안&수락 ( Offer & Answer ) 모델로 정의하였음.

SDP는 SIP 뿐만 아니라 MGCP와 Megaco에서도 사용함.



![image-20201222173957606](README%20assets/image-20201222173957606.png)

SDP는 Capability를 협상하기 위해 SIP call flow 절차를 활용함. SIP의 요청&응답 ( Request & Response ) 에서 사용되는  SIP 메시지 바디에 포함되어 전달함. 예를 들어, SIP INVITE 메시지에 SDP Offer가 포함되고 200 OK 응답 메시지에 SDP Answer가 포함됨.



#### SDP 메시지 분석

SDP는 멀티미디어를 전달하는 RTP 프로토콜에 대한 세부적인 내용을 협상함. SDP는 SIP와 다른 메시지 포맷을 사용하지만 텍스트 기반이므로 이해하기 쉬움.

```
v = 0
o = alice 2890844526 2890844526 IN IP4 atlanta.com
s =
c = IN IP4 10.1.3.33
t = 0 0
m = audio 49172 RTP/AVP 0 8 18 101
a = rtpmap:0 PCMU/8000
```

1. ##### v = 0 (필수)

   - SDP 프로토콜의 버전을 표시, SDP 버전은 0

     

2. ##### o = alice 2890844526 2890844526 IN IP4 atlanta.com (필수)

   - SDP 메시지를 생상한 Owner/Creator를 표시

   - 순서대로 Username, Session-ID, Session Version, Network Type, Address Type, Unicast Address

     

3. ##### s = (필수)

   - 세션 이름 표시

     

4. ##### c = IN IP4 10.1.3.33 (옵션)

   - RTP 프로토콜이 사용할 주소를 정의

   - 순서대로 Network Type, Address Type, Connection-Address

     

5. ##### t = 0 0 (필수)

   - Timing으로 start-time과 stop-time을 표시.

   - 0 0 은 고정 세션을 의미

     

6. ##### m = audio 49172 RTP/AVP 0 8 18 101

   - 어떤 Media를 쓸 것인지에 대한 설명
   - 순서대로 Media, Port, Protocol, Format 
   - Media : RTP 프로토콜의 페이로드가 무엇인지를 선언
     - audio, video, text, application, message 중에 표시
   - Port : 미디어가 전송될 전송 포트 표시
     - UDP 16384 ~ 32767 사이의 번호를 무작위로 선택
   - Protocol : UDP, RTP/AVP, RTP/SAVP 중에 표시
     - AVP는 Audio Video Profile 의 약자
   - Format : 미디어의 포맷을 서브 필드 a=으로 표시함을 의미
     - Payload Type 0 8 18의 순서는 코덱 협상의 우선순위를 표시
     - Payload Type 101은 DTMF 이벤트를 정의
     - 여기서 0 8 18 101이 있으므로 세개의 우선순의의 미디어 속성이  3개의 a=rtpmap으로 표시되고 각자의 미디어 패킷 한개가 포함한 시간 정보인 a=ptime 3개가 있으며, 마지막에는 a=fmtp로 DTMF인 미디어 포맷에 대한 파라미터를 정의 

7. ##### a = rtpmap:0 PCMU/8000

   - 생략. . .  등등





### 결론

![image-20201222180348753](README%20assets/image-20201222180348753.png)

결국,  여러개의 코덱 중 하나를 선택하는 과정에서 SDP와 SIP를 사용한다. 



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





# IEEE 802.11

|                            | **IEEE  802.11n**        | **IEEE  802.11ac**  | **IEEE  802.11ax**  | **IEEE  802.11ax**  |
| -------------------------- | ------------------------ | ------------------- | ------------------- | ------------------- |
| Spectrum Bands             | 2.4GHz & 5GHz            | 5GHz                | 2.4GHz & 5GHz       | 2.4, 5 & 6 GHz      |
| Channel bandwidth (MHz)    | 20, 40                   | (n), 80, 80+80, 160 | (ac)                | (ax), 320           |
| Subcarrier spacing (KHz)   | 312.5                    | 312.5               | 78.125              | 78.125              |
| Symbol time (us)           | 3.2                      | 3.2                 | 12.8                | 12.8                |
| Guard interval (us)        | 0.8                      | 0.8, 0.4            | 0.8, 1.6, 3.2       | 0.8, 1.6, 3.2       |
| MU-MIMO                    | No                       | Downlink            | Uplink and Downlink | Uplink and Downlink |
| MIMO order                 | 4                        | 8                   | 8                   | 16                  |
| Modulation                 | OFDM                     | OFDM                | OFDM, OFDMA         | OFDM, OFDMA         |
| Data subcarrier modulation | BPSK, QPSK, 16QAM, 64QAM | (n), 256QAM         | (ac), 1024 QAM      | (ax), 4K QAM        |





# RTP

**RTP(Real-time Transport Protocol)**은 **IP 네트워크 상에서 오디오와 비디오를 전달하기 위한 통신 프로토콜**이다. 전화, 그리고 WebRTC, 텔레비전 서비스, 웹 기반 push-to-talk 기능을 포함한 화상 통화 분야 등의 스트리밍 미디어를 수반하는 통신, 엔터테이먼트 시스템에 사용된다.

일반적으로 UDP위에서 동작하며 RTCP(RTP Control Protocol)와 결합하여 사용. RTP는 오디오/비디오와 같은 미디어 스트림을 전달하는 반면, RTCP는 전송 통계와 QoS를 모니터링하고 다중 스트림의 동기화를 도와준다. 

SIP(Session Initiation Protocol)과 같은 시그널링 프로토콜과 결합해서 사용하기도 한다.

![img](README%20assets/1381_1.JPG)

1. IP/UDP에 실린 RTP 구조
   ![img](README%20assets/3394_1.JPG)

   

2. RTP 패킷 구조
   ![img](README%20assets/3394_2.JPG)

   

3. RTP 패킷 각 필드별 설명

   - 제어 비트 (9 비트)
     - Ver : 2 비트
       현재 RTP 버젼은, 2 (RFC 3550)

     - P (padding) : 1 비트
       1 : 실제 유료부하 끝에 덧붙여진 패딩 데이터 있음
       	응용프로그램이 32 비트 같은 정수배 단위로 RTP 패킷 페이로드 구성을 위함

     - X (extension)   : 1 비트
       1 : 가변길이 헤더 확장(Extension Header)이 있음을 나타냄

     - CC (CSRC Count) : 4 비트
       기본 헤더 바로 뒤에 나타나는 CSRC(Countributing SouRCe) ID의 갯수
       여러 미디어가 합성되는 경우에, 그 개수를 CC로써 나타내고, 모두의 기준 동기를 맞추려면 SRRC ID로써 이를 나타냄

     - M (Marker)      : 1 비트
       . 이벤트 발생이 시작되었음을 알림

       

   - Payload type : (7 비트)  오디오/비디오 인코딩(코덱) 종류

     - 오디오 타입 번호 (예시)
       . 0 -> G.711 PCM(mu-law), 샘플링주파수 8000 Hz
       . 3 -> GSM,               샘플링주파수 8000 Hz
       . 4 -> G.723,             샘플링주파수 8000 Hz
       . 6 -> DVI4 (ADPCM),      샘플링주파수 16000 Hz
       . 7 -> LPC,               샘플링주파수 8000 Hz
       . 8 -> G.711 PCM(A-Law)   샘플링주파수 8000 Hz
       . 9 -> G.722,             샘플링주파수 8000 Hz
       . 14 -> MPEG 오디오,      샘플링주파수 90000 Hz
       . 15 -> G.728,            샘플링주파수 8000 Hz

     - 비디오 타입 번호 (예시)
       . 26 -> 화상 JPEG, 31 -> H.261, 32 -> MPEG-1 또는 MPEG-2 비디오, 
       . 33 -> MPEG-2 TS 등

     - 기타 임의 지정 가능(dynamic payload type) : 96~127

     - RTP 내 Payload type 표준 목록 ☞ IANA RTP Parameters
           . RFC 3551에서 오디오 신호/비디오 신호의 인코딩 방법,샘플링 주파수 등이 기술됨

       

   - Sequence number : (16 비트)

      - 패킷 손실 검출 및 순서 재구성 
      
      - 초기값은 랜덤이고, 매 패킷 마다 1씩 증가
        
      - 수신측은 패킷 재전송 요청 보다는 패킷 손실 검출 및 뒤바뀐 순서 복구를 위함
      
         
      
   - Timestamp : (32 비트)
      P 스트림 내 각 RTP 패킷이 샘플링된 시간관계를 나타냄
      랜덤한 초기값부터 시작하며, 통상적으로 카운터에 의해 1씩 증가시킴

      - 타임스탬프 간격은 Payload Type에 따라 정해진 샘플링 간격을 기준 

      - 대부분의 오디오 RTP 패킷의 경우 => 1 패킷 당 디폴트 시간 간격을 20 ms으로 함
                예시) G.711 (PCM A-Law) 오디오 페이로드 패킷 크기 
                         = (유료부하 코덱 데이터율) x (1 패킷 당 시간 간격)
                         = (64 kbps G.711 코덱) x (20 ms)
                         = (8000 samples x 8 bits)/sec x (0.02 sec)
                         = 160 바이트    

      - 타임스탬프 값의 연속성 의미 구분

         - 예1) 일련의 패킷들의 타임스탬프 값이 같은 경우 : 특정 비디오 장면이 같은 시간에 샘플링되었음을 의미

         - 예2) 일련의 패킷들의 타임스탬프 값이 단순 증가하지 않는 경우 : MPEG 화면 픽처 처럼 시간 순서가 어긋나며 전후 화면으로부터 예측되었음을 의미

         - 예3) 일련의 패킷들의 타임스탬프 값이 연속 증가되는 번호 순서를 갖음 : 오디오 패킷 흐름일 경우 등

            

   - 동기 발신 식별자 (SSRC ID, Synchronization SouRCe ID) : (32 비트)

      - 원래의 정보 스트림에 대한 식별 (즉, RTP 세션에서 소스 구분하는 고유 번호)
             하나의 RTP 세션 내에서는, 
             		각각의 발송지는 무작위로 선택된 SSRC ID로 나타내고, 
             		다만, 타 발송지와의 구별을 위해 중복 불허하며, 
             		결국, 하나의 웹브라우저에서 2개 동영상 가능 등

   - 기여 발신 식별자 (CSRC ID, Contributor SouRCe ID) 목록 : (32 비트) 1 이상 가변 갯수

      - 믹서(Mixer)를 통해 혼합되어 단일의 정보열로 되면, 원래의 각각에 대한 식별 역할
                . 여러 미디어가 혼합되면, CC(CSRC Count:4 비트)에 총 개수를 지정하고, 
                . SSRC 외에, 추가된 스트림들에 대한 식별자를 CSRC ID 값으로 함

 



# 6LoWPAN

- 개념

  - IEEE 802.15.4를 PHY/MAC으로 하는 저전력 WPAN상의 인터넷 프로토콜 IPv6를 결합하여 USN 네트워크를 통합해 광범위한 확장성과 이동성을 보장하기 위한 기술 - IETF 6LoWPAN Working Group에서 추진중인 IP-USN 관련 표준으로 센서 네트워크와 IPv6 네트워크를 직접 연동하는 기술



**I.** **6LoWPAN****의 개요**

 가. 6LoWPAN(IPv6 over Low Power WPAN)의 정의

 \- IEEE 802.15.4를 PHY/MAC으로 하는 저전력 WPAN상의 인터넷 프로토콜 IPv6를 결합하여 USN 네트워크를 통합해 광범위한 확장성과 이동성을 보장하기 위한 기술

 \- IETF 6LoWPAN Working Group에서 추진중인 IP-USN 관련 표준으로 센서 네트워크와 IPv6 네트워크를 직접 연동하는 기술

 

 나. 6LoWPAN의 특성

 \- IP인프라를 기반으로 Sensor node, Gateway, Sync node등의 USN 통합

 \- USN의 저가격, 저전력 특성과 이동성, 광역성, 보안성을 결합

 

다. 6LoWPAN의 특징

![img](README%20assets/20160908135938671.png)

 라. 6LoWPAN의 표준기술 범위

![img](README%20assets/20160908135950146.png)

 

**II.** **6LoWPAN****의 구성도 및 스택 아키텍처**

 가. 6LoWPAN의 구성도

 ![img](README%20assets/20160908135957818.png)

-  외부 IP네트워크와 6LoWPAN 각각에 대해 16비트 주소를 Mapping하여 IPv6패킷을 압축
-  IEEE 802.15.4 MAC과PHY위에 적응계층(Adaptation Layer)으로서 단편화, 커미셔닝, ND(Neighbor Discovery) 최적화, 메쉬라우팅, IPv6, TCP/UDP, 소켓API, SNMP, 서비스 네이밍, 센서응용 등으로 구성
-  기존 IP 인프라를 활용
-  기존의 IP 네트워크에 seamless한 무선망과의 통합
-  2.4GHz ISM 대역에서 표준 IEEE 802.15.4
-  공동 주파수대 사용(2.4GHz)으로 인한 간섭 - 위피, 블루투스, 지그비
-  멀티 임베디드 애플 리케이션에서 배터리 수명 1 년
-  소프트웨어 API 개발이 간단
-  6LoWPAN 네트워크 스택은 자동으로 IEEE802.15.4 네트워크(최대 127 바이트)의 IPv6 패킷 (최대 1280 바이트) 전송 패킷을 처리

 

**III.** **Zigbee와 6LoWPAN의 비교**

| **구분**   | **Zigbee**                                                   | **6LoWPAN**                                                  |
| ---------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| MAC Layer  | IEEE 802.15.4 표준                                           |                                                              |
| 표준화단체 | Zigbee Alliance                                              | IETF 6Lo WPAN WG                                             |
| 주소       | 16비트 혹은 64비트                                           | IPv6주소                                                     |
| 주소유일성 | 하나의 PAN내에서 유일                                        | 전세계에서 유일                                              |
| 장점       | 상대적으로 간단함상용 서비스 제공 중                         | 토폴로지 확장성, 이동성 우수IPv6 네트워크 호환성통합망 관리용이 (SNMP V3) |
| 단점       | 토폴로지 확장성 낮음IPv6 네트웍 호환성 낮음통합망 관리 어려움 | 보안 고려중국가 Pilot 프로젝트 진행중                        |

 

**IV. 6LoWPAN과 IPv4망 연동**

![img](README%20assets/2016090814003696.png)

 

-  6LoWPAN을 지원하는 장치는 기존 망에 붙어서 하나의 네트워크 구성 요소로 동작하게 되며 이런 노드들이 무수히 많아 졌을 때   이것이 하나의 클라우드 형태로 전파가 되게 되고 장치기반으로의 인터넷 망의 확장으로 인하여 다양한 응용 분야를 만들어냄

| 구분             | 필요기술                                                     |
| ---------------- | ------------------------------------------------------------ |
| IPv4/IPv6 라우터 | - 내부 네트워크에서 IPv6를 지원하고 외부로 나가는 인터넷망은 IPv4를 지원 할 수 있도록 듀얼 스택을 지원- IPv4망과의 연동을 위한 주소 변환 기술들이 적용 |
| Edge 라우터      | - IPv6에서 IPv4를 거쳐 IPv6로 가기위한 터널링 기법이 필요    |

 

**V.** **6LoWPAN의 적용 및 전망**

 가. 6LoWPAN의 적용

 \- 6LoWPAN은 IP를 사용함으로써 기존의 구축된 통신 및 응용서비스 인프라를 그대로 이용할 수 있어서 비용이 절감

 \- 잘 알려지고 검증된 IP 기술들을 사용할 수 있어 신뢰성과 안정성을 도모

 \- LoWPAN에서는 기존 네트워크들에 비해 상당히 많은 수의 노드가 배치되어야 하므로 큰 주소 공간과 자동 주소설정과 같은 기능을 내장하고 있는 IPv6가 적합

 \- 6LoWPAN에서는 IPv4는 고려하지 않는다.

 

 나. 6LoWPAN의 전망

 \- 이동성 및 글로벌 네트워크 특징을 지닌 센서네트워크 시장 선점

 \- Zigbee 시장과 공존 및 차별화 전략 필요

 \- IP-USN 기술개발, 표준화, 시범사업이 동시에 진행되어야 함.

 

[참고] 6LoWPAN 프로토콜 스택

![img](README%20assets/2016090814011929.png)

![img](README%20assets/20160908140147574.png)

 

6LoWPAN 게이트웨이 구조

![img](README%20assets/20160908140158478.png)

 

헤더압축기법

![img](README%20assets/20160908140207427.png)

 

패킷분할

![img](README%20assets/20160908140215790.png)

 

주소자동설정

![http://upload.wikimedia.org/wikipedia/commons/thumb/4/44/6LoWPAN_comparaison_ND_v2.JPG/880px-6LoWPAN_comparaison_ND_v2.JPG](README%20assets/20160908140232880.png)

 

 

프로토콜 비교

![img](README%20assets/20160908140247588.png)


메쉬 네트워크

![img](README%20assets/20160908140300942.png)



#  CoAP Protocol

## 개요

CoAP 은 Constrained Application Protocol 의 약자로 RFC 7252 로 표준으로 등록되었다. 표준 이름에서도 알 수 있듯이 작은 센서 장치 등과 같이 CPU, 메모리, 통신 bandwidth 등이 제한된(constrained) 기기를 위한 application protocol 이다.

CoAP의 주요 목표는 아래와 같다고 할 수 있다.

- IP 기반, HTTP RESTful 개념을 적용
- 작은 메모리를 위하여 최대한 단순화 및 낮은 전송 대역을 위한 데이타 최소화

즉, CoAP는 아래와 같은 기기를 위하여 만들어진 프로토콜이다.

- 8bit 프로세서와 같은 저사양 센서 모듈에서도 구현 가능한 사양
- 802.15.4 기반의 무선 프로토콜([thread](http://threadgroup.org/), [zigbee](http://www.zigbee.org/)) 대응 (손실, 작은 패킷 크기)
- IPv6 기반 ([6LoWPAN](https://tools.ietf.org/html/rfc4944))
- 쉽게 web(HTTP)에 연동하기 위한 경량화된 RESTful
- CoAP-HTTP gateway에서도 stateless 로도 쉽게 HTTP로 변환하여 전달 가능

[MQTT](https://blog.humminglab.io/mqtt-protocol/)와 비교해보면 아래와 같다.

| 구분                     | CoAP                                        | MQTT                       |
| :----------------------- | :------------------------------------------ | :------------------------- |
| Communication Model      | Request-Response, Publish-Subscribe         | Publish-Subscribe          |
| RESTful                  | Yes                                         | No                         |
| Transport Layer Protocol | UDP (TCP도 가능은 함)                       | TCP (MQTT-SN으로 UDP 대응) |
| Security                 | DTLS                                        | SSL/TLS                    |
| Header                   | 4 Bytes                                     | 2 Bytes                    |
| Encoding                 | Binary                                      | Binary                     |
| 메시지 종류              | 4                                           | 16                         |
| QoS                      | Yes (Confirmable / Non confirmable message) | Yes (3 levels)             |
| Dynamic Discovery        | Yes                                         | No                         |
| Messaging                | Asynchronous & Synchronous                  | Asynchronous               |

MQTT는 쉽게 말하면 채팅 개념을 이용하여 Machine-to-Machine 통신을 하는 것이다. 채팅 특성상 1:N 도 가능하고 이런 경우 동기 통신(명령에 대한 응답을 바로 받기) 자체는 불가능하므로 비동기 통신 기반의 메시지 큐 프로토콜 이라고 볼 수 있다. 채팅 서버 처럼 메시지를 중계할 서버(MQTT broker)가 필요하고, 이 서버와 상시 연결이 필요하고, 받고 싶은 메시지(채팅방 개념)를 정의할 subscribe 메시지도 필요하고, 비동기 방식의 특성을 감안하여 전달되는 메시지가 최소한 다음 노드 (MQTT broker)까지는 전달 되었는지를 확인하기 위한 QoS를 제공한다. 이로 인하여 메시지 종류도 16개 정도 된다.

반면에 CoAP는 한마디로 말해서 HTTP를 사용하면 좋겠는데, 리소스 제한상 사용 못하는 제한된 WPAN(Wireless Personal Area Network) 기기를 위한 lightweight 버전이라고 볼 수 있다. Payload 최대 크기가 127 바이트인 802.15.4 를 고려하여 헤더 필드 등 최대한 줄일 수 있는 만큼 줄이고, binary로 인코딩하고, 단순한 UDP를 사용하고, 암호화는 [RFC 6347 DTLS](https://tools.ietf.org/html/rfc6347) 를 이용하여 datagram 단위로 하고, multicasting을 이용하여 discovery 기능을 제공하고, Request-Response 의 polling 방식 이외에 Publish-Subscribe event 방식을 추가한 것이라고 볼 수 있다. 이를 public network에서 운영하는 것도 가능하지만, 이 보다는 WPAN 에게 연결된 gateway까지만 이를 사용하고, 상위 단은 HTTP 등으로 변환하여 기존 웹 프레임웍을 활용하기 위한 것이라고 할 수 있다.

두 프로토콜의 근본이 다르기 때문에 어떤 용도로 사용하느냐에 따라서 각각의 기능이 장점이 될 수도 있고, 단점이 될 수도 있을 것이다. 어느 정도 명확한 구분점은 단말 기기가 802.3, 802.11 기반의 LAN protocol 을 지원하는지, 802.15.4 기반의 PAN protocol 을 지원하는지 일 것이다. 기존에 다른 기기를 위한 웹 플랫폼이 구축되어 있거나, PAN 기반의 장치를 접목한다면 CoAP 이 적절할 수 있고, LAN 기반의 기기를 접목한다면 MQTT 를 이용하는 것이 전체 구성이 단순해질 수 있을 것이다.

## 프로토콜

### 네트워크 구성

CoAP은 아래 그림과 같은 활용을 염두에 두고 설계되었다고 할 수 있다.

![CoAP](README%20assets/coap.png)

개념적으로 HTTP와 비교해보면 아래와 같은 layer 구조를 가진다.

![CoAP Layer](README%20assets/coap-diagram.png)

### 헤더 구조

CoAP의 Header는 아래와 같이 고정 4Bytes header와 가변의 optional 필드로 구성된다.

![CoAP Header](README%20assets/coap-header.png)

각각의 필드 설명은 다음과 같다.

- Version(VER): 2bits. RFC 7252 CoAP 버전은 1로 고정
- Type(T): 2bits. 메시지 종류 구분. CoAp는 총 4종류의 메시지가 있다.
  - Confirmable (0), Non-confirmable (1), Acknowledgement (2), Reset (3) Reset (3)
- Token Length(TKL): 4bits. Token 필드의 길이를 0~8까지로 Token 필드가 있는 경우 해당 크기 표시.
- Code: 8bits. 3bits.5bits로 나누어서 보면 HTTP의 response code와 유사한 형태가 된다. 예를 들어 401 이 HTTP 에서 ‘Unauthorized’ 인 것처럼 4.01 (이진수로 100.00001b) 은 ‘Unauthorized’이다. 앞의 class 부분만 보면 indicate a request (0), success response (2), client error response (4), server error response (5) 로 나뉘어 진다.
- Message ID: 메시지 재전송 시 중복여부를 확인하기 위한 ID

위와 같이 총 4bytes(32bits)가 고정 헤더이고, token length(TKL) 값에 따라 추가적인 token 필드가 추가된다. Token 필드 뒤에는 option 필드들이 붙는다. Option 필드는 DHCP의 option 필드와 유사하게 TLV(type-length-value) 형태이나, 크기를 최대한 줄이기 위하여 독특한 방법을 사용하였다. Option 번호(Type)는 크기를 줄이기 위하여 이전 option 값과의 차이를 기록한다. 즉, 이전 option type 이 3 이 었고, 이번 option 이 10이라면 Option delta에는 7을 기록 한다.

![CoAP Payload](README%20assets/coap-payload.png)

- Option Delta: Option delta의 경우도 크기를 줄이기 위하여 0~12는 그대로 이 필드에 기록을 하고, 이 보다 증분이 클때는 13,14 값을 적는다. 이 경우 확장된 Option Delta가 추가로 붙고, 이 확장 필드의 크기는 13인 경우 1bytes, 14인 경우 2bytes가 된다. Option Delta는 0이나 양수이므로 옵션 필드의 순서는 정렬된 순서로 패킷에 들어가야 한다.
- Option Length: Option length도 option delta와 동일하게 0~12는 이 필드만 사용하고, 13,14인 경우에는 각각 1, 2bytes의 Option Length(extended)필드가 추가된다.

최종적인 Option 항목의 끝은 1 byte의 0xff 값을 가진 payload marker가 들어간다. 이 payload marker 이후는 전송할 데이타이 payload가 들어간다.

프로토콜 상에서는 전체 패킷의 크기를 위한 필드가 없다. 이와 같은 이유는 datagram으로 보내기 때문에 항상 앞의 header 정보를 제외한 나머지를 모두 payload로 처리한다. 이것도 크기를 줄이기 위한 것이다.

여기에 6LoWPAN은 link-local address (single hop) 인 경우 IPv6는 2 octets, UDP는 4 octets로 헤더압축이 되고, DTLS도 7bytes payload가 최대 127 octets인 802.15.4 에서도 fragmentation없이 데이터 전송이 가능하다.

### 메시지 전송

HTTP의 request method와 response code는 헤더의 Code 필드를 이용한다. 위에서도 설명하였듯이 8bits의 Code 필드는 3bits.5bits 로 나뉘어 지고, 이를 중간에 .(점)을 넣어 표기한다. 예를 들어 이진수로 100.00001b은 4.01 ‘Unauthorized’ 코드가 된다. 앞부분의 3bit가 response class로 HTTP와 동일하게 2는 성공, 4는 client error, 5는 server error가 된다. 그리고 0은 method 용도로 다음과 같이 정의된다.

- 0.01: GET
- 0.02: POST
- 0.03: PUT
- 0.04: DELETE

이 8bits code 필드로 이와 같이 구분하면 HTTP와 유사한 형태의 request-response 메시지를 정의할 수 있다.

CoAP은 기본적으로 UDP를 사용하기 때문에 request에 대한 response 응답이 없는 경우 일정 시간 후 request 측에서 재전송 하여야 한다. 이 경우 수신측에서 중복된 메시지 인지를 구분할 수 있도록 Token 필드를 이용한다.

하지만 request에 대하여 응답이 길어지는 경우가 있는 경우 이 방식으로는 문제가 있다. 예를 들어 온도 측정을 요청했는데, 기기에서 온도 값을 얻는데 10초이상 소요될 수도 있다. CoAP에서는 응답이 없는 경우 재전송 주기는 2초이다. 이렇게 응답이 늦어지는 경우 송신측에서는 response가 올때까지 계속 재전송을 하여야 한다. 수신측이야 Token 필드를 이용하여 중복된 메시지 인지를 구분하여 한번의 응답만 가도록 할 수 있으나, 송신측에서는 수신측이 받았는지, 않받았는지를 확인할 방법이 없다. 이와 같은 수신여부를 확인하기 위하여 Type(T)과 Message ID 필드를 사용한다. Type 필드가 0인 경우에는 confirmable로 수신측은 필드값 2로 ACK 응답을 하여야 한다.

일반적인 경우에는 ACK 응답시 아래 처럼 CoAP response message 를 보낸다(piggybacked).

![CoAP flow 1](README%20assets/coap-flow1.png)

만일 수신측에서 바로 응답을 할 수 없는 경우에는 아래처럼 ACK 응답만 먼저하고, 나중에 request에 대한 response를 할 수 있다. Message ID(MID)는 confirmable-ack transaction 단위로만 사용하여, 아래와 같은 경우에는 response 시에는 다른 MID를 사용하여야 한다.

![CoAP flow 2](README%20assets/coap-flow2.png)

위의 예에서 response 시에도 Type(T)필드를 Confirmable(CON)로 요청을 해서 ACK를 받았는데, 이 부분은 Confirmable로 응답할 지, Non-confirmable로 응답할 지는 수신측 마음이다.

Reset Type(T) 필드는 잘못된 응답을 받았을 때 상대방에게 알려주기 위한 것이다.

이와 같이하여 UDP 상에서 효율은 떨어지지만 TCP와 유사한 reliability를 제공할 수 있다.

### Cache & Proxy

제한된 성능의 종단 노드를 대신하여 border gateway에서 어느정도 cache를 해줄 수도 있다. 예를 들어 한번 온도 센서에서 온도값을 읽어오면 일정 시간 내에서는 다시 종단 노드에게 물어보는 것이 아니라 기존에 cache된 값을 사용할 수 있다.

이 부분은 option 필드의 Max-Age와 ETag를 사용한다. 이를 사용하는 예는 아래와 같다. (아래 그림은 ‘[Introduction to Resource-Oriented Applications in Constrained Networks](https://www.iab.org/wp-content/IAB-uploads/2011/04/Shelby.pdf)’ 에서 발췌)

![CoAP Proxy](README%20assets/coap-proxy.png)

### Observation

Polling 이 아니라 event 방식을 이용하는 것도 Observe option 필드를 이용하여 지원한다.

![CoAP Observation](README%20assets/coap-observation.png)

### 마치며

이 외에도 큰 사이즈 데이타 전송을 위한 black transfer도 지원하고, multicast address를 이용한 여러 소스에서 응답 받기, service/resource discovery 도 지원하여 별도의 설정없이 자동으로 시스템 구성도 할 수 있다.

Zigbee 보다는 좀더 오픈 마인드로 [Thread](http://threadgroup.org/) 그룹에서 802.15.4, 6LoWPAN, CoAP 등 오픈 표준을 통합하여 표준화 정의 및 호환성 검증을 하기 때문에 센서 네트워크에서 CoAP 이 업계 표준 프로토콜로 자리잡을 가능성은 상당히 커 보인다.

무엇보다도 CoAP 의 장점으로 볼 수 있는 것은 IP 기반이라는 것이다. HTTP나 [SIP](https://www.ietf.org/rfc/rfc3261.txt) 처럼 IP 기반으로 proxy 로 확장을 할 수 있는 구조로, 기존 Gateway방식의 센서 네트워크를 꾸미는 것보다 다음과 같은 면에서 장점이 있을 것이다.

- 보안: 순순한 IP 망이면 end-to-end 암호화도 가능하므로 중간의 proxy를 신뢰도가 낮아도 됨. Gateway 방식은 IP 네트워크와 센서 네트워크간의 데이타 변환을 위하여 복호화 수행하므로 보안 레벨이 높아야 함.
- 확장성: Proxy는 stateless로도 운영 가능하며 변경없이 새로운 서비스 적용 가능
- 투명성: 순수 IPv6 네트워크





# Proxy

### Proxy란?

클라이언트에서 어떤 인터넷 주소의 정보를 요청 했을때 그 주소에 해당하는 정보를 사전에 저장해둔 서버에서 찾아보고 있으면 바로 응답을 해주고, 없으면 해당 주소의 웹서버에 접속해서 요청 정보를 가져와 저장 후 응답해 주는 역할을 말한다.



## Forward Proxy란?

![image-20201223160103366](README%20assets/image-20201223160103366.png)

클라이언트가 웹 서버에 접근하려고 할때 클라이언트의 요청이 웹서버에게 직접 전송되는 것이 아니고 중간에 Proxy 서버에게 전달되어 Proxy 서버는 그 요청을 웹 서버에게 전달하여 응답을 받아오는 방식이다.

**추천 용도**

- Content Filtering
- eMail security
- NAT'ing
- Compliance Reporting



## Reverse Proxy란?

![image-20201223160326606](README%20assets/image-20201223160326606.png)

클라이언트는 웹 서버의 주소가 아닌 Reverse Proxy로 설정된 주소로 요청을 하게 되고, Proxy 서버가 받아서 그 뒷단에 있는 웹 서버에게 다시 요청을 하는 방식으로 클라이언트는 진짜 웹 서버의 정보를 알 수가 없다.

**추천 용도**

- Application Delivery including
- Load Balancing(TCP Multiplexing)
- SSL Offload/Acceleration (SSL Multiplexing)
- Caching
- Compression
- Content Switching/Redirection
- Application Firewall
- Server Obfuscation
- Authentication
- Single SIgn On



## MANET

### 무선매체

![image-20201223175423557](README%20assets/image-20201223175423557.png)