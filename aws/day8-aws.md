# Day 8 - NAT Gateway 및 Private EC2 인터넷 통신

## 오늘의 목표

Private Subnet에 있는 EC2가 외부에서 직접 접근되지는 않으면서
인터넷으로 나갈 수 있도록 NAT Gateway를 구성한다.

---

## 1. NAT Gateway 생성

다음 설정으로 NAT Gateway를 생성했다.

- 이름: `practice-nat-gateway`
- 가용성 모드: 영역별(Zonal)
- 연결 유형: Public
- Subnet: `practice-public-subnet-2a`
- Elastic IP: 새로 할당

NAT Gateway는 Public Subnet에 생성하고
Elastic IP를 연결했다.

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
Internet
```

---

## 2. Private Route Table 수정

Private Route Table에 다음 경로를 추가했다.

```text
Destination: 0.0.0.0/0
Target: NAT Gateway
```

이 경로를 통해 Private EC2의 인터넷 트래픽이
NAT Gateway로 전달되도록 구성했다.

---

## 3. NAT Gateway 연결 전·후 비교

### 연결 전

Private EC2에서 인터넷 접속을 시도했을 때
연결 시간 초과가 발생했다.

```bash
curl --connect-timeout 5 https://checkip.amazonaws.com
```

결과:

```text
Connection timeout
```

### 연결 후

NAT Gateway와 Private Route Table을 설정한 후
다시 실행했다.

```bash
curl --connect-timeout 10 https://checkip.amazonaws.com
```

NAT Gateway에 연결된 Elastic IP가 출력되었다.

```text
NAT Gateway의 Elastic IP 출력
```

이를 통해 Private EC2가 NAT Gateway를 통해
인터넷으로 나갈 수 있다는 것을 확인했다.

---

## 4. 패키지 저장소 접속 확인

Private EC2에서 다음 명령어를 실행했다.

```bash
sudo apt update
```

Ubuntu 패키지 저장소에 정상적으로 접속했고
패키지 목록을 내려받는 데 성공했다.

이를 통해 NAT Gateway를 통한 외부 통신이
정상적으로 동작한다는 것을 확인했다.

---

## 5. Private EC2에 Nginx 설치

NAT Gateway가 연결된 상태에서 Private EC2에
Nginx 웹 서버를 설치했다.

```bash
sudo apt install nginx -y
```

서비스 상태를 확인했다.

```bash
sudo systemctl status nginx --no-pager
```

확인 결과:

```text
Active: active (running)
```

Private EC2 내부에서 Nginx 응답도 확인했다.

```bash
curl http://localhost
```

Nginx 기본 페이지 HTML이 출력되었다.

---

## 6. Public EC2에서 Private EC2의 Nginx 접속

Private Security Group에 다음 인바운드 규칙을 추가했다.

```text
유형: HTTP
포트: 80
소스: Public EC2의 Security Group
```

Public EC2에서 Private EC2의 Private IP로
HTTP 요청을 보냈다.

```bash
curl http://10.0.2.129
```

Nginx 기본 페이지 HTML이 정상적으로 출력되었다.

```text
Public EC2
    |
    | HTTP 80
    v
Private EC2 (10.0.2.129)
    |
    v
Nginx
```

Public EC2를 통해 Private EC2의 웹 서버에
정상적으로 접근할 수 있다는 것을 확인했다.

---

## 7. 실습 후 리소스 정리

비용이 발생하지 않도록 다음 순서로 정리했다.

1. Private Route Table의 NAT Gateway 경로 삭제
2. NAT Gateway 삭제
3. NAT Gateway 삭제 완료 확인
4. Elastic IP 릴리스

NAT Gateway는 삭제되었고 Elastic IP도 릴리스했다.

---

## 오늘 배운 핵심

```text
NAT Gateway
= Private EC2가 인터넷으로 나가기 위한 출구

Public NAT Gateway
= Public Subnet에 생성
= Elastic IP 필요
= 외부에서 Private EC2로 직접 들어오는 것은 허용하지 않음

Private Route Table
= 0.0.0.0/0을 NAT Gateway로 전달

Security Group
= Public EC2에서 Private EC2의 HTTP 80 요청만 허용

Nginx
= Private EC2에서 실행되는 웹 서버
```

최종 구조:

```text
인터넷
   |
   v
Internet Gateway
   |
   +-- Public Subnet
   |      |
   |      +-- Public EC2
   |      +-- NAT Gateway
   |
   +-- Private Subnet
          |
          +-- Private EC2
                 |
                 +-- Nginx
```

Day 8에서는 NAT Gateway를 통해
Private EC2가 인터넷으로 나갈 수 있게 구성했고,
Public EC2에서 Private EC2의 Nginx에 접근하는 것까지 확인했다.
