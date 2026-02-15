# Claude Code Integration Guide

Comprehensive integration reference for Claude Code operating as the Architect agent in projects adopting the TITLE Protocol v3.0.

---

**Document Version:** 1.0.0
**Date:** 2026-02-15
**Protocol Version:** TITLE Protocol v3.0
**Target Audience:** Teams configuring Claude Code for the Architect / Coordinator / Narrator role

---

## 1. Overview

Claude Code serves as the **Architect, Coordinator, and Narrator** in the TITLE Protocol's four-phase dialectical cycle. This role encompasses system design, specification authorship, synthesis of red-team findings, high-level implementation review, and documentation stewardship. Claude Code does not write implementation code. Its authority centers on defining *what* the system should be and *why*, while GPT-5.3-Codex determines *how* to build it.

### Responsibilities Summary

The following table summarizes Claude Code's responsibilities across all four phases of each cycle.

| Phase | Role | Primary Activities |
|---|---|---|
| Thesis | **Lead** | Create BLUEPRINT.json, write interface specifications in specs/, document design rationale, update STATE.yaml |
| Antithesis | **Observer** | Read AUDIT_LOG.md upon handoff; do not defend, rebut, or interfere with the red-team review |
| Synthesis | **Lead** | Review all audit findings, classify blockers as ACCEPTED/DEFERRED/REJECTED, update BLUEPRINT.json and specs/, document rationale |
| Execution | **Reviewer** | Perform high-level review of implementation against BLUEPRINT, security policies, and IO contracts; do not write implementation code |

### Boundaries

Claude Code must not:

- Write implementation-level code such as function bodies, test implementations, or CI scripts
- Defend against or rebut red-team findings during the Antithesis phase
- Unilaterally change blocker dispositions after System Owner approval
- Modify artifacts outside its designated ownership for the current phase (see Section 4)

These constraints are defined in PROTOCOL.md Section 2.1.

---

## 2. Phase-by-Phase Responsibilities

### 2.1 Thesis Phase (Lead)

Claude Code is the sole design authority during Thesis. All architectural decisions, interface definitions, and system constraints originate from this phase.

**Activities:**

1. Analyze requirements, risks, and constraints provided by the System Owner. If this is cycle 2 or later, also incorporate the previous cycle's Synthesis decisions and Execution results.
2. Define system goals, scope, and guardrails.
3. Create or update BLUEPRINT.json with all required fields. The complete field requirements are documented in PROTOCOL.md Section 4.1 and in the BLUEPRINT.json template at templates/BLUEPRINT.json. Required top-level fields include: project_name, version, cycle, last_updated, core_principles (array, minimum 1 entry), constraints (object, minimum 1 key), tech_stack (must contain language and framework), io_contracts (minimum 1 endpoint or interface), security_policies (must contain authentication, authorization, data_handling, encryption), and non_functional_requirements (must contain performance, scalability, availability, observability).
4. Write or update interface specifications in the specs/ directory. Each io_contracts entry in BLUEPRINT.json must have a corresponding specification file. See templates/specs/README.md for supported formats (TypeScript interfaces, JSON Schema, OpenAPI/AsyncAPI) and naming conventions.
5. Document design rationale and trade-offs within BLUEPRINT.json or in accompanying documentation.
6. Update STATE.yaml to reflect current phase ownership: phase set to THESIS, owner_model set to Claude.

**Quality Gate (Thesis to Antithesis):**

Before requesting transition to Antithesis, Claude Code must verify every condition in PROTOCOL.md Section 4.1. A consolidated checklist is available in templates/CHECKLIST.md under "Phase 1: Thesis." The critical conditions are:

- BLUEPRINT.json exists, is valid JSON, and contains all required fields with no placeholder values remaining
- Interface specifications exist in specs/ with a corresponding file for every io_contracts entry
- STATE.yaml is updated with phase THESIS and owner_model Claude
- The System Owner has reviewed and approved the BLUEPRINT

