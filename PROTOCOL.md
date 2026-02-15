# TITLE Protocol v3.0 -- Protocol Specification

The single-source protocol document for multi-agent development with Claude Code and GPT-5.3-Codex.

---

## 1. Purpose and Principles

This protocol orchestrates two frontier AI models -- **Claude Code** (Architect) and **GPT-5.3-Codex** (Builder/Red Team) -- in a closed-loop development workflow. A human **System Owner** supervises phase transitions and resolves deadlocks.

### Core Principles

1. **Security First** -- Security and stability are first-class requirements, equal in priority to functionality. Every design decision must account for attack surface, data exposure, and failure modes.

2. **Stateless Design** -- All protocol state is persisted in explicit files (`BLUEPRINT.json`, `STATE.yaml`, `AUDIT_LOG.md`, Git history). Agents can be restarted or replaced at any phase boundary without data loss.

3. **Clear Role Separation** -- Claude Code and GPT-5.3-Codex have fixed, non-overlapping responsibilities per phase and artifact. Neither agent may act outside its designated role within a given phase.

---

## 2. Roles

### 2.1 Claude Code -- Architect / Coordinator / Narrator

| Responsibility | Description |
|---|---|
| System Design | Structures requirements, risks, and constraints into a coherent logical architecture |
| Blueprint Authorship | Creates and maintains `BLUEPRINT.json` and interface specifications |
| Synthesis Decisions | Interprets red-team findings; classifies each blocker as ACCEPTED, DEFERRED, or REJECTED |
| Documentation | Maintains README, architecture docs, operation guides, and CHANGELOG |
| High-Level Review | Reviews implementation against BLUEPRINT, security policies, and IO contracts |

**MUST NOT:**
- Write implementation-level code (function bodies, test implementations, CI scripts)
- Defend against or rebut red-team findings during the Antithesis phase
- Unilaterally change blocker dispositions after System Owner approval

### 2.2 GPT-5.3-Codex -- Builder / Red Team / Executor

| Responsibility | Description |
|---|---|
| Red-Team Review | Audits BLUEPRINT for memory leaks, concurrency issues, security flaws, and AI misuse vectors |
| Implementation | Writes production code in `src/`, test suites in `tests/`, CI/deployment scripts |
| PoC and Benchmarks | Designs and executes proof-of-concept exploits, load tests, and security benchmarks |
| Migration Planning | Evaluates implementation difficulty and proposes step-by-step migration plans |
| Feasibility Assessment | During Synthesis, evaluates migration risks and test strategies for proposed changes |

**MUST NOT:**
- Modify architectural decisions or BLUEPRINT fields directly (submit proposals as comments/issues)
- Set final policy or priority decisions (defer to Claude Code or System Owner)
- Intervene in phases where it is not the designated lead

### 2.3 System Owner -- Human Supervisor

| Responsibility | Description |
|---|---|
| Phase Approval | Approves each phase transition after reviewing quality gate conditions |
| Deadlock Resolution | Resolves CONSENSUS_BLOCKED states when agents cannot agree |
| Requirements Input | Provides initial requirements, constraints, and priorities |
| Final Authority | Makes binding decisions on disputed blockers or design trade-offs |

**The System Owner may communicate in any language.** Agents always produce artifacts in English (see LANGUAGE_POLICY.md).

---

## 3. Execution Loop

The protocol follows a four-phase dialectical cycle. Each complete pass is one **cycle**.

```
 +---> [ 1. THESIS ] ---> [ 2. ANTITHESIS ] ---> [ 3. SYNTHESIS ] ---> [ 4. EXECUTION ] ---+
 |          Claude              Codex               Claude leads           Codex leads       |
 |                                                  Codex assists          Claude reviews     |
 +-----------------------------------------------------------------------------------------+
```

### 3.1 Phase 1 -- Thesis (Claude Code leads)

**Inputs:**
- User requirements, constraints, existing system context
- Previous cycle's Synthesis decisions and Execution results (if cycle > 1)

**Activities:**
1. Analyze requirements, risks, and constraints
2. Define system goals, scope, and guardrails
3. Create or update `BLUEPRINT.json` with all required fields (see Section 4.1)
4. Write or update interface specifications in `specs/`
5. Document design rationale and trade-offs

