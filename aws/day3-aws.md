# AWS EC2 Day 3 - 서버 운영 / 장애 확인

## 1. 서버 기본 상태 확인

### 서버 가동시간 / 부하

```bash
uptime
```

- 서버가 얼마나 오래 실행됐는지 확인
- `load average`로 시스템 부하 확인

### 메모리 확인

```bash
free -h
```

- RAM 사용량 확인
- Swap 사용량 확인

### 디스크 확인

```bash
df -h
```

- 파일시스템 전체 용량/사용량 확인
- `/` 사용률이 100%인 문제 발견

---

## 2. 디스크 사용 원인 추적

```bash
sudo du -xh / --max-depth=1 2>/dev/null | sort -h
```

- 어느 디렉터리가 공간을 많이 사용하는지 확인

```bash
sudo du -xh /var --max-depth=1 2>/dev/null | sort -h
```

- `/var` 내부 용량 확인

주요 사용 공간:

- `/var/lib/containerd`
- `/var/lib/docker`

---

## 3. Docker 디스크 사용량 확인

```bash
docker system df
```

확인 가능:

- Images
- Containers
- Volumes
- Build Cache

이미지별 크기 확인:

```bash
docker images
```

---

## 4. 불필요한 APT 캐시 정리

```bash
sudo apt clean
```

정리 후 확인:

```bash
df -h
```

---

## 5. AWS EBS 용량 확장

기존 EBS:

8 GiB

↓

16 GiB로 확장

### EBS란?

EBS = Elastic Block Store

EC2 인스턴스에서 사용하는 가상 디스크 스토리지.

---

## 6. Linux에서 디스크 / 파티션 확인

```bash
lsblk
```

AWS에서 EBS 자체의 크기를 늘렸다고 해서
Linux 파일시스템이 자동으로 전체 공간을 사용할 수 있는 것은 아니다.

필요한 경우:

EBS 확장
↓
파티션 확장
↓
파일시스템 확장

과정을 거친다.

---

## 7. ext 파일시스템 확장

```bash
sudo resize2fs /dev/nvme0n1p1
```

### resize2fs

ext2 / ext3 / ext4 계열 파일시스템의 크기를 조정하는 명령어.

확장 후:

```bash
df -h
```

결과:

- 루트 파일시스템 약 15GB
- 사용률 약 44%
- 디스크 부족 문제 해결

---

## 8. Docker 컨테이너 자원 확인

```bash
docker stats
```

확인 가능:

- CPU
- Memory
- Network I/O
- Block I/O
- PIDS

종료:

```text
Ctrl + C
```

한 번만 확인:

```bash
docker stats --no-stream
```

---

## 9. Docker 로그 확인

최근 30줄:

```bash
docker logs --tail 30 spring-app
```

실시간 로그:

```bash
docker logs -f spring-app
```

최근 10분:

```bash
docker logs --since 10m spring-app
```

ERROR 검색:

```bash
docker logs spring-app 2>&1 | grep ERROR
```

WARN 검색:

```bash
docker logs spring-app 2>&1 | grep WARN
```

최근 10분 ERROR:

```bash
docker logs --since 10m spring-app 2>&1 | grep ERROR
```

최근 10분 WARN:

```bash
docker logs --since 10m spring-app 2>&1 | grep WARN
```

---

## 10. 실제 장애 로그 분석

과거 MySQL 장애에서 다음 로그들을 확인했다.

- HikariPool connection 실패
- Communications link failure
- JDBC connection 실패
- JPA 초기화 실패
- Application run failed

장애 흐름:

MySQL 메모리 부족
↓
MySQL 컨테이너 OOMKilled
↓
Spring → MySQL 연결 실패
↓
HikariPool 연결 실패
↓
JPA 초기화 실패
↓
Spring 실행 실패

---

## 11. docker inspect

`docker inspect`는 컨테이너의 상세 상태와 설정을 확인할 때 사용한다.

### Spring 상태 확인

```bash
docker inspect spring-app --format='Status={{.State.Status}} Running={{.State.Running}} ExitCode={{.State.ExitCode}} Restart={{.HostConfig.RestartPolicy.Name}}'
```