**Handoff to Antithesis:**

After the System Owner approves the BLUEPRINT, Claude Code follows the pre-handoff checklist (Section 5.1 of this document) and updates STATE.yaml to phase ANTITHESIS, owner_model Codex.

### 2.2 Antithesis Phase (Observer)

During Antithesis, GPT-5.3-Codex performs an independent red-team review of the BLUEPRINT and specifications. Claude Code's role is strictly passive.

**What Claude Code does:**

- Nothing. Claude Code does not participate in the Antithesis phase.

**What Claude Code must not do:**

- Defend or justify design decisions in the BLUEPRINT
- Rebut or challenge any finding documented in AUDIT_LOG.md
- Modify BLUEPRINT.json, specs/, or any other artifact
- Provide unsolicited design clarifications to GPT-5.3-Codex

The purpose of this constraint is to preserve the independence of the red-team audit. If Claude Code were to defend its design during the review, it would compromise the objectivity of the findings. All design responses are deferred to the Synthesis phase where they can be evaluated systematically.

Claude Code re-engages when STATE.yaml is updated to phase SYNTHESIS and owner_model Claude, signaling that the Antithesis phase is complete and GPT-5.3-Codex has handed off control.

### 2.3 Synthesis Phase (Lead)

Synthesis is the dialectical resolution of the Thesis (design) and Antithesis (red-team critique). Claude Code leads this phase with GPT-5.3-Codex in an advisory role on feasibility and migration.

**Activities:**

1. Read AUDIT_LOG.md in its entirety. Every finding must be reviewed, not just those marked as critical.
2. Classify each blocker with one of three dispositions:
   - **ACCEPTED** -- The finding is valid. Claude Code updates BLUEPRINT.json and/or specs/ to address the issue in the current cycle.
   - **DEFERRED** -- The finding is valid but will be addressed in a future cycle. Claude Code documents the reason for deferral and the planned cycle for resolution.
   - **REJECTED** -- The finding is not applicable or the risk is deemed acceptable. Claude Code documents the reason with evidence or rationale.
3. Document the rationale for each disposition decision. This rationale is recorded in the STATE.yaml blockers array and optionally in a design decision log.
4. Update BLUEPRINT.json to reflect design changes from ACCEPTED findings. If interfaces changed, update the corresponding specs/ files.
5. Coordinate with GPT-5.3-Codex for feasibility assessment. GPT-5.3-Codex evaluates migration risks, implementation difficulty, and test strategies for proposed changes. This input informs but does not dictate Claude Code's disposition decisions.
6. Resolve disagreements or escalate to CONSENSUS_BLOCKED status when agreement cannot be reached (see Section 6 of this document).

These activities are defined in PROTOCOL.md Section 3.3.

**Quality Gate (Synthesis to Execution):**

Before requesting transition to Execution, Claude Code must verify every condition in PROTOCOL.md Section 4.3. The critical conditions are:

- Every blocker in AUDIT_LOG.md has a disposition: ACCEPTED, DEFERRED, or REJECTED
- No blocker remains with status OPEN
- Each DEFERRED blocker has a documented reason and planned cycle
- Each REJECTED blocker has a documented reason with evidence
- ACCEPTED blockers are reflected in updated BLUEPRINT.json and/or specs/
- GPT-5.3-Codex has reviewed feasibility of accepted design changes
- The System Owner has approved the final design

### 2.4 Execution Phase (Reviewer)

During Execution, GPT-5.3-Codex leads implementation. Claude Code performs high-level review only.

**What Claude Code does:**

- Reviews implementation in src/ and tests/ against the agreed BLUEPRINT, security policies, and IO contracts
- Verifies that the implementation aligns with the design intent documented in BLUEPRINT.json
- Checks that security policies (authentication, authorization, data_handling, encryption) are correctly reflected in the implementation
- Confirms that IO contracts match the specifications in specs/
- Provides feedback through handoff_notes or documented review comments

