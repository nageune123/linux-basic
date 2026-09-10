AWS Day 12 - 시작 템플릿과 Auto Scaling으로 자동 복구

목표

EC2 생성과 장애 시 서버 교체를 자동화하고, ALB를 통해 Private EC2 웹 서버에 접속한다.

진행 순서

기존 AMI practice-private-ami를 사용해 시작 템플릿 practice-launch-template을 만든다.

t3.micro, practice-private-sg, docker-practice-key, Public IP 없음

시작 템플릿을 사용해 Auto Scaling 그룹 practice-asg-day12를 만든다.

Private Subnet 사용

희망/최소/최대 용량: 2 / 2 / 2

Internet-facing Application Load Balancer practice-alb-day12를 만든다.

Public Subnet 2a·2c 사용

practice-target-group을 ALB와 Auto Scaling 그룹에 연결한다.

ALB에는 practice-alb-sg, EC2에는 practice-private-sg만 연결한다.

두 EC2가 대상 그룹에서 Healthy인지 확인하고 ALB DNS로 웹사이트에 접속한다.

ASG 인스턴스 한 대를 종료해 새 인스턴스가 자동으로 생성되는지 확인한다.

결과

인터넷 사용자 → ALB → 대상 그룹 → Private EC2 2대
                                      ↑
                         장애 시 ASG가 새 인스턴스 생성

한 인스턴스를 종료하자 Auto Scaling이 새 인스턴스를 생성했고, 원하는 용량 2대를 유지했다. 새 인스턴스가 상태 검사와 대상 그룹 헬스 체크를 통과한 뒤 ALB 접속도 정상적으로 유지됐다.

핵심

시작 템플릿: EC2 생성 설계도

Auto Scaling: 인스턴스 수를 유지하고 장애 인스턴스를 교체

대상 그룹: 인스턴스 상태 확인

ALB: Healthy 인스턴스로 요청 분산

현재 설정은 용량 2 고정이며, 트래픽 기반 자동 확장 정책은 아직 설정하지 않았다.
