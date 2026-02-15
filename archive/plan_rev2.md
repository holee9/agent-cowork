> **ARCHIVED** -- This document has been superseded by [PROTOCOL.md](../PROTOCOL.md). Retained for historical reference.

# TITLE Protocol v3.0 AI -- Multi-Agent System (Claude Code x GPT-5.3-Codex 기반)

작성일: 2026‑02‑14
작성자: System Owner
참여 모델: Claude Code (Architect), GPT‑5.3‑Codex (Builder/Red Team)[file:1][web:23]

---

## 1. Philosophy

- **목표**
  - Claude Code와 GPT‑5.3‑Codex 두 모델만으로, 설계–레드팀–합의–실행이 닫힌 루프로 돌아가는 개발 프로토콜을 정의한다.[file:1][web:24]
  - 동일한 프로토콜을 다양한 프로젝트(보안 AI 비서, Internal Mesh 모니터링 등)에 재사용 가능하게 한다.[file:1]

- **핵심 원칙**
  - Security First: 보안·안정성을 기능과 동일한 1급 요구사항으로 취급한다.[file:1]
  - Stateless 지향: 가능한 한 상태는 명시적인 파일(BLUEPRINT, STATE.yaml, Git 이력)에만 남기고, 에이전트/프로세스는 언제든 재시작 가능해야 한다.[file:1]
  - Clear Role Separation: Claude와 Codex가 서로의 역할을 침범하지 않도록, Phase와 아티팩트 단위로 책임을 고정한다.[file:1][web:23]

---

## 2. Roles

### 2.1 Claude Code – Architect / Coordinator / Narrator

- 책임
  - 요구사항·위험·제약을 구조화하고, 시스템의 **논리 구조와 정책**을 설계한다.[file:1][web:23]
  - `.aisync/BLUEPRINT.json`, 핵심 인터페이스 스펙(TypeScript/JSON Schema)을 작성·갱신한다.[file:1]
  - 레드팀 결과(`AUDIT_LOG.md`)를 해석하여 어떤 리스크를 수용/보류/반려할지 결정하고, 최종 설계를 정리한다.[file:1][web:23]
  - README, 아키텍처 문서, 운영 가이드, CHANGELOG 등 **내러티브와 문맥**을 관리한다.[file:1][web:25]

- 하지 말아야 할 일
  - 세부 구현 코드(함수 레벨)까지 직접 작성·수정하려고 하지 않는다.
  - 레드팀 Phase(2)에서 Codex의 공격을 방어/반박하려 하지 않고, Synthesis Phase(3)에서만 판단한다.[file:1]

### 2.2 GPT‑5.3‑Codex – Builder / Red Team / Executor

- 책임
  - Antithesis Phase에서 `BLUEPRINT.json`과 스펙을 기반으로 **레드팀 리뷰**를 수행하고, `.aisync/AUDIT_LOG.md`를 작성한다.[file:1][web:13]
  - Execution Phase에서 `src/` 코드 구현, 테스트 코드 작성, CI/배포 스크립트를 포함한 **실행 가능한 시스템**을 만든다.[file:1][web:27]
  - 메모리/동시성/보안/하드웨어/AI 악용 시나리오에 대한 PoC, 테스트, 벤치마크를 설계·실행한다.[web:13][web:30]

- 하지 말아야 할 일
  - BLUEPRINT 수준의 구조적 설계를 직접 변경하지 않는다.(설계 변경 제안은 이슈/코멘트로만 남긴다.)[file:1][web:26]
  - 최종 정책·우선순위를 스스로 결정하지 않고, 항상 Claude 또는 System Owner의 결정을 기다린다.[file:1]

---

## 3. Execution Loop

프로토콜은 네 개의 Phase를 순환하면서 진행된다: Thesis → Antithesis → Synthesis → Execution.[file:1][web:14]

### 3.1 Phase 1 – Thesis (Claude 전담)

- 입력
  - 사용자 요구사항, 제약 조건, 기존 시스템 정보, 과거 Synthesis 결정.[file:1]
