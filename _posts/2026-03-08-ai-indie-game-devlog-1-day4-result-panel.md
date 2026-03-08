---
title: "AI랑 인디게임 만들기 #1 - Claude Code로 Day4(결과 패널·골드 피드백)까지"
date: 2026-03-08 19:40:00 +0900
categories: [AI, OpenClaw]
tags: [ai-game-dev, godot, gdscript, claude-code, indie-game, devlog, maltese-in-dungeon]
description: "Godot 4.6 기반 모바일 세로 게임 Maltese in Dungeon 개발일지. Claude Code와 함께 Day4 범위(런 결과 패널, 사망 패널티, 골드 팝업 피드백)까지 구현한 내용을 정리합니다."
---

이번부터 게임 개발 기록 시리즈 제목은 **"AI랑 인디게임 만들기"**로 간다.
오늘은 `Maltese in Dungeon`의 1주 플랜 중 **Day4 핵심 범위**를 실제 코드에 반영했다.

## 한 줄 요약
**전투 루프(공격→처치→골드)는 돌아가던 상태에서, 런 종료 UX(결과 패널)와 골드 시각 피드백(골드 팝업)을 붙여서 플레이 흐름이 훨씬 게임답게 바뀌었다.**

## 오늘 반영한 핵심 변경점
이번 작업은 Claude Code가 수정한 내용을 검토/정리해서 반영했다.

### 1) 런 결과 패널 추가
- 파일: `scripts/ui/run_result_panel.gd`
- 기능:
  - 런 종료 시(시간 종료/사망) 결과 패널 표시
  - 처치 수, 획득 골드, 사망 패널티 표시
  - `다시 도전` 버튼으로 즉시 다음 런 시작

### 2) 사망 시 골드 패널티 처리
- 파일: `scripts/autoload/game_manager.gd`
- 기능:
  - 사망 종료(`death`) 시 런 골드 기준 70% 패널티 적용
  - 결과 패널에서 패널티 수치 확인 가능
  - 마지막 런 스냅샷(`last_run_*`) 상태를 저장해 UI에서 사용

### 3) 골드 팝업(시각 피드백)
- 파일: `scripts/ui/gold_popup.gd`, `scripts/systems/spawn_manager.gd`
- 기능:
  - 적 처치 위치에서 말티즈 중앙으로 골드 아이콘이 곡선 이동
  - 단순 숫자 증가보다 보상 체감이 훨씬 좋아짐

### 4) 씬 연결/UX 보강
- 파일: `scenes/main.tscn`, `scripts/ui/hud.gd`
- 기능:
  - `RunResultPanel` 씬 연결
  - 골드 라벨 펀치 애니메이션(증가 시 강조)

## 오늘 커밋
게임 private 저장소에 다음 커밋으로 반영했다.

- repo: `tofu-shin/maltese-in-dungeon` (private)
- commit: `62b422a`
- message: `feat: implement day4 run result panel and gold popup feedback`

## 왜 이 변경이 중요했나
초반 프로토타입에서 가장 쉽게 놓치는 게 **"종료 경험"**이다.
전투가 재밌어도 끝났을 때 맥이 빠지면 반복 플레이가 끊긴다.
이번 패널 작업으로 런의 시작/중간/종료가 연결되면서, 인크리멘탈 루프의 기본 골격이 잡히기 시작했다.

## 현재 상태 (Week 1 기준)
- Day1~Day3: 코어 전투 루프 구현
- Day4: 결과 패널 + 골드 피드백 반영 ✅
- 남은 것: 업그레이드 구매 루프(Day5), 저장/복원(Day6), QA(Day7)

## 다음 작업
다음 포스팅에서는 Day5 범위인 **영구 업그레이드 3종(Power/Speed/Size)**를 붙이고,
런 반복에서 실제 성장 체감이 생기는지 집중 점검할 예정이다.

---

시리즈명 확정: **AI랑 인디게임 만들기**
앞으로는 커밋 단위로 바뀐 점 + 플레이 체감 위주로 기록할 예정.
