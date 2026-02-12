---
title: "2. OpenClaw 설치하기 - 처음부터 끝까지"
date: 2026-02-12 10:35:00 +0900
categories: [AI, OpenClaw]
tags: [openclaw, installation, setup, tutorial]
---

# OpenClaw 설치 완벽 가이드

![OpenClaw GitHub](/assets/img/posts/openclaw-github.jpg)
_OpenClaw GitHub 저장소 (188k ⭐)_

이 글에서는 OpenClaw를 처음 설치하는 과정을 단계별로 설명합니다. 복사-붙여넣기만 해도 따라할 수 있도록 모든 명령어와 설정을 포함했습니다.

## 시작하기 전에

### 지원 환경

**운영체제:**
- ✅ macOS (Intel/Apple Silicon)
- ✅ Linux (Ubuntu, Debian, Fedora 등)
- ❌ Windows (WSL2에서 가능하지만 비공식)

**요구사항:**
- Node.js 18 이상 (자동 설치됨)
- 인터넷 연결 (API 사용 시)
- 터미널 기본 지식

**선택사항:**
- 24시간 켜둘 서버 (맥 미니, 라즈베리파이 등)
- AI API 키 (Claude, OpenAI 등) 또는 Ollama

### 필요한 것 준비

**1. AI 모델 선택 (둘 중 하나 또는 둘 다)**

**옵션 A: 클라우드 API (추천 - 시작하기 쉬움)**
- **Claude (Anthropic)**: https://console.anthropic.com
  - 무료 크레딧 제공
  - 성능 좋음
  - API 키 발급 5분
  
- **OpenAI (GPT-4)**: https://platform.openai.com
  - 유료 (API 사용량 과금)
  - 널리 사용됨

**옵션 B: 로컬 모델 (무료, 오프라인)**
- **Ollama**: https://ollama.com
  - 완전 무료
  - 인터넷 불필요
  - GPU/RAM 필요 (최소 8GB RAM)

**이 가이드에서는 Claude API + Ollama fallback 구성을 설명합니다.**

---

## Step 1: Homebrew 설치

Homebrew는 macOS/Linux 패키지 관리자입니다. OpenClaw 설치에 필요합니다.

**이미 설치되어 있는지 확인:**
```bash
brew --version
```

**없으면 설치:**
```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

**Linux 사용자:** 설치 후 PATH 추가 필요 (스크립트가 안내해줌)
```bash
echo 'eval "$(/home/linuxbrew/.linuxbrew/bin/brew shellenv)"' >> ~/.profile
eval "$(/home/linuxbrew/.linuxbrew/bin/brew shellenv)"
```

---

## Step 2: OpenClaw 설치

**공식 저장소 추가 후 설치:**
```bash
brew tap openclaw/tap
brew install openclaw
```

**설치 확인:**
```bash
openclaw --version
```

출력 예시:
```
openclaw/2026.2.9
```

**만약 "command not found" 에러가 나면:**
```bash
# PATH 확인
echo $PATH | grep homebrew

# 없으면 추가 (macOS)
echo 'export PATH="/opt/homebrew/bin:$PATH"' >> ~/.zshrc
source ~/.zshrc

# 없으면 추가 (Linux)
echo 'export PATH="/home/linuxbrew/.linuxbrew/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
```

---

## Step 3: Anthropic API 키 발급 (Claude 사용 시)

**1. Anthropic Console 접속**
- https://console.anthropic.com 방문
- GitHub 또는 Google 계정으로 로그인

**2. API Keys 메뉴 이동**
- 좌측 메뉴에서 "API Keys" 클릭
- "Create Key" 버튼 클릭

**3. 키 생성**
- Name: `OpenClaw` (아무거나)
- Create 클릭
- **즉시 복사!** (다시 볼 수 없음)

**형식 예시:**
```
sk-ant-api03-xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

**4. 안전하게 보관**
```bash
# 임시로 터미널에 변수로 저장 (이 세션에서만 유효)
export ANTHROPIC_API_KEY="sk-ant-api03-여기에-당신의-키"

# 또는 메모장에 복사해두기
```

**⚠️ 주의:**
- API 키는 비밀번호처럼 취급
- GitHub/공개 저장소에 절대 올리지 말 것
- `.gitignore`에 설정 파일 추가 필수

---

## Step 4: Ollama 설치 (로컬 모델 사용 시)

**Ollama 설치:**
```bash
brew install ollama
```

**Ollama 서비스 시작:**
```bash
# macOS (백그라운드 실행)
brew services start ollama

# Linux (수동 실행)
ollama serve &
```

