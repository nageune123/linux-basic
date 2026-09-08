# AWS Day 10 - Application Load Balancer(ALB)로 Private EC2 웹 서버 연결

## 1. 오늘의 목표

인터넷에서 직접 접근할 수 없는 Private EC2의 Nginx 웹 페이지를 Application Load Balancer(ALB)를 통해 접속한다.

```text
내 PC 브라우저
    ↓ HTTP 80
Internet-facing ALB
    ↓ VPC 내부 통신
Private EC2(10.0.2.129)
    ↓
Nginx
```

Public EC2는 웹 요청 경로가 아니라 Private EC2에 SSH로 접속하기 위한 점프 서버로 사용한다.

## 2. 네트워크 구성

| 구성 요소       | 설정                                                             |
| ----------- | -------------------------------------------------------------- |
| VPC         | `practice-vpc` (`10.0.0.0/16`)                                 |
| 퍼블릭 서브넷 1   | `practice-public-subnet-2a` (`10.0.1.0/24`, `ap-northeast-2a`) |
| 퍼블릭 서브넷 2   | `practice-public-subnet-2c` (`10.0.3.0/24`, `ap-northeast-2c`) |
| 프라이빗 서브넷    | `practice-private-subnet-2a` (`10.0.2.0/24`)                   |
| Private EC2 | `10.0.2.129`, Nginx HTTP 80                                    |

Internet-facing ALB는 고가용성을 위해 서로 다른 가용 영역의 서브넷을 최소 2개 사용한다.

## 3. ALB 구성

* ALB 이름: `practice-alb`
* 체계: Internet-facing
* IP 주소 유형: IPv4
* 퍼블릭 서브넷: `practice-public-subnet-2a`, `practice-public-subnet-2c`
* 리스너: HTTP 80
* 기본 작업: 대상 그룹으로 전달

## 4. 보안 그룹

### ALB 보안 그룹

* 이름: `practice-alb-sg`
* 인바운드: HTTP 80
* 소스: `0.0.0.0/0`

### Private EC2 보안 그룹

* SSH 22: Public EC2 보안 그룹에서 허용
* HTTP 80: `practice-alb-sg`에서만 허용

보안 그룹은 서브넷에 연결하는 것이 아니라 ALB와 EC2 같은 리소스에 연결하는 가상 방화벽이다.

## 5. 대상 그룹

* 이름: `practice-target-group`
* 대상 유형: 인스턴스
* 프로토콜 및 포트: HTTP 80
* 대상: `practice-private-ec2` (`10.0.2.129`)
* 상태 검사: HTTP `/`

대상 그룹은 ALB가 요청을 전달할 백엔드 서버들의 묶음이다. 현재는 EC2 한 대만 등록했으며, 여러 대를 등록하면 정상 상태인 서버로 요청을 분산할 수 있다.

## 6. 접속 확인

ALB DNS 주소로 접속했다.

```bash
curl http://practice-alb-507427440.ap-northeast-2.elb.amazonaws.com
```

브라우저와 터미널에서 Private EC2의 사용자 지정 Nginx 페이지가 표시되어 다음 흐름을 확인했다.

```text
내 PC → ALB → Private EC2 → Nginx
```

ALB가 웹 요청을 받아 Private EC2로 전달하므로 Public EC2와 NAT Gateway는 이 HTTP 접속 경로에 사용되지 않는다.

## 7. NAT Gateway와의 차이

| 장치          | 역할                             |
| ----------- | ------------------------------ |
| ALB         | 인터넷 사용자의 웹 요청을 Private EC2로 전달 |
| NAT Gateway | Private EC2가 인터넷으로 나가는 요청을 전달  |

NAT Gateway는 Private EC2의 `apt update`나 외부 API 호출처럼 아웃바운드 통신이 필요할 때 사용한다.

## 8. 실습 후 정리

* `practice-alb` 생성 및 접속 테스트 완료
* 테스트 후 `practice-alb` 삭제 완료
* NAT Gateway 및 Elastic IP는 이전 실습에서 삭제 완료
* 사용하지 않는 EC2는 중지하여 컴퓨팅 요금을 방지한다.
* EC2를 중지해도 EBS 디스크 저장 비용은 남을 수 있다.

## 9. Billing 확인 메모

* Billing 화면의 `권한 필요`는 현재 IAM 사용자에게 청구서 조회 권한이 없다는 뜻이다.
* 권한 오류가 요금 발생을 의미하는 것은 아니다.
* 루트 계정에서 IAM 결제 정보 액세스를 활성화하고 Billing 조회 권한을 부여하면 청구서를 확인할 수 있다.
* 무료 플랜 안내가 표시되더라도 사용하지 않는 EC2는 중지하는 것이 안전하다.
