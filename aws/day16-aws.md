AWS Day 16 - S3 Gateway Endpoint

목표

NAT Gateway 없이 Private EC2에서 S3에 접근한다.

구성

VPC: practice-vpc

S3 Gateway Endpoint: practice-s3-gateway-endpoint-day16

서비스: com.amazonaws.ap-northeast-2.s3

라우팅 테이블: practice-private-rt

Private EC2: 10.0.2.129

IAM 역할: practice-day15-ec2-s3-readonly-role

S3 버킷: practice-day15-s3-794813215931-ap-northeast-2-an

진행

VPC 엔드포인트에서 S3 Gateway 유형을 선택했다.

practice-vpc와 practice-private-rt를 연결했다.

Private EC2에 S3 읽기 전용 IAM 역할을 연결했다.

Private EC2에 AWS CLI를 설치하고 S3 접근을 확인했다.

aws s3 ls s3://practice-day15-s3-794813215931-ap-northeast-2-an/ \
  --region ap-northeast-2 --no-cli-pager

aws s3 cp \
  's3://practice-day15-s3-794813215931-ap-northeast-2-an/Hello S3! Day15 IAM Role practice..txt' \
  - --region ap-northeast-2

결과

Private EC2에서 S3 객체 목록 확인 성공

S3 객체 내용 읽기 성공

NAT Gateway나 퍼블릭 IP 없이 S3 접근 성공

aws sts get-caller-identity는 STS용 경로가 없어 응답하지 않음

핵심

Gateway Endpoint는 VPC와 S3 사이의 사설 통신 경로다.

Endpoint는 통신 경로만 제공하고, IAM 역할이 S3 권한을 결정한다.

aws s3 cp S3경로 -에서 -는 파일 대신 터미널에 출력한다.

S3 Gateway Endpoint는 S3 전용이며 STS·인터넷 전체를 연결하지 않는다.

이력서 문장

AWS VPC Gateway Endpoint를 Private 라우팅 테이블에 연결하고, IAM Role 기반 Private EC2에서 NAT Gateway 없이 S3 객체 목록 조회와 다운로드를 검증했다.
