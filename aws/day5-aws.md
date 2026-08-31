# AWS EC2 Day 5 - CloudWatch Agent / RAM 모니터링 / Memory Alarm

## 1. 오늘의 목표

EC2의 RAM 사용률을 CloudWatch에서 모니터링하고,
메모리 사용률이 일정 수준을 넘으면 SNS를 통해 이메일 알림을 받는 환경을 구성했다.

전체 구조:

EC2
↓
Ubuntu OS
↓
CloudWatch Agent
↓
RAM 사용률 수집
↓
CloudWatch
↓
mem_used_percent
↓
CloudWatch Alarm
↓
SNS
↓
Email


---

# 2. CPU와 RAM 복습

## CPU

CPU는 실제 계산과 명령을 처리하는 장치이다.

예:

Spring 요청 처리
Docker 프로세스 실행
MySQL 쿼리 처리

CPU 사용률이 높다는 것은
CPU가 많은 작업을 처리하고 있다는 의미이다.


## RAM

RAM은 현재 실행 중인 프로그램이 사용하는 작업 공간이다.

예:

Spring Boot
MySQL
Docker
Ubuntu OS

등이 실행되면서 RAM을 사용한다.


쉽게 기억:

CPU = 일하는 사람

RAM = 일할 때 사용하는 책상

Disk = 자료를 장기간 보관하는 창고


---

# 3. OS란?

OS = Operating System (운영체제)

컴퓨터의 하드웨어와 프로그램 사이에서
CPU, RAM, Disk, Network 등의 자원을 관리한다.

현재 EC2에서는 Ubuntu Linux를 사용하고 있다.

구조:

EC2 가상 서버
↓
Ubuntu OS
↓
Docker
↓
Spring Boot / MySQL


---

# 4. 기본 EC2 CloudWatch 지표의 한계

EC2의 기본 CloudWatch 지표에서는 CPU 사용률 등을 확인할 수 있다.

대표적인 지표:

CPUUtilization

하지만 RAM 사용률은 기본 EC2 지표로 제공되지 않는다.

RAM은 EC2 내부 OS가 관리하는 정보이기 때문이다.


---

# 5. CloudWatch Agent

EC2 내부의 RAM, Disk 등의 정보를 CloudWatch로 보내기 위해
CloudWatch Agent를 사용할 수 있다.

구조:

EC2
↓
Ubuntu
↓
CloudWatch Agent
↓
CloudWatch

CloudWatch Agent가 EC2 내부 정보를 수집하여
CloudWatch로 전송한다.


---

# 6. IAM Role

CloudWatch Agent가 CloudWatch에 데이터를 보내기 위해서는
AWS에 데이터를 전송할 수 있는 권한이 필요하다.

이때 IAM Role을 이용한다.

쉽게 기억:

IAM Role = AWS 서비스가 사용하는 권한증

구조:

CloudWatch Agent
↓
IAM Role의 권한 사용
↓
CloudWatch에 Metric 전송


---

# 7. CloudWatch Agent Metric 확인

CloudWatch에서 CWAgent 네임스페이스를 확인했다.

대표적으로 확인한 지표:

mem_used_percent
disk_used_percent
swap_used_percent
cpu_usage_system
cpu_usage_iowait
diskio_io_time

이번 실습에서는

mem_used_percent

를 사용했다.


---

# 8. Linux에서 RAM 확인

EC2에 SSH로 접속하여 다음 명령어로 RAM 상태를 확인했다.

```bash
free -h

주요 항목:

total = 전체 RAM
used = 사용 중인 RAM
free = 완전히 비어 있는 RAM
buff/cache = Linux가 캐시 등에 사용하는 메모리
available = 프로그램이 추가로 사용할 수 있는 메모리

Swap도 함께 확인할 수 있다.

9. CloudWatch Memory Alarm 생성

CloudWatch에서 다음 조건으로 메모리 경보를 생성했다.

Alarm Name:

EC2-High-Memory-Alarm

Metric:

mem_used_percent

Statistic:

Average

Period:

5 minutes

Threshold:

mem_used_percent > 70%

Datapoints to alarm:

1 / 1

의미:

5분 단위로 평균 RAM 사용률을 계산하고
1개의 데이터 포인트가 70%를 초과하면
ALARM 상태로 전환한다.

10. SNS 연결

Memory Alarm이 ALARM 상태가 되었을 때
기존 SNS Topic으로 알림을 보내도록 설정했다.

구조:

Memory > 70%
↓
CloudWatch Alarm
↓
ALARM
↓
SNS Topic
↓
Email

11. 메모리 부하 테스트

메모리 경보가 실제로 작동하는지 확인하기 위해
EC2에서 메모리 사용량을 증가시켰다.

테스트 과정에서 stress-ng를 설치했다.

설치 확인:

stress-ng --version

그리고 메모리 부하를 발생시켜
RAM 사용률을 증가시켰다.

12. 실제 결과

CloudWatch의 mem_used_percent가 점차 증가했다.

약 40%
↓
약 60%
↓
70% 초과

최종적으로:

EC2-High-Memory-Alarm

상태가

OK
↓
ALARM

으로 변경되었다.

SNS를 통해 이메일 알림도 정상적으로 수신했다.

13. 부하 종료 후 확인

메모리 부하를 종료한 뒤:

free -h

를 실행하여 메모리가 다시 확보되는 것을 확인했다.

즉,

부하 발생
↓
RAM 증가
↓
CloudWatch Agent 수집
↓
CloudWatch Metric 증가
↓
70% 초과
↓
Alarm 발생
↓
SNS
↓
Email

전체 모니터링 흐름을 실제로 확인했다.

14. Day 4와 Day 5 비교
Day 4

CPU 모니터링

EC2
↓
CloudWatch
↓
CPUUtilization
↓
CPU Alarm
↓
SNS
↓
Email

Day 5

RAM 모니터링

EC2
↓
Ubuntu
↓
CloudWatch Agent
↓
mem_used_percent
↓
Memory Alarm
↓
SNS
↓
Email

가장 중요한 차이:

CPUUtilization
= EC2 기본 CloudWatch Metric

mem_used_percent
= CloudWatch Agent가 수집해서 보내는 Metric

15. 핵심 암기 구조
서버 내부 확인

CPU
→ top

RAM
→ free -h

Disk
→ df -h

AWS 모니터링

Metric
= 무엇을 측정할 것인가

Statistic
= 측정값을 어떻게 계산할 것인가

Period
= 몇 분 단위로 계산할 것인가

Threshold
= 어느 값을 문제 기준으로 정할 것인가

전체 구조

EC2
↓
CloudWatch Agent
↓
CloudWatch Metric
↓
Alarm
↓
SNS
↓
Email

Day 5 핵심 한 줄

CloudWatch Agent를 이용해 EC2 내부의 RAM 사용률을 CloudWatch로 전송하고,
Memory Alarm과 SNS를 연결하여 메모리 이상 상태를 이메일로 알림받는 환경을 구축했다.