**모델 다운로드:**
```bash
# 추천: Qwen 2.5 Coder (14B - 코딩 특화)
ollama pull qwen2.5-coder:14b

# 또는: Llama 3.2 (3B - 가벼움)
ollama pull llama3.2:3b

# 또는: Mistral (7B - 범용)
ollama pull mistral:7b
```

**다운로드 확인:**
```bash
ollama list
```

**모델 크기 참고:**
- 3B 모델: ~2GB, 최소 8GB RAM
- 7B 모델: ~4GB, 최소 16GB RAM
- 14B 모델: ~8GB, 최소 16GB RAM (권장 32GB)

---

## Step 5: OpenClaw 초기 설정

**설정 마법사 실행:**
```bash
openclaw wizard
```

**대화형 설정이 시작됩니다. 아래 가이드를 따라하세요:**

### 5-1. API 키 입력

```
? Enter your Anthropic API key: 
```
**→ 위에서 복사한 Claude API 키 붙여넣기**

```
? Would you like to configure additional providers? (y/N)
```
**→ `N` (나중에 추가 가능)**

### 5-2. 기본 모델 선택

```
? Select your primary model:
  ❯ anthropic/claude-sonnet-4-5
    anthropic/claude-opus-4
    ollama/qwen2.5-coder:14b
    ollama/llama3.2:3b
```
**→ `claude-sonnet-4-5` 선택 (화살표 키 + Enter)**

### 5-3. Fallback 모델 (선택)

```
? Add a fallback model? (Y/n)
```
**→ `Y`**

```
? Select fallback model:
  ❯ ollama/qwen2.5-coder:14b
    ollama/llama3.2:3b
    Skip
```
**→ Ollama 모델 선택 (인터넷 끊겨도 작동)**

### 5-4. 워크스페이스 설정

```
? Workspace directory: (~/.openclaw/workspace)
```
**→ Enter (기본값 사용)**

### 5-5. 채널 설정 (나중에 가능)

```
? Configure messaging channels now? (y/N)
```
**→ `N` (다음 글에서 설명)**

### 5-6. 완료

```
✓ Configuration saved to ~/.openclaw/openclaw.json
✓ Workspace created at ~/.openclaw/workspace

Next steps:
  1. Start the gateway: openclaw gateway start
  2. Test it: claude "hello"
```

---

## Step 6: Gateway 시작

OpenClaw의 핵심 서버를 시작합니다.

**시작:**
```bash
openclaw gateway start
```

**출력 예시:**
```
🦞 OpenClaw Gateway starting...
✓ Loaded config from ~/.openclaw/openclaw.json
✓ Workspace: ~/.openclaw/workspace
✓ Primary model: claude-sonnet-4-5
✓ Server listening on http://127.0.0.1:18789
✓ Gateway ready!
```

**확인:**
```bash
openclaw status
```

**출력 예시:**
```
Gateway: ✓ Running (PID: 12345)
Port: 18789
Uptime: 5 seconds
Model: claude-sonnet-4-5
```

**백그라운드 실행 (재부팅 시 자동 시작):**
```bash
# macOS
brew services start openclaw

# Linux (systemd)
sudo systemctl enable openclaw
sudo systemctl start openclaw
```

---

## Step 7: 첫 명령 실행

**터미널에서 대화:**
```bash
claude "안녕? 자기소개 해줘"
```

**정상 작동 시 출력:**
```
안녕하세요! 저는 OpenClaw를 통해 실행되는 AI 비서입니다.

현재 상태:
- 모델: Claude Sonnet 4.5
- 워크스페이스: ~/.openclaw/workspace
- 실행 환경: macOS

무엇을 도와드릴까요?
```

**파일 작업 테스트:**
```bash
claude "현재 디렉토리 파일 목록 보여줘"
```

**출력 예시:**
```
$ ls -la

total 16
drwxr-xr-x   5 user  staff   160 Feb 12 10:30 .
drwxr-xr-x  20 user  staff   640 Feb 12 10:25 ..
-rw-r--r--   1 user  staff  1234 Feb 12 10:28 file1.txt
-rw-r--r--   1 user  staff  5678 Feb 12 10:29 file2.md

총 4개 파일입니다.
```

**시스템 정보 확인:**
```bash
claude "현재 시스템 사용량 알려줘"
```

---

## Step 8: 설정 파일 확인

OpenClaw 설정은 JSON 파일로 저장됩니다.

**위치:**
```bash
~/.openclaw/openclaw.json
```

**확인:**
```bash
cat ~/.openclaw/openclaw.json
```

**주요 섹션:**