**Outputs:**
- `BLUEPRINT.json` -- complete with all required fields
- `specs/` -- interface and contract definitions
- Updated `STATE.yaml` -- phase: THESIS, owner_model: Claude

**Agent Constraints:**
- Claude Code: sole author of BLUEPRINT and specs
- GPT-5.3-Codex: may observe and note review points; no direct design input

**Exit Criteria:** See Section 4.1 (Thesis to Antithesis gate).

### 3.2 Phase 2 -- Antithesis (GPT-5.3-Codex leads)

**Inputs:**
- Finalized `BLUEPRINT.json` and `specs/`
- Any documentation from Phase 1

**Activities:**
1. Perform red-team review against BLUEPRINT and specifications
2. Evaluate all 5 audit areas (see templates/AUDIT_LOG.md):
   - Memory/Resource Management
   - Concurrency/Race Conditions
   - Hardware/Infrastructure Lifecycle
   - Security Vulnerabilities
   - AI Misuse Scenarios
3. Document each finding with evidence, impact, and suggested mitigation
4. Classify findings as Critical Issues/Blockers or Recommendations
5. Optionally write PoC code or test scripts in `tests/`

**Outputs:**
- `AUDIT_LOG.md` -- complete red-team report covering all 5 areas
- Optional PoC code/tests

**Agent Constraints:**
- GPT-5.3-Codex: sole author of AUDIT_LOG
- Claude Code: reads AUDIT_LOG only; does not defend or rebut during this phase

**Exit Criteria:** See Section 4.2 (Antithesis to Synthesis gate).

### 3.3 Phase 3 -- Synthesis (Claude Code leads, GPT-5.3-Codex assists)

**Inputs:**
- `BLUEPRINT.json`, `AUDIT_LOG.md`, PoC results, existing code/docs

**Activities:**
1. Review every finding in the AUDIT_LOG
2. Classify each blocker: **ACCEPTED**, **DEFERRED**, or **REJECTED**
3. Document the rationale for each disposition
4. Update BLUEPRINT, interfaces, and policies to reflect accepted changes
5. GPT-5.3-Codex evaluates feasibility, migration risks, and test strategies for proposed changes
6. Resolve or escalate any disagreements (see Section 6)

**Outputs:**
- Updated `BLUEPRINT.json` -- reflecting design changes from accepted findings
- Updated `specs/` -- if interfaces changed
- Blocker dispositions recorded in `STATE.yaml`
- Design decision log

**Agent Constraints:**
- Claude Code: final authority on blocker dispositions (subject to System Owner override)
- GPT-5.3-Codex: advisory role on feasibility and migration

**Exit Criteria:** See Section 4.3 (Synthesis to Execution gate).

### 3.4 Phase 4 -- Execution (GPT-5.3-Codex leads, Claude Code reviews)

**Inputs:**
- Agreed BLUEPRINT, interface specifications, design documents

**Activities:**
1. Implement production code in `src/`
2. Write unit, integration, load, and security tests in `tests/`
3. Create or update CI/deployment scripts
4. Run all tests; self-correct until stabilized
5. Claude Code performs high-level review against BLUEPRINT, security policies, and IO contracts
6. Update documentation (README, CHANGELOG, architecture docs)

**Outputs:**
- Implementation in `src/` and `tests/`
- CI/deployment scripts
- Updated documentation
- Test results and quality metrics

**Agent Constraints:**
- GPT-5.3-Codex: owns all code in `src/` and `tests/`
- Claude Code: review-only; does not write implementation code

**Exit Criteria:** See Section 4.4 (Execution to Thesis gate).

---

## 4. Phase Transition Gates

Each phase transition requires all conditions to be met. The System Owner verifies and approves.

### 4.1 Thesis to Antithesis

All of the following must be true:

- [ ] `BLUEPRINT.json` exists and is valid JSON
- [ ] `project_name` is set (not a placeholder)
- [ ] `core_principles` array has at least 1 entry
- [ ] `constraints` object has at least 1 key
- [ ] `tech_stack` contains `language` and `framework`
- [ ] `io_contracts` has at least 1 endpoint/interface defined
- [ ] `security_policies` contains `authentication`, `authorization`, `data_handling`, `encryption`
- [ ] `non_functional_requirements` contains `performance`, `scalability`, `availability`, `observability`
- [ ] Interface specifications exist in `specs/`
- [ ] `STATE.yaml` updated: phase=THESIS, owner_model=Claude
- [ ] System Owner has reviewed and approved the BLUEPRINT