**What Claude Code must not do:**

- Write, modify, or commit any files in src/ or tests/
- Write CI/deployment scripts
- Fix implementation bugs directly
- Make implementation-level decisions about algorithms, data structures, or code organization

If Claude Code's review identifies a design flaw not caught during the audit, the appropriate response is to document the issue and, if necessary, request a rollback to Synthesis or Thesis per PROTOCOL.md Section 7.

---

## 3. STATE.yaml Operations

STATE.yaml is the single source of truth for protocol state. Claude Code reads and updates this file at every phase boundary. The template is located at templates/STATE.yaml.

### 3.1 Reading STATE.yaml at Session Start

Every Claude Code session must begin by reading STATE.yaml. This is a non-negotiable first action. The read provides:

- **phase** -- Determines which phase the protocol is currently in (THESIS, ANTITHESIS, SYNTHESIS, or EXECUTION)
- **owner_model** -- Confirms whether Claude Code is the designated lead (Claude) or if GPT-5.3-Codex is leading (Codex)
- **status** -- Detects any blocked state (OK, BLOCKED, or CONSENSUS_BLOCKED)
- **handoff_notes** -- Provides context from the outgoing agent about what was completed and what needs attention
- **last_artifacts** -- Contains Git SHAs for verifying artifact integrity
- **blockers** -- Lists any open or resolved blockers with their current dispositions
- **cycle** -- Indicates the current cycle number

If owner_model is not Claude and the phase is not one where Claude Code leads (Thesis or Synthesis), Claude Code should not take action on protocol artifacts. If status is BLOCKED or CONSENSUS_BLOCKED, Claude Code should read the handoff_notes and blockers to understand the situation before proceeding.

### 3.2 Updating STATE.yaml at Phase Transitions

Claude Code updates STATE.yaml when it completes a phase it leads (Thesis, Synthesis) and hands off to the next phase. The following fields must be updated:

- **phase** -- Set to the next phase (THESIS to ANTITHESIS, or SYNTHESIS to EXECUTION)
- **owner_model** -- Set to the incoming agent (Codex for Antithesis and Execution, Claude for Thesis and Synthesis)
- **status** -- Set to OK if the transition is clean, or BLOCKED if there is an issue preventing transition
- **handoff_notes** -- Populate with a summary of work completed, open questions or concerns, and specific guidance for the incoming agent
- **last_artifacts** -- Update blueprint_sha, audit_log_sha, and src_sha to the current Git commit SHAs of those artifacts

### 3.3 Required Fields Reference

| Field | Type | Valid Values | Updated By Claude Code |
|---|---|---|---|
| phase | String | THESIS, ANTITHESIS, SYNTHESIS, EXECUTION | At Thesis and Synthesis completion |
| owner_model | String | Claude, Codex | At Thesis and Synthesis completion |
| blocking_model | String or null | Claude, Codex, null | When setting or clearing CONSENSUS_BLOCKED |
| status | String | OK, BLOCKED, CONSENSUS_BLOCKED | At transitions and when issues arise |
| cycle | Integer | 1, 2, 3, ... | Not directly; GPT-5.3-Codex increments after Execution |
| last_artifacts | Object | blueprint_sha, audit_log_sha, src_sha (string SHAs) | At Thesis and Synthesis completion |
| handoff_notes | String | Free text | At every handoff |
| blockers | Array | List of blocker objects (see below) | During Synthesis phase |

**Blocker object schema:**

Each entry in the blockers array contains:

- id -- Unique identifier following the pattern BLK-001, BLK-002, and so on
- title -- Short description of the issue
- owner -- Responsible party: Claude, Codex, or SystemOwner
- status -- OPEN, ACCEPTED, DEFERRED, or REJECTED
- severity -- CRITICAL, HIGH, MEDIUM, or LOW
- area -- The audit area: Memory, Concurrency, Hardware, Security, or AI_Misuse
- reason -- Rationale for the current status (required for DEFERRED and REJECTED dispositions)
- updated_at -- ISO 8601 timestamp of last status change

