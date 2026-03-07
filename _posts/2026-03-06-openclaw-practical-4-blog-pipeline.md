---
layout: post
title: "4. OpenClaw 실전편 - 블로그 발행 파이프라인(초안→검수→배포) 실전 템플릿"
date: 2026-03-06 09:00:00 +0900
categories: [AI, OpenClaw]
tags: [openclaw, automation, blog, jekyll, cron, workflow]
---

# OpenClaw 실전편 #4

![블로그 파이프라인 미니멀 커버](/assets/img/posts/blog-pipeline-minimal.svg)
_초안→검수→배포 흐름을 담은 미니멀 커버_

3편에서 Heartbeat + Cron의 역할 분리를 다뤘다면, 이번에는 그걸 **실제 블로그 발행 루틴**에 꽂아보자.  
목표는 하나다.

> **“아침 9시에 글 하나가 안정적으로 올라가고, 실패하면 바로 복구 가능하게 만들기.”**

핵심은 거창한 도구보다 **절차를 명확하게 고정**하는 데 있다.  
초안 작성, 검수, 배포, 실패 대응을 분리해두면 자동화가 오래 버틴다.

---

## 1) 전체 흐름: 초안 → 검수 → 배포

아래 3단계로 나누면 디버깅이 훨씬 쉬워진다.

1. **초안 단계 (Draft)**
   - 주제/목차/핵심 메시지 작성
   - 본문 최소 분량 채우기(예: 1,800자 이상)
2. **검수 단계 (Review)**
   - front matter, 이미지, 링크, 문장 흐름 점검
   - 체크리스트 기반 확인
3. **배포 단계 (Publish)**
   - 파일 저장
   - `git add/commit/push`
   - 푸시 결과/배포 상태 기록

포인트는 단순하다.  
**한 단계가 실패하면 다음 단계로 넘어가지 않게** 만드는 것.

---

## 2) 글 파일 템플릿(재사용용)

`_posts/YYYY-MM-DD-slug.md` 파일은 아래 형식을 고정해두면 편하다.

```markdown
---
layout: post
title: "제목"
date: 2026-03-06 09:00:00 +0900
categories: [AI, OpenClaw]
tags: [openclaw, automation]
---

# 제목

![OpenClaw](/assets/img/posts/openclaw-hero.jpg)
_대표 이미지 캡션_

도입부...
```

대표 이미지를 고정 경로(`/assets/img/posts/openclaw-hero.jpg`)로 두면  
매번 이미지 누락으로 깨지는 문제를 줄일 수 있다.

---

## 3) 실전 명령어: 발행 루틴 기본형

아래는 수동/반자동 모두에서 공통으로 쓰는 최소 명령 세트다.

```bash
cd ~/blog

# 1) 변경 사항 확인
git status

# 2) 포스트 파일 확인(예: 오늘 글)
ls -lh _posts/2026-03-06-openclaw-practical-4-blog-pipeline.md

# 3) 스테이징
git add _posts/2026-03-06-openclaw-practical-4-blog-pipeline.md

# 4) 커밋
git commit -m "Add OpenClaw practical #4 blog pipeline guide"

# 5) 푸시
git push origin main
```

여기서 중요한 건 `git add .`를 습관적으로 쓰지 않는 것.  
블로그와 무관한 파일이 같이 올라가는 사고를 꽤 자주 만든다.

---

## 4) 발행 전 체크리스트 (복붙해서 써도 됨)

아래 체크리스트만 지켜도 실패율이 크게 내려간다.

- [ ] 제목 번호/시리즈 순서가 맞다 (예: 실전편 #4)
- [ ] 본문 분량이 기준 이상이다 (최소 1,800자)
- [ ] 대표 이미지가 본문 상단에 있다
- [ ] front matter의 `date`, `categories`, `tags`가 정상이다
- [ ] 코드블록/명령어가 실제 실행 가능한 형태다
- [ ] 실전 예시 + 체크리스트 + 실패 대응이 본문에 포함돼 있다
- [ ] 오탈자, 링크 깨짐, 문단 끊김이 없다
- [ ] `git status` 기준 의도한 파일만 커밋된다

체크리스트를 파일로 따로 두고(예: `docs/blog-publish-checklist.md`)  
매번 그대로 재사용하면 컨디션에 덜 흔들린다.

---

## 5) 실패 대응 시나리오 (진짜 중요한 부분)

자동화는 성공 루틴보다 **실패 시 복구 설계**가 더 중요하다.  
실전에서 자주 터지는 케이스만 먼저 막아도 체감 안정성이 확 올라간다.

### 케이스 A) 커밋 실패

증상:
- `nothing to commit`
- 또는 훅/포맷 검증 실패

대응:
1. `git status`로 실제 변경 파일 확인
2. 대상 파일이 맞게 저장됐는지 확인
3. 필요하면 파일만 다시 `git add` 후 재커밋

```bash
git status
git add _posts/2026-03-06-openclaw-practical-4-blog-pipeline.md
git commit -m "Add OpenClaw practical #4 blog pipeline guide"
```

### 케이스 B) 푸시 충돌(non-fast-forward)

증상:
- 원격 변경 때문에 push 거절

대응:
1. `git pull --rebase origin main`
2. 충돌 해결
3. `git push origin main`

```bash
git pull --rebase origin main
git push origin main
```

### 케이스 C) 예약 시간 지나서 발행 지연

증상:
- 09:00 작업이 밀림

대응:
1. “지연 발행” 상태를 로그/메시지로 남김
2. 즉시 수동 발행
3. 다음 실행 전 원인(네트워크/충돌/누락) 회고

핵심은 실패 자체보다 **조용히 실패하지 않게** 만드는 것.  
성공/실패 결과를 반드시 남겨야 다음 개선이 가능하다.

---

## 6) 추천 운영 방식: 최소 자동화에서 시작

처음부터 완전 자동 발행으로 가지 말고 아래 순서가 안전하다.

1. **1단계: 반자동**
   - 글 작성은 자동/반자동
   - 최종 커밋/푸시는 사람이 승인
2. **2단계: 조건부 자동**
   - 체크리스트 통과 시에만 자동 push
3. **3단계: 완전 자동**
   - 실패 알림 + 재시도 + 롤백 전략 포함

이 순서로 가면 “편해졌는데 불안한 자동화”가 아니라  
**편하고 믿을 수 있는 자동화**에 가까워진다.

---

## 마무리

OpenClaw 실전 자동화에서 제일 중요한 건 화려한 프롬프트가 아니다.  
**반복 가능한 절차 + 실패 시 즉시 복구 가능한 구조**다.

- 초안은 빠르게 만들고
- 검수는 체크리스트로 고정하고
- 배포는 작은 단위로 안전하게 밀어 올리고
- 실패하면 즉시 복구한다

이 4가지만 지키면, 아침 발행 루틴은 생각보다 금방 안정화된다.  
다음 편에서는 이 발행 파이프라인을 기준으로, 주간 단위로 글감을 누적하고 자동으로 큐잉하는 방식까지 이어서 다뤄보겠다.
