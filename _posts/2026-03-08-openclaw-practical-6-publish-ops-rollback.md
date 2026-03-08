---
layout: post
title: "6. OpenClaw 실전편 - 발행 실패를 막는 운영 템플릿(검증·커밋·롤백)"
date: 2026-03-08 09:00:00 +0900
categories: [AI, OpenClaw]
tags: [openclaw, automation, blog, jekyll, git, rollback, checklist]
---

# OpenClaw 실전편 #6

![OpenClaw](/assets/img/posts/openclaw-hero.jpg)
_실패를 전제로 설계한 블로그 발행 운영 템플릿_

3편에서 Heartbeat + Cron의 역할 분리를 잡았고, 4편에서 초안→검수→배포 파이프라인을 만들었다.  
이번 6편은 그 다음 단계다.

> **"오늘도 09:00 발행이 돌아가게 만드는 운영 습관"**

자동화는 “한 번 성공”보다 “계속 성공”이 훨씬 어렵다.  
실전에서 진짜 많이 터지는 건 글 작성이 아니라 **마지막 10% (검증/커밋/푸시/복구)** 구간이다.

그래서 이번 글은 아래 4가지를 한 번에 묶어 다룬다.

- 발행 직전 검증 루틴
- 안전한 git 커밋/푸시 순서
- 충돌/실패 대응 표준 절차
- 내일부터 바로 쓸 수 있는 체크리스트

---

## 1) 운영 원칙: "글쓰기"와 "발행"을 분리한다

발행 실패를 줄이려면 먼저 관점을 바꿔야 한다.

- 글쓰기 = 콘텐츠 작업
- 발행 = 운영 작업

이 둘을 섞으면, 본문 수정하다가 배포 이슈를 놓치거나, 반대로 배포에 쫓겨서 품질이 무너진다.  
추천 흐름은 아래처럼 고정하면 된다.

1. 본문 완성 (분량/구성 충족)
2. 발행 검증 (형식/이미지/명령어 점검)
3. 커밋/푸시 (의도한 파일만)
4. 결과 기록 (성공/실패 로그)

핵심은 단순하다.  
**검증을 통과하지 못하면 커밋하지 않는다.**

---

## 2) 발행 직전 검증: 3분 점검으로 사고를 줄인다

아래 명령어를 발행 직전에 순서대로 실행한다.

```bash
cd ~/tofu-shin.github.io

# 1) 오늘 포스트 파일 확인
ls -lh _posts/2026-03-08-openclaw-practical-6-publish-ops-rollback.md

# 2) front matter / 대표 이미지 경로 확인
grep -n "^title:\|^date:\|^categories:\|^tags:" _posts/2026-03-08-openclaw-practical-6-publish-ops-rollback.md
grep -n "/assets/img/posts/openclaw-hero.jpg" _posts/2026-03-08-openclaw-practical-6-publish-ops-rollback.md

# 3) 최소 분량(예: 1800자) 확인
python3 - <<'PY'
from pathlib import Path
p = Path('_posts/2026-03-08-openclaw-practical-6-publish-ops-rollback.md')
text = p.read_text(encoding='utf-8')
# front matter 제외한 본문 길이 대략 계산
parts = text.split('---')
body = '---'.join(parts[2:]) if len(parts) >= 3 else text
print('body_chars=', len(body.strip()))
PY
```

여기서 분량만 채우고 끝내면 안 된다.  
실전에서는 아래 항목이 더 자주 문제를 만든다.

