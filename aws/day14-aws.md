# AWS Day 14 - IAM 최소 권한 구성

## 목표

IAM의 사용자·그룹·정책·역할을 구성하고 최소 권한을 확인한다.

## 진행

- 고객 관리형 정책 `practice-day14-ec2-describe-only` 생성
- EC2 인스턴스 조회만 허용

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "ec2:DescribeInstances",
      "Resource": "*"
    }
  ]
}
```

- 정책을 `practice-day14-ec2-readonly-group`에 연결
- 콘솔 로그인·액세스 키가 없는 `practice-day14-user`를 그룹에 추가
- EC2가 사용할 `practice-day14-ec2-readonly-role` 생성
- 역할의 신뢰 주체를 EC2(`ec2.amazonaws.com`)로 설정
- IAM 정책 시뮬레이터로 권한 검증

## 결과

| 작업 | 결과 |
| --- | --- |
| `ec2:DescribeInstances` | 허용됨 |
| `ec2:StartInstances` | 거부됨(암시적 거부) |

## 핵심

- 정책은 허용·거부할 작업을 정의한다.
- 그룹은 사용자에게 정책을 묶어서 적용한다.
- 역할은 EC2 같은 AWS 서비스가 임시로 사용하는 권한 신분이다.
- 신뢰 정책은 누가 역할을 사용할 수 있는지, 권한 정책은 무엇을 할 수 있는지 정한다.
- 여러 정책의 허용 권한은 합쳐진다.

## 이력서 문장

> AWS IAM에서 EC2 조회 전용 최소 권한 정책을 생성하고 사용자 그룹과 EC2 역할에 연결한 뒤, 정책 시뮬레이터로 허용·거부 권한을 검증했다.
