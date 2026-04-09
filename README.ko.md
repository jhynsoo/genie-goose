# Genie Goose

[English](README.md) | **한국어**

Claude Code와 Codex용 조합형 개발 워크플로우 플러그인.

브레인스토밍, 아키텍처 설계, 설계 의도 문서화, 구현 계획, 평가 기준, 코드 리뷰, PR 생성, 검증된 완료까지 — 태스크 규모에 맞게 조합할 수 있는 스킬 세트를 제공합니다.

## 설치

### Claude Code

```bash
# marketplace 등록
claude plugin marketplace add jhynsoo/genie-goose

# 플러그인 설치
claude plugin install genie-goose@genie-goose
```

팀과 공유하려면 프로젝트 스코프로 설치:

```bash
claude plugin install genie-goose@genie-goose --scope project
```

### Codex

Codex 플러그인은 marketplace를 통해 설치합니다. 이 저장소에는 `./plugin`을 가리키는 repo-local marketplace 파일 `.agents/plugins/marketplace.json`이 포함되어 있으므로, 별도 복사 없이 Codex에서 바로 genie-goose를 테스트할 수 있습니다.

1. 이 저장소를 받은 뒤 Codex를 재시작해 repo-local marketplace를 다시 읽게 합니다.
2. Codex를 실행하고 `/plugins`로 플러그인 디렉터리를 엽니다.
3. `Genie Goose Local` marketplace를 선택합니다.
4. `genie-goose`를 설치합니다.
5. 새 스레드를 열고 `@genie-goose`로 플러그인 엔트리포인트를 호출합니다.
6. 라우터를 거치지 않고 특정 스킬만 바로 실행하고 싶다면 `$goose`, `$lamp`, `$implement`처럼 개별 스킬을 직접 호출합니다.
7. `@genie-goose`는 플러그인/라우터 엔트리포인트로 보고, 전체 9단계 파이프라인을 바로 실행하고 싶을 때는 `$goose`를 사용합니다.

### Codex 고급 설정 선택 사항

기본 Codex 사용 경로는 단일 에이전트 기준으로 동작하며, 추가 설정 없이 사용할 수 있습니다.

리뷰 보조용 커스텀 에이전트 템플릿도 함께 쓰고 싶다면 아래처럼 Codex 에이전트 디렉터리로 복사합니다. 프로젝트 범위는 `.codex/agents/`, 개인 범위는 `~/.codex/agents/`를 사용하면 됩니다.

```bash
mkdir -p .codex/agents
cp ./plugin/assets/codex-agents/*.toml .codex/agents/
```

이 템플릿은 선택 사항입니다. Codex는 사용자가 명시적으로 요청할 때만 subagent를 띄우므로, 파일을 복사해도 기본 `@genie-goose` 동작이 자동으로 바뀌지는 않습니다.

## 설정

설치 후 프로젝트에 conventions 파일을 생성합니다. 플러그인은 이 파일을 사용하여 코드 리뷰 평가 기준을 생성합니다.

이 저장소에서 작업 중이거나 repo-local Codex marketplace를 쓰는 경우:

```bash
mkdir -p .goose
cp ./plugin/.goose/conventions.template.yaml .goose/conventions.yaml
```

Claude marketplace로 설치한 경우:

```bash
cp ~/.claude/plugins/cache/*/genie-goose/*/plugin/.goose/conventions.template.yaml .goose/conventions.yaml
```

`.goose/conventions.yaml`을 팀의 코딩 표준에 맞게 수정하세요. Flat YAML 구조를 사용합니다:

```yaml
general:
  - id: GEN-001
    rule: >
      A single function should not exceed 30 lines.
    stacks: [all]
```

선택적으로 아키텍처 결정 기록 파일을 생성할 수 있습니다:

이 저장소에서 작업 중이거나 repo-local Codex marketplace를 쓰는 경우:

```bash
cp ./plugin/.goose/decisions.template.yaml .goose/decisions.yaml
```

Claude marketplace로 설치한 경우:

```bash
cp ~/.claude/plugins/cache/*/genie-goose/*/plugin/.goose/decisions.template.yaml .goose/decisions.yaml
```