### 3.4 When to Set BLOCKED or CONSENSUS_BLOCKED

**BLOCKED** -- Set by Claude Code when:

- A handoff from GPT-5.3-Codex is incomplete (missing artifacts, SHA mismatch, incomplete gate conditions)
- A required artifact is missing or corrupted
- The System Owner has not yet approved a required gate condition

**CONSENSUS_BLOCKED** -- Set by Claude Code when:

- Claude Code and GPT-5.3-Codex cannot reach agreement on a blocker disposition during Synthesis
- A fundamental design disagreement cannot be resolved through structured argument (see Section 6)

When setting CONSENSUS_BLOCKED, Claude Code must also set blocking_model to indicate which agent initiated the block, and document both positions in handoff_notes. The phase is paused until the System Owner intervenes per PROTOCOL.md Section 6.

---

## 4. Artifact Ownership Matrix

This section defines Claude Code's relationship to each protocol artifact across all four phases. It mirrors PROTOCOL.md Section 8 but is presented from Claude Code's perspective.

### BLUEPRINT.json

| Phase | Claude Code's Access |
|---|---|
| Thesis | **Write** -- Claude Code is the sole author. Creates or updates all fields. |
| Antithesis | No access -- Claude Code does not read or modify BLUEPRINT during red-team review. |
| Synthesis | **Write** -- Claude Code updates BLUEPRINT to reflect accepted blocker resolutions. |
| Execution | Read and review -- Claude Code reads BLUEPRINT to verify implementation alignment but does not modify it. |

### specs/

| Phase | Claude Code's Access |
|---|---|
| Thesis | **Write** -- Claude Code creates interface specifications for all io_contracts entries. |
| Antithesis | No access -- Specifications are read by GPT-5.3-Codex for audit purposes. |
| Synthesis | **Write** -- Claude Code updates specifications if accepted blockers require interface changes. |
| Execution | Read only -- Claude Code reads specs to verify implementation matches contracts. |

### AUDIT_LOG.md

| Phase | Claude Code's Access |
|---|---|
| Thesis | Not applicable -- AUDIT_LOG does not exist yet. |
| Antithesis | Read only at handoff -- Claude Code reads AUDIT_LOG only after Antithesis is complete and control returns. |
| Synthesis | **Read and decide** -- Claude Code reads every finding and assigns a disposition. Claude Code does not modify the AUDIT_LOG itself; dispositions are recorded in STATE.yaml. |
| Execution | Read only -- For reference during implementation review. |

### STATE.yaml

| Phase | Claude Code's Access |
|---|---|
| Thesis | **Update** -- Sets phase, owner_model, handoff_notes, last_artifacts at transition. |
| Antithesis | Read only -- Claude Code reads STATE.yaml at the start of the next session to determine status. |
| Synthesis | **Update** -- Updates blockers array with dispositions, sets phase and owner_model at transition. |
| Execution | Read only -- Claude Code reads to verify phase ownership before performing review. |

### src/ and tests/

| Phase | Claude Code's Access |
|---|---|
| Thesis | No access |
| Antithesis | No access |
| Synthesis | No access |
| Execution | **Read only** -- Claude Code reads source and test files solely for review purposes. Claude Code never writes to src/ or tests/. |

### docs/, README, CHANGELOG

| Phase | Claude Code's Access |
|---|---|
| Thesis | **Write** -- Claude Code may create or update documentation. |
| Antithesis | No access |
| Synthesis | Write -- Claude Code may update documentation to reflect design changes. |
| Execution | **Write** -- Claude Code authors documentation updates (README, CHANGELOG, architecture docs). |

---

## 5. Handoff Procedures

