---
title: "OpenClaw 시작하기 - AI 비서를 터미널에서"
date: 2026-02-12 09:26:00 +0900
categories: [Development, AI]
tags: [openclaw, ai, automation, claude, tutorial]
---

# OpenClaw란?

OpenClaw는 Anthropic의 Claude AI를 로컬 환경에서 사용할 수 있게 해주는 오픈소스 프로젝트입니다. 터미널, 텔레그램, Discord 등 다양한 채널을 통해 AI 비서와 대화하고 작업을 자동화할 수 있습니다.

## 왜 OpenClaw인가?

- **항상 켜져있는 AI 비서**: 맥 미니 같은 서버에서 24/7 실행
- **다양한 채널 지원**: Telegram, Discord, 터미널 등
- **강력한 자동화**: 파일 관리, 시스템 모니터링, 크론잡 설정
- **확장 가능**: MCP (Model Context Protocol) 서버 연동

## 설치

### 1. Homebrew로 설치 (macOS/Linux)

```bash
brew install openclaw/tap/openclaw
```

### 2. 초기 설정

```bash
openclaw wizard
```

대화형 설정 마법사가 실행됩니다:
- Anthropic API 키 입력
- 기본 모델 선택 (claude-sonnet-4 추천)
- 워크스페이스 위치 설정

### 3. Gateway 시작

```bash
openclaw gateway start
```

## 기본 사용법

### 터미널에서 대화

```bash
claude "안녕? 파일 목록 보여줘"
```

### 채널 연동

#### Telegram 연동

1. BotFather로 봇 생성
2. API 토큰 받기
3. 설정에 추가:

```bash
openclaw configure --channel telegram
```

#### Discord 연동

```bash
openclaw configure --channel discord
```

## 실전 활용 예시

### 1. 파일 정리 자동화

```
너: "ComfyUI 폴더 18GB나 차지하는데 정리해줘"
AI: [분석 후 불필요한 모델 18GB 삭제]
```

### 2. 크론잡 설정

```
너: "10분 뒤에 작업 완료됐는지 알려줘"
AI: [크론잡 자동 설정 후 시간 되면 알림]
```

### 3. GitHub 블로그 자동 배포

```
너: "GitHub Pages 블로그 만들어줘"
AI: [Jekyll 설치, 테마 적용, 첫 포스트 작성, 푸시까지 자동화]
```

## 강력한 기능들

### MCP 서버 연동

ComfyUI, 파일 시스템 등 외부 툴과 연동 가능:

```bash
# ComfyUI MCP 서버 실행
cd ~/mcp-server/comfyui-mcp-server
python server.py
```

### 세션 관리

```bash
# 세션 목록
openclaw sessions list

# 특정 세션에 메시지 보내기
openclaw sessions send --session main "작업 확인해줘"
```

### 브라우저 자동화

```
너: "CivitAI에서 픽셀아트 워크플로우 찾아줘"
AI: [브라우저 열고 검색 후 결과 정리]
```

## 설정 파일 위치

```
~/.openclaw/openclaw.json    # 메인 설정
~/.openclaw/workspace/        # 작업 공간
~/.openclaw/workspace/memory/ # 메모리 파일
```

## 유용한 팁

### 1. 메모리 관리

AI는 세션마다 초기화됩니다. 중요한 정보는 메모리 파일에 저장:

```
~/.openclaw/workspace/memory/YYYY-MM-DD.md
```

### 2. 워크플로우 자동화

`AGENTS.md`, `TOOLS.md` 등으로 AI의 행동 커스터마이징 가능

### 3. 비밀번호 관리

절대 비밀번호를 AI에게 알려주지 마세요! 
- 대화 로그에 평문으로 남음
- SSH 키 인증 등 대안 사용 권장

## 문제 해결

### Gateway 재시작

```bash
openclaw gateway restart
```

### 로그 확인

```bash
openclaw logs --follow
```

### 상태 확인

```bash
openclaw status
```

## 마무리

OpenClaw는 단순한 챗봇이 아니라, 진짜 일을 해주는 AI 비서입니다. 파일 정리부터 블로그 배포까지, 귀찮은 작업들을 맡기고 더 중요한 일에 집중하세요.

## 참고 링크

- [OpenClaw 공식 문서](https://docs.openclaw.ai)
- [GitHub Repository](https://github.com/openclaw/openclaw)
- [Discord 커뮤니티](https://discord.com/invite/clawd)

---

*이 블로그도 OpenClaw의 도움으로 만들어졌습니다. 😊*
