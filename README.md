<p align="center">
  <img src="assets/nanoclaw-logo.png" alt="NanoClaw" width="400">
</p>

<p align="center">
  에이전트를 자체 컨테이너에서 안전하게 실행하는 AI 비서. 가볍고, 이해하기 쉬우며, 사용자 필요에 맞게 완전히 커스터마이징 가능합니다.
</p>

<p align="center">
  <a href="https://github.com/TaegyuHO/nanoclaw">GitHub</a>&nbsp; • &nbsp;
  <a href="https://hov.kr">HOV</a>
</p>

<p align="center">
  <a href="https://github.com/qwibitai/nanoclaw">qwibitai/nanoclaw</a> 기반 커스텀 포크.
  도구 중립 가이드, Codex 연동, OpenAI Credential Proxy 등을 추가했습니다.
  <br>변경 내역은 <a href="#원본-대비-변경사항">원본 대비 변경사항</a>을 참조하세요.
</p>

Claude Code를 사용하여 NanoClaw는 코드를 동적으로 수정해 사용자의 필요에 맞는 기능을 구성합니다.

**신규:** 최초의 [Agent Swarms](https://code.claude.com/docs/en/agent-teams) 지원 AI 비서. 에이전트 팀을 생성하여 채팅에서 협업할 수 있습니다.

## NanoClaw를 만든 이유

[OpenClaw](https://github.com/openclaw/openclaw)는 인상적인 프로젝트이지만, 이해하지 못하는 복잡한 소프트웨어에 내 생활 전체에 대한 접근 권한을 주고 잠들 수는 없었습니다. OpenClaw는 약 50만 줄의 코드, 53개의 설정 파일, 70개 이상의 의존성을 가지고 있습니다. 보안은 OS 수준의 격리가 아닌 애플리케이션 수준(허용 목록, 페어링 코드)입니다. 모든 것이 공유 메모리를 가진 하나의 Node 프로세스에서 실행됩니다.

NanoClaw는 동일한 핵심 기능을 제공하면서도 이해할 수 있을 만큼 작은 코드베이스를 유지합니다: 하나의 프로세스와 몇 개의 파일. Claude 에이전트는 단순한 권한 검사가 아닌, 파일시스템이 격리된 자체 Linux 컨테이너에서 실행됩니다.

## 빠른 시작

```bash
git clone https://github.com/TaegyuHO/nanoclaw.git
cd nanoclaw
claude
```

그런 다음 `/setup`을 실행하세요. Claude Code가 의존성, 인증, 컨테이너 설정, 서비스 구성을 모두 처리합니다.

> **참고:** `/`로 시작하는 명령어(예: `/setup`, `/add-whatsapp`)는 [Claude Code 스킬](https://code.claude.com/docs/en/skills)입니다. 일반 터미널이 아닌 `claude` CLI 프롬프트 안에서 입력하세요. Claude Code가 설치되어 있지 않다면 [claude.com/product/claude-code](https://claude.com/product/claude-code)에서 받으세요.

## 철학

**이해할 수 있을 만큼 작게.** 하나의 프로세스, 몇 개의 소스 파일, 마이크로서비스 없음. NanoClaw 코드베이스 전체를 이해하고 싶다면 Claude Code에게 설명을 요청하세요.

**격리를 통한 보안.** 에이전트는 Linux 컨테이너(macOS의 Apple Container 또는 Docker)에서 실행되며, 명시적으로 마운트된 것만 볼 수 있습니다. Bash 접근은 컨테이너 내부에서 실행되므로 안전합니다.

**개인 사용자를 위한 설계.** NanoClaw는 모놀리식 프레임워크가 아니라 각 사용자의 정확한 요구에 맞는 소프트웨어입니다. 비대해지는 대신 맞춤형으로 설계되었습니다. 자신의 포크를 만들고 Claude Code로 필요에 맞게 수정하세요.

**커스터마이징 = 코드 변경.** 설정 파일 난립 없음. 다른 동작을 원하시나요? 코드를 수정하세요. 코드베이스가 충분히 작아서 안전하게 변경할 수 있습니다.

**AI 네이티브.**
- 설치 마법사 없음 — Claude Code가 설정을 안내합니다.
- 모니터링 대시보드 없음 — Claude에게 현재 상태를 물어보세요.
- 디버깅 도구 없음 — 문제를 설명하면 Claude가 수정합니다.

**기능보다 스킬.** 코드베이스에 기능(예: Telegram 지원)을 추가하는 대신, 기여자들은 `/add-telegram` 같은 [Claude Code 스킬](https://code.claude.com/docs/en/skills)을 제출하여 포크를 변환합니다. 결과적으로 정확히 필요한 것만 수행하는 깔끔한 코드를 갖게 됩니다.

**최고의 하네스, 최고의 모델.** NanoClaw는 Claude Agent SDK 위에서 실행됩니다. Claude Code를 직접 실행하는 것과 같으며, 뛰어난 코딩 및 문제 해결 능력으로 NanoClaw를 수정하고 확장하여 각 사용자에게 맞춤화할 수 있습니다.

## 지원 기능

- **멀티채널 메시징** — WhatsApp, Telegram, Discord, Slack, Gmail에서 비서와 대화. `/add-whatsapp`, `/add-telegram` 등의 스킬로 채널을 추가. 동시에 여러 채널 운영 가능.
- **격리된 그룹 컨텍스트** — 각 그룹은 자체 `CLAUDE.md` 메모리, 격리된 파일시스템을 가지며, 해당 파일시스템만 마운트된 자체 컨테이너 샌드박스에서 실행.
- **메인 채널** — 관리 제어를 위한 프라이빗 채널(셀프 채팅). 모든 그룹은 완전히 격리됨.
- **예약 작업** — Claude를 실행하고 메시지를 보낼 수 있는 반복 작업.
- **웹 접근** — 웹에서 콘텐츠 검색 및 가져오기.
- **컨테이너 격리** — 에이전트는 Apple Container(macOS) 또는 Docker(macOS/Linux)에서 샌드박스 처리.
- **Agent Swarms** — 복잡한 작업에 협업하는 전문 에이전트 팀 구성. NanoClaw는 Agent Swarms를 지원하는 최초의 개인 AI 비서.
- **선택적 통합** — Gmail(`/add-gmail`) 등 스킬을 통한 추가 기능.

## 도구 중립 가이드

NanoClaw의 핵심 지식(설치, 채널 설정, 디버깅 등)은 `guides/` 디렉터리에 도구 중립적인 마크다운 가이드로 제공됩니다. Claude Code뿐만 아니라 **Codex, Copilot, Windsurf** 등 어떤 AI 코딩 도구에서든 활용할 수 있으며, 사람이 직접 따라할 수도 있습니다.

```
guides/
├── setup/guide.md              # 설치 및 서비스 설정
├── debug/guide.md              # 컨테이너 디버깅 및 트러블슈팅
├── add-telegram/guide.md       # Telegram 채널 추가
├── add-whatsapp/guide.md       # WhatsApp 채널 추가
├── add-slack/guide.md          # Slack 채널 추가
├── add-discord/guide.md        # Discord 채널 추가
├── add-gmail/guide.md          # Gmail 통합
├── add-voice-transcription/    # 음성 메시지 전사 (Whisper)
├── add-pdf-reader/             # PDF 텍스트 추출
├── add-image-vision/           # 이미지 첨부파일 처리
├── add-ollama-tool/            # 로컬 Ollama 모델 통합
├── add-reactions/              # WhatsApp 이모지 반응
├── add-compact/                # 세션 컨텍스트 압축
└── index.yaml                  # 머신 리더블 인덱스
```

진입점: [`AGENTS.md`](AGENTS.md) — Codex/Copilot이 자동으로 발견하는 가이드 목록.

Claude Code 스킬(`.claude/skills/`)은 실행 로직만 담당하고, 도메인 지식은 가이드를 참조합니다.

## 사용법

트리거 단어(기본: `@Andy`)로 비서에게 말하세요:

```
@Andy 매일 아침 9시에 영업 파이프라인 개요를 보내줘 (Obsidian vault 폴더에 접근 가능)
@Andy 매주 금요일 지난주 git 히스토리를 검토하고 README에 변경 사항이 있으면 업데이트해줘
@Andy 매주 월요일 오전 8시에 Hacker News와 TechCrunch에서 AI 개발 뉴스를 정리해서 브리핑해줘
```

메인 채널(셀프 채팅)에서 그룹과 작업을 관리할 수 있습니다:
```
@Andy 모든 그룹의 예약 작업 목록을 보여줘
@Andy 월요일 브리핑 작업을 일시 중지해줘
@Andy 가족 채팅 그룹에 참여해줘
```

## 커스터마이징

NanoClaw는 설정 파일을 사용하지 않습니다. 변경하려면 Claude Code에게 원하는 것을 말하세요:

- "트리거 단어를 @Bob으로 변경해줘"
- "앞으로 응답을 더 짧고 직접적으로 해줘"
- "좋은 아침이라고 하면 맞춤 인사를 해줘"
- "매주 대화 요약을 저장해줘"

또는 `/customize`를 실행하여 안내에 따라 변경하세요.

코드베이스가 충분히 작아서 Claude가 안전하게 수정할 수 있습니다.

## 기여하기

기능 추가보다 스킬 방식을 권장합니다. 새로운 채널이나 기능은 별도 브랜치에서 작업한 후 `skill/` 브랜치로 머지하세요. `/add-telegram` 같은 스킬로 필요한 기능만 선택적으로 적용할 수 있습니다.

## 요구사항

- macOS 또는 Linux
- Node.js 20+
- [Claude Code](https://claude.ai/download)
- [Apple Container](https://github.com/apple/container) (macOS) 또는 [Docker](https://docker.com/products/docker-desktop) (macOS/Linux)

## 아키텍처

```
채널 --> SQLite --> 폴링 루프 --> 컨테이너 (Claude Agent SDK) --> 응답
```

단일 Node.js 프로세스. 채널은 스킬을 통해 추가되고 시작 시 자동 등록됩니다 — 오케스트레이터가 자격 증명이 있는 채널을 연결합니다. 에이전트는 파일시스템이 격리된 Linux 컨테이너에서 실행됩니다. 마운트된 디렉터리만 접근 가능합니다. 그룹별 메시지 큐와 동시성 제어. 파일시스템을 통한 IPC.

전체 아키텍처 세부사항은 [docs/SPEC.md](docs/SPEC.md)를 참조하세요.

주요 파일:
- `src/index.ts` — 오케스트레이터: 상태, 메시지 루프, 에이전트 호출
- `src/channels/registry.ts` — 채널 레지스트리 (시작 시 자동 등록)
- `src/ipc.ts` — IPC 감시자 및 작업 처리
- `src/router.ts` — 메시지 포맷팅 및 아웃바운드 라우팅
- `src/group-queue.ts` — 글로벌 동시성 제한이 있는 그룹별 큐
- `src/container-runner.ts` — 스트리밍 에이전트 컨테이너 생성
- `src/task-scheduler.ts` — 예약 작업 실행
- `src/db.ts` — SQLite 작업 (메시지, 그룹, 세션, 상태)
- `groups/*/CLAUDE.md` — 그룹별 메모리
- `guides/` — 도구 중립적 가이드 ([AGENTS.md](AGENTS.md) 참조)

## FAQ

**왜 Docker인가요?**

Docker는 크로스 플랫폼 지원(macOS, Linux, WSL2를 통한 Windows)과 성숙한 생태계를 제공합니다. macOS에서는 `/convert-to-apple-container`를 통해 더 가벼운 네이티브 런타임인 Apple Container로 전환할 수 있습니다.

**Linux에서 실행할 수 있나요?**

네. Docker가 기본 런타임이며 macOS와 Linux 모두에서 작동합니다. `/setup`을 실행하세요.

**안전한가요?**

에이전트는 애플리케이션 수준의 권한 검사가 아닌 컨테이너에서 실행됩니다. 명시적으로 마운트된 디렉터리만 접근할 수 있습니다. 실행하는 것을 여전히 검토해야 하지만, 코드베이스가 충분히 작아서 실제로 검토할 수 있습니다. 전체 보안 모델은 [docs/SECURITY.md](docs/SECURITY.md)를 참조하세요.

**왜 설정 파일이 없나요?**

설정 파일 난립을 원하지 않습니다. 모든 사용자가 코드가 정확히 원하는 것을 수행하도록 NanoClaw를 커스터마이징해야 합니다. 설정 파일을 선호한다면 Claude에게 추가하라고 말하세요.

**서드파티 또는 오픈소스 모델을 사용할 수 있나요?**

네. NanoClaw는 Claude API 호환 모델 엔드포인트를 지원합니다. `.env` 파일에 다음 환경 변수를 설정하세요:

```bash
ANTHROPIC_BASE_URL=https://your-api-endpoint.com
ANTHROPIC_AUTH_TOKEN=your-token-here
```

사용 가능한 옵션:
- API 프록시를 통한 [Ollama](https://ollama.ai) 로컬 모델
- [Together AI](https://together.ai), [Fireworks](https://fireworks.ai) 등에 호스팅된 오픈소스 모델
- Anthropic 호환 API를 사용하는 커스텀 모델 배포

참고: 최상의 호환성을 위해 모델은 Anthropic API 형식을 지원해야 합니다.

**문제를 어떻게 디버깅하나요?**

Claude Code에게 물어보세요. "스케줄러가 왜 실행 안 돼?" "최근 로그에 뭐가 있어?" "이 메시지는 왜 응답이 없어?" 이것이 NanoClaw의 근간인 AI 네이티브 접근 방식입니다.

**설정이 작동하지 않아요.**

설정 중 문제가 있으면 Claude가 동적으로 해결을 시도합니다. 그래도 안 되면 `claude`를 실행한 후 `/debug`를 실행하세요. Claude가 다른 사용자에게도 영향을 줄 수 있는 문제를 발견하면, setup SKILL.md를 수정하는 PR을 열어주세요.

**어떤 변경이 코드베이스에 수용되나요?**

보안 수정, 버그 수정, 명확한 개선만 기본 설정에 수용됩니다. 그것뿐입니다.

그 외 모든 것(새로운 기능, OS 호환성, 하드웨어 지원, 개선사항)은 스킬로 기여해야 합니다.

이렇게 하면 기본 시스템을 최소한으로 유지하면서 모든 사용자가 원하지 않는 기능을 물려받지 않고 자신의 설치를 커스터마이징할 수 있습니다.

## 원본 대비 변경사항

이 레포지토리는 [qwibitai/nanoclaw](https://github.com/qwibitai/nanoclaw)를 기반으로 하며, 다음과 같은 변경이 적용되었습니다.

### 도구 중립 가이드 시스템 (`guides/`)

원본 NanoClaw는 모든 지식과 실행 로직이 Claude Code 전용 스킬 파일(`.claude/skills/*/SKILL.md`)에 혼재되어 있어 다른 AI 도구에서 활용할 수 없었습니다.

**변경 내용:**
- **`guides/` 디렉터리 신설** — 13개 가이드를 도구 중립적 마크다운으로 분리. 설치, 디버깅, 채널 설정(Telegram, WhatsApp, Slack, Discord, Gmail), 기능 확장(음성 전사, PDF, 이미지, Ollama, 반응, 세션 압축) 포함.
- **SKILL.md 경량화** — 기존 스킬에서 도메인 지식을 가이드로 추출하고, SKILL.md에는 Claude Code 실행 로직만 남김. `guide` frontmatter 필드로 가이드를 참조.
- **`AGENTS.md` 추가** — Codex, Copilot 등이 자동으로 가이드를 발견할 수 있는 진입점.
- **`guides/index.yaml` 추가** — 머신 리더블 가이드 인덱스 (name, path, description, tags, requires).
- **`.github/copilot-instructions.md` 추가** — GitHub Copilot용 가이드 참조.

**효과:**
- Claude Code 외에 **Codex, Copilot, Windsurf** 등에서도 동일한 설치/설정 지식 활용 가능
- 사람이 직접 따라할 수 있는 독립적 문서로도 기능
- 기존 Claude Code 스킬 워크플로우는 100% 호환 유지

### OpenSpec 스펙 관리 도입

[OpenSpec](https://github.com/Fission-AI/OpenSpec)을 도입하여 구조화된 변경 관리를 적용했습니다.

- `openspec/` 디렉터리에 프로젝트 스펙(guide-format, guide-discovery, skill-guide-separation) 관리
- `.claude/commands/opsx/` — OpenSpec 워크플로우 명령어 (new, continue, apply, verify, archive 등)

### Codex CLI 연동 및 Credential Proxy 확장

OpenAI Codex CLI에서도 NanoClaw를 바로 사용할 수 있도록 설정했습니다.

- **`.codex/config.toml` 추가** — Codex 프로젝트 설정. `CLAUDE.md`를 폴백 지시 파일로 등록하여 기존 프로젝트 컨텍스트를 Codex가 자동으로 인식.
- **`AGENTS.md`** — Codex가 기본으로 읽는 프로젝트 지시 파일. 가이드 목록이 포함되어 있어 Codex에서 `/setup`, 채널 추가 등의 가이드를 바로 참조 가능.
- **`container/Dockerfile` 업데이트** — `@openai/codex`를 글로벌 패키지에 추가. API 키는 이미지에 포함하지 않음.
- **OpenAI Credential Proxy 추가** — Anthropic과 동일한 격리 패턴으로 `OPENAI_API_KEY`도 credential proxy(포트 3002)를 통해 주입. 컨테이너는 placeholder 키만 받고, 실제 키는 프록시가 런타임에 교체.

```
.env (호스트)                    컨테이너 (격리)
├─ ANTHROPIC_API_KEY=sk-ant-xxx  │  ANTHROPIC_BASE_URL=http://host:3001
├─ OPENAI_API_KEY=sk-xxx         │  OPENAI_BASE_URL=http://host:3002
│                                │  ANTHROPIC_API_KEY=placeholder
Credential Proxies               │  OPENAI_API_KEY=placeholder
├─ :3001 → Anthropic API        │
├─ :3002 → OpenAI API           │  실제 키는 컨테이너에 도달하지 않음
```

```bash
# Codex 설치
npm install -g @openai/codex

# .env에 키 추가 (프록시가 자동으로 읽음)
echo "OPENAI_API_KEY=sk-xxxxx" >> .env

# 프로젝트에서 실행
cd nanoclaw
codex "setup 가이드 따라서 설치해줘"
```

> **참고:** `.env`에 `OPENAI_API_KEY`가 없으면 OpenAI 프록시는 자동으로 스킵됩니다. Claude만 사용하는 기존 환경에서는 변경 없이 동작합니다.

### README 한글화

README.md를 한국어로 번역했습니다.

## 변경 이력

주요 변경사항 및 마이그레이션 안내는 [CHANGELOG.md](CHANGELOG.md)를 참조하세요.

## 라이선스

MIT