- Claude의 작업
  - 요구사항·위험·제약을 구조화하고, 시스템의 목표·범위·가드레일을 정의한다.[file:1][web:23]
  - `.aisync/BLUEPRINT.json` 작성/갱신:
    - `project_name`, `core_principles`, `constraints`, `io_contracts`, `security_policies`, `non_functional_requirements` 등.[file:1]
  - `specs/01_interface_def.ts` 또는 동등한 인터페이스 정의 파일을 작성해 IO 계약을 명시한다.[file:1]
- Codex의 역할
  - 이 Phase에서는 설계에 직접 개입하지 않고, 나중 레드팀에서 활용할 관찰 포인트만 메모(선택 사항).[web:23]

- 종료 조건
  - BLUEPRINT의 필수 필드가 채워지고, System Owner가 Thesis를 승인한다.[file:1]

### 3.2 Phase 2 – Antithesis (Codex 전담 레드팀)

- 입력
  - 확정된 `BLUEPRINT.json`, `specs/01_interface_def.ts`, 관련 문서.[file:1]
- Codex의 작업
  - `.aisync/AUDIT_LOG.md`에 레드팀 리뷰를 작성:
    - 메모리/리소스 누수, 동시성/레이스, 하드웨어·인프라 수명, 보안·AI 악용 시나리오 등 체크리스트 기반 평가.[file:1][web:13]
    - 각 리스크에 대해 근거와 잠재 PoC 또는 테스트 아이디어를 제시.
    - Critical Issues / Blockers, Recommendations 섹션을 채운다.[file:1]
  - 필요시 PoC 코드나 테스트 스크립트를 `tests/` 또는 별도 디렉토리에 추가한다.[web:13]

- Claude의 역할
  - Codex가 작성한 AUDIT_LOG를 읽을 준비만 하고, 이 Phase에서는 방어/정당화를 하지 않는다.[file:1]

- 종료 조건
  - 정의된 체크리스트 항목이 충분히 채워지고, Blocker가 명시된 상태에서 System Owner 또는 .ai_sync가 Phase 전환을 허용한다.[file:1]

### 3.3 Phase 3 – Synthesis (Claude 주도, Codex 보조)

- 입력
  - `BLUEPRINT.json`, `AUDIT_LOG.md`, 레드팀 PoC/테스트, 기존 코드/문서.[file:1]
- Claude의 작업
  - 레드팀 결과를 해석하고, 각 Blocker·리스크를 **수용/보류/반려**로 분류한다.[file:1]
  - 그 결정 이유를 BLUEPRINT 또는 별도 설계 문서에 기록한다.[file:1][web:23]
  - 필요 시 아키텍처·인터페이스·정책을 수정하고, `BLUEPRINT.json`과 `specs/`를 업데이트한다.[file:1]

- Codex의 작업
  - Claude가 제안한 설계 변경에 대해:
    - 실행 난이도, 마이그레이션 리스크, 테스트 전략을 평가한다.[web:23][web:29]
    - 복잡한 변경에 대해서는 단계별 마이그레이션 계획/스크립트 초안을 제안한다.

- 종료 조건
  - 모든 Blocker가:
    - 해결되었거나,
    - 보류/반려되었고 그 이유가 문서화되었으며,
  - System Owner가 최종 설계를 승인했다.[file:1]

### 3.4 Phase 4 – Execution (Codex 주도 구현/테스트)

- 입력
  - 합의된 BLUEPRINT, 인터페이스 스펙, 설계 문서.[file:1]
- Codex의 작업
  - `src/` 코드와 `tests/`를 구현/수정하고, CI/배포 스크립트를 작성·갱신한다.[file:1][web:27]
  - 단위/통합/부하/보안 테스트를 실행하고, 실패 시 자가 수정 루프를 돌며 안정화한다.[web:22][web:30]

- Claude의 작업
  - 구현이 BLUEPRINT와 설계 의도, 보안 정책, IO 계약을 만족하는지 **고수준 리뷰**를 수행한다.[file:1][web:23]
  - README, 운영 가이드, CHANGELOG 등을 업데이트하여 결정과 변경사항을 기록한다.[file:1]

- 종료 조건
  - 정의된 테스트/품질 기준을 만족하고, System Owner가 릴리즈 또는 다음 기능으로의 전환을 승인한다.[file:1]

---

## 4. .ai_sync Manager & STATE.yaml