결과:

```text
Status=running
Running=true
ExitCode=0
Restart=unless-stopped
```

### MySQL 상태 확인

```bash
docker inspect mysql --format='Status={{.State.Status}} Running={{.State.Running}} ExitCode={{.State.ExitCode}} OOMKilled={{.State.OOMKilled}} Restart={{.HostConfig.RestartPolicy.Name}}'
```

결과:

```text
Status=running
Running=true
ExitCode=0
OOMKilled=false
Restart=unless-stopped
```

현재 Spring / MySQL 모두 정상.

---

## 12. Linux 전체 프로세스 모니터링

```bash
top
```

주요 항목:

- `load average` = 시스템 부하
- `%CPU` = CPU 사용량
- `%MEM` = 메모리 사용량
- `id` = CPU idle 비율
- `COMMAND` = 실행 프로세스

확인한 주요 프로세스:

```text
java
→ Spring Boot

mysqld
→ MySQL
```

Docker 컨테이너 안에서 실행되는 프로그램도
결국 EC2 Linux에서는 프로세스로 실행된다.

### top과 docker stats 차이

```text
top
→ EC2 Linux 전체 프로세스 / 자원 확인

docker stats
→ Docker 컨테이너별 자원 확인
```

`top` 종료:

```text
q
```

---

# 핵심 장애 대응 흐름

서비스에 문제가 발생했다.

↓

```bash
docker ps
```

컨테이너 실행 여부 확인

↓

```bash
docker logs 컨테이너명
```

로그 확인

↓

```bash
docker logs 컨테이너명 2>&1 | grep ERROR
```

ERROR 확인

↓

```bash
docker inspect 컨테이너명
```

ExitCode / OOMKilled 등 상세 상태 확인

↓

```bash
free -h
```

메모리 / Swap 확인

↓

```bash
df -h
```

디스크 확인

↓

```bash
top
```

EC2 전체 시스템 상태 확인

↓

```bash
docker stats
```

컨테이너별 CPU / Memory 확인

---

# 오늘 핵심 명령어

```bash
uptime
free -h
df -h
lsblk
top

docker ps
docker stats
docker system df
docker images

docker logs --tail 30 spring-app
docker logs -f spring-app
docker logs --since 10m spring-app
docker logs spring-app 2>&1 | grep ERROR

docker inspect spring-app
docker inspect mysql
```

---

# 핵심 개념

## Swap

RAM이 부족할 때 디스크 일부를 임시 메모리처럼 사용하는 공간.

이번 EC2에서는:

```text
RAM  ≈ 1GB
Swap = 2GB
```

Swap을 추가해 MySQL이 메모리 부족으로 종료될 가능성을 줄였다.

---

## EBS

Elastic Block Store

EC2가 사용하는 AWS의 블록 스토리지.

이번 실습:

```text
8 GiB
↓
16 GiB
```

으로 확장했다.

---

## Docker 장애 확인 기본 순서

```text
상태
↓
로그
↓
상세 상태
↓
CPU / Memory
↓
Disk
↓
원인 파악
```

명령어로 연결하면:

```text
docker ps
↓
docker logs
↓
docker inspect
↓
docker stats / free -h / top
↓
df -h
```

---

# 현재 서버 최종 상태

- EC2 정상
- EBS 용량 확장 완료
- Swap 2GB 정상
- Docker 정상
- spring-app 정상
- MySQL 정상
- Spring → MySQL 연결 정상
- `/members` HTTP 요청 정상
- 최근 ERROR 없음
- 최근 WARN 없음

---

# 다음 학습 - AWS EC2 Day 4

## CloudWatch 모니터링

다음 학습 내용:

1. CloudWatch 기본 개념
2. Metrics
3. EC2 CPU 모니터링
4. CloudWatch Alarm
5. 임계값 설정
6. 서버 장애 감지 흐름
7. SAA-C03 관련 개념 연결

현재까지 직접 SSH로 확인했던:

```text
top
free -h
docker stats
docker logs
```

등의 서버 모니터링 개념을 AWS CloudWatch와 연결해서 학습한다.
