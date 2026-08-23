# AWS Day 2 - EC2 운영 기초

## 1. 오늘의 목표

EC2에서 실행 중인 Spring Boot + MySQL Docker 환경을
서버 재부팅이나 EC2 중지/시작 이후에도 자동으로 복구되도록 설정하고 확인했다.

---

## 2. EC2 메모리 확인

현재 서버의 메모리 상태를 확인했다.

```bash
free -h
```

초기 상태:

- RAM 약 1GB
- Swap 없음

MySQL 컨테이너가 Exit Code 137로 종료되는 문제가 발생했다.

확인:

```bash
docker inspect mysql --format='OOMKilled={{.State.OOMKilled}} ExitCode={{.State.ExitCode}}'
```

결과:

```text
OOMKilled=true ExitCode=137
```

메모리 부족(OOM)으로 MySQL 컨테이너가 종료된 것을 확인했다.

---

## 3. Swap 2GB 생성

RAM이 부족할 때 디스크 일부를 보조 메모리처럼 사용할 수 있도록
2GB Swap 파일을 생성했다.

```bash
sudo fallocate -l 2G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
```

확인:

```bash
free -h
```

Swap:

```text
2.0Gi
```

---

## 4. Swap 영구 설정

현재 설정만으로는 서버 재부팅 후 Swap이 자동 활성화되지 않을 수 있기 때문에
`/etc/fstab`에 등록했다.

```bash
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```

확인:

```bash
tail -n 5 /etc/fstab
```

### 명령어 정리

- `tee -a` : 기존 파일 내용을 유지하면서 파일 마지막에 내용 추가
- `tail` : 파일의 마지막 부분 확인
- `tail -n 5` : 파일의 마지막 5줄 확인

---

## 5. Docker Restart Policy 확인

컨테이너의 자동 재시작 정책을 확인했다.

```bash
docker inspect spring-app --format='{{.HostConfig.RestartPolicy.Name}}'
docker inspect mysql --format='{{.HostConfig.RestartPolicy.Name}}'
```

기존 결과:

```text
no
```

서버가 재부팅되었을 때 컨테이너가 자동으로 실행되도록 설정했다.

```bash
docker update --restart unless-stopped mysql
docker update --restart unless-stopped spring-app
```

다시 확인:

```bash
docker inspect mysql --format='{{.HostConfig.RestartPolicy.Name}}'
docker inspect spring-app --format='{{.HostConfig.RestartPolicy.Name}}'
```

결과:

```text
unless-stopped
```

### unless-stopped

사용자가 명시적으로 중지하지 않은 컨테이너는
Docker 또는 서버가 재시작될 때 자동으로 다시 실행한다.

---

## 6. EC2 재부팅 테스트

Ubuntu 서버 자체를 재부팅했다.

```bash
sudo reboot
```

### sudo

뒤의 명령어를 관리자(root) 권한으로 실행한다.

### reboot

Linux 운영체제를 재부팅한다.

`exit`은 SSH 접속만 종료하지만,
`reboot`는 서버의 운영체제 자체를 재부팅한다.

---

## 7. 재부팅 후 확인

SSH로 다시 접속한 뒤 다음을 확인했다.

### Swap 확인

```bash
free -h
```

Swap 2GB가 자동으로 활성화되어 있었다.

### Docker 컨테이너 확인

```bash
docker ps
```

결과:

- spring-app : Up
- mysql : Up

두 컨테이너 모두 자동으로 실행되었다.

### Spring API 확인

```bash
curl -i http://localhost:8080/members
```

결과:

```text
HTTP/1.1 200
[]
```

Spring Boot → JPA → MySQL 연결이 정상적으로 동작하는 것을 확인했다.

---

## 8. EC2 Reboot와 Stop/Start 차이

### sudo reboot

```text
Ubuntu OS 재부팅
        ↓
EC2 인스턴스는 그대로 유지
        ↓
일반적으로 Public IPv4 유지
```

### EC2 Stop → Start

AWS Console에서:

```text
EC2 중지
   ↓
EC2 시작
   ↓
Public IPv4가 변경될 수 있음
```

