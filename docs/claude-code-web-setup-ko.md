# Claude Code 웹 환경에서 MMK Skills 설정 가이드

> 이 문서는 [Claude Code 웹](https://claude.ai/code) 환경에서 MMK Skills를 설치하고 사용하기 위한 전체 설정 과정을 안내합니다.

## 목차

- [개요](#개요)
- [사전 요구사항](#사전-요구사항)
- [1단계: MMK CLI 설치](#1단계-mmk-cli-설치)
- [2단계: 환경변수 설정](#2단계-환경변수-설정)
- [3단계: 인증](#3단계-인증)
- [4단계: Skills 설치](#4단계-skills-설치)
- [5단계: SessionStart Hook 설정](#5단계-sessionstart-hook-설정)
- [설정 검증](#설정-검증)
- [사용 예시](#사용-예시)
- [문제 해결](#문제-해결)

---

## 개요

Claude Code 웹 환경은 브라우저 기반의 클라우드 샌드박스에서 실행됩니다. 로컬 터미널과 달리 **세션이 시작될 때마다 환경이 초기화**되므로, 매번 CLI 도구를 재설치하고 인증을 다시 설정해야 합니다.

이 문제를 해결하기 위해 **SessionStart Hook**을 활용하면 세션 시작 시 자동으로 환경을 구성할 수 있습니다.

### 아키텍처 요약

```
Claude Code 웹 세션 시작
  │
  ├─ SessionStart Hook 실행
  │   ├─ MMK CLI 설치 (npm install -g)
  │   ├─ 서버 URL 설정 (mmk config set)
  │   └─ 인증 토큰 복원 (환경변수에서)
  │
  ├─ Skills 로드 (.claude/skills/)
  │   └─ mmk-shared (자동 로드) + 서비스별 스킬
  │
  └─ 사용자 작업 시작
```

---

## 사전 요구사항

| 항목 | 설명 |
|------|------|
| Claude Code 웹 접속 | [claude.ai/code](https://claude.ai/code) 계정 필요 |
| MMK 서버 엔드포인트 | Magic Meal Kits 서버 URL (관리자에게 문의) |
| MMK 인증 정보 | `mmk auth login`으로 발급받은 토큰 |
| Node.js / npm | Claude Code 웹 환경에 기본 포함 |

---

## 1단계: MMK CLI 설치

Claude Code 웹 터미널에서 MMK CLI를 전역 설치합니다.

```bash
npm install -g @magic-meal-kits/cli@beta
```

설치 확인:

```bash
mmk --version
```

> **참고:** Claude Code 웹 환경은 세션 종료 시 전역 패키지가 초기화됩니다. 매 세션마다 재설치가 필요하며, 이를 자동화하려면 [SessionStart Hook](#5단계-sessionstart-hook-설정) 섹션을 참고하세요.

---

## 2단계: 환경변수 설정

MMK CLI가 필요로 하는 핵심 환경변수를 설정합니다.

### 필수 환경변수

| 변수명 | 설명 | 예시 |
|--------|------|------|
| `MMK_SERVER` | MMK 서버 엔드포인트 URL | `https://api.magicmealkits.com` |
| `MMK_AUTH_TOKEN` | 인증 토큰 (선택 — 토큰 기반 자동 인증 시 사용) | `mmk_tok_xxxxx` |

### Claude Code 웹에서 환경변수 지정 방법

Claude Code 웹에서는 **프로젝트 설정**에서 환경변수를 관리할 수 있습니다.

**방법 1: 세션 내 직접 설정**

```bash
export MMK_SERVER="https://your-server-url.com"
mmk config set server "$MMK_SERVER"
```

**방법 2: `.claude/settings.json`에서 Hook을 통한 자동 설정**

SessionStart Hook에서 환경변수를 읽어 자동 구성합니다 (5단계에서 상세 설명).

### 서버 URL 설정

```bash
mmk config set server <your-server-url>
```

설정 확인:

```bash
mmk config get server
```

---

## 3단계: 인증

MMK CLI는 토큰 기반 인증을 사용합니다.

### 대화형 로그인

```bash
mmk auth login
```

브라우저 기반 인증 플로우가 시작됩니다. Claude Code 웹 환경에서는 출력되는 URL을 별도 브라우저 탭에서 열어 인증을 완료합니다.

### 인증 상태 확인

```bash
mmk auth status
```

### 인증 토큰 위치

인증 완료 후 토큰은 아래 경로에 저장됩니다:

```
~/.config/mmk/config.yaml
```

> **팁:** 세션 간 토큰을 유지하려면 해당 파일의 토큰 값을 별도로 보관해 두고, SessionStart Hook에서 복원할 수 있습니다.

---

## 4단계: Skills 설치

### 전체 Skills 설치 (권장)

```bash
npx skills add https://github.com/magic-meal-kits/mmk-skills
```

이 명령은 아래의 모든 스킬을 한번에 설치합니다:
- `mmk-shared` — 인증, 글로벌 플래그, 에러 처리 (백그라운드 자동 로드)
- `mmk-notion` — Notion 관련 41개 명령어
- `mmk-paymint` — 결제/청구 관련 7개 명령어
- `mmk-threads` — Threads 관련 3개 명령어
- `mmk-youtube` — YouTube 관련 3개 명령어
- 레시피 스킬 6개 (배치 작업, 워크플로우)

### 서비스별 개별 설치

필요한 서비스만 선택적으로 설치할 수도 있습니다:

```bash
# Notion만 설치
npx skills add https://github.com/magic-meal-kits/mmk-skills/tree/main/skills/mmk-notion

# Paymint만 설치
npx skills add https://github.com/magic-meal-kits/mmk-skills/tree/main/skills/mmk-paymint

# 특정 하위 명령어만 설치
npx skills add https://github.com/magic-meal-kits/mmk-skills/tree/main/skills/mmk-notion-page
npx skills add https://github.com/magic-meal-kits/mmk-skills/tree/main/skills/mmk-paymint-send
```

### 플러그인 마켓플레이스를 통한 설치

```bash
/plugin marketplace add magic-meal-kits/mmk-skills
/plugin install mmk-skills@mmk-plugins
```

---

## 5단계: SessionStart Hook 설정

SessionStart Hook은 Claude Code 세션이 시작될 때 자동으로 실행되는 셸 명령어입니다. 웹 환경에서 매 세션마다 반복되는 설정 작업을 자동화할 수 있습니다.

### Hook 설정 파일

`.claude/settings.json` 파일에 Hook을 정의합니다:

```json
{
  "hooks": {
    "SessionStart": [
      {
        "matcher": "",
        "hooks": [
          {
            "type": "command",
            "command": "bash .claude/hooks/session-start.sh"
          }
        ]
      }
    ]
  }
}
```

### Hook 스크립트 작성

`.claude/hooks/session-start.sh` 파일을 생성합니다:

```bash
#!/usr/bin/env bash
set -euo pipefail

echo "[MMK Setup] 환경 초기화 시작..."

# 1. MMK CLI 설치 (미설치 시에만)
if ! command -v mmk &>/dev/null; then
  echo "[MMK Setup] MMK CLI 설치 중..."
  npm install -g @magic-meal-kits/cli@beta
  echo "[MMK Setup] MMK CLI 설치 완료: $(mmk --version)"
else
  echo "[MMK Setup] MMK CLI 이미 설치됨: $(mmk --version)"
fi

# 2. 서버 URL 설정
if [ -n "${MMK_SERVER:-}" ]; then
  mmk config set server "$MMK_SERVER"
  echo "[MMK Setup] 서버 설정 완료: $MMK_SERVER"
else
  echo "[MMK Setup] 경고: MMK_SERVER 환경변수가 설정되지 않았습니다."
  echo "[MMK Setup] 수동으로 설정하세요: mmk config set server <url>"
fi

# 3. 인증 토큰 복원 (환경변수에 토큰이 있는 경우)
if [ -n "${MMK_AUTH_TOKEN:-}" ]; then
  mkdir -p ~/.config/mmk
  cat > ~/.config/mmk/config.yaml <<YAML
server: ${MMK_SERVER:-}
token: ${MMK_AUTH_TOKEN}
YAML
  echo "[MMK Setup] 인증 토큰 복원 완료"
else
  echo "[MMK Setup] 참고: MMK_AUTH_TOKEN이 설정되지 않았습니다."
  echo "[MMK Setup] 수동으로 인증하세요: mmk auth login"
fi

# 4. 설정 검증
echo "[MMK Setup] 환경 검증 중..."
mmk auth status 2>/dev/null && echo "[MMK Setup] 인증 상태: 정상" || echo "[MMK Setup] 인증 필요: mmk auth login 실행하세요"

echo "[MMK Setup] 환경 초기화 완료"
```

### 스크립트에 실행 권한 부여

```bash
chmod +x .claude/hooks/session-start.sh
```

### 디렉토리 구조

설정이 완료되면 프로젝트의 `.claude/` 디렉토리는 다음과 같습니다:

```
.claude/
├── hooks/
│   └── session-start.sh       <- SessionStart Hook 스크립트
├── settings.json              <- Hook 설정 파일
└── skills/
    └── skill-sync/
        └── SKILL.md           <- 스킬 동기화 자동화 (기존)
```

---

## 설정 검증

모든 설정이 완료된 후 아래 명령어로 환경을 검증합니다:

```bash
# CLI 설치 확인
mmk --version

# 서버 연결 확인
mmk config get server

# 인증 상태 확인
mmk auth status

# 전체 환경 진단
mmk doctor
```

`mmk doctor`는 CLI 설정, 서버 연결, 인증 상태를 한번에 점검합니다.

### Skills 설치 확인

Claude Code에서 `/`를 입력하면 설치된 스킬 목록이 표시됩니다:

```
/mmk-notion             — Notion 명령어 개요
/mmk-notion-page        — 페이지 초대, 게시, 복제 등 (13개 명령어)
/mmk-paymint            — 결제/청구 개요
/mmk-paymint-send       — 청구서 발송
/mmk-threads            — Threads 개요
/mmk-youtube            — YouTube 개요
...
```

---

## 사용 예시

### Notion 페이지에 사용자 초대

```
/mmk-notion-page
> kim@example.com을 에디터로 https://notion.so/my-page-abc123에 초대해줘
```

### Paymint 청구서 발송

```
/mmk-paymint-send
> 01012345678로 50,000원 청구서 발송해줘. 이름: 김철수, 상품: 워크샵 참가비, 만료일: 2026-05-01
```

### Notion 데이터베이스에서 배치 청구서 발송

```
/mmk-paymint-notion-invoice
> 데이터베이스 abc123-def456에서 청구서 일괄 발송해줘
```

### YouTube 영상 자막 가져오기

```
/mmk-youtube-transcript
> https://youtube.com/watch?v=abc123 자막 가져와줘
```

### Threads 최근 게시물 확인

```
/mmk-threads-posts
> 최근 게시물 10개 보여줘
```

---

## 문제 해결

### "not authenticated" 오류

```bash
# 인증 상태 확인
mmk auth status

# 재인증
mmk auth login

# 서버 URL 확인
mmk config get server
```

### SessionStart Hook이 실행되지 않을 때

1. `.claude/settings.json` 파일이 올바른 JSON 형식인지 확인
2. Hook 스크립트에 실행 권한이 있는지 확인 (`chmod +x`)
3. 스크립트를 수동으로 실행하여 오류 확인:

```bash
bash .claude/hooks/session-start.sh
```

### CLI 명령어가 실패할 때

```bash
# 전체 환경 진단
mmk doctor

# CLI 최신 버전 확인
mmk upgrade --check

# 서버 상태 확인
mmk status
```

### 일반적인 HTTP 오류 코드

| 코드 | 의미 | 대응 |
|------|------|------|
| 400 | 잘못된 요청 (필수 파라미터 누락) | 필수 플래그 확인 |
| 403 | 권한 없음 | 접근 권한 확인 |
| 404 | 리소스 없음 | block-id 또는 이메일 확인 |

---

## 참고 자료

- [MMK Skills README](../README.md) — 전체 스킬 목록 및 아키텍처
- [MMK Shared](../skills/mmk-shared/SKILL.md) — 인증, 글로벌 플래그, 에러 처리
- [CONTEXT.md](../CONTEXT.md) — CLI 명령어 구문 참고
- [Claude Code 공식 문서](https://docs.anthropic.com/en/docs/claude-code) — Claude Code 사용법
