<p align="center">
  <img src="assets/logo.png" alt="Archon" width="160" />
</p>

<h1 align="center">Archon</h1>

<p align="center">
  AI 코딩을 위한 최초의 오픈소스 하네스 빌더. AI 코딩을 결정론적이고 반복 가능하게 만드세요.
</p>

<p align="center">
  <a href="https://trendshift.io/repositories/13964" target="_blank"><img src="https://trendshift.io/api/badge/repositories/13964" alt="coleam00%2FArchon | Trendshift" style="width: 250px; height: 55px;" width="250" height="55"/></a>
</p>

<p align="center">
  <a href="LICENSE"><img src="https://img.shields.io/badge/License-MIT-blue.svg" alt="라이선스: MIT" /></a>
  <a href="https://github.com/coleam00/Archon/actions/workflows/test.yml"><img src="https://github.com/coleam00/Archon/actions/workflows/test.yml/badge.svg" alt="CI" /></a>
  <a href="https://archon.diy"><img src="https://img.shields.io/badge/docs-archon.diy-blue" alt="문서" /></a>
</p>

---

Archon은 AI 코딩 에이전트를 위한 워크플로우 엔진입니다. 개발 프로세스를 YAML 워크플로우로 정의하세요 - 계획, 구현, 검증, 코드 리뷰, PR 생성 - 그리고 모든 프로젝트에서 안정적으로 실행하세요.

Dockerfile이 인프라에, GitHub Actions가 CI/CD에 한 것처럼 - Archon은 AI 코딩 워크플로우에 그것을 합니다. n8n과 같지만, 소프트웨어 개발을 위한 것입니다.

## 왜 Archon인가?

AI 에이전트에게 "이 버그를 고쳐줘"라고 요청하면, 무슨 일이 일어나는지는 모델의 상태에 따라 달라집니다. 계획을 건너뛸 수도 있습니다. 테스트 실행을 잊을 수도 있습니다. 당신의 템플릿을 무시한 PR 설명을 작성할 수도 있습니다. 매번 실행이 다릅니다.

Archon이 이것을 해결합니다. 개발 프로세스를 워크플로우로 인코딩하세요. 워크플로우는 단계, 검증 게이트, 아티팩트를 정의합니다. AI는 각 단계에서 지능을 채우지만, 구조는 결정론적이며 당신이 소유합니다.

- **반복 가능** - 매번 동일한 워크플로우, 동일한 순서. 계획, 구현, 검증, 리뷰, PR.
- **격리됨** - 모든 워크플로우 실행은 자체 git 워크트리를 가집니다. 충돌 없이 5개의 수정을 병렬로 실행하세요.
- **실행 후 잊기** - 워크플로우를 시작하고 다른 작업을 하세요. 리뷰 코멘트가 달린 완성된 PR로 돌아오세요.
- **조합 가능** - 결정론적 노드(bash 스크립트, 테스트, git 작업)와 AI 노드(계획, 코드 생성, 리뷰)를 혼합하세요. AI는 가치를 더하는 곳에서만 실행됩니다.
- **이식 가능** - `.archon/workflows/`에 워크플로우를 한 번 정의하고 리포지토리에 커밋하세요. CLI, Web UI, Slack, Telegram, 또는 GitHub에서 동일하게 작동합니다.

## 어떻게 생겼나요?

다음은 계획하고, 테스트가 통과할 때까지 루프에서 구현하고, 승인을 받은 다음 PR을 생성하는 Archon 워크플로우 예시입니다:

```yaml
# .archon/workflows/build-feature.yaml
nodes:
  - id: plan
    prompt: "코드베이스를 탐색하고 구현 계획을 만드세요"

  - id: implement
    depends_on: [plan]
    loop:                                      # AI 루프 - 완료될 때까지 반복
      prompt: "계획을 읽으세요. 다음 작업을 구현하세요. 검증을 실행하세요."
      until: ALL_TASKS_COMPLETE
      fresh_context: true                      # 각 반복마다 새로운 세션

  - id: run-tests
    depends_on: [implement]
    bash: "bun run validate"                   # 결정론적 - AI 없음

  - id: review
    depends_on: [run-tests]
    prompt: "계획에 대비하여 모든 변경 사항을 검토하세요. 문제를 수정하세요."

  - id: approve
    depends_on: [review]
    loop:                                      # 인간 승인 게이트
      prompt: "검토를 위해 변경 사항을 제시하세요. 피드백을 처리하세요."
      until: APPROVED
      interactive: true                        # 일시 중지하고 인간 입력을 기다림

  - id: create-pr
    depends_on: [approve]
    prompt: "변경 사항을 푸시하고 풀 리퀘스트를 생성하세요"
```

코딩 에이전트에게 원하는 것을 말하면, Archon이 나머지를 처리합니다:

```
사용자: settings 페이지에 다크 모드를 추가하기 위해 archon을 사용해줘

에이전트: 이를 위해 archon-idea-to-pr 워크플로우를 실행하겠습니다.
       → archon/task-dark-mode 브랜치에 격리된 워크트리 생성 중...
       → 계획 수립 중...
       → 구현 중 (작업 1/4)...
       → 구현 중 (작업 2/4)...
       → 테스트 실패 - 반복 중...
       → 2번 반복 후 테스트 통과
       → 코드 리뷰 완료 - 0개 이슈
       → PR 준비됨: https://github.com/you/project/pull/47
```

## 이전 버전