### 4.1 STATE.yaml 스키마 개요

```yaml
phase: THESIS | ANTITHESIS | SYNTHESIS | EXECUTION
owner_model: Claude | Codex
blocking_model: Claude | Codex | null
status: OK | BLOCKED | CONSENSUS_BLOCKED

last_artifacts:
  blueprint_sha: string
  audit_log_sha: string
  src_sha: string

blockers:
  - id: string
    title: string
    owner: Claude | Codex | SystemOwner
    status: OPEN | ACCEPTED | DEFERRED | REJECTED
    reason: string
    updated_at: timestamp
```

- **`phase`**: 현재 프로토콜 단계.[file:1]
- **`owner_model`**: 현재 Phase를 주도하는 모델(Thesis/Synthesis는 Claude, Antithesis/Execution은 Codex).[file:1]
- **`blocking_model`**: 합의를 막고 있는 쪽이 있다면 명시 (예: Synthesis에서 Claude가 수용을 거부하는 경우).[file:1]
- **`status`**: 전체 진행 상태 (정상, Blocker 때문에 중단, 모델 간 합의 실패).[file:1]
- **`last_artifacts`**: Git 기준 마지막 아티팩트 상태를 추적한다.
- **`blockers`**: 주요 이슈의 상태와 책임자를 기록한다.

### 4.2 .ai_sync Manager 역할

#### Phase 전환 규칙 강제

- **THESIS → ANTITHESIS**: Claude가 BLUEPRINT 필수 필드를 채우고 Owner가 승인했을 때, phase=ANTITHESIS, owner_model=Codex로 전환.[file:1]
- **ANTITHESIS → SYNTHESIS**: Codex가 AUDIT_LOG를 채우고 최소 체크리스트 기준을 충족하면 phase=SYNTHESIS, owner_model=Claude.[file:1]
- **SYNTHESIS → EXECUTION**: Blocker 상태가 정리되면 phase=EXECUTION, owner_model=Codex.[file:1]
- **EXECUTION → THESIS**: 구현·테스트·배포 완료 후 다음 사이클로.[file:1]

#### 합의 실패 처리

충돌 시:

- status=CONSENSUS_BLOCKED, blocking_model 및 reason 기록.[file:1]
- System Owner에게 알림을 보내 인간 개입을 요청.

#### 인터페이스

- `ai-sync status`: STATE.yaml 요약 표시.
- `ai-sync next`: 전환 가능 여부 체크 후 Phase 전환 또는 이유 출력.
- `ai-sync check`: 필수 파일(BLUEPRINT, AUDIT_LOG, specs 등) 존재·형식 검사.[file:1]

---

## 5. Templates (요약)

### 5.1 .aisync/BLUEPRINT.json (Claude 담당)

```json
{
  "project_name": "ProjectName",
  "core_principles": [
    "Security First",
    "Stateless where possible"
  ],
  "constraints": {
    "network": "Internal Mesh only, no public exposure"
  },
  "tech_stack": {
    "language": "Python 3.12",
    "framework": "FastAPI"
  },
  "io_contracts": { },
  "security_policies": { },
  "non_functional_requirements": { }
}
```

Claude가 작성·갱신, Codex는 참조만 한다.[file:1]

### 5.2 .aisync/AUDIT_LOG.md (Codex 담당)

```text
# Technical Audit Report – TITLE Protocol v3.0 AI

- Reviewer: GPT-5.3-Codex
- Target Spec: specs/01_interface_def.ts

## Critical Issues / Blockers

1. [ID] Title
   - Area: Memory | Concurrency | Hardware | Security | AI Misuse
   - Description:
   - Evidence / PoC Idea:
   - Impact:
   - Suggested Mitigation:

## Recommendations

- ...
```

Codex가 레드팀 체크리스트 기반으로 작성, Claude는 Synthesis에서 해석·결정.[file:1][web:13]

### 5.3 Artifact–모델 매핑