### 4.2 Antithesis to Synthesis

All of the following must be true:

- [ ] `AUDIT_LOG.md` exists and follows the template structure
- [ ] All 5 audit areas have been evaluated:
  - [ ] Memory/Resource Management
  - [ ] Concurrency/Race Conditions
  - [ ] Hardware/Infrastructure Lifecycle
  - [ ] Security Vulnerabilities
  - [ ] AI Misuse Scenarios
- [ ] Each finding has: ID, Title, Area, Severity, Description, Evidence/PoC, Impact, Suggested Mitigation
- [ ] Critical issues are clearly separated from recommendations
- [ ] Blockers are documented with severity ratings
- [ ] `STATE.yaml` updated: phase=ANTITHESIS, owner_model=Codex

### 4.3 Synthesis to Execution

All of the following must be true:

- [ ] Every blocker in `AUDIT_LOG.md` has a disposition: ACCEPTED, DEFERRED, or REJECTED
- [ ] Each DEFERRED blocker has: documented reason, planned cycle for resolution
- [ ] Each REJECTED blocker has: documented reason with evidence or rationale
- [ ] ACCEPTED blockers are reflected in updated `BLUEPRINT.json` and/or `specs/`
- [ ] `STATE.yaml` blockers array is updated with dispositions
- [ ] No blocker has status: OPEN
- [ ] GPT-5.3-Codex has reviewed feasibility of accepted design changes
- [ ] System Owner has approved the final design

### 4.4 Execution to Thesis (Next Cycle)

All of the following must be true:

- [ ] All planned implementation is complete per the BLUEPRINT
- [ ] Unit tests pass
- [ ] Integration tests pass (if applicable)
- [ ] Security tests pass (if applicable)
- [ ] Claude Code has performed high-level review against BLUEPRINT, security policies, and IO contracts
- [ ] Documentation is updated (README, CHANGELOG, architecture docs)
- [ ] `STATE.yaml` updated: cycle incremented, phase=THESIS, owner_model=Claude
- [ ] System Owner has approved the release or next-cycle transition

---

## 5. Handoff Procedures

When control transfers between agents at a phase boundary, the following procedure applies.

### 5.1 Pre-Handoff Checklist (Outgoing Agent)

Before updating STATE.yaml to signal phase completion:

1. **Verify exit criteria** -- confirm all quality gate conditions for the current phase are met
2. **Commit artifacts** -- ensure all modified artifacts are committed to Git with descriptive messages
3. **Write handoff notes** -- populate `STATE.yaml` `handoff_notes` with:
   - Summary of work completed
   - Any open questions or areas of concern
   - Specific guidance for the incoming agent
4. **Update artifact SHAs** -- set `last_artifacts` in STATE.yaml to the current Git SHAs

### 5.2 State Transfer

The outgoing agent updates STATE.yaml:

```yaml
phase: <NEXT_PHASE>
owner_model: <INCOMING_AGENT>
status: OK
handoff_notes: "<context for the incoming agent>"
last_artifacts:
  blueprint_sha: "<current SHA>"
  audit_log_sha: "<current SHA>"
  src_sha: "<current SHA>"
```

### 5.3 Post-Handoff Checklist (Incoming Agent)

Upon receiving control:

1. **Read STATE.yaml** -- confirm phase, owner_model, and status
2. **Read handoff notes** -- understand context from the outgoing agent
3. **Verify artifact integrity** -- confirm artifact SHAs match the committed versions
4. **Review relevant artifacts** -- read BLUEPRINT, AUDIT_LOG, specs as needed for the phase
5. **Begin phase activities** -- proceed with the activities defined in Section 3

### 5.4 Handoff Failure

If the incoming agent detects a problem (missing artifacts, SHA mismatch, incomplete gate conditions):

1. Set `STATE.yaml` status to BLOCKED
2. Document the issue in `handoff_notes`
3. Notify the System Owner for resolution

---

## 6. Conflict Resolution