원래 Python 기반 Archon(작업 관리 + RAG)을 찾고 계신가요? [`archive/v1-task-management-rag`](https://github.com/coleam00/Archon/tree/archive/v1-task-management-rag) 브랜치에 완전히 보존되어 있습니다.

## 시작하기

> **대부분의 사용자는 [전체 설정](#전체-설정-5분)으로 시작해야 합니다** - 자격 증명을 안내하고, 프로젝트에 Archon 스킬을 설치하며, 웹 대시보드를 제공합니다.
>
> **이미 Claude Code를 가지고 있고 CLI만 원하시나요?** [빠른 설치](#빠른-설치-30초)로 이동하세요.

### 전체 설정 (5분)

리포지토리를 클론하고 안내된 설정 마법사를 사용하세요. 자격 증명, 플랫폼 통합을 구성하고 대상 프로젝트에 Archon 스킬을 복사합니다.

<details>
<summary><b>사전 요구 사항</b> - Bun, Claude Code, GitHub CLI</summary>

**Bun** - [bun.sh](https://bun.sh)

```bash
# macOS/Linux
curl -fsSL https://bun.sh/install | bash

# Windows (PowerShell)
irm bun.sh/install.ps1 | iex
```

**GitHub CLI** - [cli.github.com](https://cli.github.com/)

```bash
# macOS
brew install gh

# Windows (winget을 통해)
winget install GitHub.cli

# Linux (Debian/Ubuntu)
sudo apt install gh
```

**Claude Code** - [claude.ai/code](https://claude.ai/code)

```bash
# macOS/Linux/WSL
curl -fsSL https://claude.ai/install.sh | bash

# Windows (PowerShell)
irm https://claude.ai/install.ps1 | iex
```

</details>

```bash
git clone https://github.com/coleam00/Archon
cd Archon
bun install
claude
```

그런 다음 말하세요: **"Archon 설정해줘"**

설정 마법사가 모든 것을 안내합니다: CLI 설치, 인증, 플랫폼 선택, 대상 리포지토리에 Archon 스킬 복사.

### 빠른 설치 (30초)

이미 Claude Code가 설정되어 있나요? 독립 실행형 CLI 바이너리를 설치하고 마법사를 건너뛰세요.

**macOS / Linux**
```bash
curl -fsSL https://archon.diy/install | bash
```

**Windows (PowerShell)**
```powershell
irm https://archon.diy/install.ps1 | iex
```

**Homebrew**
```bash
brew install coleam00/archon/archon
```

> **컴파일된 바이너리는 `CLAUDE_BIN_PATH`가 필요합니다.** 빠른 설치 바이너리는
> Claude Code를 번들로 포함하지 않습니다. 별도로 설치한 다음 Archon이 그것을 가리키도록 하세요:
>
> ```bash
> # macOS / Linux / WSL
> curl -fsSL https://claude.ai/install.sh | bash
> export CLAUDE_BIN_PATH="$HOME/.local/bin/claude"
>
> # Windows (PowerShell)
> irm https://claude.ai/install.ps1 | iex
> $env:CLAUDE_BIN_PATH = "$env:USERPROFILE\.local\bin\claude.exe"
> ```
>
> 또는 `~/.archon/config.yaml`에서 `assistants.claude.claudeBinaryPath`를 설정하세요.
> Docker 이미지는 Claude Code가 사전 설치되어 있습니다. 자세한 내용은 [AI 어시스턴트 → 바이너리 경로 구성](https://archon.diy/docs/getting-started/ai-assistants/#binary-path-configuration-compiled-binaries-only)을 참조하세요.

### Archon 사용 시작

설정 경로 중 하나를 완료했으면, 프로젝트로 이동하여 작업을 시작하세요:

```bash
cd /path/to/your/project
claude
```

```
이슈 #42를 고치기 위해 archon을 사용해줘
```

```
어떤 archon 워크플로우가 있어? 각각을 언제 사용하면 되지?
```

코딩 에이전트가 워크플로우 선택, 브랜치 이름 지정, 워크트리 격리를 처리합니다. 프로젝트는 처음 사용될 때 자동으로 등록됩니다.

> **중요:** 항상 Archon 리포지토리가 아닌 대상 리포지토리에서 Claude Code를 실행하세요. 설정 마법사가 Archon 스킬을 프로젝트에 복사하여 거기서 작동합니다.

## Web UI

Archon에는 코딩 에이전트와 채팅하고, 워크플로우를 실행하며, 활동을 모니터링하는 웹 대시보드가 포함되어 있습니다. 바이너리 설치: `archon serve`를 실행하여 한 단계로 Web UI를 다운로드하고 시작하세요. 소스에서: 코딩 에이전트에게 Archon 리포지토리에서 프론트엔드를 실행하도록 요청하거나, 리포지토리 루트에서 직접 `bun run dev`를 실행하세요.

채팅 사이드바에서 "프로젝트" 옆의 **+**를 클릭하여 프로젝트를 등록하세요 - GitHub URL 또는 로컬 경로를 입력하세요. 그런 다음 대화를 시작하고, 워크플로우를 호출하고, 실시간으로 진행 상황을 지켜보세요.

**주요 페이지:**
- **채팅** - 실시간 스트리밍 및 도구 호출 시각화가 있는 대화 인터페이스
- **대시보드** - 실행 중인 워크플로우를 모니터링하는 미션 컨트롤, 프로젝트, 상태, 날짜별 필터링 가능한 기록
- **워크플로우 빌더** - 루프 노드가 있는 DAG 워크플로우를 만들기 위한 시각적 드래그 앤 드롭 편집기
- **워크플로우 실행** - 실행 중이거나 완료된 워크플로우의 단계별 진행 뷰

**모니터링 허브:** 사이드바는 **모든 플랫폼**의 대화를 보여줍니다 - 웹만이 아닙니다. CLI에서 시작된 워크플로우, Slack 또는 Telegram의 메시지, GitHub 이슈 상호작용 - 모든 것이 한 곳에 나타납니다.

전체 문서는 [Web UI 가이드](https://archon.diy/adapters/web/)를 참조하세요.

## 무엇을 자동화할 수 있나요?

Archon은 일반적인 개발 작업을 위한 워크플로우와 함께 제공됩니다:

| 워크플로우 | 설명 |
|----------|-------------|
| `archon-assist` | 일반 Q&A, 디버깅, 탐색 - 모든 도구를 갖춘 완전한 Claude Code 에이전트 |
| `archon-fix-github-issue` | 이슈 분류 → 조사/계획 → 구현 → 검증 → PR → 스마트 리뷰 → 자체 수정 |
| `archon-idea-to-pr` | 기능 아이디어 → 계획 → 구현 → 검증 → PR → 5개 병렬 리뷰 → 자체 수정 |
| `archon-plan-to-pr` | 기존 계획 실행 → 구현 → 검증 → PR → 리뷰 → 자체 수정 |
| `archon-issue-review-full` | GitHub 이슈를 위한 포괄적인 수정 + 전체 멀티 에이전트 리뷰 파이프라인 |
| `archon-smart-pr-review` | PR 복잡도 분류 → 대상 리뷰 에이전트 실행 → 결과 종합 |
| `archon-comprehensive-pr-review` | 자동 수정이 있는 멀티 에이전트 PR 리뷰 (5개 병렬 리뷰어) |
| `archon-create-issue` | 문제 분류 → 컨텍스트 수집 → 조사 → GitHub 이슈 생성 |
| `archon-validate-pr` | 메인 및 기능 브랜치 모두 테스트하는 철저한 PR 검증 |
| `archon-resolve-conflicts` | 병합 충돌 감지 → 양측 분석 → 해결 → 검증 → 커밋 |
| `archon-feature-development` | 계획에서 기능 구현 → 검증 → PR 생성 |
| `archon-architect` | 아키텍처 검토, 복잡성 감소, 코드베이스 건강 개선 |
| `archon-refactor-safely` | 타입 체크 훅과 동작 검증을 통한 안전한 리팩토링 |
| `archon-ralph-dag` | PRD 구현 루프 - 완료될 때까지 스토리를 반복 |
| `archon-remotion-generate` | AI로 Remotion 비디오 컴포지션 생성 또는 수정 |
| `archon-test-loop-dag` | 루프 노드 테스트 워크플로우 - 완료될 때까지 반복 카운터 |
| `archon-piv-loop` | 반복 간 인간 리뷰가 있는 안내된 계획-구현-검증 루프 |

Archon은 17개의 기본 워크플로우를 제공합니다 - `archon workflow list`를 실행하거나 원하는 것을 설명하면 라우터가 올바른 것을 선택합니다.

**또는 직접 정의하세요.** 기본 워크플로우는 좋은 시작점입니다 - `.archon/workflows/defaults/`에서 하나를 복사하고 커스터마이즈하세요. 워크플로우는 `.archon/workflows/`의 YAML 파일이고, 명령어는 `.archon/commands/`의 마크다운 파일입니다. 리포지토리의 동일한 이름의 파일이 번들된 기본값을 재정의합니다. 커밋하면 팀 전체가 동일한 프로세스를 실행합니다.

[워크플로우 작성](https://archon.diy/guides/authoring-workflows/) 및 [명령어 작성](https://archon.diy/guides/authoring-commands/)을 참조하세요.

## 플랫폼 추가

Web UI와 CLI는 바로 사용할 수 있습니다. 선택적으로 원격 접근을 위한 채팅 플랫폼을 연결하세요:

| 플랫폼 | 설정 시간 | 가이드 |
|----------|-----------|-------|
| **Telegram** | 5분 | [Telegram 가이드](https://archon.diy/adapters/telegram/) |
| **Slack** | 15분 | [Slack 가이드](https://archon.diy/adapters/slack/) |
| **GitHub 웹훅** | 15분 | [GitHub 가이드](https://archon.diy/adapters/github/) |
| **Discord** | 5분 | [Discord 가이드](https://archon.diy/adapters/community/discord/) |

## 아키텍처

```
┌─────────────────────────────────────────────────────────┐
│  플랫폼 어댑터 (Web UI, CLI, Telegram, Slack,           │
│                Discord, GitHub)                          │
└──────────────────────────┬──────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────┐
│                     오케스트레이터                       │
│          (메시지 라우팅 & 컨텍스트 관리)                │
└─────────────┬───────────────────────────┬───────────────┘
              │                           │
      ┌───────┴────────┐          ┌───────┴────────┐
      │                │          │                │
      ▼                ▼          ▼                ▼
┌───────────┐  ┌────────────┐  ┌──────────────────────────┐
│  명령어   │  │  워크플로우 │  │    AI 어시스턴트 클라이언트│
│  핸들러   │  │  실행기     │  │      (Claude / Codex)    │
│  (슬래시) │  │  (YAML)    │  │                          │
└───────────┘  └────────────┘  └──────────────────────────┘
      │              │                      │
      └──────────────┴──────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────┐
│              SQLite / PostgreSQL (7개 테이블)           │
│   코드베이스 • 대화 • 세션 • 워크플로우 실행            │
│    격리 환경 • 메시지 • 워크플로우 이벤트              │
└─────────────────────────────────────────────────────────┘
```

## 문서

전체 문서는 **[archon.diy](https://archon.diy)**에서 확인할 수 있습니다.

| 주제 | 설명 |
|-------|-------------|
| [시작하기](https://archon.diy/getting-started/overview/) | 설정 가이드 (Web UI 또는 CLI) |
| [Archon의 책](https://archon.diy/book/) | 10장 내러티브 튜토리얼 |
| [CLI 참조](https://archon.diy/reference/cli/) | 전체 CLI 참조 |
| [워크플로우 작성](https://archon.diy/guides/authoring-workflows/) | 커스텀 YAML 워크플로우 만들기 |
| [명령어 작성](https://archon.diy/guides/authoring-commands/) | 재사용 가능한 AI 명령어 만들기 |
| [구성](https://archon.diy/reference/configuration/) | 모든 설정 옵션, 환경 변수, YAML 설정 |
| [AI 어시스턴트](https://archon.diy/getting-started/ai-assistants/) | Claude 및 Codex 설정 세부 정보 |
| [배포](https://archon.diy/deployment/) | Docker, VPS, 프로덕션 설정 |
| [아키텍처](https://archon.diy/reference/architecture/) | 시스템 설계 및 내부 구조 |
| [문제 해결](https://archon.diy/reference/troubleshooting/) | 일반적인 문제 및 해결 방법 |

## 텔레메트리

Archon은 워크플로우가 시작될 때마다 단일 익명 이벤트 — `workflow_invoked` — 를 전송하여, 유지 관리자가 어떤 워크플로우가 실제로 사용되는지 파악하고 그에 따라 우선 순위를 정할 수 있습니다. **PII는 절대 없습니다.**

**수집되는 것:** 워크플로우 이름, 워크플로우 설명(YAML에서 작성한 것), 트리거한 플랫폼(`cli`, `web`, `slack` 등), Archon 버전, `~/.archon/telemetry-id`에 저장된 임의의 설치 UUID. 그 외에는 없습니다.

**수집되지 않는 것:** 코드, 프롬프트, 메시지, git 원격, 파일 경로, 사용자 이름, 토큰, AI 출력, 워크플로우 노드 세부 정보 — 아무것도 없습니다.

**옵트아웃:** 환경에서 다음 중 하나를 설정하세요:

```bash
ARCHON_TELEMETRY_DISABLED=1
DO_NOT_TRACK=1        # Astro, Bun, Prisma, Nuxt 등이 준수하는 사실상의 표준
```

`POSTHOG_API_KEY` 및 `POSTHOG_HOST`를 설정하여 PostHog를 자체 호스팅하거나 다른 프로젝트를 사용하세요.

## 기여하기

기여를 환영합니다! 작업할 항목은 공개 [이슈](https://github.com/coleam00/Archon/issues)를 확인하세요.

풀 리퀘스트를 제출하기 전에 [CONTRIBUTING.md](CONTRIBUTING.md)를 읽어주세요.

## 라이선스

[MIT](LICENSE)
