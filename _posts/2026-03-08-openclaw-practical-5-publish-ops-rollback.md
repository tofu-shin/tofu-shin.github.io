---
layout: post
title: "5. OpenClaw 실전편 - 발행 운영 표준안(검증·커밋·롤백)"
date: 2026-03-08 09:00:00 +0900
categories: [OpenClaw, AI Workflow]
tags: [openclaw, automation, blog, jekyll, git, rollback, checklist]
---

# OpenClaw 실전편 #5

![OpenClaw](/assets/img/posts/openclaw-hero.jpg)
_실패를 줄이는 발행 운영 루틴을 표준 절차로 정리했다._

3편에서 Heartbeat/Cron 역할 분리를 잡았고, 4편에서 초안→검수→배포 파이프라인을 만들었다.  
이번 5편은 그 파이프라인을 **실패에 강한 운영 절차**로 고정하는 단계다.

핵심 목표는 하나다.

> **"09:00 발행을 한 번 성공시키는 게 아니라, 매일 안정적으로 반복한다."**

실전에서 문제는 보통 본문 작성보다 마지막 구간에서 난다.

- front matter 누락
- 대표 이미지 경로 오타
- 의도하지 않은 파일 커밋
- 원격 충돌로 push 실패

그래서 이번 글은 아래 4가지를 한 번에 다룬다.

1. 발행 직전 검증 절차
2. 안전한 git 커밋/푸시 순서
3. 실패 시 표준 대응(재시도/복구)
4. 바로 복붙 가능한 체크리스트

---

## 1) 운영 원칙: 작성과 발행을 분리한다

블로그 자동화가 흔들리는 가장 큰 이유는 “글쓰기 작업”과 “운영 작업”이 뒤섞이기 때문이다.

- **작성 작업**: 메시지 품질, 구조, 사례, 문장 흐름
- **운영 작업**: 형식 검증, 커밋 단위, 충돌 처리, 결과 기록

권장 순서는 고정한다.

1. 본문 완성(분량/구성 충족)
2. 발행 검증(이미지/형식/명령어 확인)
3. 커밋/푸시(대상 파일 제한)
4. 결과 기록(성공/실패, 원인 1줄)

이렇게 분리해두면 실패 원인이 명확해지고, 복구 속도가 빨라진다.

---

## 2) 발행 직전 3분 검증 루틴

아래 루틴은 실제 발행 전 마지막 점검용이다.

```bash
cd ~/tofu-shin.github.io

# 1) 대상 파일 존재 확인
ls -lh _posts/2026-03-08-openclaw-practical-5-publish-ops-rollback.md

# 2) front matter 핵심 필드 확인
grep -n "^title:\|^date:\|^categories:\|^tags:" _posts/2026-03-08-openclaw-practical-5-publish-ops-rollback.md

# 3) 대표 이미지 경로 확인
grep -n "/assets/img/posts/openclaw-hero.jpg" _posts/2026-03-08-openclaw-practical-5-publish-ops-rollback.md

# 4) 본문 분량 점검(최소 1800자)
python3 - <<'PY'
from pathlib import Path
p = Path('_posts/2026-03-08-openclaw-practical-5-publish-ops-rollback.md')
t = p.read_text(encoding='utf-8')
parts = t.split('---')
body = '---'.join(parts[2:]) if len(parts) >= 3 else t
print('body_chars =', len(body.strip()))
PY
```

포인트는 “한 번에 완벽”이 아니라 “누락을 사전에 제거”하는 데 있다.

---

## 3) 배포 명령어: 작게 커밋하고 명확하게 푸시

안전한 운영의 기본 규칙은 단순하다.

> **`git add .` 대신 파일 경로 지정 add를 기본값으로 쓴다.**

```bash
cd ~/tofu-shin.github.io

# 변경 파일 확인
git status --short

# 이번 글 파일만 스테이징
git add _posts/2026-03-08-openclaw-practical-5-publish-ops-rollback.md

# 커밋
git commit -m "Revise OpenClaw practical #5 publish ops runbook"

# 푸시
git push origin main
```

이 방식이면 임시 파일, 메모, 테스트 산출물이 섞여 올라가는 사고를 크게 줄일 수 있다.

---

## 4) 실패 대응 표준안 (자주 터지는 케이스)

### 케이스 A) `nothing to commit`

원인:
- 파일 저장 누락
- 대상 경로 착각

대응:
1. `git status`로 변경사항 확인
2. 파일 저장 여부 확인
3. 파일 지정 add 후 재커밋

```bash
git status
git add _posts/2026-03-08-openclaw-practical-5-publish-ops-rollback.md
git commit -m "Revise OpenClaw practical #5 publish ops runbook"
```

### 케이스 B) `non-fast-forward` push 거절

원인:
- 원격 브랜치 선행 커밋 존재

대응:
1. rebase pull
2. 충돌 해결
3. push 재시도

```bash
git pull --rebase origin main
git push origin main
```

### 케이스 C) 예약 시간 지연(09:00 초과)

원인:
- 검증 단계 지연
- 충돌 처리 시간 초과
- 네트워크 이슈

대응:
1. 즉시 수동 발행
2. 지연 원인 1줄 기록
3. 다음 실행에서 동일 원인 차단 룰 추가

핵심은 “실패 제로”가 아니라 **동일 실패 재발 방지**다.

---

## 5) 발행 체크리스트 (실전용)

아래는 발행 전에 그대로 확인하면 되는 최소 체크리스트다.

- [ ] 시리즈 번호/제목 일치: OpenClaw 실전편 #5
- [ ] 본문 1,800자 이상
- [ ] 상단 대표 이미지 포함: `/assets/img/posts/openclaw-hero.jpg`
- [ ] 실전 예시/명령어/체크리스트/실패 대응 섹션 포함
- [ ] front matter(`title`, `date`, `categories`, `tags`) 정상
- [ ] 코드블록 명령어 복사 실행 가능
- [ ] `git status` 기준 의도한 파일만 커밋
- [ ] `git push origin main` 성공
- [ ] 발행 결과(성공/실패) 기록

체크리스트는 숙련자일수록 더 중요하다. 대부분의 장애는 “몰라서”가 아니라 “익숙해서 생략해서” 발생한다.

---

## 6) 운영 팁: 자동화는 단계적으로 올린다

처음부터 완전 자동 배포를 강행하기보다, 신뢰도를 쌓으면서 단계적으로 올리는 편이 안정적이다.

1. **반자동**: 작성 자동화 + 최종 푸시 수동 승인
2. **조건부 자동**: 체크리스트 통과 시 자동 푸시
3. **완전 자동**: 실패 알림/재시도/복구 루틴 포함

이 순서를 지키면 자동화 품질이 올라가면서도 사고 비용을 통제할 수 있다.

---

## 마무리

실전 자동화의 핵심은 화려한 문장이 아니라 **운영 표준화**다.

- 검증 없이 커밋하지 않고
- 커밋 범위를 좁히고
- 실패를 표준 절차로 처리하고
- 재발 방지 기록을 남긴다

이 4가지를 루틴으로 고정하면, 블로그 발행은 “운”이 아니라 “시스템”이 된다.
다음 편에서는 이 운영 표준안을 바탕으로 주간 백로그 큐를 자동 소진하는 운영 방식까지 확장해보겠다.