When agents disagree on a design decision, blocker disposition, or implementation approach:

### 6.1 Escalation Ladder

**Level 1 -- Self-Resolve (within current phase):**
- The leading agent considers the other agent's input and makes a decision
- Decision and reasoning are documented in the relevant artifact
- Applies to: minor design choices, non-critical implementation details

**Level 2 -- Structured Argument:**
- Each agent documents its position with evidence and trade-off analysis
- The leading agent for the current phase makes the final call
- The dissenting position is recorded for future reference
- Applies to: significant design disagreements, blocker disposition disputes

**Level 3 -- CONSENSUS_BLOCKED:**
- If Level 2 does not resolve the conflict, either agent may set:
  ```yaml
  status: CONSENSUS_BLOCKED
  blocking_model: <the model that initiated the block>
  ```
- Both agents document their positions in `handoff_notes` or a designated conflict log
- The phase is paused until the System Owner intervenes

**Level 4 -- System Owner Decision:**
- The System Owner reviews both positions
- The System Owner makes a binding decision
- The decision is recorded in `STATE.yaml` blockers with `owner: SystemOwner`
- The protocol resumes from the point of interruption

### 6.2 Recording Resolutions

Every conflict resolution must be recorded:

```yaml
# In STATE.yaml blockers array:
- id: "BLK-XXX"
  title: "Description of the disagreement"
  owner: SystemOwner  # or Claude/Codex if self-resolved
  status: ACCEPTED    # or DEFERRED/REJECTED
  severity: HIGH
  area: Security       # or relevant area
  reason: "System Owner decided X because Y"
  updated_at: "YYYY-MM-DDTHH:MM:SSZ"
```

---

## 7. Rollback and Recovery

### 7.1 Valid Rollback Paths

Rollback is permitted in the following scenarios:

| Current Phase | Rollback To | Condition |
|---|---|---|
| Antithesis | Thesis | BLUEPRINT found to be fundamentally incomplete or incorrect |
| Synthesis | Antithesis | Audit was incomplete; additional red-team review needed |
| Synthesis | Thesis | Design requires major rework beyond blocker fixes |
| Execution | Synthesis | Implementation reveals design flaws not caught in audit |
| Execution | Thesis | Requirements changed significantly during implementation |

**Not permitted:** Rolling back from Thesis (it is the starting phase of each cycle).

### 7.2 Rollback Procedure

1. The agent or System Owner identifies the need for rollback
2. Update STATE.yaml:
   ```yaml
   phase: <TARGET_PHASE>
   owner_model: <APPROPRIATE_AGENT>
   status: OK
   handoff_notes: "ROLLBACK from <CURRENT_PHASE>: <reason>"
   ```
3. The cycle counter does **not** increment on rollback
4. Previous artifacts are preserved (not deleted); the target phase builds on them

### 7.3 Cycle Abandonment

If a cycle is unrecoverable (e.g., requirements invalidated, project pivot):

1. System Owner sets:
   ```yaml
   status: BLOCKED
   handoff_notes: "CYCLE ABANDONED: <reason>"
   ```
2. All current-cycle artifacts are preserved in Git history
3. A new cycle begins from Thesis with `cycle` incremented

---

## 8. Artifact Ownership Matrix

| Artifact | Thesis | Antithesis | Synthesis | Execution |
|---|---|---|---|---|
| `BLUEPRINT.json` | Claude: **write** | Codex: read | Claude: **write** | Codex: read, Claude: review |
| `AUDIT_LOG.md` | -- | Codex: **write** | Claude: read, decide | Codex: read |
| `specs/` | Claude: **write** | Codex: read | Claude: **write** | Codex: read, implement |
| `src/` | -- | -- | -- | Codex: **write**, Claude: review |
| `tests/` | -- | Codex: write (PoC) | -- | Codex: **write**, Claude: review |
| `STATE.yaml` | Claude: update | Codex: update | Claude: update | Codex: update |
| `docs/`, README | Claude: **write** | -- | Claude: write | Claude: **write** |
| CHANGELOG | -- | -- | -- | Claude: **write** |

**Legend:** **write** = primary author for this phase; read = consumes but does not modify; review = reads and provides feedback; update = modifies specific fields; -- = no interaction.
