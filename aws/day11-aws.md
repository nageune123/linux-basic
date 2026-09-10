AWS Day 11 - AMI로 Private EC2 복제 및 ALB 장애 테스트

목표

기존 Private EC2를 AMI로 복제하고, 두 대를 ALB에 연결해 한 대가 중지되어도 웹사이트가 계속 접속되는지 확인한다.

진행 순서

기존 Nginx Private EC2로 AMI practice-private-ami를 생성한다.

AMI에서 두 번째 Private EC2 practice-private-ec2-2를 생성한다.

두 EC2를 practice-target-group에 HTTP 80으로 등록한다.

새 Internet-facing ALB를 Public Subnet에 만들고 대상 그룹을 연결한다.

ALB DNS로 접속한 뒤 EC2 한 대를 중지한다.

중지한 대상은 비정상 또는 사용되지 않음으로 바뀌고, 다른 정상 대상이 요청을 처리하는지 확인한다.

결과

인터넷 사용자 → ALB → Healthy Private EC2

EC2 한 대를 중지해도 다른 EC2를 통해 웹사이트에 접속할 수 있었다. 실습 후 테스트용 ALB는 삭제하고 EC2는 중지했다.

핵심

AMI: 설정이 끝난 EC2의 복사본

대상 그룹: ALB가 연결할 서버 목록과 상태 확인 목록

ALB: 정상 상태인 서버로 웹 요청을 전달

Day 11은 서버를 수동으로 복제하고 등록한 실습이다.
