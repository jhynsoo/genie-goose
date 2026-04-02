# Genie Goose

[English](README.md) | **한국어**

Claude Code용 파이프라인 기반 개발 워크플로우 플러그인.

**brainstorm → architecture → design-intent → evaluation-criteria → implement → review** 6단계 파이프라인을 통해 모든 기능이 체계적인 설계, 설계 의도 문서화, 근거 기반 코드 리뷰를 거치도록 합니다.

## 설치

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

## 설정

설치 후 프로젝트에 conventions 파일을 생성합니다. 플러그인은 이 파일을 사용하여 코드 리뷰 평가 기준을 생성합니다.

```bash
mkdir -p .goose
cp ~/.claude/plugins/cache/*/genie-goose/*/plugin/.goose/conventions.template.yaml .goose/conventions.yaml
```

`.goose/conventions.yaml`을 팀의 코딩 표준에 맞게 수정하세요. 간단한 YAML 구조를 사용합니다:

```yaml
categories:
  - name: general
    rules:
      - id: GEN-001
        title: Function length limit
        stacks: [all]
        level: recommended
        description: >
          A single function should not exceed 30 lines.
```

`.goose-artifacts/`를 `.gitignore`에 추가하세요 — 플러그인이 브랜치별 산출물을 이 디렉토리에 생성합니다:

```bash
echo ".goose-artifacts/" >> .gitignore
```

## 사용법

### 전체 파이프라인

하나의 명령으로 전체 워크플로우를 실행합니다:

```
/genie-goose:goose 사용자 인증 시스템 구축
```

6단계를 순서대로 실행하며, 각 단계 사이에 사용자 입력을 기다립니다.

### 개별 단계

각 단계를 독립적으로 실행할 수도 있습니다:

| 명령 | 설명 |
|------|------|
| `/genie-goose:brainstorm <주제>` | 브레인스토밍 — 질문을 통해 요구사항을 파악하고 2-3가지 접근 방식 제안 |
| `/genie-goose:architecture` | 브레인스토밍 결과를 바탕으로 기술 아키텍처 설계 |
| `/genie-goose:design-intent` | 설계 결정, 트레이드오프, 의도적 제외 사항 문서화 |
| `/genie-goose:evaluation-criteria` | 관련 컨벤션을 추출하여 리뷰 평가 기준 수립 |
| `/genie-goose:implement` | 아키텍처와 설계 의도 문서를 기반으로 구현 |
| `/genie-goose:review` | 근거 기반 코드 리뷰 — 치명적 이슈 자동 수정 |

### 파이프라인 흐름

```
brainstorm ──→ design.md
                  │
architecture ──→ architecture.md
                  │
design-intent ─→ intent.md ──────────────┐
                  │                       │
evaluation    ─→ criteria.md ─────┐      │
 (conventions.yaml)               │      │
                                  ▼      ▼
implement     ←── architecture.md + intent.md
                                  │      │
review        ←── criteria.md + intent.md + git diff
```

모든 산출물은 `.goose-artifacts/{branch-name}/`에 저장됩니다.

## 리뷰 방식

리뷰 단계는 전용 subagent가 모든 변경 사항을 평가 기준에 대해 검사합니다. 모든 코멘트는 구체적인 기준을 인용해야 하며, 근거 없는 리뷰는 허용되지 않습니다.

**우선순위:**

| 등급 | 의미 | 후속 처리 |
|------|------|----------|
| `[p1]` | 반드시 수정 (버그, 기준 위반) | 자동 수정 루프 (최대 3회) |
| `[p2]` | 강력 권장 | 사용자에게 보고 |
| `[p3]` | 권장 | 사용자에게 보고 |
| `[p4]` | 사소한 개선 | 사용자에게 보고 |

## 로컬 개발

플러그인을 로컬에서 테스트하려면:

```bash
claude --plugin-dir ./plugin
```

## 라이선스

MIT
