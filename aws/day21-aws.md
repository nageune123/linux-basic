# AWS Day 21 — ECS Fargate + ALB

- 실습일: 2026-09-17~18
- 목표: Spring Boot 컨테이너를 AWS에 배포하고 외부에서 API 호출하기.

## 진행

- ECR 이미지를 Fargate에서 실행하고 ALB·대상 그룹 연결.
- 태스크 정의 `practice-ecs-task:3`에 RDS 접속 환경 변수 설정.
- ECS 서비스의 원하는 태스크 수를 1개로 설정해 재배포.

## 문제와 해결

- ALB와 ECS 보안 그룹 연결 및 8080 허용 규칙 수정.
- 상태 검사 `/`의 404 오류: 경로를 `/docker-test`로 변경.
- 앱 시작 시간을 고려해 상태 검사 유예 기간을 60초에서 300초로 조정.

## 확인 결과

- ECS 배포 성공, 태스크 1개 실행.
- `GET /docker-test`: `Docker Build Test v2!`
- `POST /members`: 201 응답, 테스트 회원 생성.
- `GET /members`: 생성한 `ecs-test` 조회 성공.

## 배운 점

- 앱 실행 여부, ALB 상태 검사, 실제 API 동작을 각각 확인해야 한다.
