---
layout: post
title: "6. OpenClaw 실전편 - 실패를 10분 안에 복구하는 대응 템플릿"
date: 2026-03-09 09:00:00 +0900
categories: [AI, OpenClaw]
tags: [openclaw, automation, blog, incident-response, git, checklist, rollback]
---

# OpenClaw 실전편 #6

![OpenClaw](/assets/img/posts/openclaw-hero.jpg)
_발행 자동화에서 진짜 차이를 만드는 건 성공률이 아니라 복구 속도다._

3편에서 Heartbeat와 Cron 역할을 분리했고, 4편에서 초안→검수→발행 파이프라인을 만들었고, 5편에서 운영 템플릿을 정리했다.  
이번 6편은 그 흐름의 핵심 보강이다.

> **목표: 발행 실패가 나도 10분 안에 복구하고, 같은 실수를 반복하지 않게 만드는 것**

실전에서는 “실패를 완전히 없애는 것”보다 “실패했을 때 빠르게 정상화하는 것”이 더 중요하다.  
특히 오전 09:00 발행 루틴은 작은 오타 하나, 원격 충돌 하나로 바로 지연되기 쉽다.  
그래서 이번 글에서는 실제로 바로 적용 가능한 **실패 대응 템플릿**을 다룬다.

---

## 1) 실패를 사건(Incident)으로 다루는 기준

발행 자동화에서 아래 조건 중 하나라도 해당하면 “사건”으로 본다.

- 09:00 기준 발행 미완료
- `git push` 실패로 배포 지연
- 대표 이미지/링크 오류로 게시물 품질 미달
- 잘못된 파일 커밋으로 롤백 필요

중요한 건 감정이 아니라 절차다.  
"망했다"에서 멈추지 말고, 바로 아래 4단계로 들어가면 된다.

1. **탐지(Detect)**: 무엇이 실패했는지 1줄로 정의  
2. **격리(Contain)**: 피해 확산 차단(추가 커밋/무분별한 수정 중지)  
3. **복구(Recover)**: 최소 조치로 정상 발행 복원  
4. **회고(Review)**: 재발 방지 규칙 1개 추가

---

## 2) 10분 복구 플레이북 (실전 명령어)

아래는 블로그 포스트 발행 실패 시 바로 쓰는 기본 플레이북이다.

```bash
cd ~/tofu-shin.github.io

# 0) 현재 상태 스냅샷
pwd
git status --short
git log --oneline -5

# 1) 원격 최신 반영 (충돌 원인 파악)
git fetch origin

# 2) 충돌 없는 경우: 리베이스 후 푸시
git pull --rebase origin main
git push origin main
```

### `non-fast-forward`가 계속 나는 경우

```bash
# 충돌 파일 확인
git status

# 충돌 해결 후
git add <conflict-resolved-file>
git rebase --continue

# 최종 푸시
git push origin main
```

### 잘못된 커밋을 올린 경우(빠른 복구)

```bash
# 최근 커밋 되돌리기(히스토리 보존)
git revert <bad_commit_sha>

git push origin main
```

여기서 핵심은 `reset --hard` 남발이 아니다.  
공유 브랜치(main)에서는 **revert 우선**이 복구 안정성이 높다.

---

## 3) 실전 예시: 08:59 충돌, 09:07 복구

상황:
- 08:59 `git push` 실행
- 원격에 선반영 커밋이 있어 push 거절
- 09:00 발행 실패

대응:
1. `git pull --rebase origin main`로 최신 반영  
2. 충돌 파일 1개 해결 후 `git rebase --continue`  
3. `git push origin main` 재실행  
4. 09:07 발행 완료, 지연 7분 기록

이 케이스에서 중요한 건 “왜 늦었는지”를 남기는 것이다.  
원인을 남기지 않으면 다음 주에 같은 실패가 다시 나온다.

---

## 4) 실패 유형별 대응 템플릿

### A. 본문 품질 미달 (분량/구성 부족)
증상:
- 1,800자 미만
- 예시/명령어/체크리스트/실패 대응 섹션 누락

즉시 대응:
- 누락 섹션부터 보강
- 품질 체크 통과 전 커밋 금지

검증 예시:

```bash
python3 - <<'PY'
from pathlib import Path
p = Path('_posts/2026-03-09-openclaw-practical-6-failure-response-template.md')
text = p.read_text(encoding='utf-8')
body = '---'.join(text.split('---')[2:]).strip()
print('body_chars=', len(body))
PY
```

### B. 대표 이미지 누락/경로 오류
증상:
- 상단 이미지 미노출
- 잘못된 경로로 404

즉시 대응:
- 본문 상단에 아래 라인 고정

```markdown
![OpenClaw](/assets/img/posts/openclaw-hero.jpg)
```

### C. 의도하지 않은 파일 커밋
증상:
- `_posts` 외 임시 파일까지 함께 커밋

즉시 대응:
- 필요한 파일만 골라 새 커밋
- 불필요 변경은 별도 정리

안전 습관:

```bash
git status --short
git add _posts/2026-03-09-openclaw-practical-6-failure-response-template.md
```

---

## 5) 발행 전/후 체크리스트 (복붙용)

### 발행 전 (Pre-flight)
- [ ] 실전편 번호/제목/날짜 일치
- [ ] 본문 1,800자 이상
- [ ] 대표 이미지 포함 (`/assets/img/posts/openclaw-hero.jpg`)
- [ ] 실전 예시/명령어/체크리스트/실패 대응 섹션 포함
- [ ] `git status`에서 의도한 변경만 존재

### 발행 후 (Post-flight)
- [ ] `git push origin main` 성공
- [ ] 원격 반영 확인
- [ ] 지연 여부 기록(정시/지연 분)
- [ ] 실패 원인/대응/재발 방지 1줄 로그 작성

이 체크리스트는 길지 않아야 한다.  
길고 복잡하면 결국 생략되고, 생략되면 사고가 난다.

---

## 6) 재발 방지 룰: 한 번 실패하면 규칙 하나 추가

실패 대응의 마지막 단계는 “반성”이 아니라 **규칙 추가**다.

예시:
- 충돌이 잦다 → 08:55에 `git fetch origin` 사전 점검 추가
- 이미지 누락이 났다 → 포스트 템플릿에 대표 이미지 라인 고정
- 분량 미달이 반복된다 → 초안 단계에서 최소 소제목 6개 강제

규칙은 크게 만들 필요 없다.  
작은 규칙 1~2개만 추가해도 다음 실패 확률이 눈에 띄게 내려간다.

---

## 마무리

자동화 운영에서 실력 차이는 "실패를 안 하는 사람"이 아니라, **실패에서 빨리 복구하고 시스템을 더 단단하게 만드는 사람**에게서 나온다.

이번 6편의 핵심은 세 가지다.

1. 실패를 사건으로 정의하고 절차로 대응한다.  
2. 10분 복구 플레이북을 미리 준비한다.  
3. 실패마다 재발 방지 규칙 1개를 추가한다.

이 흐름이 자리 잡으면 09:00 발행 루틴은 훨씬 안정적이 된다.  
다음 7편에서는 이 복구 템플릿을 기반으로, 주간 백로그 큐를 운영해 발행이 끊기지 않는 구조를 만든다.
