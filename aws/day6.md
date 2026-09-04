# Day 6 - AWS VPC 네트워크 실습

## 오늘의 목표

AWS에서 VPC 네트워크를 직접 구성하면서 다음 구조를 이해한다.

Internet
   |
   v
Internet Gateway
   |
   v
VPC (10.0.0.0/16)
   |
   +-- Public Subnet (10.0.1.0/24)
   |
   +-- Private Subnet (10.0.2.0/24)


---

## 1. VPC

VPC(Virtual Private Cloud)는 AWS에서 사용하는
논리적으로 격리된 가상 네트워크이다.

이번 실습에서는 다음 VPC를 생성했다.

- 이름: practice-vpc
- CIDR: 10.0.0.0/16

VPC 내부에 여러 Subnet을 만들 수 있다.


---

## 2. CIDR과 Subnet

VPC:

10.0.0.0/16

Public Subnet:

10.0.1.0/24

Private Subnet:

10.0.2.0/24

두 Subnet 모두 VPC의 10.0.0.0/16 범위 안에 존재한다.

Subnet은 VPC의 IP 주소 범위를 더 작은 네트워크로
나누어 사용하는 것이다.


---

## 3. Public Subnet

생성한 Public Subnet:

- 이름: practice-public-subnet-2a
- CIDR: 10.0.1.0/24
- AZ: ap-northeast-2a

Public Subnet의 Route Table:

10.0.0.0/16 -> local
0.0.0.0/0   -> Internet Gateway

0.0.0.0/0은 위의 구체적인 내부 경로에 해당하지 않는
IPv4 목적지에 사용하는 기본 경로이다.

Internet Gateway로 향하는 기본 경로가 있기 때문에
인터넷 연결이 가능한 Public Subnet 구조가 된다.


---

## 4. Internet Gateway

Internet Gateway(IGW)는 VPC와 인터넷 사이의
통신을 가능하게 하는 AWS 네트워크 구성 요소이다.

구조:

EC2
 |
Subnet
 |
Route Table
 |
Internet Gateway
 |
Internet

IGW는 EC2에 직접 연결하는 것이 아니라
VPC에 연결한다.


---

## 5. Public EC2

Public Subnet에 EC2를 생성했다.

설정:

- Ubuntu Server 24.04 LTS
- t3.micro
- VPC: practice-vpc
- Subnet: practice-public-subnet-2a
- Public IPv4 자동 할당 활성화
- Security Group: practice-web-sg

Public EC2는 Public IP와 Private IP를 가진다.

Public IP:
인터넷에서 EC2에 접근할 때 사용

Private IP:
VPC 내부에서 EC2끼리 통신할 때 사용


---

## 6. Security Group

Security Group은 EC2 등의 리소스에 적용되는
Stateful 가상 방화벽이다.

SSH:

TCP 22
Source: 내 공인 IP /32

/32는 하나의 IPv4 주소만 의미한다.

예:

211.xxx.xxx.xxx/32

즉 특정 공인 IP에서 오는 SSH 연결만 허용할 수 있다.

카페나 다른 네트워크로 이동하면 ISP에서 사용하는
공인 IP가 달라질 수 있으므로 Security Group의
'내 IP' 값도 달라질 수 있다는 것을 실습 중 확인했다.


---

## 7. Private Subnet

생성한 Private Subnet:

- 이름: practice-private-subnet-2a
- CIDR: 10.0.2.0/24
- AZ: ap-northeast-2a

Private Route Table:

10.0.0.0/16 -> local

Public Subnet과 다르게

0.0.0.0/0 -> Internet Gateway

경로를 설정하지 않았다.

따라서 인터넷으로 직접 연결되는 경로가 없는
Private Subnet 구조이다.


---

## 8. local Route

VPC Route Table에 자동으로 존재하는 경로:

10.0.0.0/16 -> local

이 경로를 통해 같은 VPC 내부의 Subnet끼리
라우팅할 수 있다.

예:

Public EC2
10.0.1.x
    |
    | local
    v
Private EC2
10.0.2.x

단, 실제 통신이 성공하려면 Security Group과
NACL 등의 보안 규칙도 허용해야 한다.


---

## 9. Security Group과 NACL

Security Group

- EC2/네트워크 인터페이스 수준
- Stateful
- 허용된 연결의 응답 트래픽을 자동으로 인식

NACL

- Subnet 수준
- Stateless
- Inbound / Outbound를 각각 검사
- Allow / Deny 규칙 사용
- 낮은 규칙 번호부터 검사


---

## 10. NAT Gateway

