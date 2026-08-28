# AWS EC2 Day 4 - CloudWatch 모니터링 / CPU 경보

## 1. CloudWatch란?

Amazon CloudWatch는 AWS 리소스와 애플리케이션의 상태를 모니터링하기 위한 서비스이다.

EC2에 직접 SSH로 접속해서 확인했던:

```bash
top
free -h
df -h
docker stats
docker logs
```

등의 직접적인 서버 확인 방식과 함께 CloudWatch를 이용하면 AWS 리소스의 지표를 지속적으로 모니터링하고 조건에 따라 경보를 발생시킬 수 있다.

기본 흐름:

```text
EC2
 ↓
Metric 수집
 ↓
CloudWatch
 ↓
Alarm
 ↓
SNS
 ↓
이메일 알림
```

---

# 2. Metric

## Metric이란?

Metric(메트릭)은 시스템의 상태를 나타내는 **측정값 / 지표**이다.

EC2에서 확인한 대표적인 Metric:

```text
CPUUtilization
NetworkIn
NetworkOut
CPU Credit 관련 지표
```

예:

```text
시간        CPUUtilization
09:00           10%
09:05           20%
09:10           85%
09:15           95%
```

즉,

```text
Metric = 무엇을 측정하는가?
```

이번 실습에서는:

```text
CPUUtilization
```

을 사용했다.

---

# 3. EC2 기본 Metric과 메모리 Metric

EC2 콘솔의 모니터링 탭에서는 CPU 사용률과 네트워크 등의 지표를 기본적으로 확인할 수 있다.

하지만 Linux 내부의 RAM 사용률은 EC2 기본 Metric에 포함되지 않는다.

Linux에서 RAM을 확인할 때 사용했던 명령어:

```bash
free -h
```

디스크 사용량 확인:

```bash
df -h
```

암기:

```text
free = RAM
df   = Disk
```

RAM, 디스크 사용률 등 OS 내부의 추가 정보를 CloudWatch로 수집하려면 **CloudWatch Agent**를 사용할 수 있다.

구조:

```text
EC2 Linux
   ↓
RAM / Disk 등 OS 정보
   ↓
CloudWatch Agent
   ↓
CloudWatch
```

---

# 4. CloudWatch Metric 통계

CloudWatch에서는 Metric을 그대로 보는 것뿐 아니라 일정 기간의 데이터를 통계로 계산할 수 있다.

대표적인 통계:

```text
Average     = 평균
Maximum     = 최대값
Minimum     = 최소값
SampleCount = 데이터 개수
Sum         = 합계
```

예:

```text
CPU 사용률

20%
25%
100%
22%
20%
```

순간적인 CPU 폭증을 확인할 때는 Maximum이 유용할 수 있다.

Average를 사용하면 일정 기간의 전체적인 CPU 사용 상태를 확인할 수 있다.

---

# 5. Period

Period(기간)는 Metric 데이터를 **어느 시간 단위로 묶어서 통계를 계산할 것인지**를 의미한다.

이번 실습:

```text
Period = 5분
Statistic = Average
```

예:

```text
09:00 ~ 09:05 → 평균 CPU 계산
09:05 ~ 09:10 → 평균 CPU 계산
09:10 ~ 09:15 → 평균 CPU 계산
```

핵심:

```text
Metric    = 무엇을 측정?
Statistic = 어떻게 계산?
Period    = 어느 시간 단위로 계산?
```

이번 실습:

```text
Metric    = CPUUtilization
Statistic = Average
Period    = 5분
```

---

# 6. CloudWatch Alarm

CloudWatch Alarm은 Metric을 감시하다가 설정한 조건을 만족하면 경보 상태로 변경한다.

이번 실습에서는 EC2 CPU 사용률을 감시했다.

조건:

```text
CPUUtilization > 80%
```

즉,

```text
CPU Metric
   ↓
CloudWatch Alarm
   ↓
80% 초과 여부 판단
```

---

# 7. Threshold

Threshold = 임계값

정상과 비정상을 구분하기 위해 설정하는 기준값이다.

이번 실습:

```text
Threshold = 80%
```

따라서:

```text
CPU 30% → 정상
CPU 60% → 정상
CPU 79% → 정상
----------------
CPU 80% → 설정한 비교 연산에 따라 판단
CPU 85% → 경보 조건 충족
CPU 95% → 경보 조건 충족
```

이번에는 조건을:

```text
CPUUtilization > 80%
```

으로 설정했다.

---

# 8. 경보 데이터 포인트

이번 실습에서는:

```text
1 / 1
```

로 설정했다.

의미:

```text
평가한 데이터 포인트 1개 중
1개가 임계값을 위반하면
↓
ALARM
```

예를 들어 `2 / 3`이라면:

```text
최근 평가 데이터 3개 중
2개가 조건을 위반하면
↓
ALARM
```

여러 데이터 포인트를 이용하면 순간적인 일시적 변화 때문에 불필요한 경보가 발생하는 것을 줄일 수 있다.

---

# 9. SNS

SNS = Simple Notification Service

AWS에서 알림을 전달하는 서비스이다.

CloudWatch Alarm과 연결하면 경보가 발생했을 때 이메일 등의 방법으로 알림을 전달할 수 있다.

구조:

```text
CloudWatch Alarm
      ↓
   SNS Topic
      ↓
   Subscriber
      ↓
     Email
```

역할:

```text
CloudWatch Alarm
→ 문제가 발생했는지 판단

SNS
→ 발생한 알림을 전달
```

---

# 10. SNS Topic / Subscription

## Topic

SNS에서 메시지를 전달하는 통로이다.

이번 실습에서는 CloudWatch Alarm과 SNS Topic을 연결했다.

## Subscription

SNS Topic에서 보내는 메시지를 받을 대상을 등록하는 것이다.

이메일을 등록하면 AWS에서 구독 확인 메일이 발송된다.

```text
SNS Topic 생성
 ↓
이메일 등록
 ↓
Subscription Confirmation 메일
 ↓
Confirm subscription
 ↓
구독 완료
```

구독 확인을 완료해야 실제 SNS 알림을 이메일로 받을 수 있다.

---

# 11. CloudWatch CPU Alarm 생성

이번 실습에서 생성한 경보:

```text
이름:
EC2-High-CPU-Alarm
```

설정:

```text
Metric     = CPUUtilization
Statistic  = Average
Period     = 5분
Threshold  = 80%
Condition  = CPUUtilization > 80%
Datapoints = 1 / 1
```

그리고:

```text
ALARM
 ↓
SNS Topic
 ↓
Email
```

구조로 알림을 연결했다.

---

# 12. Alarm 상태

CloudWatch Alarm에서는 대표적으로 다음 상태를 확인할 수 있다.

```text
OK
ALARM
INSUFFICIENT_DATA
```

## OK

현재 Metric이 설정한 경보 조건을 위반하지 않는 상태.

## ALARM

Metric이 설정한 임계값 조건을 위반한 상태.

## INSUFFICIENT_DATA

경보를 평가할 데이터가 충분하지 않은 상태.

경보를 처음 생성했을 때 잠시 `데이터 부족` 상태가 나타났고 이후 데이터가 수집되면서 정상 상태로 변경되는 것을 확인했다.

---

# 13. CPU 부하 테스트

CloudWatch Alarm이 실제로 동작하는지 확인하기 위해 EC2 CPU에 의도적으로 부하를 발생시켰다.

기존 CPU 상태 확인:

```bash
top
```

초기 상태:

```text
id ≈ 99%
```

`id`는 CPU Idle 비율이다.

즉:

```text
id 99%
→ CPU가 약 99% 쉬고 있음
→ CPU 사용량은 매우 낮음
```

---

# 14. yes 명령어로 CPU 부하 발생

테스트를 위해 실행:

```bash
yes > /dev/null
```

`yes`가 계속 데이터를 생성하고 이를 `/dev/null`로 버리면서 CPU 연산을 계속 수행한다.

첫 번째 `yes` 실행 후:

```text
yes → CPU 약 100%

전체 CPU idle ≈ 49%
```

EC2가 2 vCPU 환경이었기 때문에 `yes` 프로세스 하나가 하나의 vCPU를 거의 모두 사용하면서 전체 CPU 기준으로는 약 절반 정도의 부하가 발생했다.

---

# 15. 두 vCPU에 부하 발생

`yes`를 하나 더 실행:

```bash
yes > /dev/null
```

`top` 확인 결과:

```text
yes    ≈ 100%
yes    ≈ 100%

CPU idle = 0%
```

즉 두 vCPU가 거의 모두 사용되는 상태를 만들었다.

```text
id = 0%
→ CPU가 쉬는 시간이 거의 없음
→ CPU 전체 사용률 거의 100%
```

---

# 16. CloudWatch Alarm 실제 발생

CPU 부하를 유지한 뒤 CloudWatch에서:

```text
정상(OK)
 ↓
경보(ALARM)
```

상태 변화를 확인했다.

CloudWatch에서 수집된 CPUUtilization은 약:

```text
99.99%
```

까지 상승했다.

설정한 임계값:

```text
80%
```

을 초과했기 때문에:

```text
CPUUtilization ≈ 99.99%
        ↓
Threshold 80% 초과
        ↓
CloudWatch Alarm
        ↓
ALARM
```

이 발생했다.

---

# 17. SNS 이메일 알림 확인

CloudWatch Alarm이 ALARM 상태가 된 후 SNS를 통해 실제 이메일이 도착하는 것을 확인했다.