- 대표 이미지 경로 오타
- `date` 시간대 누락 (`+0900` 빠짐)
- 제목 번호(실전편 #N) 불일치
- 코드블록 안 명령어 오탈자

---

## 3) 안전한 배포 명령어: "작게, 명확하게" 커밋

가장 중요한 규칙 하나:

> **`git add .` 대신 파일 지정 add를 기본값으로 쓴다.**

```bash
cd ~/tofu-shin.github.io

# 변경점 확인
git status --short

# 이번 글 파일만 스테이징
git add _posts/2026-03-08-openclaw-practical-6-publish-ops-rollback.md

# 커밋
git commit -m "Add OpenClaw practical #6 publish ops and rollback guide"

# 푸시
git push origin main
```

이렇게 하면 의도하지 않은 파일(임시 메모, 테스트 파일, 로컬 설정)이 같이 올라가는 사고를 거의 막을 수 있다.

---

## 4) 실패 대응 표준 절차 (자주 터지는 3가지)

### 케이스 A: nothing to commit

증상:
- `git commit` 했는데 변경사항이 없다고 나옴

대응:
1. 파일이 실제 저장됐는지 확인
2. 경로가 맞는지 확인
3. 다시 add 후 commit

```bash
git status
git add _posts/2026-03-08-openclaw-practical-6-publish-ops-rollback.md
git commit -m "Add OpenClaw practical #6 publish ops and rollback guide"
```

### 케이스 B: non-fast-forward (원격 충돌)

증상:
- push 거절

대응:
1. 원격 변경을 rebase로 당겨오기
2. 충돌 해결
3. 다시 push

```bash
git pull --rebase origin main
git push origin main
```

### 케이스 C: 발행 지연 (09:00 넘김)

증상:
- 예약 시각이 지났는데 글이 아직 미발행

대응:
1. 즉시 수동 발행
2. 지연 원인 기록(충돌/네트워크/검증 누락)
3. 다음 실행에 같은 실패 재발 방지 룰 추가

중요한 건 완벽한 무오류가 아니다.  
**같은 실패를 두 번 반복하지 않는 운영 기록**이 핵심이다.

---

## 5) 그대로 복붙해서 쓰는 발행 체크리스트

아래 체크리스트는 실제로 매번 확인하는 용도로 만든 템플릿이다.

- [ ] 시리즈 번호/제목이 맞다 (OpenClaw 실전편 #6)
- [ ] 본문 1,800자 이상 충족
- [ ] 상단 대표 이미지 포함: `/assets/img/posts/openclaw-hero.jpg`
- [ ] 실전 예시/명령어/체크리스트/실패 대응 섹션 포함
- [ ] front matter (`title`, `date`, `categories`, `tags`) 정상
- [ ] 명령어 블록 복사 실행 가능한 형태
- [ ] `git status` 기준 의도한 파일만 커밋
- [ ] `git push origin main` 성공 확인
- [ ] 발행 결과(성공/실패 + 원인) 한 줄 기록

체크리스트는 "잘 아는 내용"이어도 반드시 눈으로 확인하는 게 좋다.  
실수는 지식 부족보다 **루틴 생략**에서 더 많이 나온다.

---

## 6) 운영 팁: 자동화는 "신뢰"가 쌓여야 확장된다

처음부터 모든 걸 완전 자동으로 돌리면 편해 보이지만, 초기에는 오히려 불안정해지기 쉽다.  
추천 단계는 아래 순서다.

1. **반자동 운영**: 작성 자동화 + 최종 배포 수동 승인
2. **조건부 자동**: 체크리스트 통과 시 자동 push
3. **완전 자동**: 실패 알림/재시도/롤백까지 포함

이 순서대로 올리면, 루틴이 깨졌을 때 어디를 고쳐야 하는지 바로 보인다.

---

## 마무리

실전 자동화의 핵심은 멋진 프롬프트가 아니라, **실패를 다루는 운영 템플릿**이다.

- 검증 없이 커밋하지 않고
- 작은 단위로 안전하게 푸시하고
- 실패하면 표준 절차로 복구하고
- 재발 방지 기록을 남긴다

이 4가지만 지켜도 아침 발행 루틴의 안정성이 확 달라진다.  
다음 편에서는 이 운영 템플릿을 기반으로, 주간 글감 큐를 자동으로 소진하는 방식(우선순위 + 백로그 관리)까지 이어서 다뤄보겠다.
