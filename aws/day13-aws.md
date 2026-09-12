AWS Day 13 - CloudWatch 기반 동적 Auto Scaling

목표

CloudWatch Target Tracking 정책으로 트래픽에 따라 Auto Scaling 그룹의 EC2 수가 자동으로 늘고 줄어드는지 확인한다.

진행

practice-asg-day12 용량을 최소/희망/최대 2 / 2 / 3으로 설정

Auto Scaling 그룹 지표의 CloudWatch 수집 활성화

CPU 정책 practice-cpu-target-tracking 생성(평균 CPU 5%)

정적 Nginx 요청은 CPU 부하가 낮아 ALB 요청 수 정책으로 전환

practice-alb-request-count 생성(대상당 10개/분, 워밍업 60초)

CloudShell에서 ALB DNS로 반복 요청을 전송

결과

요청 수 증가 → 정책 알람 실행

희망 용량 2 → 3, 새 EC2 i-0cb8c200295593b45 생성

요청 중단 → 저부하 알람 실행

희망 용량 3 → 2, ALB Connection Draining 후 인스턴스 종료 진행

모니터링 그래프에서 원하는 용량과 서비스 중 인스턴스가 2 → 3 → 2로 변화

핵심

CloudWatch 지표가 목표값을 넘으면 Auto Scaling이 확장하고, 낮아지면 축소한다.

활동 탭은 실제 확장·축소 작업과 원인을 확인하는 곳이다.

모니터링 탭은 그룹 용량 변화를 그래프로 보여준다.

ALB는 인스턴스 종료 전에 연결을 정리해 서비스 중단을 줄인다.

이력서 문장

AWS Auto Scaling에 CloudWatch Target Tracking 정책을 구성하고, ALB 요청 부하로 EC2 인스턴스의 2→3 자동 확장과 3→2 자동 축소를 검증했다.