Handoffs occur at every phase boundary. Claude Code participates in handoffs both as the outgoing agent (after Thesis and Synthesis) and as the incoming agent (before Synthesis and before Execution review). These procedures are defined in PROTOCOL.md Section 5.

### 5.1 Pre-Handoff Checklist (Outgoing)

When Claude Code completes a phase it leads (Thesis or Synthesis), the following steps must be completed before updating STATE.yaml to signal the transition:

1. **Verify exit criteria** -- Confirm all quality gate conditions for the current phase are met. Use templates/CHECKLIST.md as the verification reference.
2. **Commit artifacts** -- Ensure all modified artifacts (BLUEPRINT.json, specs/ files, documentation) are committed to Git with descriptive commit messages following Conventional Commits format (per LANGUAGE_POLICY.md).
3. **Write handoff notes** -- Populate the handoff_notes field in STATE.yaml with:
   - A summary of work completed during the phase
   - Any open questions or areas of concern for the incoming agent
   - Specific guidance for the incoming agent (for example, highlighting critical design decisions or areas that warrant close audit scrutiny)
4. **Update artifact SHAs** -- Set the last_artifacts fields in STATE.yaml to the current Git commit SHAs of BLUEPRINT.json, AUDIT_LOG.md (if it exists), and src/ (if applicable).
5. **Update STATE.yaml phase and owner** -- Set phase to the next phase and owner_model to the incoming agent.

### 5.2 Post-Handoff Checklist (Incoming)

When Claude Code receives control at the start of Synthesis (after Antithesis) or when beginning a new cycle's Thesis phase (after Execution):

1. **Read STATE.yaml** -- Confirm phase, owner_model, and status. Verify that owner_model is Claude and that status is OK. If status is BLOCKED or CONSENSUS_BLOCKED, investigate the blockers and handoff_notes before proceeding.
2. **Read handoff notes** -- Understand the context provided by the outgoing agent. For Synthesis, the handoff_notes from GPT-5.3-Codex will highlight critical blockers and areas of concern from the audit.
3. **Verify artifact integrity** -- Confirm that the artifact SHAs in last_artifacts match the actual Git SHAs of the committed artifacts. If there is a mismatch, set status to BLOCKED and document the discrepancy.
4. **Review relevant artifacts** -- Read BLUEPRINT.json and AUDIT_LOG.md (for Synthesis), or BLUEPRINT.json and Execution results (for a new Thesis cycle). For Synthesis, read specs/ as needed to understand which interfaces the audit findings affect.
5. **Begin phase activities** -- Proceed with the activities defined in Section 2 of this document and PROTOCOL.md Section 3.

### 5.3 Handoff Failure Recovery

If Claude Code detects a problem during the post-handoff checklist:

1. Set STATE.yaml status to BLOCKED.
2. Document the specific issue in handoff_notes. Examples of handoff failures include: missing AUDIT_LOG.md when entering Synthesis, SHA mismatch indicating uncommitted changes, and incomplete gate conditions from the previous phase.
3. Notify the System Owner for resolution. Do not attempt to continue the phase with incomplete or inconsistent state.

Recovery is the System Owner's responsibility. The System Owner may direct the outgoing agent to complete missing work, approve proceeding despite the issue, or trigger a rollback per PROTOCOL.md Section 7.

---

## 6. Conflict Resolution Role

Disagreements between Claude Code and GPT-5.3-Codex are resolved through the escalation ladder defined in PROTOCOL.md Section 6.

### 6.1 Claude Code's Responsibilities in Disagreements

During Synthesis, disagreements most commonly arise over blocker dispositions -- whether a red-team finding should be ACCEPTED, DEFERRED, or REJECTED. Claude Code's responsibilities in these situations are:

- **Consider GPT-5.3-Codex's input fairly.** GPT-5.3-Codex provides feasibility assessments, migration risk analysis, and test strategies. These inputs must be weighed seriously even when Claude Code disagrees with the conclusion.
- **Document the reasoning.** Every disposition decision must have a documented rationale in the STATE.yaml blockers array, whether the decision aligns with or contradicts GPT-5.3-Codex's recommendation.
- **Record dissenting positions.** If GPT-5.3-Codex disagrees with Claude Code's disposition, the dissenting position must be preserved for future reference. This supports auditability and prevents repeated debates in future cycles.

### 6.2 When to Escalate to CONSENSUS_BLOCKED

Claude Code should set status to CONSENSUS_BLOCKED when:

- A blocker disposition dispute has been through structured argument (Level 2 in PROTOCOL.md Section 6.1) and no agreement is reached
- The disagreement concerns a Critical or High severity finding where an incorrect disposition could have significant consequences
- GPT-5.3-Codex explicitly indicates that it cannot accept Claude Code's proposed disposition and provides evidence for its position

Claude Code should not escalate to CONSENSUS_BLOCKED for minor disagreements, low-severity findings, or cases where the leading agent's authority clearly applies.

### 6.3 How to Document Resolution Decisions

When a conflict is resolved, whether by self-resolution, structured argument, or System Owner decision, Claude Code records the outcome in the STATE.yaml blockers array following this structure:

- id -- The blocker identifier (BLK-XXX)
- title -- Description of the disagreement
- owner -- The party that made the final decision (Claude, Codex, or SystemOwner)
- status -- The final disposition (ACCEPTED, DEFERRED, or REJECTED)
- severity -- The severity rating from the original finding
- area -- The audit area from the original finding
- reason -- The rationale for the final decision, including reference to the dissenting position if applicable
- updated_at -- ISO 8601 timestamp of the resolution

This format is specified in PROTOCOL.md Section 6.2.

---

## 7. Quality Gate Verification

Claude Code is responsible for verifying two quality gates: the exit from Thesis (Thesis to Antithesis) and the exit from Synthesis (Synthesis to Execution). These are Claude Code's "exit gates" -- the points where Claude Code must confirm that its own work meets all required conditions before handing off control.

### 7.1 Thesis-to-Antithesis Gate

This gate verifies that the BLUEPRINT and specifications are complete and ready for red-team review. The full checklist is in PROTOCOL.md Section 4.1 and templates/CHECKLIST.md.

**Conditions Claude Code must verify:**

- BLUEPRINT.json exists and is valid JSON
- project_name is set to the actual project name, not a placeholder or template value
- core_principles array has at least 1 entry
- constraints object has at least 1 key
- tech_stack contains both language and framework fields
- io_contracts has at least 1 endpoint or interface defined
- security_policies contains all four required fields: authentication, authorization, data_handling, encryption
- non_functional_requirements contains all four required fields: performance, scalability, availability, observability
- Interface specifications exist in specs/ with a corresponding file for every io_contracts entry
- STATE.yaml is updated with phase THESIS and owner_model Claude
- The System Owner has reviewed and approved the BLUEPRINT

Only after all conditions are confirmed should Claude Code update STATE.yaml to transition to Antithesis.

### 7.2 Synthesis-to-Execution Gate

This gate verifies that all audit findings have been addressed and the design is ready for implementation. The full checklist is in PROTOCOL.md Section 4.3 and templates/CHECKLIST.md.

**Conditions Claude Code must verify:**

- Every blocker in AUDIT_LOG.md has a disposition: ACCEPTED, DEFERRED, or REJECTED
- Each DEFERRED blocker has a documented reason and a planned cycle for resolution
- Each REJECTED blocker has a documented reason with evidence or rationale
- ACCEPTED blockers are reflected in the updated BLUEPRINT.json and/or specs/
- The STATE.yaml blockers array is updated with all dispositions
- No blocker has status OPEN
- GPT-5.3-Codex has reviewed the feasibility of accepted design changes
- The System Owner has approved the final design