```json
{
  "auth": {
    "profiles": {
      "anthropic:claude-code": {
        "provider": "anthropic",
        "mode": "token"
      }
    }
  },
  "models": {
    "providers": {
      "anthropic": {
        "baseUrl": "https://api.anthropic.com",
        "models": []
      },
      "ollama": {
        "baseUrl": "http://127.0.0.1:11434",
        "models": []
      }
    }
  },
  "agents": {
    "defaults": {
      "model": {
        "primary": "anthropic/claude-sonnet-4-5",
        "fallbacks": ["ollama/qwen2.5-coder:14b"]
      },
      "workspace": "/Users/you/.openclaw/workspace"
    }
  },
  "gateway": {
    "port": 18789,
    "bind": "loopback"
  }
}
```

**수동 편집:**
```bash
# 안전하게 백업
cp ~/.openclaw/openclaw.json ~/.openclaw/openclaw.json.backup

# 에디터로 열기
nano ~/.openclaw/openclaw.json
# 또는
code ~/.openclaw/openclaw.json
```

**변경 후 재시작:**
```bash
openclaw gateway restart
```

---

## 트러블슈팅

### 1. "command not found: openclaw"

**원인:** PATH에 Homebrew가 없음

**해결:**
```bash
# Homebrew 위치 확인
which brew

# 없으면 PATH 추가
echo 'export PATH="/opt/homebrew/bin:$PATH"' >> ~/.zshrc
source ~/.zshrc
```

### 2. "API key not found"

**원인:** API 키가 설정되지 않음

**해결:**
```bash
# 설정 파일 확인
openclaw config get

# API 키 재설정
openclaw configure --provider anthropic
```

### 3. "Port 18789 already in use"

**원인:** Gateway가 이미 실행 중이거나 포트 충돌

**해결:**
```bash
# 실행 중인 Gateway 종료
openclaw gateway stop

# 포트 사용 확인
lsof -i :18789

# 다른 포트 사용 (config 파일 수정)
# gateway.port를 18790 등으로 변경
```

### 4. Ollama 연결 실패

**원인:** Ollama 서비스가 안 켜져 있음

**해결:**
```bash
# Ollama 실행 확인
ps aux | grep ollama

# 없으면 시작
ollama serve &

# 또는
brew services start ollama

# 연결 테스트
curl http://localhost:11434/api/tags
```

### 5. "Model not found"

**원인:** Ollama 모델이 다운로드 안 됨

**해결:**
```bash
# 모델 목록 확인
ollama list

# 없으면 다운로드
ollama pull qwen2.5-coder:14b
```

### 6. 느린 응답 속도

**원인:** 
- 인터넷 느림 (API 사용 시)
- Ollama 모델이 너무 큼 (로컬 시)

**해결:**
```bash
# 더 작은 모델 사용
ollama pull llama3.2:3b

# config에서 primary 모델 변경
# "primary": "ollama/llama3.2:3b"
```

---

## 다음 단계

설치가 완료되었습니다! 🎉

**지금 할 수 있는 것:**
- ✅ 터미널에서 AI와 대화
- ✅ 파일 읽기/쓰기
- ✅ 시스템 명령 실행

**아직 안 된 것:**
- ❌ Telegram/Discord 연동
- ❌ 자동화/크론잡
- ❌ 브라우저 제어

**다음 글 예고:**
- **3편**: Telegram 연동 - 어디서든 명령하기
- **4편**: 실전 활용 - 파일 관리 자동화
- **5편**: 크론잡으로 스케줄링

---

## 유용한 명령어 모음

```bash
# Gateway 관리
openclaw gateway start      # 시작
openclaw gateway stop       # 종료
openclaw gateway restart    # 재시작
openclaw status            # 상태 확인
openclaw logs --follow     # 로그 실시간 보기

# 설정
openclaw configure         # 대화형 설정
openclaw config get        # 현재 설정 보기
openclaw doctor           # 진단

# AI 대화
claude "메시지"            # 기본
claude -m qwen "메시지"    # 모델 지정
claude --help             # 옵션 보기

# 세션
openclaw sessions list     # 세션 목록
openclaw sessions history  # 대화 기록
```

---

## 참고 자료

- [OpenClaw 공식 문서](https://docs.openclaw.ai)
- [Anthropic API 문서](https://docs.anthropic.com)
- [Ollama 모델 목록](https://ollama.com/library)
- [OpenClaw GitHub](https://github.com/openclaw/openclaw)
- [Discord 커뮤니티](https://discord.com/invite/clawd)

---

*설치 중 문제가 생기면 댓글로 남겨주세요. 같이 해결해봅시다!*
