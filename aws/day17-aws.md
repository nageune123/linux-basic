AWS Day 17 - CloudTrail and CloudWatch Dashboard

목표

CloudTrail에서 AWS API 호출 기록을 확인하고, CloudWatch 대시보드에 EC2 지표를 추가한다.

CloudTrail 확인

리전: ap-northeast-2

이벤트 이름: CreateVpcEndpoint

이벤트 소스: ec2.amazonaws.com

이벤트 시간: 2026-09-14T06:29:58Z

VPC: vpc-0f3e3aca8754b183e (practice-vpc)

라우팅 테이블: rtb-0217a303f26ce6052 (practice-private-rt)

서비스: com.amazonaws.ap-northeast-2.s3

CloudTrail 이벤트 상세 정보에서 실행된 API, 리전, 요청 주체, 요청 파라미터를 확인했다.

CloudWatch Dashboard

CloudWatch에서 practice-day17-dashboard 대시보드를 생성했다.

CloudWatch 지표에서 EC2 > Per-Instance Metrics > CPUUtilization을 선택했다.

EC2 인스턴스 1개의 CPU 사용률 행 그래프 위젯을 추가하고 저장했다.

선택한 인스턴스에 최근 지표가 없어 그래프가 비어 있을 수 있지만, 대시보드와 위젯 생성은 정상적으로 완료됐다.

핵심

CloudTrail은 AWS 콘솔·CLI에서 실행된 API 호출을 기록한다.

eventName은 실행한 작업, eventSource는 해당 AWS 서비스다.

CloudWatch Dashboard는 여러 리소스의 지표를 한 화면에서 확인하는 공간이다.

EC2가 실행 중이어야 CPU 지표가 수집되며, 지표 반영에는 시간이 걸릴 수 있다.

이력서 문장

AWS CloudTrail에서 VPC Endpoint 생성 API 호출을 검증하고, CloudWatch Dashboard에 EC2 CPUUtilization 지표를 구성했다.
