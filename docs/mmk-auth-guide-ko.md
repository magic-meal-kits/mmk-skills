# MMK CLI 인증 가이드

> 환경에 따라 적합한 인증 방법을 선택하세요. 이 문서는 로컬 PC, Claude Code 웹, CI/CD 등 다양한 환경에서의 인증 방법을 안내합니다.

## 목차

- [인증 명령어 요약](#인증-명령어-요약)
- [환경별 인증 방법](#환경별-인증-방법)
  - [로컬 PC (브라우저 있음)](#로컬-pc-브라우저-있음)
  - [Claude Code 웹 / 클라우드 / 서버 (브라우저 없음)](#claude-code-웹--클라우드--서버-브라우저-없음)
- [Long-Lived 토큰 관리](#long-lived-토큰-관리)
  - [토큰 생성](#토큰-생성)
  - [토큰 확인](#토큰-확인)
  - [토큰 폐기](#토큰-폐기)
  - [스코프 제한](#스코프-제한)
- [환경별 설정 요약표](#환경별-설정-요약표)
- [토큰 저장 위치](#토큰-저장-위치)
- [문제 해결](#문제-해결)

---

## 인증 명령어 요약

| 명령어 | 설명 |
|--------|------|
| `mmk auth login` | 인증 (브라우저 PKCE / headless device flow 자동 전환) |
| `mmk auth status` | 현재 인증 상태 확인 |
| `mmk auth token` | 현재 토큰 정보 표시 |
| `mmk auth token --create` | Long-lived 토큰 생성 (유효기간 1년) |
| `mmk auth token --show` | 현재 액세스 토큰 원문 출력 |
| `mmk auth token --revoke <token>` | 특정 토큰 폐기 |
| `mmk auth logout` | 토큰 폐기 및 로컬 인증 정보 삭제 |

---

## 환경별 인증 방법

### 로컬 PC (브라우저 있음)

브라우저가 있는 일반 PC 환경에서는 `mmk auth login`만으로 인증이 완료됩니다.

```bash
# 1. 서버 설정
mmk config set server <your-server-url>

# 2. 로그인 (브라우저가 자동으로 열림)
mmk auth login
```

**동작 방식:**
- OAuth 2.0 PKCE 플로우 사용
- 기본 브라우저가 자동으로 열리며 인증 페이지로 이동
- 인증 완료 후 CLI가 자동으로 토큰 수신 및 저장
- 토큰은 `~/.config/mmk/config.yaml`에 저장

**이후 할 일 — Long-lived 토큰 발급:**

로컬 PC에서 인증한 뒤, Claude Code 웹이나 CI/CD 등 headless 환경에서 사용할 long-lived 토큰을 미리 발급해 둡니다:

```bash
mmk auth token --create --name "claude-code-web"
```

출력된 토큰 값을 안전하게 보관하세요. 이 토큰은 headless 환경에서 사용합니다.

---

### Claude Code 웹 / 클라우드 / 서버 (브라우저 없음)

브라우저가 없는 환경에서는 두 가지 방법이 있습니다.

#### 방법 1: Device Flow (권장 — 최초 1회)

`mmk auth login`은 브라우저를 감지하지 못하면 **device flow**로 자동 전환됩니다.

```bash
# 1. 서버 설정
mmk config set server <your-server-url>

# 2. 로그인 (device flow 자동 전환)
mmk auth login
```

**동작 방식:**
- CLI가 인증 URL과 코드를 터미널에 출력
- 해당 URL을 **다른 기기의 브라우저**(스마트폰, 다른 PC 등)에서 열기
- 출력된 코드를 입력하여 인증 승인
- CLI가 자동으로 토큰을 수신하고 저장

```
예시 출력:
  인증을 위해 아래 URL을 브라우저에서 여세요:
  https://your-server.com/device
  코드: ABCD-1234
  대기 중...
```

> **참고:** Claude Code 웹 환경에서는 출력된 URL을 복사하여 로컬 PC 브라우저에 붙여넣으면 됩니다.

#### 방법 2: Long-Lived 토큰 주입 (권장 — 반복 세션)

로컬 PC에서 미리 발급한 long-lived 토큰을 환경변수로 주입합니다. **매 세션마다 인증 과정을 건너뛸 수 있으므로** Claude Code 웹에서 가장 편리한 방법입니다.

```bash
# 토큰을 config 파일에 직접 작성
mkdir -p ~/.config/mmk
cat > ~/.config/mmk/config.yaml <<YAML
server: https://your-server-url.com
token: mmk_tok_xxxxxxxxxxxxxxxxxx
YAML
```

또는 환경변수를 사용:

```bash
export MMK_SERVER="https://your-server-url.com"
export MMK_AUTH_TOKEN="mmk_tok_xxxxxxxxxxxxxxxxxx"

# config 파일에 반영
mmk config set server "$MMK_SERVER"
```

**SessionStart Hook과 조합하면 완전 자동화됩니다.** 자세한 내용은 [Claude Code 웹 설정 가이드](./claude-code-web-setup-ko.md#5단계-sessionstart-hook-설정)를 참고하세요.

---

## Long-Lived 토큰 관리

Long-lived 토큰은 **유효기간 1년**의 장기 토큰입니다. CI/CD, GitHub Actions, Claude Code 웹, 서버 등 headless 환경에서 반복적인 로그인 없이 인증을 유지할 수 있습니다.

### 토큰 생성

> **사전 조건:** `mmk auth login`으로 먼저 인증된 상태여야 합니다 (로컬 PC에서 수행 권장).

```bash
# 기본 토큰 생성 (유효기간 1년)
mmk auth token --create

# 용도를 식별할 수 있도록 이름 지정 (권장)
mmk auth token --create --name "claude-code-web"
mmk auth token --create --name "github-actions"
mmk auth token --create --name "dev-server"
```

출력 예시:

```
Token created successfully
Name:    claude-code-web
Token:   mmk_tok_xxxxxxxxxxxxxxxxxx
Expires: 2027-04-10

Save this token securely — it will not be shown again.
```

> **주의:** 토큰 값은 생성 시 한 번만 표시됩니다. 반드시 안전한 곳에 저장하세요.

### 토큰 확인

```bash
# 현재 토큰 정보 표시
mmk auth token

# 원시 토큰 값 출력 (스크립트에서 사용)
mmk auth token --show

# JSON 형식으로 출력
mmk auth token -o json
```

### 토큰 폐기

토큰이 유출되었거나 더 이상 필요하지 않으면 즉시 폐기합니다:

```bash
mmk auth token --revoke mmk_tok_xxxxxxxxxxxxxxxxxx
```

### 스코프 제한

특정 서비스만 접근 가능하도록 토큰 스코프를 제한할 수 있습니다:

```bash
# YouTube만 접근 가능한 토큰
mmk auth token --create --name "youtube-only" --scope youtube

# Notion만 접근 가능한 토큰
mmk auth token --create --name "notion-bot" --scope notion
```

스코프를 제한하면 토큰 유출 시 피해 범위를 최소화할 수 있습니다.

---

## 환경별 설정 요약표

| 환경 | 인증 방법 | 토큰 유형 | 자동화 |
|------|-----------|-----------|--------|
| **로컬 PC** (macOS/Windows/Linux) | `mmk auth login` (브라우저 PKCE) | 세션 토큰 | 불필요 (토큰 자동 저장) |
| **Claude Code 웹** | Device flow 또는 long-lived 토큰 주입 | Long-lived 토큰 권장 | SessionStart Hook |
| **CI/CD** (GitHub Actions 등) | Long-lived 토큰을 Secret으로 주입 | Long-lived 토큰 | 워크플로우에서 config 작성 |
| **원격 서버** (SSH 등) | Device flow 또는 long-lived 토큰 주입 | Long-lived 토큰 권장 | 시스템 환경변수 또는 config 파일 |
| **Docker 컨테이너** | Long-lived 토큰을 환경변수로 전달 | Long-lived 토큰 | `docker run -e MMK_AUTH_TOKEN=...` |

### 환경별 권장 플로우

```
로컬 PC (최초 설정)
  │
  ├─ mmk auth login          ← 브라우저 PKCE로 인증
  ├─ mmk auth token --create ← long-lived 토큰 발급
  │
  └─ 토큰을 각 환경에 배포:
       ├─ Claude Code 웹  → SessionStart Hook의 MMK_AUTH_TOKEN
       ├─ GitHub Actions  → Repository Secret
       ├─ 원격 서버       → 환경변수 또는 config.yaml
       └─ Docker          → -e MMK_AUTH_TOKEN=...
```

---

## 토큰 저장 위치

| 항목 | 경로 / 위치 |
|------|-------------|
| CLI 설정 파일 | `~/.config/mmk/config.yaml` |
| 설정 파일 오버라이드 | `--config <path>` 플래그 |
| 환경변수 (서버) | `MMK_SERVER` |
| 환경변수 (토큰) | `MMK_AUTH_TOKEN` |

`config.yaml` 파일 구조:

```yaml
server: https://your-server-url.com
token: mmk_tok_xxxxxxxxxxxxxxxxxx
```

---

## 문제 해결

### "not authenticated" 오류

```bash
# 1. 인증 상태 확인
mmk auth status

# 2. 토큰 유효성 확인
mmk auth token

# 3. 재인증 (브라우저 또는 device flow)
mmk auth login

# 4. 서버 URL 확인
mmk config get server
```

### Device flow에서 코드 입력 후 응답 없음

- 서버 URL이 올바른지 확인: `mmk config get server`
- 네트워크 연결 상태 확인
- 코드 유효시간 만료 시 `mmk auth login`을 다시 실행

### Long-lived 토큰이 작동하지 않음

```bash
# 토큰이 올바르게 저장되었는지 확인
mmk auth token --show

# 토큰 유효기간 확인
mmk auth token -o json

# 토큰이 폐기되었을 수 있음 — 새 토큰 발급
mmk auth token --create --name "replacement"
```

### Claude Code 웹에서 매 세션마다 인증 초기화됨

Long-lived 토큰을 SessionStart Hook으로 자동 주입하세요:

1. 로컬 PC에서 `mmk auth token --create --name "claude-code-web"` 실행
2. 토큰 값을 `MMK_AUTH_TOKEN` 환경변수로 설정
3. [SessionStart Hook 설정](./claude-code-web-setup-ko.md#5단계-sessionstart-hook-설정) 참고

---

## 참고 자료

- [Claude Code 웹 설정 가이드](./claude-code-web-setup-ko.md) — 전체 환경 설정 및 SessionStart Hook
- [MMK Shared](../skills/mmk-shared/SKILL.md) — 인증, 글로벌 플래그, 에러 처리
- [CONTEXT.md](../CONTEXT.md) — CLI 명령어 구문 참고
