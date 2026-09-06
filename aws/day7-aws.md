# Day 7 - Public 및 Private 네트워크 검증

## 오늘의 목표

Day 6에서 구성한 VPC 네트워크가 실제로 어떻게 동작하는지 확인한다.

- Private EC2의 네트워크 정보 확인
- Public EC2와 Private EC2의 라우팅 비교
- Private EC2의 DNS 동작 확인
- Public EC2와 Private EC2의 인터넷 연결 차이 확인
- SSH ProxyJump를 통한 Private EC2 접속 확인

---

## 1. SSH ProxyJump로 Private EC2 접속

Private EC2에는 Public IP가 없기 때문에
Public EC2를 경유하여 접속했다.

```text
내 PC
  |
  | SSH ProxyJump
  v
Public EC2
  |
  | VPC 내부 통신
  v
Private EC2
```

접속 명령어 형식:

```bash
ssh -i <키파일경로> \
-o "ProxyCommand=ssh -i <키파일경로> -W %h:%p ubuntu@<PUBLIC_EC2_PUBLIC_IP>" \
ubuntu@<PRIVATE_EC2_PRIVATE_IP>
```

Private EC2 접속 후 다음 프롬프트를 확인했다.

```text
ubuntu@ip-10-0-2-129
```

이를 통해 Public EC2를 Jump Server로 사용하여
Private EC2에 정상적으로 접속할 수 있다는 것을 확인했다.

---

## 2. Private EC2의 기본 정보 확인

Private EC2에서 다음 명령어를 실행했다.

```bash
hostname
hostname -I
ip route
```

### hostname 결과

```text
ip-10-0-2-129
```

현재 접속한 서버가 Private EC2임을 확인했다.

### Private IP 결과

```text
10.0.2.129
```

Private EC2는 `10.0.2.129` 주소를 사용하고 있다.

### 네트워크 장치

```text
ens5
```

`ens5`는 EC2에서 사용하는 네트워크 인터페이스이다.

---

## 3. Private EC2의 라우팅 테이블

Private EC2에서 확인한 주요 라우팅 정보:

```text
default via 10.0.2.1 dev ens5 proto dhcp src 10.0.2.129 metric 100
10.0.0.2 via 10.0.2.1 dev ens5 proto dhcp src 10.0.2.129 metric 100
10.0.2.0/24 dev ens5 proto kernel scope link src 10.0.2.129 metric 100
10.0.2.1 dev ens5 proto dhcp scope link src 10.0.2.129 metric 100
```

### 주요 항목

- `10.0.2.129`: Private EC2의 IP 주소
- `10.0.2.1`: Private Subnet에서 사용하는 VPC 라우터
- `10.0.2.0/24`: Private Subnet의 네트워크 범위
- `ens5`: EC2의 네트워크 장치
- `default`: 별도의 경로가 없을 때 사용하는 기본 경로

`10.0.2.0/24` 범위의 주소는 같은 Private Subnet에
직접 연결되어 있으므로 `ens5`를 통해 통신한다.

---

## 4. Private EC2의 DNS 조회 확인

다음 명령어를 실행했다.

```bash
getent hosts google.com
```

Google 도메인의 IP 주소가 정상적으로 조회되었다.

이를 통해 Private EC2에서 AWS VPC가 제공하는
DNS 이름 해석 기능은 정상적으로 동작한다는 것을 확인했다.

단, DNS 조회가 성공했다고 해서 인터넷 접속까지
가능하다는 뜻은 아니다.

```text
DNS 이름 해석   가능
인터넷 웹 접속  별도 경로가 필요
```

---

## 5. Private EC2의 인터넷 연결 확인

다음 명령어를 실행했다.

```bash
curl --connect-timeout 5 https://checkip.amazonaws.com
```

결과:

```text
curl: (28) Connection timeout
```

Private EC2에서 인터넷 연결이 되지 않는 것을 확인했다.

현재 Private Route Table에는 인터넷으로 나가는
NAT Gateway 또는 Internet Gateway 경로가 없기 때문이다.

```text
Private EC2
    |
    v
VPC 라우터
    |
    v
Private Route Table
    |
    v
인터넷으로 향하는 경로 없음
```

---

## 6. Public EC2와 비교

Public EC2에서도 같은 명령어를 실행했다.

```bash
curl --connect-timeout 5 https://checkip.amazonaws.com
```

