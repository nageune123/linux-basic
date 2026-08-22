# AWS EC2 + Docker 배포 실습

## 1. 오늘의 목표

로컬에서 만든 Spring Boot Docker 이미지를 AWS EC2에 배포하고
MySQL 컨테이너와 연결하여 외부에서 API에 접속한다.

---

## 2. AWS EC2 생성

- Ubuntu 26.04 LTS
- 인스턴스 유형: t3.micro
- SSH 접속용 Key Pair 생성
- `.pem` 키 사용

SSH 접속을 위해 개인키 권한 변경:

```bash
chmod 400 ~/.ssh/docker-practice-key.pem
EC2 접속 후 확인:

whoami
hostname
pwd
cat /etc/os-release
3. EC2에 Docker 설치

Docker 설치 후 버전 확인:

docker --version

Docker 서비스 상태 확인:

sudo systemctl status docker

컨테이너 확인:

docker ps
docker ps -a
4. Docker Hub에서 Spring 이미지 받기
sudo docker pull <DockerHub-ID>/spring-member:latest

이미지 확인:

sudo docker images
5. Spring + MySQL 컨테이너 실행

구성:

EC2
 └─ Docker
     ├─ spring-app : 8080
     └─ mysql      : 3306

Spring의 DB 주소:

jdbc:mysql://mysql:3306/memberdb

여기서 mysql은 IP 주소가 아니라
Docker Network에서 사용하는 MySQL 컨테이너 이름이다.

6. Docker Network

Spring과 MySQL이 서로 통신하려면
같은 Docker Network에 연결되어 있어야 한다.

docker network connect spring-network mysql

Spring 재시작:

docker restart spring-app
7. 발생한 문제 1 - UnknownHostException

오류:

java.net.UnknownHostException: mysql

원인:

Spring 컨테이너가 mysql이라는 컨테이너 이름을
DNS로 찾을 수 없었다.

Spring과 MySQL이 같은 Docker Network에 연결되어 있지 않았기 때문이다.

해결:

docker network connect spring-network mysql
docker restart spring-app
8. 발생한 문제 2 - MySQL 종료

확인:

docker ps -a

MySQL 상태:

Exited (137)

추가 확인:

docker inspect mysql --format='OOMKilled={{.State.OOMKilled}} ExitCode={{.State.ExitCode}}'

결과:

OOMKilled=true
ExitCode=137

즉 EC2의 메모리가 부족해서
Linux OOM Killer가 MySQL 프로세스를 종료한 것이었다.

9. Swap 2GB 추가

EC2 메모리 확인:

free -h

Swap 생성:

sudo fallocate -l 2G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile

확인:

free -h

결과:

Swap: 2.0Gi
10. API 테스트

EC2 내부에서:

curl -i http://localhost:8080/members

결과:

HTTP/1.1 200
Content-Type: application/json

[]

200은 요청이 정상 처리되었다는 의미이고,
[]는 현재 회원 데이터가 없다는 의미이다.

11. AWS Security Group

외부 PC에서 Spring Boot에 접근하기 위해
EC2 Security Group에 TCP 8080 인바운드 규칙을 추가했다.

실습에서는 가능하면:

TCP 8080
Source: My IP

처럼 필요한 IP만 허용한다.

MySQL의 3306 포트는 인터넷 전체에 공개하지 않는다.

12. 최종 구조
내 PC 브라우저
      |
      | HTTP :8080
      v
AWS Security Group
      |
      v
EC2 Ubuntu
      |
      v
Docker
  |
  +-- spring-app :8080
  |       |
  |       | JDBC
  |       v
  +-- mysql :3306

외부 브라우저에서도 /members 요청 결과:

[]

정상 확인.

오늘 배운 핵심
EC2는 AWS에서 사용하는 가상 서버이다.
SSH Key Pair로 EC2에 접속할 수 있다.
Security Group은 EC2의 방화벽 역할을 한다.
Docker Hub 이미지를 EC2에서 Pull하여 실행할 수 있다.
컨테이너끼리 이름으로 통신하려면 같은 Docker Network가 필요하다.
UnknownHostException: mysql은 Docker 네트워크/DNS 문제를 의심할 수 있다.
ExitCode 137과 OOMKilled=true는 메모리 부족을 의심한다.
작은 EC2에서는 Spring + MySQL 실행 시 메모리 관리가 중요하다.
HTTP 200은 서버가 요청을 정상적으로 처리했다는 뜻이다.
