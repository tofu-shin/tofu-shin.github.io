---
layout: post
title: "OpenClaw 실전편 #3 - 하트비트와 크론으로 자동화 루틴 만들기"
date: 2026-03-05 09:00:00 +0900
categories: [OpenClaw, Automation]
tags: [openclaw, heartbeat, cron, automation, ai-agent]
---

이전 글에서 OpenClaw 기본 사용법을 다뤘다면,
이번엔 **매일 알아서 일하게 만드는 루틴**을 정리해본다.

핵심은 두 가지다.

- **Heartbeat**: 주기적 상태 확인(느슨한 주기, 문맥 기반)
- **Cron**: 정확한 시간 실행(엄격한 스케줄)

## 1) Heartbeat
가벼운 점검 작업에 적합하다.
- 이메일/멘션 확인
- 캘린더 24~48시간 점검
- 필요할 때만 알림

## 2) Cron
정시 실행 작업에 적합하다.
- 매일 09:00 포스팅 발행
- 주간 요약 자동 생성
- 야간 점검 로그 기록

## 3) 추천 운영 패턴
1. Heartbeat로 재료 수집
2. Cron으로 08:50 점검, 09:00 발행
3. 실패 시 재시도 + 즉시 알림

## 4) 마무리
Heartbeat는 상황 인지, Cron은 정시 실행이다.  
둘을 함께 쓰면 자동화가 안정적으로 굴러간다.