실제로 EC2를 중지 후 다시 시작했더니
기존 Public IPv4와 다른 새로운 Public IPv4가 할당되었다.

---

## 9. 새로운 Public IP로 SSH 접속

EC2 Stop → Start 이후 기존 IP 대신
새로 할당된 Public IPv4를 사용해서 SSH 접속했다.

```bash
ssh -i ~/.ssh/docker-practice-key.pem ubuntu@새로운_PUBLIC_IP
```

정상적으로 접속되었다.

---

## 10. Stop → Start 이후 최종 점검

### 메모리 / Swap

```bash
free -h
```

Swap 2GB가 정상적으로 유지되었다.

### Docker

```bash
docker ps
```

spring-app과 mysql이 자동으로 실행되었다.

### API

```bash
curl -i http://localhost:8080/members
```

결과:

```text
HTTP/1.1 200
[]
```

EC2를 완전히 중지했다가 시작한 후에도
Spring Boot + MySQL 서비스가 자동으로 정상 복구되는 것을 확인했다.

---

## 11. Elastic IP 개념

일반 EC2 Public IPv4는 Stop → Start 과정에서 변경될 수 있다.

서버 주소가 계속 변경되면 SSH 접속 주소나
서비스 접속 주소를 계속 수정해야 하는 문제가 생긴다.

Elastic IP는 EC2에 연결해서 사용할 수 있는
고정 Public IPv4 주소이다.

```text
일반 Public IPv4
      ↓
EC2 Stop → Start
      ↓
IP 변경 가능

Elastic IP
      ↓
고정 Public IPv4 사용
```

---

# 오늘 사용한 주요 명령어

```bash
free -h

docker ps
docker ps -a

docker inspect mysql
docker inspect spring-app

docker inspect mysql --format='OOMKilled={{.State.OOMKilled}} ExitCode={{.State.ExitCode}}'

docker update --restart unless-stopped mysql
docker update --restart unless-stopped spring-app

sudo fallocate -l 2G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile

tail -n 5 /etc/fstab

sudo reboot

curl -i http://localhost:8080/members
```

---

# 오늘의 전체 흐름

```text
EC2 서버
   ↓
메모리 부족 문제 확인
   ↓
MySQL OOMKilled / Exit Code 137 확인
   ↓
Swap 2GB 생성
   ↓
/etc/fstab에 Swap 영구 등록
   ↓
Docker Restart Policy 설정
   ↓
sudo reboot
   ↓
Swap + Docker 자동 복구 확인
   ↓
Spring API 200 확인
   ↓
EC2 Stop
   ↓
EC2 Start
   ↓
Public IPv4 변경 확인
   ↓
새 IP로 SSH 접속
   ↓
Swap 확인
   ↓
Docker 확인
   ↓
Spring API 확인
   ↓
HTTP 200 OK
```

---

# 핵심 정리

서버 운영에서는 단순히 애플리케이션을 실행하는 것뿐만 아니라

- 서버 상태 확인
- 메모리 관리
- 장애 원인 확인
- 서비스 자동 재시작
- 서버 재부팅 후 복구
- 네트워크/IP 관리
- 최종 서비스 동작 확인

과 같은 운영 과정도 중요하다.

이번 실습에서 가장 중요한 흐름은 다음과 같다.

```text
EC2
 ↓
Ubuntu
 ↓
Docker
 ↓
Spring Boot + MySQL
 ↓
서버 상태/메모리 확인
 ↓
문제 발생 시 로그 및 상태 확인
 ↓
재부팅/중지 후 자동 복구
 ↓
curl로 최종 서비스 확인
```

특히 다음 명령어들은 EC2 서버를 운영하거나 문제를 확인할 때 자주 사용할 수 있다.

```bash
free -h
docker ps
docker ps -a
docker logs 컨테이너명
docker inspect 컨테이너명
curl -i http://localhost:8080/members
```

단순히 "Docker 컨테이너를 실행하는 것"에서 끝나는 것이 아니라,
서버가 재부팅되거나 문제가 발생해도 다시 서비스를 정상 상태로 만드는 것이
운영에서 중요하다는 것을 실습했다.