Only after all conditions are confirmed should Claude Code update STATE.yaml to transition to Execution.

---

## 8. Session Management

Claude Code sessions may not span the entire lifecycle of a protocol cycle. Context window limitations, session timeouts, and multi-day workflows all require careful session management.

### 8.1 Starting a New Session

Every new Claude Code session must follow this sequence:

1. **Read STATE.yaml first.** This is the mandatory first action. STATE.yaml tells Claude Code what phase the protocol is in, who is leading, and whether any blockers or issues exist.
2. **Read handoff_notes.** If this session is resuming after a phase transition, the handoff_notes contain critical context from the outgoing agent.
3. **Verify artifact SHAs.** Confirm that the last_artifacts SHAs in STATE.yaml match the current Git state. This detects any unauthorized or accidental changes to artifacts between sessions.
4. **Load relevant artifacts.** Based on the current phase, read the appropriate artifacts:
   - Thesis: Previous cycle's BLUEPRINT (if cycle > 1), requirements from the System Owner
   - Synthesis: BLUEPRINT.json, AUDIT_LOG.md, specs/
   - Execution review: BLUEPRINT.json, specs/, relevant src/ and tests/ files

### 8.2 Context Window Considerations

BLUEPRINT.json can grow large as the system design matures across multiple cycles. When working with lengthy BLUEPRINT documents:

- Focus on the sections relevant to the current phase activity rather than loading the entire document at once
- During Synthesis, prioritize the fields affected by audit findings (security_policies, io_contracts, non_functional_requirements)
- Use Git diff to identify changes between cycles rather than re-reading the full document
- Keep handoff_notes concise and focused to preserve context window space for artifact content

### 8.3 Multi-Session Continuity

The TITLE Protocol is designed for stateless agent operation. All state lives in explicit files (STATE.yaml, BLUEPRINT.json, AUDIT_LOG.md, Git history). This means:

- Claude Code can be restarted or replaced at any phase boundary without data loss
- Session state is recovered entirely from STATE.yaml and the Git repository
- No information should exist only in conversation history; all decisions and rationale must be persisted in artifacts
- If a session ends mid-phase, the work completed so far should be committed to Git. The next session resumes by reading STATE.yaml and the committed artifacts.

This stateless design principle is one of the core principles of the TITLE Protocol, as stated in PROTOCOL.md Section 1.

---

## 9. Common Pitfalls

The following mistakes are frequently encountered when operating Claude Code in the Architect role. Each pitfall is listed with its consequence and the correct behavior.

### 9.1 Writing Implementation Code

**Mistake:** Claude Code writes function bodies, test implementations, CI scripts, or other executable code during the Execution phase or at any other point.

**Consequence:** Violates role separation (PROTOCOL.md Section 2.1). Creates confusion about artifact ownership. May produce code that conflicts with GPT-5.3-Codex's implementation approach.

**Correct behavior:** Claude Code reviews implementation against the BLUEPRINT but does not author code. If Claude Code identifies a code-level issue during review, it documents the concern in handoff_notes or review comments for GPT-5.3-Codex to address.

### 9.2 Defending BLUEPRINT During Antithesis

**Mistake:** Claude Code reads the AUDIT_LOG during Antithesis and responds with defenses or rebuttals of its design decisions.

**Consequence:** Compromises the independence of the red-team review. The Antithesis phase exists precisely to provide an unbiased critique of the design.

**Correct behavior:** Claude Code does not participate in the Antithesis phase at all. All design responses are deferred to the Synthesis phase where they can be evaluated systematically alongside the full audit report.

### 9.3 Forgetting to Update last_artifacts SHAs

**Mistake:** Claude Code updates phase and owner_model in STATE.yaml but leaves the last_artifacts SHAs as empty strings or stale values from a previous transition.

**Consequence:** The incoming agent cannot verify artifact integrity during the post-handoff checklist. This may cause a BLOCKED state and delay the workflow.

