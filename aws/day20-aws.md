# AWS Day 20 - ECR

## 목표
Docker 이미지를 Amazon ECR private repository에 업로드한다.

## 구성
- 리전: ap-northeast-2
- ECR 저장소: practice-ecr-repo
- 로컬 이미지: spring-member-app:latest

## 실습 흐름
1. Docker Desktop과 WSL 연동
2. AWS CLI 설치 및 IAM 액세스 키 설정
3. AWS 계정 인증 확인
4. Docker를 ECR에 로그인
5. 기존 이미지에 ECR 주소 태그 지정
6. ECR에 이미지 Push
7. 콘솔에서 latest 이미지 확인

## 핵심 명령어
aws sts get-caller-identity

aws ecr get-login-password --region ap-northeast-2 | docker login --username AWS --password-stdin 794813215931.dkr.ecr.ap-northeast-2.amazonaws.com

docker tag spring-member-app:latest 794813215931.dkr.ecr.ap-northeast-2.amazonaws.com/practice-ecr-repo:latest

docker push 794813215931.dkr.ecr.ap-northeast-2.amazonaws.com/practice-ecr-repo:latest

## 결과
ECR에 latest 이미지 업로드 성공
이미지 크기: 약 163MB

## 다음 학습
21일차: ECS Fargate + ALB로 ECR 이미지 배포