`.goose-artifacts/`를 `.gitignore`에 추가하세요 — 플러그인이 브랜치별 산출물을 이 디렉토리에 생성합니다:

```bash
echo ".goose-artifacts/" >> .gitignore
```

## 사용법

genie-goose를 사용하는 세 가지 방법이 있습니다:

### 1. 전체 파이프라인

하나의 명령으로 전체 9단계 워크플로우를 실행합니다:

```text
Claude Code: /genie-goose:goose 사용자 인증 시스템 구축
Codex 라우터 엔트리포인트: @genie-goose 사용자 인증 시스템 구축
Codex 전체 파이프라인 스킬: $goose 사용자 인증 시스템 구축
```

**rub → architecture → intent → write-plan → criteria → implement → honk → pr (선택) → finish** 순서로 실행하며, 각 단계 사이에 사용자 입력을 기다립니다. 중대형 기능 개발에 권장됩니다.

### 2. 추천 경로

`lamp` 스킬 라우터는 태스크 규모를 분류하고 맞춤 경로를 추천합니다:

| 태스크 규모 | 추천 경로 |
|------------|----------|
| 대형 기능 | `goose` (전체 9단계 파이프라인), 또는 `rub → architecture → intent → write-plan → criteria → implement → honk → finish` |
| 중형 태스크 | `rub → write-plan → implement → honk → finish` |
| 소형 태스크 | `implement → finish` (또는 스킬 불필요) |
| 디버그 | `debug` |
| 문서 관리 | `update-docs` |

하고 싶은 작업을 설명하면 lamp이 적절한 경로를 제안합니다. 수락, 조정, 건너뛰기 모두 가능합니다.

Claude Code에서는 `lamp`가 번들된 SessionStart 훅을 통해 세션 시작 시 자동 주입됩니다.

Codex에서는 플러그인 번들 내부의 훅이 자동 로드되지 않습니다. 기본 사용 경로는 `@genie-goose`로 시작해 플러그인 라우터에 태스크를 맡기거나, `$lamp`, `$goose`, `$implement`처럼 특정 번들 스킬을 직접 호출하는 방식입니다.

Codex에서도 세션 시작 시 자동 라우팅을 원하면 기존 `plugin/hooks/session-start` 스크립트를 `~/.codex/hooks.json` 또는 `<repo>/.codex/hooks.json`에 연결해야 합니다. Codex 공식 문서는 훅을 플러그인 매니페스트가 아니라 Codex 설정 레이어 옆에서 읽는다고 안내합니다.

### 3. 직접 호출

각 스킬을 독립적으로 실행할 수 있습니다:

| 명령 | 설명 |
|------|------|
| `Claude: /genie-goose:goose <주제>` `Codex: $goose <주제>` | 전체 9단계 파이프라인 프리셋 |
| `Codex: @genie-goose <주제>` | 플러그인/라우터 엔트리포인트. 요청을 분류한 뒤 다음 스킬을 고릅니다. |
| `Claude: /genie-goose:rub <주제>` `Codex: $rub <주제>` | 브레인스토밍 — 질문을 통해 요구사항을 파악하고 2-3가지 접근 방식 제안 |
| `Claude: /genie-goose:architecture` `Codex: $architecture` | 브레인스토밍 결과를 바탕으로 기술 아키텍처 설계 |
| `Claude: /genie-goose:intent` `Codex: $intent` | 설계 의도 문서화 + 컨벤션/결정 충돌 감지 및 변경 제안 |
| `Claude: /genie-goose:write-plan` `Codex: $write-plan` | 마이크로 태스크와 테스트 코드가 포함된 상세 구현 계획 작성 |
| `Claude: /genie-goose:criteria` `Codex: $criteria` | 관련 컨벤션/결정을 추출하여 리뷰 평가 기준 수립 |
| `Claude: /genie-goose:implement` `Codex: $implement` | 계획 체크리스트를 단계별로 실행 |
| `Claude: /genie-goose:honk` `Codex: $honk` | 근거 기반 코드 리뷰 — ACCEPT/REJECT 판정 |
| `Claude: /genie-goose:receive-review` `Codex: $receive-review` | 리뷰 피드백 처리 — 수락 전 검증, 근거 있는 반론 |
| `Claude: /genie-goose:pr` `Codex: $pr` | 파이프라인 산출물로 PR body 생성 |
| `Claude: /genie-goose:finish` `Codex: $finish` | 워크플로우 완료 — 검증, 병합/PR/유지/폐기 |
| `Claude: /genie-goose:debug` `Codex: $debug` | 체계적 디버깅 — 재현, 격리, 원인 증명, 수정 |
| `Claude: /genie-goose:update-docs` `Codex: $update-docs` | conventions.yaml과 decisions.yaml 관리 |
| `Claude: /genie-goose:polish` `Codex: $polish` | 검증 게이트 — 근거 없이 완료 주장 불가 |

