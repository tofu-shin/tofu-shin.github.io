---
title: "AI 자동화 실패 복구 가이드: 스케줄러/배포/메시지 꼬였을 때"
date: 2026-05-08 22:24:00 +0900
categories: [AI, Automation, Runbook]
tags: [AI 자동화 장애 복구, 스케줄러 문제 해결, Runbook, CI/CD, 운영]
---

AI 자동화 파이프라인은 평소엔 조용히 잘 돌지만, 한 번 꼬이면 스케줄 누락·중복 배포·메시지 폭주가 동시에 발생할 수 있다. 이 문서는 운영자가 즉시 따라 할 수 있는 장애 대응 Runbook이다. 목표는 “빠른 정상화 → 데이터 무결성 확인 → 재발 방지”다.

## 1) 장애 유형 분류

### A. 스케줄러 장애
- 증상: 작업 미실행, 같은 잡 중복 실행, 특정 시간대 지연
- 대표 영향: 리포트 누락, 예약 발행 실패, 비용 급증

### B. 배포 파이프라인 장애
- 증상: 빌드 성공인데 서비스 미반영, 롤백 실패, 환경변수 누락
- 대표 영향: 최신 프롬프트/모델 설정 미적용, API 오류 급증

### C. 메시지/알림 장애
- 증상: 같은 알림 다중 발송, 전송 실패 후 재시도 무한 루프, 채널 순서 역전
- 대표 영향: 사용자 혼란, 운영 알림 신뢰도 저하

## 2) 원인 진단

진단은 “시간축 정렬”이 핵심이다. 스케줄러 로그, 배포 로그, 메시지 큐 이벤트를 같은 타임존으로 맞춘다.

```bash
# 1) 최근 실패 잡 확인
crontab -l
journalctl -u cron --since "2 hours ago"

# 2) CI/CD 최근 실행
gh run list -L 20
gh run view <run-id> --log

# 3) 큐/워커 상태 (예: Redis)
redis-cli LLEN message:queue
redis-cli GET worker:last_heartbeat
```

체크리스트:
- [ ] 장애 시작 시각(T0)과 최초 증상 시각이 일치하는가?
- [ ] 최근 변경(배포, 시크릿 교체, 스케줄 수정)이 있었는가?
- [ ] 재시도 정책(backoff, max retry)이 의도대로 동작했는가?
- [ ] idempotency key(중복 방지 키)가 누락되었는가?

## 3) 즉시 조치

### 3-1. 확산 차단
1. 스케줄러 일시 정지 또는 문제 잡 disable
2. 메시지 발송 워커 scale down (폭주 차단)
3. 자동 배포 일시 중단

```bash
# 예시: 워커 확산 차단
kubectl scale deploy msg-worker --replicas=0 -n prod

# 예시: GitHub Actions 임시 비활성(저장소 설정 또는 워크플로우 disable)
gh workflow disable deploy.yml
```

### 3-2. 서비스 복구
1. 마지막 정상 릴리스로 롤백
2. 큐 적체량 확인 후 안전 재처리
3. 누락 작업만 수동 재실행(전체 재실행 금지)

```bash
# 롤백 예시
kubectl rollout undo deploy/ai-automation-api -n prod
kubectl rollout status deploy/ai-automation-api -n prod

# 특정 잡만 재실행
python scripts/replay_job.py --job-id <failed_job_id> --dry-run
python scripts/replay_job.py --job-id <failed_job_id>
```

### 3-3. 무결성 확인
- 중복 발송 건수, 누락 건수, 처리 지연 P95 확인
- 사용자 영향 범위(시간/채널/건수) 산출
- 필요한 경우 정정 메시지 1회 발송

## 4) 재발 방지

1. **Idempotency 기본화**: 메시지/배포 이벤트에 고유 키 강제
2. **회로 차단기 적용**: 실패율 임계치 초과 시 자동 정지
3. **관측성 강화**: “스케줄 지연, 큐 길이, 재시도 횟수” 대시보드화
4. **런북 자동화**: 자주 쓰는 복구 명령을 `make incident-*`로 표준화
5. **게임데이 운영**: 월 1회 장애 리허설로 복구 시간(RTO) 측정

운영 팁: AI 자동화 장애 복구는 “빨리 많이 고치는 것”보다 “더 꼬이지 않게 순서대로 고치는 것”이 중요하다. 스케줄러 문제 해결도 같은 원칙이다. 확산 차단 → 최소 복구 → 검증 → 재가동 순서를 팀 표준으로 고정하면, 실제 장애에서 판단 비용이 크게 줄어든다.