Public IP 주소가 정상적으로 출력되었다.

따라서 다음과 같은 차이를 확인했다.

```text
Public EC2
- 인터넷 접속 가능
- Public Subnet에 위치
- Route Table에 0.0.0.0/0 -> Internet Gateway 경로 존재

Private EC2
- 인터넷 직접 접속 불가
- Private Subnet에 위치
- Internet Gateway로 직접 향하는 경로 없음
```

Public EC2의 라우팅 정보도 확인했다.

```bash
ip route
```

Public EC2의 기본 게이트웨이는 다음과 같은 형태이다.

```text
default via 10.0.1.1 dev ens5
```

Private EC2의 기본 게이트웨이는 다음과 같다.

```text
default via 10.0.2.1 dev ens5
```

각 EC2는 서로 다른 Subnet의 VPC 라우터를 사용한다.

---

## 7. Public Subnet과 Private Subnet 비교

| 구분 | Public Subnet | Private Subnet |
|---|---|---|
| 네트워크 범위 | 10.0.1.0/24 | 10.0.2.0/24 |
| EC2 예시 | Public EC2 | Private EC2 |
| Public IP | 있음 | 없음 |
| 인터넷 직접 접속 | 가능 | 불가능 |
| 기본 경로 | Internet Gateway | 내부 라우터 |
| 외부 인터넷 접속 | 직접 가능 | NAT Gateway 필요 |
| 외부에서 직접 SSH | 가능 | 불가능 |
| 관리 접속 방법 | 직접 SSH | Jump Server 또는 SSM |

---

## 8. 오늘 확인한 전체 구조

```text
인터넷
   |
   v
Internet Gateway
   |
   v
VPC (10.0.0.0/16)
   |
   +-- Public Subnet (10.0.1.0/24)
   |      |
   |      +-- Public EC2
   |      |      |
   |      |      +-- 인터넷 접속 가능
   |      |
   |      +-- 0.0.0.0/0 -> Internet Gateway
   |
   +-- Private Subnet (10.0.2.0/24)
          |
          +-- Private EC2
          |      |
          |      +-- 인터넷 직접 접속 불가
          |      +-- Public EC2 경유 접속 가능
          |
          +-- local route
```

---

## 9. 오늘 배운 핵심

### Public Subnet

Route Table에 다음 경로가 있는 Subnet이다.

```text
0.0.0.0/0 -> Internet Gateway
```

인터넷 통신이 가능하려면 일반적으로 다음 조건도 필요하다.

- Internet Gateway 연결
- Public IP 또는 Elastic IP
- 적절한 Security Group 규칙
- 올바른 Route Table 연결

### Private Subnet

인터넷에서 직접 접근할 수 없도록 구성한 Subnet이다.

Private EC2는 Public IP가 없으며,
외부에서 직접 SSH 접속할 수 없다.

관리자가 접속해야 할 경우에는 다음 방법을 사용할 수 있다.

```text
내 PC -> Public EC2 -> Private EC2
```

또는 AWS Systems Manager Session Manager를 사용할 수 있다.

### NAT Gateway

Private EC2가 인터넷으로 요청을 보낼 때 사용하는
인터넷 출구이다.

```text
Private EC2
    |
    v
NAT Gateway
    |
    v
Internet Gateway
    |
    v
인터넷
```

NAT Gateway는 Private EC2로 외부에서 직접 들어오는
입구 역할은 하지 않는다.

---

## 10. Day 7 결론

이번 실습을 통해 다음 내용을 확인했다.

1. Private EC2는 `10.0.2.129` Private IP를 사용한다.
2. Private EC2의 네트워크 장치는 `ens5`이다.
3. Private EC2의 기본 게이트웨이는 `10.0.2.1`이다.
4. Private EC2에서도 DNS 조회는 가능하다.
5. NAT Gateway가 없으면 Private EC2는 인터넷에 직접 접속할 수 없다.
6. Public EC2는 Internet Gateway를 통해 인터넷에 접속할 수 있다.
7. Public EC2를 Jump Server로 사용하여 Private EC2에 접속할 수 있다.
8. Private EC2에 외부에서 직접 접근하지 못하게 하는 구조를 확인했다.

Day 7에서는 새로운 AWS 서비스를 추가하지 않고,
Day 6에서 만든 VPC 네트워크의 동작 결과를 검증했다.