선행 산출물이 없으면 경고만 표시되며 차단되지 않습니다 — 축소된 컨텍스트로 진행할 수 있습니다.

### 워크플로우 다이어그램

```
rub ───────────→ design.md
                    │
architecture ───→ architecture.md
                    │
intent ─────────→ intent.md ──────────────────┐
 (conventions.yaml    │                       │
  decisions.yaml)     │                       │
                      │                       │
write-plan ─────→ plan.md                     │
                    │                         │
criteria ───────→ criteria.md ────────┐      │
 (conventions.yaml    │               │      │
  decisions.yaml)     │               │      │
                      │               ▼      ▼
implement       ←── plan.md + intent.md
                                      │      │
honk            ←── criteria.md + intent.md + git diff
                    │
                    → review-report.md
                                      │
receive-review  ←── review-report.md (선택, 엄격한 리뷰 처리용)
                                      │
pr (선택)       ←── intent.md + review-report.md + git diff
                    │
                    → pr-body.md
                                      │
finish          ←── review-report.md (선택) + pr-body.md (선택)
                    │
                    → merge / PR / keep / discard

debug           ←── (독립 실행, 선행 조건 없음)
                    │
                    → debug-report.md
```

모든 산출물은 `.goose-artifacts/{branch-name}/`에 저장됩니다.

## 가드 레일

두 스킬은 모든 워크플로우에 적용됩니다 — 전체 파이프라인, 추천 경로, 직접 호출 모두:

- **polish** — 완료 주장 전 근거 기반 검증. 5단계 게이트: IDENTIFY → RUN → READ → VERIFY → CLAIM.
- **finish** — 병합/PR/유지/폐기 옵션이 포함된 구조화된 마무리. review-report.md 없이도 동작합니다.

## 리뷰 방식

리뷰 단계(honk)는 모든 변경 사항을 평가 기준에 대해 검사합니다. 모든 코멘트는 구체적인 기준을 인용해야 하며, 근거 없는 리뷰는 허용되지 않습니다. Codex 기본 경로에서는 현재 스레드에서 처리하고, 커스텀 에이전트 템플릿은 명시적 위임이 필요할 때만 선택적으로 사용합니다.

**우선순위:**

| 등급 | 의미 | 후속 처리 |
|------|------|----------|
| `[p1]` | 반드시 수정 (버그, 기준 위반) | 사용자 판정 → 수정 |
| `[p2]` | 강력 권장 | 사용자 판정 |
| `[p3]` | 권장 | 사용자 판정 |
| `[p4]` | 사소한 개선 | 사용자 판정 |

각 코멘트는 사용자로부터 **ACCEPT** (수정) 또는 **REJECT** (건너뛰기) 판정을 받습니다. ACCEPT된 항목은 자동으로 수정 및 검증됩니다.

## 로컬 개발

Claude Code에서 로컬 테스트:

```bash
claude --plugin-dir ./plugin
```

Codex에서 로컬 테스트:

1. `.agents/plugins/marketplace.json`을 다시 읽도록 Codex를 재시작합니다.
2. `/plugins`를 엽니다.
3. `Genie Goose Local`에서 `genie-goose`를 설치하거나 재설치합니다.
4. 새 스레드에서 `@genie-goose`를 플러그인/라우터 엔트리포인트로 사용합니다.
5. 라우터를 우회하고 싶을 때만 특정 `$skill`을 사용합니다. 예: 전체 파이프라인은 `$goose`.

## 라이선스

MIT