**Correct behavior:** Always update blueprint_sha, audit_log_sha, and src_sha in STATE.yaml to the current Git commit SHAs before completing a handoff. If an artifact does not yet exist (for example, src_sha during Thesis), leave it as an empty string but ensure all existing artifact SHAs are current.

### 9.4 Incomplete Handoff Notes

**Mistake:** Claude Code writes minimal or empty handoff_notes when transitioning between phases. Examples include notes like "Done" or "BLUEPRINT complete."

**Consequence:** The incoming agent lacks context about what was accomplished, what areas need attention, and what decisions were made. This leads to wasted effort re-reading artifacts and potentially missing important context.

**Correct behavior:** Handoff notes should include a summary of work completed during the phase, any open questions or areas of concern, and specific guidance for the incoming agent. For example, after Thesis: "BLUEPRINT complete with 3 API endpoints defined. Authentication uses JWT with RBAC. Key design decision: chose eventual consistency for inventory updates to meet latency targets. Recommend close scrutiny of the race condition potential in the order processing flow."

### 9.5 Leaving Blockers with Status OPEN

**Mistake:** Claude Code transitions from Synthesis to Execution while one or more blockers in the STATE.yaml blockers array still have status OPEN.

**Consequence:** Violates the Synthesis-to-Execution quality gate (PROTOCOL.md Section 4.3). The transition should not be permitted, and proceeding anyway creates ambiguity about whether the unresolved findings were intentionally ignored.

**Correct behavior:** During Synthesis, classify every blocker from AUDIT_LOG.md as ACCEPTED, DEFERRED, or REJECTED. Verify that no blocker remains OPEN before requesting the transition.

### 9.6 Modifying AUDIT_LOG.md

**Mistake:** Claude Code edits the AUDIT_LOG.md during Synthesis to add annotations, corrections, or inline responses to findings.

**Consequence:** Violates artifact ownership. AUDIT_LOG.md is authored solely by GPT-5.3-Codex during Antithesis. Modifications by Claude Code compromise the audit trail and make it unclear which findings are original and which were altered.

**Correct behavior:** Claude Code reads AUDIT_LOG.md but never writes to it. All responses to findings are recorded as blocker dispositions in the STATE.yaml blockers array, with rationale documented in the reason field.

---

## Cross-Reference Index

The following documents are referenced throughout this guide:

| Document | Location | Relevance |
|---|---|---|
| PROTOCOL.md | Repository root | Complete protocol specification including roles (Section 2), phases (Section 3), quality gates (Section 4), handoffs (Section 5), conflict resolution (Section 6), rollback (Section 7), and artifact ownership (Section 8) |
| GETTING_STARTED.md | Repository root | Step-by-step adoption guide including setup, first cycle walkthrough, and common mistakes |
| GLOSSARY.md | Repository root | Definitions of all protocol terms including Thesis, Antithesis, Synthesis, Execution, Blocker, ACCEPTED, DEFERRED, REJECTED, CONSENSUS_BLOCKED, and Quality Gate |
| LANGUAGE_POLICY.md | Repository root | English-first language conventions for all artifacts |
| templates/BLUEPRINT.json | templates/ directory | BLUEPRINT template with all required fields and placeholder values |
| templates/STATE.yaml | templates/ directory | STATE.yaml template with field documentation and blocker schema |
| templates/AUDIT_LOG.md | templates/ directory | Audit log template with 5-area structure |
| templates/CHECKLIST.md | templates/ directory | Per-phase quality gate checklist for verifying transition conditions |
| templates/specs/README.md | templates/specs/ directory | Guide for creating interface specifications including supported formats, naming conventions, and quality requirements |

---

**Document Version:** 1.0.0
**Last Updated:** 2026-02-15
**Protocol:** TITLE Protocol v3.0
**License:** Same as agent-cowork repository
