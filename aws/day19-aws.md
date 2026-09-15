# AWS Day 19 - Private RDS 구성 및 접속

## 목표

Private Subnet에 RDS MySQL을 배치하고 Private EC2를 통해 접속한다.

## 구성

- VPC: practice-vpc
- DB Subnet Group: practice-rds-subnet-group
- Private Subnet:
  - practice-private-subnet-2a
  - practice-private-subnet-2c
- RDS: practice-rds-mysql
- 엔진: MySQL 8.4.9
- 인스턴스: db.t4g.micro
- 스토리지: gp2 20GiB
- 퍼블릭 액세스: 아니요
- RDS 보안 그룹: practice-rds-sg
- 인바운드: TCP 3306
- 소스: practice-private-sg

## 접속 흐름

로컬 PC → SSH 터널 → Private EC2 → Private RDS

## 검증

- RDS MySQL 접속 성공
- MySQL 버전 8.4.9 확인
- practice_day19 데이터베이스 생성
- connection_test 테이블 생성
- 데이터 입력 및 조회 성공

## 핵심

DB Subnet Group은 RDS가 사용할 Private Subnet들을 묶은 그룹이다.
RDS 보안 그룹은 Private EC2 보안 그룹에서 오는 3306만 허용한다.