전체 흐름:

```text
yes 프로세스 실행
 ↓
EC2 CPU 사용률 증가
 ↓
CPUUtilization Metric 증가
 ↓
CloudWatch가 Metric 평가
 ↓
5분 평균 CPU > 80%
 ↓
CloudWatch Alarm
 ↓
SNS Topic
 ↓
Email
```

이를 통해 CloudWatch → Alarm → SNS 알림 구조가 실제로 동작하는 것을 확인했다.

---

# 18. CPU 부하 종료

테스트가 끝난 뒤 각각의 `yes` 프로세스를 실행한 터미널에서:

```text
Ctrl + C
```

로 종료했다.

다시:

```bash
top
```

으로 확인한 결과:

```text
id ≈ 99%
```

로 돌아왔다.

즉 CPU 부하가 제거되고 서버가 다시 정상 상태로 돌아왔다.

CloudWatch Metric에도 이후 정상화된 CPU 사용률이 반영된다.

---

# 19. top의 id 의미

`top`에서:

```text
id = idle
```

CPU가 아무 작업을 하지 않고 쉬고 있는 비율이다.

예:

```text
id = 99%
→ CPU 대부분 쉬고 있음

id = 50%
→ CPU 약 절반 사용 중

id = 0%
→ CPU가 거의 전부 사용 중
```

대략적으로:

```text
CPU 사용률 ≈ 100 - idle
```

로 이해할 수 있다.

---

# 20. Day 3와 Day 4 연결

Day 3에서는 서버에 직접 접속해서 상태를 확인했다.

```text
SSH
 ↓
top
free -h
df -h
docker stats
docker logs
```

Day 4에서는 AWS 모니터링 서비스를 이용했다.

```text
EC2
 ↓
CloudWatch Metrics
 ↓
CloudWatch Alarm
 ↓
SNS
 ↓
Email
```

즉:

```text
Day 3
직접 서버에 접속해서 장애 확인

Day 4
CloudWatch를 이용해 지속적으로 상태를 감시하고
조건 충족 시 자동으로 알림
```

---

# 핵심 개념 정리

## Metric

```text
측정값 / 지표
```

예:

```text
CPUUtilization
NetworkIn
NetworkOut
```

## Statistic

```text
Metric 데이터를 어떻게 계산할 것인지
```

예:

```text
Average
Maximum
Minimum
```

## Period

```text
어느 시간 단위로 데이터를 묶어 계산할 것인지
```

## Threshold

```text
정상 / 비정상을 판단하기 위한 기준값
```

## Alarm

```text
Metric이 설정한 조건을 만족하는지 감시하고 상태를 변경
```

## SNS

```text
알림을 사용자나 시스템에 전달
```

## CloudWatch Agent

```text
EC2 운영체제 내부의 추가 Metric을 수집하여
CloudWatch로 전송하는 에이전트
```

---

# 오늘 핵심 흐름

```text
EC2
 ↓
Metric
 ↓
CloudWatch
 ↓
Alarm
 ↓
SNS
 ↓
Email
```

실제 실습:

```text
yes 2개 실행
 ↓
CPU 거의 100%
 ↓
CPUUtilization 약 99.99%
 ↓
5분 평균 > 80%
 ↓
EC2-High-CPU-Alarm
 ↓
ALARM
 ↓
SNS
 ↓
이메일 수신
 ↓
yes 종료
 ↓
CPU 정상화
```

---

# 오늘 핵심 명령어

```bash
# EC2 전체 프로세스 / CPU 확인
top

# CPU 부하 테스트
yes > /dev/null

# 실행 중인 yes 종료
Ctrl + C
```

SSH 접속에 사용한 개인 키는 보안을 위해 권한을 제한했다.

```bash
chmod 400 ~/docker-practice-key.pem
```

---

# Day 4 핵심 암기

```text
Metric
= 무엇을 측정?

Statistic
= 어떻게 계산?

Period
= 어느 시간 단위?

Threshold
= 어디부터 문제로 볼 것인가?

Alarm
= 조건을 감시하고 상태 판단

SNS
= 알림 전달
```

한 줄로:

```text
Metric 측정
→ CloudWatch 모니터링
→ Alarm 판단
→ SNS 전달
→ Email 알림
```

---

# 다음 학습

## AWS EC2 Day 5

다음 단계에서는 Day 4에서 배운 CloudWatch 모니터링을 기반으로 다음 내용을 이어서 학습한다.

- CloudWatch Agent
- EC2 메모리(RAM) Metric 수집
- 디스크 사용률 Metric 수집
- 기본 EC2 Metric과 Agent Metric 차이
- IAM Role과 CloudWatch Agent 권한
- 실제 서버 운영 모니터링 확장
