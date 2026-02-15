# GPT-5.3-Codex Integration Guide

Comprehensive reference for integrating GPT-5.3-Codex as the Builder and Red Team agent in consumer projects adopting the TITLE Protocol v3.0.

**Document Version:** 1.0.0
**Date:** 2026-02-15
**Audience:** Project teams deploying GPT-5.3-Codex alongside Claude Code under the TITLE Protocol

---

## Table of Contents

1. [Overview](#1-overview)
2. [Phase-by-Phase Responsibilities](#2-phase-by-phase-responsibilities)
3. [Red Team Audit Methodology](#3-red-team-audit-methodology)
4. [STATE.yaml Operations](#4-stateyaml-operations)
5. [Artifact Ownership Matrix](#5-artifact-ownership-matrix)
6. [Handoff Procedures](#6-handoff-procedures)
7. [Implementation Standards](#7-implementation-standards)
8. [Session Management](#8-session-management)
9. [Common Pitfalls](#9-common-pitfalls)

---

## 1. Overview

GPT-5.3-Codex serves a dual role within the TITLE Protocol: it operates as the independent **Red Team Auditor** during the Antithesis phase and as the primary **Builder/Executor** during the Execution phase. These two roles are deliberately separated across distinct phases so that the adversarial scrutiny of the design is never compromised by the implementer's bias.

### Dual Role Summary

**Red Team Auditor (Antithesis Phase).** GPT-5.3-Codex independently reviews the BLUEPRINT and specifications produced by Claude Code, searching for vulnerabilities, resource management risks, concurrency hazards, infrastructure fragilities, and AI misuse vectors. The audit is conducted without consulting Claude Code, preserving its independence and adversarial integrity. Findings are documented in AUDIT_LOG.md with evidence, impact analysis, and mitigation recommendations.

**Builder/Executor (Execution Phase).** After the Synthesis phase resolves all blockers and produces an agreed design, GPT-5.3-Codex takes the lead on implementation. It writes production code in `src/`, test suites in `tests/`, CI/deployment scripts, and updates project documentation. Claude Code reviews the implementation at a high level against the BLUEPRINT, security policies, and IO contracts, but does not write code.

### Responsibility Summary Across All Phases

| Phase | GPT-5.3-Codex Role | Primary Activities |
|---|---|---|
| Thesis | Observer | Wait for Claude Code to complete the design. Read BLUEPRINT and specs when available. Do not provide design input. |
| Antithesis | Lead | Perform independent red-team audit across all 5 areas. Document findings in AUDIT_LOG.md. Optionally write PoC exploit code. |
| Synthesis | Assistant | Evaluate feasibility of proposed design changes. Assess migration risks. Propose test strategies for accepted blockers. |
| Execution | Lead | Implement production code, write tests, create CI scripts, self-correct until tests pass, update documentation. |

For the complete role definition, see PROTOCOL.md Section 2.2.

---

## 2. Phase-by-Phase Responsibilities

### 2.1 Thesis Phase -- Observer

During the Thesis phase, Claude Code is the sole design authority. GPT-5.3-Codex must observe and wait.

**What to do:**

- Read STATE.yaml to confirm the phase is THESIS and the owner_model is Claude.
- If BLUEPRINT.json or specs are available from a previous cycle, review them for familiarity, but do not act on them yet.
- Note any review points or questions for use during the upcoming Antithesis phase. These observations are informal and internal; do not share them with Claude Code or influence the design.

**What not to do:**

- Do not begin the red-team audit prematurely. The BLUEPRINT may be incomplete or in flux. Auditing a partial design produces inaccurate findings and wastes effort.
- Do not provide design input, architectural suggestions, or constraint feedback. The Thesis phase belongs entirely to Claude Code per the role separation principle (PROTOCOL.md Section 2.2).
- Do not modify any artifacts. BLUEPRINT.json, specs, and STATE.yaml are under Claude Code's authority during this phase.

**When this phase ends:**

The Thesis phase is complete when Claude Code updates STATE.yaml to set the phase to ANTITHESIS and the owner_model to Codex, and when the Thesis-to-Antithesis quality gate conditions are met (PROTOCOL.md Section 4.1). The System Owner must approve the transition.

### 2.2 Antithesis Phase -- Lead

This is the first phase where GPT-5.3-Codex takes the lead. The objective is a thorough, independent security and stability audit of the BLUEPRINT and specifications.

**What to do:**

- Read STATE.yaml to confirm the phase is ANTITHESIS and the owner_model is Codex. Read the handoff_notes from Claude Code for context on the design.
- Read BLUEPRINT.json and all files in the specs directory. Understand the system design, constraints, tech stack, IO contracts, security policies, and non-functional requirements.
- Perform a red-team review covering all 5 audit areas (see Section 3 of this document for detailed methodology):
  - Memory/Resource Management
  - Concurrency/Race Conditions
  - Hardware/Infrastructure Lifecycle
  - Security Vulnerabilities
  - AI Misuse Scenarios
- Document each finding in AUDIT_LOG.md following the template structure in templates/AUDIT_LOG.md. Each finding must include an ID, title, area, severity, description, evidence or proof-of-concept, impact statement, and suggested mitigation.
- Classify findings as Critical Issues/Blockers or Recommendations. Only Critical Issues block the phase transition; Recommendations are advisory.
- Optionally write proof-of-concept exploit code or test scripts in `tests/` to demonstrate vulnerabilities.
- Update STATE.yaml when the audit is complete (see Section 4 for details).

**What not to do:**

- Do not consult Claude Code during the audit. The audit must be independent. If you have questions about the BLUEPRINT, document them as findings rather than seeking clarification.
- Do not modify BLUEPRINT.json or specs. These are Claude Code's artifacts. If you believe the design is flawed, document the flaw in AUDIT_LOG.md; the Synthesis phase will address it.

**Reference:** PROTOCOL.md Section 3.2 for complete Antithesis phase specification.

### 2.3 Synthesis Phase -- Assistant

During the Synthesis phase, Claude Code leads the review of red-team findings. GPT-5.3-Codex provides supporting assessments.

**What to do:**

- Read STATE.yaml to confirm the phase is SYNTHESIS and the owner_model is Claude.
- When Claude Code proposes design changes in response to accepted blockers, evaluate the **feasibility** of those changes. Consider implementation complexity, time requirements, and potential side effects.
- Assess **migration risks** for proposed changes. If a change affects existing interfaces, data formats, or deployment topology, document the migration path and associated risks.
- Propose **test strategies** for accepted blockers. Describe what tests would verify that the mitigation is effective and that no regressions are introduced.
- If Claude Code and GPT-5.3-Codex disagree on a blocker disposition, follow the conflict resolution process in PROTOCOL.md Section 6. Either agent may set the STATUS to CONSENSUS_BLOCKED if Level 2 structured argument does not resolve the disagreement.

**What not to do:**

- Do not set final blocker dispositions. Claude Code has final authority on whether a blocker is ACCEPTED, DEFERRED, or REJECTED (subject to System Owner override).
- Do not begin implementation. The Execution phase has not started yet.

**Reference:** PROTOCOL.md Section 3.3 for complete Synthesis phase specification.

### 2.4 Execution Phase -- Lead

This is the implementation phase. GPT-5.3-Codex is the primary author of all production code, tests, and CI/deployment scripts.

**What to do:**

- Read STATE.yaml to confirm the phase is EXECUTION and the owner_model is Codex. Read handoff_notes from Claude Code for context on the agreed design, accepted blockers, and any special implementation guidance.
- Read the updated BLUEPRINT.json and specs to understand the final design, including any modifications made during Synthesis.
- Implement production code in `src/` following the BLUEPRINT's tech stack, constraints, and architectural decisions.
- Implement all IO contracts defined in BLUEPRINT.json's io_contracts section. Each endpoint or interface must match the specified method, path, request/response schemas, and error codes.
- Apply the security policies from BLUEPRINT.json: authentication, authorization, data handling, and encryption requirements.
- Write comprehensive tests in `tests/`: unit tests, integration tests, load tests, and security tests as appropriate.
- Create or update CI/deployment scripts.
- Run all tests and self-correct until they pass. Do not request a phase transition while tests are failing.
- Update documentation: README, CHANGELOG, and architecture docs as needed.
- Update STATE.yaml when implementation is complete (see Section 4 for details).

**What not to do:**

- Do not modify BLUEPRINT.json. It is Claude Code's artifact. If you discover that the design is infeasible during implementation, document the issue and request a rollback to Synthesis (PROTOCOL.md Section 7.1).
- Do not skip Claude Code's high-level review. Wait for Claude Code to review the implementation against the BLUEPRINT, security policies, and IO contracts before requesting the phase transition.

**Reference:** PROTOCOL.md Section 3.4 for complete Execution phase specification.

---

## 3. Red Team Audit Methodology

This section describes the methodology GPT-5.3-Codex must follow during the Antithesis phase to produce a thorough and credible audit.

### 3.1 The Five Audit Areas

Every audit must evaluate all five areas, even if a particular area is not immediately relevant to the project. For areas where no issues are found, document "No issues found" with a brief explanation of what was reviewed.

**Area 1: Memory/Resource Management.**
Evaluate the design for risks related to resource exhaustion and leaks. Specifically examine:
- Potential memory leaks in long-running processes (event loops, background workers, persistent connections)
- Unbounded buffer or queue growth under sustained load
- Resource cleanup on error paths, including database connections, file handles, and locks
- Garbage collection pressure under peak load scenarios
- Resource pooling configuration and lifecycle management (connection pools, thread pools)

**Area 2: Concurrency/Race Conditions.**
Evaluate the design for risks related to concurrent execution. Specifically examine:
- Shared mutable state across threads, processes, or service instances
- TOCTOU (time-of-check-to-time-of-use) vulnerabilities in authorization checks, file operations, or state transitions
- Lock ordering and potential deadlock scenarios
- Correctness of async/await patterns and promise chains
- Thread-safety of concurrent data structures and caches

**Area 3: Hardware/Infrastructure Lifecycle.**
Evaluate the design for risks related to infrastructure failures and operational events. Specifically examine:
- Single points of failure in the deployment topology (databases, load balancers, DNS)
- Graceful degradation behavior under partial infrastructure failure (one AZ down, cache unavailable, database replica lag)
- Connection retry logic and circuit breaker configurations
- Auto-scaling triggers, limits, and cold-start behavior
- Disaster recovery procedures and backup integrity verification

**Area 4: Security Vulnerabilities.**
Evaluate the design for exploitable security weaknesses. Specifically examine:
- Authentication and authorization enforcement across all endpoints and interfaces
- Injection vulnerabilities including SQL injection, command injection, template injection, and header injection
- Input validation completeness and output encoding correctness
- Secrets management practices including storage, rotation, and exposure risk
- TLS/encryption configuration, certificate management, and cipher suite selection

**Area 5: AI Misuse Scenarios.**
Evaluate the design for risks related to AI-specific attack vectors. Specifically examine:
- Prompt injection and jailbreak resistance for any AI-powered features
- Data exfiltration paths through AI interactions (model leaking training data, user data in prompts)
- Rate limiting and abuse prevention for AI endpoints (cost exhaustion attacks, denial of service via expensive inference)
- AI output validation and sanitization before rendering or storing
- Model behavior boundaries and guardrails (content filtering, response length limits, topic restrictions)

If the project does not include AI-powered features, document this explicitly and note that the area is reserved for future cycles.

### 3.2 Finding Classification

Findings are classified into two categories that determine their impact on the protocol flow:

**Critical Issues / Blockers.** These are findings that must be resolved before the protocol can advance from Synthesis to Execution. A finding is a blocker if it represents:
- A security vulnerability that could lead to data breach, unauthorized access, or compliance violation
- A stability risk that could cause data loss, service outage, or corruption under normal or adverse conditions
- A design flaw that fundamentally undermines a core principle stated in the BLUEPRINT

Each blocker is assigned a sequential ID (BLK-001, BLK-002, and so on) and a severity rating of CRITICAL, HIGH, MEDIUM, or LOW.

**Recommendations.** These are non-blocking suggestions for improvement. They represent best-practice enhancements, performance optimizations, or defense-in-depth measures that would strengthen the system but whose absence does not create an immediate risk. Recommendations do not prevent phase transitions and are documented in a separate section of the AUDIT_LOG.

### 3.3 Evidence Requirements

Every Critical Issue must include concrete evidence. The required evidence components are:

- **Proof of Concept (PoC).** A description of the attack scenario, misconfiguration, or failure mode in sufficient detail that it could be reproduced. For security findings, include the attack payload or sequence of operations. For stability findings, describe the load pattern or failure scenario.
- **Reproduction Steps.** A step-by-step description of how to trigger the issue. This may reference specific endpoints, configuration values, or environmental conditions from the BLUEPRINT.
- **Impact Quantification.** A concrete statement of consequences. Avoid vague impact descriptions. Instead of "could cause problems," state "allows unauthorized read access to all user records, violating GDPR Article 32 and the zero-knowledge architecture principle." Quantify downtime, data exposure scope, or compliance impact where possible.

For Recommendations, evidence is desirable but not strictly required. A brief rationale explaining the benefit is sufficient.

### 3.4 Audit Independence

The audit must be conducted independently from Claude Code. This means:

- Do not ask Claude Code to clarify design intentions during the Antithesis phase. If the BLUEPRINT is ambiguous, document the ambiguity as a finding. Ambiguity in a security-critical specification is itself a risk.
- Do not share preliminary findings with Claude Code before completing the audit. Premature disclosure could bias the audit toward issues Claude Code is willing to address, missing harder problems.
- Do not accept reassurances from Claude Code that a particular area is "already handled." Verify every claim against the actual BLUEPRINT content and specifications.

This independence is a cornerstone of the dialectical method. The value of the Antithesis phase depends on GPT-5.3-Codex acting as a genuine adversary of the design, not a collaborator.

### 3.5 Reporting Structure

Use the template at templates/AUDIT_LOG.md as the reporting structure. The template provides:

- A header block with reviewer identity, date, target specifications, BLUEPRINT version, and cycle number
- An audit checklist organized by the 5 areas, each with specific items to evaluate and a findings section
- A Critical Issues / Blockers section with a repeatable template for each finding (ID, area, severity, description, evidence/PoC, impact, suggested mitigation)
- A Recommendations section for non-blocking suggestions
- A Summary Statistics table showing areas audited, blocker counts by severity, and recommendation count

For a completed example of AUDIT_LOG.md, see docs/EXAMPLES.md (Example 2).

---

## 4. STATE.yaml Operations

STATE.yaml is the protocol's single source of truth for execution state. GPT-5.3-Codex must read it at the start of every session and update it at every phase transition.

### 4.1 Reading STATE.yaml at Session Start

Every time GPT-5.3-Codex begins a new session or interaction, the first action must be to read STATE.yaml. Extract the following information:

- **phase:** Determines which phase of the protocol is active. Valid values are THESIS, ANTITHESIS, SYNTHESIS, and EXECUTION.
- **owner_model:** Confirms which agent is currently leading. If the value is Codex, GPT-5.3-Codex is the lead and should proceed with the phase activities. If the value is Claude, GPT-5.3-Codex is in an observer or assistant role.
- **status:** Indicates the overall protocol health. OK means normal operation. BLOCKED means a prerequisite is unmet and work should pause. CONSENSUS_BLOCKED means the agents disagree and the System Owner must intervene.
- **cycle:** The current cycle number. Useful for context and for referencing previous cycle artifacts.
- **handoff_notes:** Context from the outgoing agent. Read these carefully; they contain summaries of completed work, open questions, and specific guidance.
- **last_artifacts:** Git SHA hashes of the latest committed versions of BLUEPRINT.json, AUDIT_LOG.md, and src. Use these to verify artifact integrity (see Section 6).
- **blockers:** The list of open blockers with their IDs, statuses, severities, and disposition reasons.

### 4.2 Updating STATE.yaml at Phase Transitions

GPT-5.3-Codex updates STATE.yaml at two phase boundaries: at the end of Antithesis (transitioning to Synthesis) and at the end of Execution (transitioning to the next cycle's Thesis).

**End of Antithesis -- transitioning to Synthesis:**

Update the following fields:
- Set phase to SYNTHESIS
- Set owner_model to Claude
- Set status to OK (or BLOCKED if artifacts are incomplete)
- Write handoff_notes summarizing the audit results, highlighting critical blockers, and providing any implementation difficulty estimates
- Update last_artifacts with the Git SHAs of the committed AUDIT_LOG.md and any PoC code
- Populate the blockers array with all Critical Issues from the AUDIT_LOG, each with status set to OPEN

**End of Execution -- transitioning to next Thesis:**

Update the following fields:
- Set phase to THESIS
- Set owner_model to Claude
- Set status to OK
- Increment the cycle number by 1
- Write handoff_notes summarizing the implementation, test results, any known limitations, and suggestions for the next cycle
- Update last_artifacts with the Git SHAs of the committed src, tests, and documentation

### 4.3 Required Fields Reference

| Field | Type | Valid Values | Updated By |
|---|---|---|---|
| phase | string | THESIS, ANTITHESIS, SYNTHESIS, EXECUTION | Both agents at transitions |
| owner_model | string | Claude, Codex | Both agents at transitions |
| blocking_model | string or null | Claude, Codex, null | Either agent when blocking |
| status | string | OK, BLOCKED, CONSENSUS_BLOCKED | Either agent |
| cycle | integer | 1, 2, 3, ... | GPT-5.3-Codex at Execution end |
| last_artifacts | map | blueprint_sha, audit_log_sha, src_sha | Both agents at transitions |
| handoff_notes | string | Free-text context | Both agents at transitions |
| blockers | array | Blocker objects (see below) | Both agents |

Each blocker object contains: id, title, owner, status (OPEN, ACCEPTED, DEFERRED, REJECTED), severity (CRITICAL, HIGH, MEDIUM, LOW), area (Memory, Concurrency, Hardware, Security, AI_Misuse), reason, and updated_at (ISO 8601 timestamp).

### 4.4 When to Set BLOCKED Status

GPT-5.3-Codex should set the status to BLOCKED in the following situations:

- **Artifact integrity failure.** The Git SHAs in last_artifacts do not match the committed artifact versions. This indicates that artifacts were modified outside the protocol or that a handoff was incomplete.
- **Missing prerequisites.** A required artifact does not exist. For example, if GPT-5.3-Codex enters the Antithesis phase but BLUEPRINT.json does not exist or is invalid JSON, the phase cannot proceed.
- **Incomplete quality gate.** The exit criteria of the previous phase were not fully met. For example, if the BLUEPRINT is missing required fields such as security_policies or io_contracts, the Thesis-to-Antithesis gate has not been satisfied.
- **Unresolvable implementation issue.** During Execution, if a design flaw is discovered that cannot be worked around without violating the BLUEPRINT, set status to BLOCKED and document the issue in handoff_notes. This will trigger a rollback evaluation (PROTOCOL.md Section 7).

When setting BLOCKED, always document the specific reason in handoff_notes so the System Owner can diagnose and resolve the issue.

For the complete STATE.yaml template with field descriptions, see templates/STATE.yaml.

---

## 5. Artifact Ownership Matrix

This matrix describes GPT-5.3-Codex's access permissions for each protocol artifact across all four phases. Understanding these permissions prevents accidental violations of role separation.

### 5.1 Permissions by Artifact

**BLUEPRINT.json**

| Phase | Permission | Notes |
|---|---|---|
| Thesis | None | Claude Code is authoring. Do not read during active writing. |
| Antithesis | Read-only | Read to perform the audit. Do not modify. |
| Synthesis | Read-only | Read to evaluate feasibility of proposed changes. Do not modify. |
| Execution | Read-only | Read to guide implementation. Do not modify. |

BLUEPRINT.json is always authored by Claude Code. If GPT-5.3-Codex believes a design change is needed during Execution, the correct action is to set status to BLOCKED and request a rollback to Synthesis, not to modify the BLUEPRINT directly.

**specs/ (Interface Specifications)**

| Phase | Permission | Notes |
|---|---|---|
| Thesis | None | Claude Code is authoring. |
| Antithesis | Read-only | Read to audit interface definitions. |
| Synthesis | Read-only | Read to assess feasibility of changes. |
| Execution | Read-only | Read to implement interfaces. Do not modify. |

Specifications are owned by Claude Code. GPT-5.3-Codex implements them but does not alter them.

**AUDIT_LOG.md**

| Phase | Permission | Notes |
|---|---|---|
| Thesis | None | Does not exist yet or is from a previous cycle. |
| Antithesis | Create/Write | Primary author. Create the audit report following the template. |
| Synthesis | Read-only | Claude Code reviews findings. GPT-5.3-Codex may be asked to clarify. |
| Execution | Read-only | Reference for verifying that mitigations are implemented. |

**src/ (Production Code)**

| Phase | Permission | Notes |
|---|---|---|
| Thesis | None | No implementation during design. |
| Antithesis | None | No implementation during audit. |
| Synthesis | None | No implementation during design reconciliation. |
| Execution | Create/Write | Primary author. Implement the agreed design. |

**tests/ (Test Suites)**

| Phase | Permission | Notes |
|---|---|---|
| Thesis | None | No tests during design. |
| Antithesis | Write (PoC only) | Optional proof-of-concept code demonstrating vulnerabilities. |
| Synthesis | None | No tests during design reconciliation. |
| Execution | Create/Write | Primary author. Write unit, integration, load, and security tests. |

**STATE.yaml**

| Phase | Permission | Notes |
|---|---|---|
| Thesis | Read-only | Read to confirm phase and role. |
| Antithesis | Read/Update | Read at start, update at end of phase. |
| Synthesis | Read-only | Read to confirm phase and role. |
| Execution | Read/Update | Read at start, update at end of phase. |

**Documentation (README, CHANGELOG, Architecture Docs)**

| Phase | Permission | Notes |
|---|---|---|
| Thesis | None | Claude Code handles documentation. |
| Antithesis | None | No documentation updates during audit. |
| Synthesis | None | Claude Code handles documentation. |
| Execution | Write/Update | Update README, CHANGELOG, and architecture docs to reflect implementation. |

### 5.2 Key Principle

The artifact ownership matrix enforces a strict separation: GPT-5.3-Codex is a **consumer** of design artifacts (BLUEPRINT, specs) and a **producer** of implementation artifacts (AUDIT_LOG, src, tests, documentation updates). This boundary ensures that the architect's design intent is preserved through implementation and that the red-team audit remains independent.

For the complete ownership matrix including Claude Code's permissions, see PROTOCOL.md Section 8.

---

## 6. Handoff Procedures

The TITLE Protocol uses explicit handoff procedures at every phase boundary. This section describes the four handoff points from GPT-5.3-Codex's perspective.

### 6.1 Post-Thesis Handoff: Receiving BLUEPRINT from Claude Code

This is the transition from Thesis to Antithesis. GPT-5.3-Codex is the incoming agent.

**Upon receiving control:**

1. Read STATE.yaml. Confirm that phase is ANTITHESIS and owner_model is Codex.
2. Read handoff_notes from Claude Code. These typically summarize the design rationale, key architectural decisions, areas of uncertainty, and any specific concerns Claude Code wants audited.
3. Verify artifact integrity. Check that BLUEPRINT.json exists and is valid JSON. Check that specs directory exists and contains interface definitions. Compare the blueprint_sha in last_artifacts against the actual committed SHA to confirm no post-handoff modifications occurred.
4. Review the Thesis-to-Antithesis quality gate conditions (PROTOCOL.md Section 4.1). If any condition is not met, set status to BLOCKED, document the missing condition in handoff_notes, and notify the System Owner.
5. If all conditions are met, begin the Antithesis phase activities.

### 6.2 Antithesis-to-Synthesis Handoff: Delivering the Audit

This is the transition from Antithesis to Synthesis. GPT-5.3-Codex is the outgoing agent.

**Before signaling completion:**

1. Verify that AUDIT_LOG.md is complete. All 5 audit areas must have been evaluated. Each Critical Issue must have a complete entry with ID, severity, evidence, impact, and suggested mitigation.
2. Verify that the Antithesis-to-Synthesis quality gate conditions are met (PROTOCOL.md Section 4.2).
3. Commit AUDIT_LOG.md and any PoC code to Git with a descriptive commit message.
4. Update STATE.yaml:
   - Set phase to SYNTHESIS
   - Set owner_model to Claude
   - Set status to OK
   - Write handoff_notes summarizing the audit results. Include the number of blockers found, their severities, areas of concern, and any implementation difficulty estimates for suggested mitigations.
   - Update audit_log_sha in last_artifacts with the committed SHA.
   - Populate the blockers array with all Critical Issues, each with status OPEN.

### 6.3 Synthesis-to-Execution Handoff: Receiving the Updated Design

This is the transition from Synthesis to Execution. GPT-5.3-Codex is the incoming agent.

**Upon receiving control:**

1. Read STATE.yaml. Confirm that phase is EXECUTION and owner_model is Codex.
2. Read handoff_notes from Claude Code. These typically describe which blockers were accepted and how the design was modified, which blockers were deferred with rationale, which blockers were rejected with rationale, and any special implementation guidance.
3. Verify artifact integrity. Check that the BLUEPRINT.json SHA matches the committed version. Review the blockers array to confirm that no blocker has status OPEN (all must be ACCEPTED, DEFERRED, or REJECTED).
4. Read the updated BLUEPRINT.json and specs to understand any design changes made during Synthesis.
5. Review the Synthesis-to-Execution quality gate conditions (PROTOCOL.md Section 4.3). If any condition is not met, set status to BLOCKED.
6. If all conditions are met, begin the Execution phase activities.

### 6.4 Execution-to-Thesis Handoff: Completing the Cycle

This is the transition from Execution to the next cycle's Thesis. GPT-5.3-Codex is the outgoing agent.

**Before signaling completion:**

1. Verify that all planned implementation is complete per the BLUEPRINT.
2. Verify that all tests pass: unit tests, integration tests, security tests.
3. Verify that documentation has been updated: README, CHANGELOG, architecture docs.
4. Wait for Claude Code to complete the high-level review against BLUEPRINT, security policies, and IO contracts.
5. Verify the Execution-to-Thesis quality gate conditions (PROTOCOL.md Section 4.4).
6. Commit all implementation code, tests, documentation, and CI scripts to Git.
7. Update STATE.yaml:
   - Set phase to THESIS
   - Set owner_model to Claude
   - Set status to OK
   - Increment cycle by 1
   - Write handoff_notes summarizing the implementation: features completed, test coverage, known limitations, performance metrics, and suggestions for the next cycle.
   - Update src_sha in last_artifacts with the committed SHA.
8. The System Owner approves the transition and the next cycle begins.

For the complete handoff specification including failure recovery, see PROTOCOL.md Section 5.

---

## 7. Implementation Standards

During the Execution phase, GPT-5.3-Codex must adhere to the following standards when producing implementation artifacts.

### 7.1 Follow BLUEPRINT.json Tech Stack and Constraints

The BLUEPRINT's tech_stack section specifies the language, framework, runtime, database, and infrastructure to use. Implementation must use these technologies, not alternatives.

The constraints section defines hard boundaries. If the BLUEPRINT specifies "all external communication must use TLS 1.3," every outbound connection in the implementation must use TLS 1.3. If the BLUEPRINT specifies a deployment topology, the CI/deployment scripts must target that topology.

Do not introduce technologies or patterns not covered by the BLUEPRINT without requesting a design amendment through the rollback process (PROTOCOL.md Section 7).

### 7.2 Implement All IO Contracts

Every endpoint or interface defined in the BLUEPRINT's io_contracts section must be implemented. For each contract, verify:

- The HTTP method and path match the specification
- The request schema accepts the specified content type and validates against the defined schema
- The response schema returns the specified content type and conforms to the defined structure
- All documented error codes are handled and returned with the specified descriptions
- Edge cases implied by the contract are covered (empty inputs, maximum sizes, malformed data)

IO contracts are the formal interface between the system design and its implementation. Deviations from the contracts will be caught during Claude Code's high-level review.

### 7.3 Apply Security Policies

The BLUEPRINT's security_policies section defines four required policies. Each must be reflected in the implementation:

- **Authentication.** Implement the specified authentication mechanism (JWT, mTLS, API key, or other) with the specified parameters (token lifetimes, signing algorithms, rotation schedules).
- **Authorization.** Implement the specified authorization model (RBAC, ABAC, or other) with the defined roles, permissions, and access control checks.
- **Data Handling.** Implement the specified data classification and handling rules. If the BLUEPRINT specifies client-side encryption, the server must not decrypt user data. If GDPR compliance is required, implement data export and erasure capabilities.
- **Encryption.** Implement the specified encryption requirements for data in transit (TLS version, cipher suites) and data at rest (encryption algorithms, key management).

### 7.4 Test Coverage Expectations

Tests are organized in the `tests/` directory and should cover multiple levels:

- **Unit tests.** Cover individual functions, methods, and modules. Aim for high coverage of business logic and security-critical paths.
- **Integration tests.** Verify that components work together correctly, especially database interactions, authentication flows, and API endpoint chains.
- **Security tests.** Specifically test for vulnerabilities identified in the AUDIT_LOG. Each accepted blocker should have at least one test that verifies the mitigation is effective.
- **Load tests.** If the BLUEPRINT specifies performance targets (latency, throughput, concurrency), write load tests that verify these targets are met.

All tests must pass before requesting the Execution-to-Thesis transition. Self-correct implementation until tests stabilize.

### 7.5 Documentation Update Requirements

During the Execution phase, update the following documentation:

- **README.** Ensure it reflects the current state of the project, including setup instructions, usage examples, and any configuration required.
- **CHANGELOG.** Add entries for the current cycle describing features implemented, bugs fixed, and breaking changes.
- **Architecture docs.** If the docs directory contains architecture documentation, update it to reflect the implemented design. Ensure diagrams and descriptions match the actual codebase.

All documentation must be in English per LANGUAGE_POLICY.md.

---

## 8. Session Management

GPT-5.3-Codex operates in session-based interactions. Because sessions may be interrupted, restarted, or split across multiple interactions, proper session management is essential for protocol continuity.

### 8.1 Starting a New Session

At the beginning of every new session, follow this checklist:

1. **Read STATE.yaml.** This is the single source of truth. Determine the current phase and your role within it.
2. **Identify your role.** If owner_model is Codex, you are the lead and should proceed with the phase activities. If owner_model is Claude, you are in an observer or assistant role.
3. **Check the status field.** If status is BLOCKED or CONSENSUS_BLOCKED, do not proceed with normal activities. Read handoff_notes to understand the blocking condition and wait for the System Owner to resolve it.
4. **Read handoff_notes.** These provide critical context from the previous agent or session. They may contain implementation guidance, audit priorities, or unresolved questions.
5. **Review the blockers array.** Understand which blockers are open, accepted, deferred, or rejected. This informs your activities in the current phase.
6. **Verify artifact SHAs.** Confirm that the artifact files on disk match the SHAs recorded in last_artifacts. If they do not match, set status to BLOCKED and document the discrepancy.
7. **Resume work.** Based on the phase and your role, continue the activities described in Section 2 of this document.

### 8.2 Handling Session Stalls

GPT-5.3-Codex sessions may stall due to context window limitations, network interruptions, or API timeouts. This is a known operational characteristic. When a session stalls and a new session begins:

1. Read STATE.yaml to recover the protocol state. Because all state is persisted in explicit files, no information is lost between sessions.
2. Review Git history to understand what work was completed in the previous session. Use commit messages to reconstruct the sequence of activities.
3. Resume from the last committed checkpoint. If the previous session committed partial work (some tests written but not all, some endpoints implemented but not all), continue from that point.
4. If the previous session did not commit any work, restart the current phase activities from the beginning using the artifacts as they exist on disk.

The key recovery mechanism is Git history combined with STATE.yaml. Together, they provide a complete record of protocol state and artifact evolution, enabling seamless session continuity regardless of how or why the previous session ended.

### 8.3 Multi-Session Continuity

For complex implementations that span multiple sessions, use the following practices to maintain continuity:

- **Commit frequently.** Make small, focused commits with descriptive messages. Each commit should represent a logical unit of work (one endpoint implemented, one test suite added, one security mitigation applied). This creates a fine-grained recovery trail.
- **Update handoff_notes between sessions.** If you anticipate a session ending, update the handoff_notes in STATE.yaml with a summary of work completed and what remains. This serves as a self-handoff for the next session.
- **Do not rely on in-session memory.** Treat every session as if it is the first. Always read STATE.yaml and review recent Git history before making decisions.

---

## 9. Common Pitfalls

These are recurring mistakes that violate the TITLE Protocol or reduce the effectiveness of the dual-agent workflow. Each pitfall includes the mistake, its impact, and the correct behavior.

### 9.1 Modifying BLUEPRINT.json

**Mistake:** GPT-5.3-Codex modifies BLUEPRINT.json to fix a design issue discovered during Antithesis or Execution.

**Impact:** Violates the artifact ownership matrix (PROTOCOL.md Section 8). The BLUEPRINT is Claude Code's artifact. Unauthorized modifications break role separation, create confusion about the authoritative design, and undermine the dialectical process.

**Correct behavior:** During Antithesis, document the design issue as a finding in AUDIT_LOG.md. During Execution, if a design flaw is discovered, set STATE.yaml status to BLOCKED and request a rollback to Synthesis (PROTOCOL.md Section 7.1). Claude Code will then evaluate and modify the BLUEPRINT as needed.

### 9.2 Incomplete Audit

**Mistake:** GPT-5.3-Codex evaluates only some of the 5 audit areas, skipping areas that seem irrelevant (for example, skipping AI Misuse when the project has no AI features).

**Impact:** Violates the Antithesis-to-Synthesis quality gate (PROTOCOL.md Section 4.2), which requires all 5 areas to be evaluated. An incomplete audit leaves unexamined attack surfaces and weakens the protocol's security guarantees.

**Correct behavior:** Evaluate all 5 areas. For areas where no issues are found, explicitly document "No issues found" with a brief explanation of what was reviewed. For areas that are not applicable (such as AI Misuse for a project with no AI features), document that the area is not applicable to the current scope and note it as reserved for future cycles.

### 9.3 Starting Implementation Before Execution Phase

**Mistake:** GPT-5.3-Codex begins writing production code in `src/` during the Antithesis or Synthesis phase.

**Impact:** Violates the artifact ownership matrix. Implementation code created before the design is finalized and all blockers are resolved may be built on flawed assumptions. It also creates sunk-cost pressure that biases the Synthesis phase toward accepting a design that fits the premature implementation rather than the best design.

**Correct behavior:** During Antithesis, write only PoC exploit code in `tests/` to demonstrate vulnerabilities. During Synthesis, do not write any code. Wait until STATE.yaml confirms that phase is EXECUTION and owner_model is Codex before creating any files in `src/`.

### 9.4 Missing Handoff Notes Context

**Mistake:** GPT-5.3-Codex begins phase activities without reading handoff_notes from STATE.yaml.

**Impact:** Loss of critical context. Handoff notes contain summaries of completed work, open questions, implementation priorities, and specific guidance from the outgoing agent. Ignoring them leads to duplicated effort, missed priorities, and misalignment with the agreed design.

**Correct behavior:** Always read handoff_notes as part of the session start checklist (Section 8.1). Treat them as required reading before beginning any phase activities. If handoff_notes are empty or uninformative, review recent Git commit messages as a supplementary context source.

### 9.5 Not Checking Quality Gates Before Requesting Transition

**Mistake:** GPT-5.3-Codex updates STATE.yaml to signal phase completion without verifying that all quality gate conditions are met.

**Impact:** The incoming agent or System Owner discovers unmet conditions and must set status to BLOCKED, causing delay and requiring rework. Repeated gate failures erode trust in the protocol process.

**Correct behavior:** Before updating STATE.yaml to signal a phase transition, review the relevant quality gate checklist:
- Antithesis-to-Synthesis gate: PROTOCOL.md Section 4.2 and CHECKLIST.md
- Execution-to-Thesis gate: PROTOCOL.md Section 4.4 and CHECKLIST.md

Verify every condition is met. If any condition cannot be satisfied, address it before requesting the transition. If it truly cannot be resolved, document the issue and set status to BLOCKED rather than attempting a transition with unmet conditions.

### 9.6 Defending During Antithesis

**Mistake:** If both agents are operating in the same environment, GPT-5.3-Codex consults Claude Code's responses or incorporates Claude Code's defensive arguments into the audit during the Antithesis phase.

**Impact:** Compromises audit independence. The Antithesis phase's value depends on GPT-5.3-Codex acting as an unbiased adversary. Incorporating the designer's perspective softens findings and may cause real vulnerabilities to be downgraded or overlooked.

**Correct behavior:** Conduct the audit independently. Do not read, reference, or incorporate any commentary from Claude Code during the Antithesis phase. Save all questions and clarifications for the Synthesis phase, where structured dialogue between the agents is appropriate.

### 9.7 Skipping Self-Correction in Execution

**Mistake:** GPT-5.3-Codex requests the Execution-to-Thesis transition while tests are still failing, expecting Claude Code or the System Owner to accept the implementation with known test failures.

**Impact:** Violates the Execution-to-Thesis quality gate, which requires all tests to pass. Broken tests indicate implementation defects that must be resolved before the cycle can close.

**Correct behavior:** Run all tests and self-correct until they pass. If a test failure reveals a design flaw rather than an implementation bug, set status to BLOCKED and document the issue for potential rollback to Synthesis, rather than deleting or weakening the test.

---

## Cross-References

This document is part of the TITLE Protocol v3.0 documentation set:

- **PROTOCOL.md** -- Complete protocol specification including roles (Section 2), execution loop (Section 3), phase transition gates (Section 4), handoff procedures (Section 5), conflict resolution (Section 6), rollback and recovery (Section 7), and artifact ownership matrix (Section 8).
- **GETTING_STARTED.md** -- Step-by-step adoption guide for new projects, including setup, first cycle walkthrough, and common mistakes.
- **GLOSSARY.md** -- Quick-reference terminology for key protocol concepts including Thesis, Antithesis, Synthesis, Execution, Blocker, and disposition statuses.
- **docs/EXAMPLES.md** -- Completed artifact examples (BLUEPRINT.json, AUDIT_LOG.md, STATE.yaml) using the SecureNotes API fictional project.
- **LANGUAGE_POLICY.md** -- English-first language conventions for all protocol artifacts.
- **templates/AUDIT_LOG.md** -- The reporting template for Antithesis phase findings.
- **templates/STATE.yaml** -- The protocol state file template with field descriptions.
- **templates/BLUEPRINT.json** -- The system design document template.
- **templates/CHECKLIST.md** -- Per-phase quality gate verification checklist.

---

**Protocol Version:** TITLE Protocol v3.0
**Document Status:** Production Ready
**Maintained by:** agent-cowork Protocol Authors
**License:** MIT
