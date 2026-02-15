# TITLE Protocol v3.0 AI

**Multi-Agent Development System**

[![Protocol](https://img.shields.io/badge/Protocol-TITLE%20v3.0-blue)]()
[![Architect](https://img.shields.io/badge/Architect-Claude%20Code-blueviolet)]()
[![Builder](https://img.shields.io/badge/Builder-GPT--5.3--Codex-green)]()
[![Language](https://img.shields.io/badge/Language-English%20First-lightgrey)]()
[![License](https://img.shields.io/badge/License-Proprietary-red)]()

---

## Overview

TITLE Protocol v3.0 AI is a multi-agent development protocol that orchestrates two frontier AI models -- **Claude Code** and **GPT-5.3-Codex** -- in a closed-loop workflow. The protocol defines clear role separation, phase-gated execution, and artifact-based state management to deliver secure, high-quality software through adversarial collaboration.

The system treats security and stability as first-class requirements, enforces stateless design through explicit artifact tracking, and prevents role overlap by binding responsibilities to specific phases and artifacts.

### Core Principles

- **Security First** -- Security and stability are treated as equal priorities alongside functionality.
- **Stateless Design** -- All state is persisted in explicit files (BLUEPRINT, STATE.yaml, Git history). Agents can be restarted at any time without loss.
- **Clear Role Separation** -- Claude Code and GPT-5.3-Codex responsibilities are fixed per phase and artifact, with no cross-boundary interference.

---

## Architecture

The protocol operates on a dual-agent model where each AI has distinct, non-overlapping responsibilities.

### Claude Code -- Architect / Coordinator / Narrator

| Responsibility | Description |
|---|---|
| System Design | Structures requirements, risks, and constraints into a coherent logical architecture |
| Blueprint Authorship | Creates and maintains `.aisync/BLUEPRINT.json` and interface specifications |
| Synthesis Decisions | Interprets red-team findings and decides to accept, defer, or reject each risk |
| Documentation | Maintains README, architecture docs, operation guides, and CHANGELOG |

Claude Code does **not** write implementation-level code or defend against red-team findings during the Antithesis phase.

### GPT-5.3-Codex -- Builder / Red Team / Executor

| Responsibility | Description |
|---|---|
| Red Team Review | Audits the blueprint for memory leaks, concurrency issues, security flaws, and AI misuse vectors |
| Implementation | Writes production code in `src/`, test suites in `tests/`, and CI/deployment scripts |
| PoC & Benchmarks | Designs and executes proof-of-concept exploits, load tests, and security benchmarks |
| Migration Planning | Evaluates implementation difficulty and proposes step-by-step migration plans for complex changes |

GPT-5.3-Codex does **not** modify architectural decisions directly. Design change proposals are submitted as issues or comments for Claude Code to resolve.

---

## Execution Loop

The protocol follows a four-phase dialectical cycle that repeats for each feature or iteration.

```
 +---> [ 1. Thesis ] ---> [ 2. Antithesis ] ---> [ 3. Synthesis ] ---> [ 4. Execution ] ---+
 |                                                                                          |
 +------------------------------------------------------------------------------------------+
```

### Phase 1 -- Thesis (Claude Code)

Claude Code analyzes requirements, risks, and constraints. It produces or updates the `BLUEPRINT.json` with project principles, IO contracts, security policies, and non-functional requirements. Interface specifications are written in `specs/`. The phase ends when the blueprint passes completeness checks and the System Owner approves.

### Phase 2 -- Antithesis (GPT-5.3-Codex)

GPT-5.3-Codex performs a red-team review against the finalized blueprint and specifications. It writes the `AUDIT_LOG.md` covering memory/resource leaks, concurrency races, hardware lifecycle risks, security vulnerabilities, and AI misuse scenarios. Each finding includes evidence, impact assessment, and suggested mitigations. Claude Code does not intervene during this phase.

### Phase 3 -- Synthesis (Claude Code leads, GPT-5.3-Codex assists)

Claude Code reviews every finding in the audit log and classifies each blocker as **Accepted**, **Deferred**, or **Rejected**, with documented reasoning. Architecture, interfaces, and policies are updated as needed. GPT-5.3-Codex evaluates the feasibility and migration risks of proposed changes. The phase ends when all blockers are resolved or documented and the System Owner approves the final design.

### Phase 4 -- Execution (GPT-5.3-Codex leads, Claude Code reviews)

GPT-5.3-Codex implements the agreed design in `src/` and `tests/`, including CI and deployment scripts. It runs unit, integration, load, and security tests with self-correction loops until stabilized. Claude Code performs high-level review against the blueprint, security policies, and IO contracts. Documentation is updated to reflect all decisions and changes.

---

## Project Structure

```
agent-cowork/
├── .aisync/                    # Protocol state and artifact directory
│   ├── BLUEPRINT.json          # System design document (Claude Code owns)
│   ├── AUDIT_LOG.md            # Red-team findings (GPT-5.3-Codex owns)
│   └── STATE.yaml              # Phase tracking and blocker status
├── specs/                      # Interface and contract definitions
│   └── 01_interface_def.ts     # IO contract specifications
├── src/                        # Production source code (GPT-5.3-Codex owns)
├── tests/                      # Test suites and PoC scripts
├── docs/                       # Documentation (Claude Code owns)
│   └── ko/                     # Korean-language documentation (optional)
├── plan_rev2.md                # Full protocol specification
├── LANGUAGE_POLICY.md          # Language usage policy
└── README.md                   # This file
```

---

## Artifacts

| Artifact | Primary Owner | Secondary Role | Format |
|---|---|---|---|
| `.aisync/BLUEPRINT.json` | Claude Code | Codex references for review | JSON |
| `.aisync/AUDIT_LOG.md` | GPT-5.3-Codex | Claude uses findings in Synthesis | Markdown |
| `.aisync/STATE.yaml` | .ai_sync Manager | Both models read/update; Owner approves | YAML |
| `specs/**` | Claude Code | Codex provides implementation feedback | TypeScript / JSON Schema |
| `src/**` and `tests/**` | GPT-5.3-Codex | Claude reviews against blueprint | Implementation language |
| `docs/**`, README, CHANGELOG | Claude Code | Codex provides code examples and CLI usage | Markdown |

---

## .ai_sync Manager

The `.ai_sync` Manager is a state management layer that enforces phase transition rules, tracks artifact integrity, and handles consensus failures between agents.

### STATE.yaml

The `STATE.yaml` file tracks:

- **phase** -- Current protocol phase (THESIS, ANTITHESIS, SYNTHESIS, EXECUTION)
- **owner_model** -- The model leading the current phase
- **blocking_model** -- The model blocking consensus, if any
- **status** -- Overall progress (OK, BLOCKED, CONSENSUS_BLOCKED)
- **last_artifacts** -- SHA hashes of the latest blueprint, audit log, and source
- **blockers** -- List of open issues with ownership, status, and resolution reasoning

### Phase Transition Rules

| Transition | Condition |
|---|---|
| Thesis to Antithesis | Blueprint required fields complete; Owner approved |
| Antithesis to Synthesis | Audit log checklist met; blockers documented |
| Synthesis to Execution | All blockers resolved, deferred, or rejected with rationale |
| Execution to Thesis | Tests pass; quality criteria met; Owner approves next cycle |

### CLI Interface

| Command | Description |
|---|---|
| `ai-sync status` | Display a summary of the current STATE.yaml |
| `ai-sync next` | Check transition eligibility and advance the phase, or explain why not |
| `ai-sync check` | Validate that required files (BLUEPRINT, AUDIT_LOG, specs) exist and conform to expected formats |

When consensus between agents fails, the manager sets `status: CONSENSUS_BLOCKED` and alerts the System Owner for human intervention.

---

## Language Policy

The project follows an **English-first** policy for all internal artifacts. This decision is grounded in token efficiency, model performance characteristics, and long-term maintainability across toolchains.

| Scope | Language | Notes |
|---|---|---|
| Code, variables, comments | English | All source code and inline comments |
| BLUEPRINT.json | English | All keys and values |
| AUDIT_LOG.md | English | Full audit content; optional Korean summary section at end |
| Specs and interface files | English | Type names, field names, JSDoc/TSDoc |
| STATE.yaml | English | All keys and values; Korean in YAML comments only |
| CLI output (ai-sync) | English | Future: `--locale=ko` flag planned |
| Documentation | English | Korean docs maintained separately under `docs/ko/` |
| Owner instructions | Korean or English | System Owner may use either language freely |
| Agent responses | English primary | Optional Korean summaries when requested |

For the complete policy and rationale, see [LANGUAGE_POLICY.md](LANGUAGE_POLICY.md).

---

## Getting Started

### 1. Initialize the project structure

Create the required directories and files:

```bash
mkdir -p .aisync specs src tests docs
```

### 2. Set up STATE.yaml

Create the initial state file at `.aisync/STATE.yaml`:

```yaml
phase: THESIS
owner_model: Claude
blocking_model: null
status: OK
last_artifacts:
  blueprint_sha: ""
  audit_log_sha: ""
  src_sha: ""
blockers: []
```

### 3. Begin the Thesis phase

Claude Code analyzes requirements and produces:
- `.aisync/BLUEPRINT.json` with project principles, constraints, tech stack, IO contracts, and security policies
- `specs/01_interface_def.ts` with interface and type definitions

### 4. Advance through the cycle

Use the `.ai_sync` Manager to progress through phases:

```bash
ai-sync status    # Check current state
ai-sync check     # Validate required artifacts
ai-sync next      # Advance to the next phase
```

### 5. Iterate

After completing Execution, the cycle returns to Thesis for the next feature or iteration. Each cycle produces a complete audit trail of design decisions, red-team findings, and implementation artifacts.

---

## References

- [plan_rev2.md](plan_rev2.md) -- Full protocol specification with detailed phase descriptions, templates, and artifact-model mapping
- [LANGUAGE_POLICY.md](LANGUAGE_POLICY.md) -- Complete language usage policy with per-artifact rules and rationale