| Artifact | 주 담당 모델 | 보조 모델 역할 |
|---|---|---|
| .aisync/BLUEPRINT.json | Claude | Codex: 레드팀 관찰 포인트 메모 |
| .aisync/AUDIT_LOG.md | Codex | Claude: Blocker 처리·설계 수정에 활용 |
| specs/01_interface_def.ts | Claude | Codex: 구현/테스트 관점 피드백 |
| src/** 코드 및 테스트 | Codex | Claude: 구조·계약·보안 정책 준수 여부 리뷰 |
| STATE.yaml | .ai_sync | 두 모델 모두 읽기·갱신, Owner 최종 승인 |
| docs/**, README, CHANGELOG | Claude | Codex: 예제 코드·CLI 사용 예시 제공 |

[file:1][web:24]

---

## Reference

### GPT‑5.3‑Codex 관련

- **GPT‑5.3‑Codex System Card**
  - OpenAI 공식 시스템 카드로, 코딩/도구 사용/취약점 탐지·테스트 생성 등에서의 강점과 한계를 정리한 문서.

- **GPT‑5.3‑Codex 출시 및 성능 정리 기사**
  - GPT‑5.3‑Codex를 Anthropic 계열 모델과 비교하며 "일반 코딩·실행·툴 사용"에 강한 포지셔닝을 설명.

- **GPT‑5.3‑Codex 보안/취약점 탐지 관련 평가**
  - 실무 보안 팀이 Codex의 취약점 탐지·PoC 작성 능력, 세션 스톨 이슈 등을 분석한 리포트.

- **GPT‑5.3‑Codex에 대한 실사용 후기/토론**
  - Hacker News 등에서 Codex를 "고수준 설계보다는 구체적인 구현/테스트/익스플로잇에 강하다"고 평가하는 스레드.

### Claude Code 및 멀티에이전트 아키텍처

- **Context Engineering for Multi‑Agent LLM Code Assistants (논문)**
  - 멀티에이전트 코딩 시스템에서 Planner(설계/조정)와 Executor(구현) 분리의 효과를 실험적으로 분석.

- **Multi‑Agent Systems: Architecture + Use Cases**
  - 멀티에이전트 시스템에서 역할 분리(Planner vs Worker, Coordinator vs Executor)의 일반적인 베스트 프랙티스를 설명.

- **Designing a State‑of‑the‑Art Multi‑Agent System**
  - 대규모 멀티에이전트 시스템에서 상태 관리, 역할 분리, 실패 복구 전략 등을 다루는 아키텍처 가이드.

- **Claude Code Sub‑agents: The 90% Performance Gain**
  - Claude Code를 중심으로 한 "설계 에이전트 + 서브 에이전트(실행)" 구조가 단일 에이전트 대비 성능 개선을 보였다는 사례.

- **When to use multi‑agent systems (Claude 공식 블로그)**
  - 멀티에이전트가 효과적인 경우, 에이전트별 책임 분할 전략 등을 설명한 글.

- **커뮤니티 멀티에이전트 아키텍처 사례(Claude Code 중심)**
  - 간단한 multi‑agent 구조로 컨텍스트 품질과 성공률을 높이는 패턴, 역할 분리 예시.

- **Claude Code 기반 멀티에이전트 오케스트레이션 예제**
  - Claude를 중심 Planner/Coordinator로 두고, 실행 에이전트들을 붙인 GitHub Gist.

### 보안·레드팀·계획/실행 패턴

- **AI Red Teaming 및 보안 체크리스트**
  - AI 시스템을 대상으로 한 레드팀 방법론과 체크리스트(위협 모델, PoC, 취약점 관리 등).

- **Architecting Resilient LLM Agents (Resilient Agent 설계 논문)**
  - 안전한 Plan–Act–Observe 루프, 권한 분리, 실패·공격에 강한 구조를 설계하는 방법.

### 사용자 계획서/구성 자체

- **사용자 제공 plan.md**
  - Protocol v3.0 AI, Thesis–Antithesis–Synthesis–Execution 루프, .aisync/BLUEPRINT.json, .aisync/AUDIT_LOG.md, STATE.yaml, specs/01_interface_def.ts, src/ 등 파일 구조와 기본 역할 정의.

---

위 레퍼런스들을 바탕으로, Claude를 Architect/Coordinator로, Codex를 Red Team/Executor로 두는 것이 현재 공개된 성능 특성과 멀티에이전트 베스트 프랙티스에 가장 잘 부합한다고 판단하고 역할 분리를 개정했습니다.