Private EC2는 인터넷에서 직접 접근하지 못하게 하면서도
인터넷으로 요청을 보내야 하는 경우가 있다.

예:

- apt update
- 패키지 다운로드
- 외부 API 요청
- Docker 이미지 다운로드

이때 NAT Gateway를 사용할 수 있다.

Private EC2
    |
    v
Private Route Table
0.0.0.0/0 -> NAT Gateway
    |
    v
NAT Gateway
(Public Subnet)
    |
    v
Internet Gateway
    |
    v
Internet

핵심:

NAT Gateway는 Private EC2가
인터넷으로 나가기 위한 출구 역할을 한다.

인터넷에서 Private EC2로 직접 접속하기 위한
입구 역할은 하지 않는다.

※ NAT Gateway는 시간 및 데이터 처리에 따른
비용이 발생할 수 있으므로 이번 실습에서는 생성하지 않았다.


---

## 11. Jump Server / Bastion Host

Private EC2에는 Public IP를 할당하지 않았다.

따라서 내 PC에서 Private EC2로 직접 SSH 접속하는 대신
Public Subnet의 EC2를 중간 서버로 사용할 수 있다.

내 PC
   |
   | SSH
   v
Public EC2
(Jump Server)
   |
   | Private IP
   v
Private EC2

Private EC2 설정:

- VPC: practice-vpc
- Subnet: practice-private-subnet-2a
- Public IP: 비활성화
- Private IP: 10.0.2.129
- Security Group: practice-private-sg

Private EC2의 SSH 22번 Source는
Public EC2가 사용하는 practice-web-sg로 설정했다.

즉 인터넷 전체에서 SSH를 허용하는 것이 아니라
Public EC2를 경유하는 구조로 제한했다.


---

## 12. SSH ProxyJump

Private Key를 Public EC2에 복사하지 않고
내 PC의 키를 사용하면서 Public EC2를 경유하기 위해
SSH ProxyJump(-J)를 사용할 수 있다.

형태:

ssh -i <KEY> -J ubuntu@<PUBLIC_EC2_IP> ubuntu@<PRIVATE_EC2_IP>

구조:

내 PC
   |
   | SSH
   v
Public EC2
   |
   | VPC 내부 통신
   v
Private EC2

실습 중 다른 네트워크의 카페에서 접속하면서
공인 IP가 변경되어 기존 Security Group의 /32 규칙으로
Public EC2 SSH 접속이 되지 않는 상황도 확인했다.

Security Group의 SSH Source를 현재 '내 IP'로
갱신한 후 Public EC2 SSH 접속이 정상적으로 되는 것을 확인했다.

SSH ProxyJump를 사용하여 Public EC2를 경유한 뒤
Private EC2에 정상적으로 접속되는 것을 확인했다.

이를 통해 Private Key를 Public EC2에 복사하지 않고도
내 PC의 키를 사용하여 Private EC2에 안전하게
접속할 수 있다는 것을 확인했다.

ProxyJump 접속 결과:

내 PC
   |
   | SSH ProxyJump
   v
Public EC2
   |
   | Private IP
   v
Private EC2

Public EC2를 Jump Server로 사용하여
Public IP가 없는 Private EC2에 정상적으로 접속했다.


---

## 오늘 이해한 핵심 구조

Internet
   |
   v
Internet Gateway
   |
   v
practice-vpc (10.0.0.0/16)
   |
   +-- Public Subnet (10.0.1.0/24)
   |      |
   |      +-- Public EC2
   |      |
   |      +-- 0.0.0.0/0 -> IGW
   |
   +-- Private Subnet (10.0.2.0/24)
          |
          +-- Private EC2
          |
          +-- local route only


Public EC2
-> 인터넷과 직접 통신할 수 있는 구조

Private EC2
-> 인터넷에서 직접 접근하지 않도록 구성

NAT Gateway
-> Private EC2가 인터넷으로 나갈 때 사용

Jump Server
-> 관리자가 Public EC2를 경유하여
   Private EC2에 접근할 때 사용


---

## Day 6 핵심 정리

VPC
= AWS의 가상 네트워크

Subnet
= VPC를 나눈 작은 네트워크

Route Table
= 트래픽이 어디로 갈지 결정

Internet Gateway
= VPC와 인터넷을 연결

Security Group
= Stateful 리소스 방화벽

NACL
= Stateless Subnet 방화벽

NAT Gateway
= Private 서버의 인터넷 출구

Jump Server
= Private 서버 접근을 위한 중간 서버

Public Subnet
= IGW로 향하는 인터넷 경로가 있는 Subnet

Private Subnet
= IGW로 직접 향하는 인터넷 경로가 없는 Subnet
