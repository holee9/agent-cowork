# Consensus Resolution

Detailed procedures for resolving disagreements between Claude Code and GPT-5.3-Codex within the TITLE Protocol v3.0 dialectical cycle.

**Version:** 1.0.0
**Date:** 2026-02-15
**Expands:** PROTOCOL.md Section 6 (Conflict Resolution)

---

## Table of Contents

1. [Purpose](#1-purpose)
2. [Escalation Ladder](#2-escalation-ladder)
3. [CONSENSUS_BLOCKED Procedure](#3-consensus_blocked-procedure)
4. [System Owner Decision Framework](#4-system-owner-decision-framework)
5. [Resolution Documentation](#5-resolution-documentation)
6. [Common Disagreement Scenarios](#6-common-disagreement-scenarios)
7. [Prevention Strategies](#7-prevention-strategies)
8. [Metrics and Tracking](#8-metrics-and-tracking)
9. [Quick Reference](#9-quick-reference)

---

## 1. Purpose

Claude Code and GPT-5.3-Codex are frontier AI models with different architectural biases, training distributions, and reasoning strengths. These differences are by design: the TITLE Protocol's dialectical structure (Thesis, Antithesis, Synthesis, Execution) depends on genuine tension between an architect perspective and a builder/red-team perspective to produce robust systems.

Legitimate disagreements arise naturally in this structure. An architect may prioritize clean abstractions and long-term maintainability. A red-team auditor may prioritize eliminating attack surface regardless of design elegance. Neither perspective is inherently wrong; the protocol requires a structured way to reconcile them.

Without a clear resolution process, disagreements produce one of two failure modes:

- **Deadlock.** Both agents refuse to proceed, and the cycle stalls indefinitely. No progress is made, and the System Owner must intervene without structured information about the dispute.
- **Silent capitulation.** One agent yields without documenting its concerns, and a genuine risk or design insight is lost. The final system is weaker than it would have been if the disagreement had been surfaced and resolved explicitly.

The consensus resolution process defined here addresses both failure modes. It provides escalation levels proportional to the severity of the disagreement, ensures that dissenting positions are always recorded for future reference, and gives the System Owner the structured information needed to make binding decisions when agents cannot agree.

This document expands on the escalation ladder and recording conventions introduced in PROTOCOL.md Section 6. All references to STATE.yaml fields, blocker schemas, and phase transitions correspond to the definitions in PROTOCOL.md Sections 4 through 6 and the STATE.yaml template.

---

## 2. Escalation Ladder

The escalation ladder has four levels. Agents should always attempt resolution at the lowest applicable level before escalating. Escalation is not a failure; it is a structured acknowledgment that a disagreement requires more formal treatment.

### Level 1: Self-Resolve

**When to use:** Minor disagreements on implementation details, stylistic preferences, non-critical design choices, or cases where one position is clearly better supported by evidence.

**Who leads:** The agent designated as the phase lead (Claude Code during Thesis and Synthesis; GPT-5.3-Codex during Antithesis and Execution), as defined in PROTOCOL.md Section 3.

**Procedure:**

1. The non-leading agent raises a concern through normal phase communication (a comment in the relevant artifact, a note in handoff material, or a remark during collaborative work).
2. The leading agent considers the concern on its merits.
3. The leading agent makes a decision and documents both the decision and the reasoning in the relevant artifact (BLUEPRINT.json design rationale, AUDIT_LOG.md finding classification, or inline documentation).
4. The dissenting position is briefly noted alongside the decision so that future cycles can revisit it if circumstances change.

**Documentation requirement:** A brief note in the relevant artifact. No STATE.yaml changes are needed beyond normal phase updates.

**Examples of Level 1 disagreements:**

- Naming conventions for an interface field
- Whether a particular warning should be classified as MEDIUM or LOW severity
- The ordering of parameters in an IO contract
- Minor phrasing differences in documentation

**Characteristics that keep a disagreement at Level 1:**

- The decision is easily reversible in a future cycle
- The risk difference between the two positions is small
- The disagreement does not touch security, core architecture, or non-functional requirements
- One agent acknowledges the other's reasoning as reasonable even if it prefers a different outcome

### Level 2: Structured Argument

**When to use:** Significant disagreements where the outcome materially affects the design, security posture, performance characteristics, or scope of the current cycle. Both agents hold defensible positions and neither is willing to yield without a formal evaluation.

**Who leads:** The phase-leading agent evaluates and decides, as defined in PROTOCOL.md Section 3.

**Procedure:**

1. The disagreement is identified as requiring structured treatment (either agent may request Level 2).
2. Each agent prepares a formal position statement containing the following elements:

   - **Position:** A clear statement of the agent's recommended course of action.
   - **Evidence:** Technical facts, benchmarks, specifications, security analyses, or other concrete data supporting the position.
   - **Impact:** What happens if this position is adopted. What are the benefits? What trade-offs are accepted?
   - **Risk Assessment:** What happens if the opposing position is adopted instead? What risks are introduced? What is the worst-case outcome?

3. The leading agent reviews both position statements.
4. The leading agent makes a decision and documents:
   - Which position was adopted and why
   - What conditions might warrant revisiting the decision in a future cycle
   - Acknowledgment of the dissenting position and its merits
5. The decision is recorded in the STATE.yaml blockers array with the leading agent as owner.

**Documentation requirement:** Both position statements are preserved in handoff_notes or the relevant artifact. The decision is recorded as a blocker entry in STATE.yaml with status ACCEPTED, DEFERRED, or REJECTED as appropriate.

**Examples of Level 2 disagreements:**

- Whether a specific security mitigation is necessary for the current cycle or can be deferred
- Choice between two architectural approaches that have different performance and maintainability trade-offs
- Disagreement on whether a red-team finding constitutes a true blocker or a recommendation
- Scope boundaries for the current cycle's implementation

**Escalation signal:** If the non-leading agent believes the decision is materially wrong and could result in a security vulnerability, a fundamental architectural flaw, or a violation of the project's core_principles, it may escalate to Level 3.

### Level 3: CONSENSUS_BLOCKED

**When to use:** Neither agent accepts the other's position after Level 2 structured argumentation. The disagreement is significant enough that proceeding with the leading agent's decision would, in the dissenting agent's assessment, compromise security, violate core principles, or create unacceptable technical risk.

**Who can trigger it:** Either agent (Claude Code or GPT-5.3-Codex), regardless of which agent is the current phase lead.

**Procedure:** See Section 3 (CONSENSUS_BLOCKED Procedure) for the detailed step-by-step process.

**Key characteristic:** This level pauses all phase activity. No further design decisions, implementation work, or artifact modifications proceed until the System Owner resolves the block.

### Level 4: System Owner Decision

**When to use:** Automatically triggered by Level 3. The System Owner is the final arbiter of all protocol disputes.

**Who decides:** The System Owner (human supervisor), as defined in PROTOCOL.md Section 2.3.

**Procedure:** See Section 4 (System Owner Decision Framework) for the detailed decision process.

**Key characteristic:** The System Owner's decision is binding. Once a Level 4 decision is recorded, both agents must accept it and proceed accordingly. An agent may note its continued disagreement in a future cycle's Thesis phase, but it must not re-litigate a decided issue within the same cycle.

---

## 3. CONSENSUS_BLOCKED Procedure

This section provides the step-by-step procedure for invoking, managing, and resolving a CONSENSUS_BLOCKED state. This procedure corresponds to Escalation Ladder Level 3 and directly references PROTOCOL.md Section 6.1.

### 3.1 Who Can Trigger CONSENSUS_BLOCKED

Either agent may trigger a CONSENSUS_BLOCKED state. There is no requirement that the triggering agent be the phase lead or the dissenting party. The trigger criteria are:

- A Level 2 Structured Argument has been conducted and the non-leading agent rejects the decision.
- The triggering agent genuinely believes that proceeding would compromise the project's security, violate core_principles from BLUEPRINT.json, or create unacceptable technical risk.
- The triggering agent can articulate its position with evidence, not merely disagreement.

A CONSENSUS_BLOCKED trigger is a serious action. It halts phase progress and requires System Owner intervention. Agents should not use it for minor disagreements that belong at Level 1 or Level 2.

### 3.2 STATE.yaml Changes Required

The triggering agent updates STATE.yaml with the following changes:

**Fields to update:**

| Field | Value | Notes |
|-------|-------|-------|
| `status` | `CONSENSUS_BLOCKED` | Replaces the previous status (typically `OK`) |
| `blocking_model` | The name of the triggering agent (`Claude` or `Codex`) | Identifies who initiated the block |
| `handoff_notes` | Structured conflict summary (see Section 3.3) | Replaces or appends to existing handoff_notes |

**Fields that remain unchanged:**

| Field | Unchanged Value | Reason |
|-------|-----------------|--------|
| `phase` | Current phase | The phase does not advance; it is paused |
| `owner_model` | Current phase lead | Ownership does not transfer |
| `cycle` | Current cycle number | No cycle increment on a block |
| `last_artifacts` | Current artifact SHAs | Artifacts are frozen at the point of the block |
| `blockers` | Current blockers array | A new blocker entry is added (not replacing existing entries) |

A new entry is added to the `blockers` array:

| Blocker Field | Value |
|---------------|-------|
| `id` | Next sequential blocker ID (e.g., `BLK-005`) |
| `title` | Short description of the consensus dispute |
| `owner` | `SystemOwner` (because resolution requires System Owner action) |
| `status` | `OPEN` |
| `severity` | As assessed by the triggering agent (`CRITICAL` or `HIGH`) |
| `area` | The relevant audit area or `Consensus` if cross-cutting |
| `reason` | Brief statement of why consensus could not be reached |
| `updated_at` | Current ISO 8601 timestamp |

### 3.3 Information Required in handoff_notes

The handoff_notes field must contain a structured conflict summary with the following sections:

**Conflict Summary Template:**

- **Dispute Title:** A concise name for the disagreement.
- **Phase and Context:** The current phase, cycle number, and what activity was in progress when the disagreement arose.
- **Position A (Agent Name):** The first agent's position statement, including evidence and risk assessment.
- **Position B (Agent Name):** The second agent's position statement, including evidence and risk assessment.
- **Level 2 Decision (if applicable):** What decision the leading agent made at Level 2 and why the dissenting agent found it unacceptable.
- **Impact of Inaction:** What happens if the dispute is not resolved. Which deliverables are blocked? What is the cost of delay?
- **Suggested Resolution Paths:** Two or three potential resolutions the System Owner might consider, including compromise positions if any exist.

Both agents must contribute their position statements honestly and completely. Omitting evidence that weakens one's own position undermines the integrity of the resolution process.

### 3.4 System Owner Notification

Once STATE.yaml is updated to CONSENSUS_BLOCKED:

1. The triggering agent produces a clear, plain-language summary of the dispute directed at the System Owner. This summary should be understandable without deep technical context.
2. The summary must reference the specific blocker ID (e.g., BLK-005) so the System Owner can locate the detailed positions in STATE.yaml and handoff_notes.
3. The System Owner is notified through whatever communication channel the project uses (direct message, issue tracker, email, or in-session prompt). The protocol does not prescribe a specific notification mechanism; this depends on the project's operational setup.
4. The notification must include:
   - The blocker ID and title
   - A one-paragraph summary of each agent's position
   - The urgency level (based on blocker severity)
   - A request for the System Owner's decision

### 3.5 Timeline Expectations

The protocol does not impose a hard deadline on System Owner resolution, as response times depend on human availability and the complexity of the dispute. However, the following guidelines apply:

- **CRITICAL severity disputes:** The System Owner should aim to respond within 24 hours. The cycle is fully halted and no progress can be made.
- **HIGH severity disputes:** The System Owner should aim to respond within 72 hours. While the specific disputed decision is blocked, the leading agent may continue work on unrelated aspects of the current phase if any exist.
- **Escalation of stale blocks:** If a CONSENSUS_BLOCKED state persists beyond one week without System Owner response, either agent may add a note to handoff_notes documenting the delay and its impact on the project timeline.

### 3.6 Resuming After Resolution

Once the System Owner has made a decision (see Section 4):

1. The System Owner (or the designated agent acting on the System Owner's instruction) updates STATE.yaml:
   - Sets `status` back to `OK`
   - Sets `blocking_model` to `null`
   - Updates the relevant blocker entry's `status` to `ACCEPTED`, `DEFERRED`, or `REJECTED`
   - Updates the blocker's `owner` to `SystemOwner`
   - Updates the blocker's `reason` with the System Owner's rationale
   - Updates `handoff_notes` with the resolution summary
   - Sets the blocker's `updated_at` to the current timestamp

2. The phase-leading agent reads the updated STATE.yaml and handoff_notes.

3. The phase-leading agent incorporates the System Owner's decision into the current phase's work:
   - If the decision requires design changes, the leading agent updates BLUEPRINT.json and/or specs/ accordingly.
   - If the decision upholds the current design, the leading agent proceeds with the existing plan.
   - If the decision introduces new constraints, the leading agent documents them in the appropriate artifact.

4. Phase activities resume from the point of interruption. The cycle counter does not change.

---

## 4. System Owner Decision Framework

When a CONSENSUS_BLOCKED state is triggered, the System Owner must evaluate both agents' positions and make a binding decision. This section provides a structured framework for that evaluation.

### 4.1 Questions to Consider

The System Owner should evaluate the dispute against the following criteria:

**Security Impact:**
- Which position better protects the system against known attack vectors?
- Does either position introduce a vulnerability that would violate the security_policies defined in BLUEPRINT.json?
- What is the worst-case security outcome of each position?

**Alignment with Core Principles:**
- Which position is more consistent with the core_principles defined in BLUEPRINT.json?
- Does either position require compromising a stated principle? If so, is the compromise justified?

**Risk and Reward:**
- What is the probability and severity of the risks identified by each agent?
- What are the concrete benefits of each position?
- Is there a middle-ground position that captures most benefits while mitigating most risks?

**Implementation Cost:**
- What is the additional development effort required by each position?
- Does either position significantly affect the project timeline?
- Can a more expensive but safer approach be deferred to a future cycle without unacceptable risk in the interim?

**Reversibility:**
- Which position is easier to reverse if it turns out to be wrong?
- Does either position create technical debt or lock-in that would be costly to undo?

**Precedent and Consistency:**
- Have similar disputes been resolved in previous cycles? If so, what decision was made and has it held up?
- Does either position conflict with decisions already made in this cycle?

### 4.2 Decision Documentation Template

The System Owner's decision should be documented using the following structure:

**Decision Summary:**
A clear statement of the decision: which position is adopted, which is rejected, or what compromise is reached.

**Rationale:**
The reasoning behind the decision, referencing the evaluation criteria from Section 4.1. The rationale should be detailed enough that both agents and future reviewers can understand why this decision was made.

**Conditions and Constraints:**
Any conditions attached to the decision. For example: "Position A is adopted for this cycle, but the concern raised in Position B must be addressed in Cycle 3 with a dedicated security review." Or: "Position B is adopted with the constraint that performance benchmarks must exceed the thresholds defined in non_functional_requirements."

**Dissent Acknowledgment:**
A brief acknowledgment of the losing position's merits. This serves two purposes: it validates that the dissenting agent raised a legitimate concern, and it preserves the reasoning for future cycles that may need to revisit the issue.

### 4.3 Binding Nature of System Owner Decisions

Once the System Owner records a decision:

- Both agents must accept the decision and proceed accordingly. An agent may not refuse to implement a System Owner decision or passively resist by producing suboptimal work.
- The decision is final for the current cycle. It may only be revisited in a future cycle's Thesis phase, where new evidence or changed circumstances may warrant reconsideration.
- If an agent believes a System Owner decision will result in a demonstrable security vulnerability, it should document this concern in the relevant artifact (AUDIT_LOG.md or handoff_notes) so that it is preserved for future review. However, it must still proceed with the decision.
- The System Owner may amend a decision within the same cycle if new information emerges. This amendment follows the same documentation template and is recorded as an update to the original blocker entry.

---

## 5. Resolution Documentation

Every conflict resolution, regardless of escalation level, must be documented for traceability and future reference.

### 5.1 Where to Record

**STATE.yaml blockers array:** The primary record of all conflict resolutions at Level 2 and above. Each resolution is a blocker entry, even if the underlying issue was not a traditional audit finding.

**handoff_notes:** Contains the narrative context for the resolution, including position statements, evaluation criteria, and decision rationale. Handoff_notes are overwritten at each phase transition, so critical resolution context should also be preserved in the blockers array's reason field.

**Relevant artifacts:** Design decisions resulting from conflict resolution should be reflected in BLUEPRINT.json (for architectural decisions), specs/ (for interface changes), or AUDIT_LOG.md (for finding dispositions).

**Git history:** Resolution commits preserve the decision in version control (see Section 5.3).

### 5.2 Required Fields for Blocker Entries

Every conflict-related blocker entry in STATE.yaml must contain the following fields, per the STATE.yaml template schema:

| Field | Description | Example |
|-------|-------------|---------|
| `id` | Unique sequential identifier | `BLK-007` |
| `title` | Short description of the disagreement | `Rate limiting strategy: token bucket vs sliding window` |
| `owner` | Party that resolved the conflict | `Claude`, `Codex`, or `SystemOwner` |
| `status` | Resolution status | `ACCEPTED`, `DEFERRED`, or `REJECTED` |
| `severity` | Impact level of the disagreement | `CRITICAL`, `HIGH`, `MEDIUM`, or `LOW` |
| `area` | Relevant domain | `Security`, `Concurrency`, `Memory`, `Hardware`, `AI_Misuse`, or `Consensus` |
| `reason` | Documented rationale for the resolution | `SystemOwner decided token bucket because it provides fairer distribution under burst load. Sliding window concern deferred to Cycle 3.` |
| `updated_at` | ISO 8601 timestamp of the resolution | `2026-02-15T14:30:00Z` |

### 5.3 Git Commit Conventions for Resolution Commits

When a conflict resolution results in artifact changes, the Git commit message should follow these conventions:

- **Prefix:** `resolve:` to distinguish resolution commits from normal development commits.
- **Blocker reference:** Include the blocker ID in the commit message.
- **Brief description:** Summarize what changed as a result of the resolution.

Examples:

- `resolve: BLK-007 adopt token bucket rate limiting per System Owner decision`
- `resolve: BLK-003 update BLUEPRINT security_policies to require mTLS`
- `resolve: BLK-012 defer input sanitization redesign to Cycle 4`

### 5.4 Referencing Resolutions in Future Cycles

When a previously resolved disagreement becomes relevant in a future cycle (for example, a deferred concern comes due, or new evidence challenges a past decision):

1. Reference the original blocker ID in the new cycle's discussion (e.g., "This revisits the concern raised in BLK-007 from Cycle 2").
2. Do not modify the original blocker entry. Create a new blocker entry in the current cycle that references the original.
3. The new blocker entry's reason field should explain why the issue is being revisited and what has changed since the original resolution.

---

## 6. Common Disagreement Scenarios

The following scenarios illustrate how the escalation ladder applies to realistic disputes. Each scenario includes the context, both agents' positions, and the recommended resolution path.

### Scenario 1: Security vs Performance

**Context:** During the Synthesis phase (Cycle 2), GPT-5.3-Codex identifies a timing side-channel vulnerability in the authentication token comparison logic. The proposed fix involves constant-time comparison, which Codex's benchmarks show would increase authentication latency by 15ms per request. Claude Code argues that this latency increase would push the system below the p99 latency threshold defined in BLUEPRINT.json non_functional_requirements (target: 50ms; projected with fix: 58ms).

**Claude Code's position:** Defer the fix to Cycle 3. The current implementation uses HTTPS-only transport, limiting the practical exploitability of the timing side-channel. Meanwhile, violating the latency NFR would affect all users immediately. Propose optimizing the comparison algorithm in the next cycle to achieve both goals.

**GPT-5.3-Codex's position:** Accept the fix now. Timing side-channels are a well-documented attack vector (CWE-208). The 15ms overhead is within measurement noise for most clients, and the p99 threshold should be revised upward rather than leaving a known vulnerability open.

**Resolution path:**

1. **Level 2 -- Structured Argument.** Both agents document their positions with evidence: Codex provides the timing analysis and CWE reference; Claude Code provides the latency benchmark data and NFR specification.
2. Claude Code, as Synthesis lead, evaluates both positions. If Claude Code accepts the fix with a revised NFR threshold, the disagreement resolves at Level 2.
3. If Claude Code defers the fix and Codex believes the vulnerability is actively exploitable, Codex may escalate to **Level 3 -- CONSENSUS_BLOCKED**.
4. At **Level 4**, the System Owner evaluates: Is the timing side-channel exploitable in the deployment context? Is the NFR threshold a hard business requirement or a target? The System Owner makes a binding decision and may impose conditions (e.g., "Accept the fix, revise the NFR to 65ms, and optimize in Cycle 3").

### Scenario 2: Scope Dispute

**Context:** During the Synthesis phase (Cycle 3), Codex's AUDIT_LOG identifies that the file upload endpoint lacks virus scanning. Claude Code classifies this as a recommendation rather than a blocker, arguing that the current scope covers only internal users with trusted workstations and that virus scanning belongs in a future hardening cycle. Codex insists the finding is a CRITICAL blocker because the BLUEPRINT's security_policies specify "defense in depth" as a core principle.

**Claude Code's position:** Classify as DEFERRED to Cycle 4. Internal-only deployment reduces risk. Adding virus scanning requires integrating a third-party service, which introduces new dependencies and testing requirements that would delay the Execution phase by an estimated two weeks.

**GPT-5.3-Codex's position:** Classify as ACCEPTED with CRITICAL severity. "Defense in depth" is a stated core_principle. An unscanned file upload is a violation of this principle regardless of user trust assumptions. The integration can use ClamAV, which is well-supported and has minimal dependency footprint.

**Resolution path:**

1. **Level 2 -- Structured Argument.** Claude Code documents the scope rationale and schedule impact. Codex documents the core_principle alignment argument and proposes a specific integration approach with effort estimate.
2. Claude Code decides: if the effort estimate is acceptable, the finding is ACCEPTED. If the two-week delay is unacceptable, Claude Code DEFERs with conditions.
3. If Codex rejects the deferral because it believes the core_principle violation is unacceptable, escalation to **Level 3** is appropriate.
4. At **Level 4**, the System Owner considers: Is the "defense in depth" principle absolute or contextual? What is the actual risk of the current deployment? Is the schedule constraint firm? The System Owner decides and documents.

### Scenario 3: Architectural Disagreement

**Context:** During the Synthesis phase (Cycle 1), both agents agree that the message queue architecture needs redesign. Claude Code proposes an event-sourcing pattern with a persistent log, arguing that it provides auditability and replay capability. Codex proposes a simpler pub/sub pattern with at-least-once delivery, arguing that event sourcing adds significant complexity and the team (or future maintainers) may not have the expertise to operate it reliably.

**Claude Code's position:** Event sourcing aligns with the "traceability" core_principle. The persistent log enables debugging, compliance auditing, and system recovery. The complexity is justified by long-term operational benefits.

**GPT-5.3-Codex's position:** Pub/sub with idempotent consumers achieves the functional requirements with 40% less code and simpler operational characteristics. Event sourcing can be added in a future cycle if the traceability requirement proves insufficient with standard logging.

**Resolution path:**

1. **Level 2 -- Structured Argument.** Both agents produce position statements with evidence: Claude Code references traceability requirements and provides an event-sourcing design sketch. Codex provides a complexity comparison and operational risk assessment.
2. Claude Code evaluates whether traceability can be achieved through standard logging plus pub/sub, or whether event sourcing is genuinely required. This evaluation should reference specific BLUEPRINT requirements, not abstract preferences.
3. If Claude Code selects event sourcing and Codex believes the operational complexity creates unacceptable risk in the Execution phase, Codex may escalate to **Level 3**.
4. At **Level 4**, the System Owner weighs: What are the actual traceability requirements? Is the team equipped to operate event sourcing? Can a phased approach work (pub/sub now, event sourcing later)?

### Scenario 4: Severity Classification

**Context:** During the Synthesis phase (Cycle 2), Claude Code reviews Codex's AUDIT_LOG and encounters finding AUD-017: "API responses include stack traces in error messages when the application runs in production mode." Codex classified this as CRITICAL (information disclosure vulnerability, CWE-209). Claude Code disagrees and classifies it as a HIGH-severity recommendation, arguing that the error messages are only visible to authenticated internal API consumers and do not expose sensitive data.

**Claude Code's position:** Reclassify as HIGH. The error messages contain stack traces but no credentials, tokens, or PII. The internal-only API surface limits exposure. Fix by adding a production error handler in the Execution phase, but do not treat it as a cycle-blocking issue.

**GPT-5.3-Codex's position:** Maintain CRITICAL classification. Stack traces reveal internal file paths, framework versions, and dependency information that an attacker can use for reconnaissance. CWE-209 is classified as a security vulnerability regardless of API visibility. The fix is straightforward (configure production error handling) and should block Execution until implemented.

**Resolution path:**

1. **Level 1 -- Self-Resolve** may suffice if Claude Code acknowledges that the fix is straightforward and agrees to ACCEPT it at HIGH severity with implementation in the Execution phase.
2. If Codex insists on CRITICAL classification and blocking behavior, escalate to **Level 2 -- Structured Argument**.
3. Claude Code, as Synthesis lead, makes the final severity classification. If the classification is HIGH and Codex accepts this (with the fix still scheduled), the disagreement resolves.
4. If Codex believes the downgrade from CRITICAL to HIGH removes urgency and the fix may be deprioritized, it may escalate to **Level 3**.
5. At **Level 4**, the System Owner evaluates the actual deployment context and makes a binding severity classification.

### Scenario 5: Implementation Feasibility

**Context:** During the Synthesis phase (Cycle 2), Claude Code's updated BLUEPRINT specifies a real-time notification system using WebSocket connections with automatic reconnection, message ordering guarantees, and exactly-once delivery semantics. Codex reviews the feasibility and reports that exactly-once delivery over WebSocket is not achievable with the current tech_stack (Node.js, Redis) without adding a dedicated message broker (such as Apache Kafka or RabbitMQ), which is outside the approved tech_stack in BLUEPRINT.json.

**Claude Code's position:** Exactly-once delivery is a stated requirement from the System Owner. The tech_stack should be updated to include a message broker. The additional infrastructure complexity is justified by the reliability requirement.

**GPT-5.3-Codex's position:** Adding a message broker changes the infrastructure profile significantly: it requires new deployment configuration, monitoring, and operational expertise. Propose relaxing the requirement to at-least-once delivery with client-side deduplication, which is achievable within the current tech_stack and provides equivalent user experience.

**Resolution path:**

1. **Level 2 -- Structured Argument.** Codex documents the technical infeasibility with specifics: what exactly-once semantics require, why the current stack cannot provide them, and what the alternative achieves. Claude Code documents the requirement source and evaluates whether at-least-once with deduplication satisfies the System Owner's intent.
2. Claude Code may need to consult the System Owner directly (outside the CONSENSUS_BLOCKED process) to clarify whether the original requirement is negotiable.
3. If Claude Code insists on exactly-once and Codex insists it is infeasible without tech_stack changes, escalation to **Level 3** is appropriate because the disagreement is about a factual constraint (technical feasibility) rather than a preference.
4. At **Level 4**, the System Owner decides: Is exactly-once delivery essential, or is at-least-once with deduplication acceptable? If exactly-once is required, the System Owner authorizes the tech_stack change.

---

## 7. Prevention Strategies

The best conflict resolution is conflict prevention. The following strategies reduce the frequency and severity of disagreements by addressing their root causes.

### 7.1 Clear Requirements in BLUEPRINT

Vague or ambiguous requirements are the most common source of agent disagreements. When BLUEPRINT.json specifies "high availability" without defining a target, both agents may have different interpretations of what constitutes acceptable availability.

**Prevention action:** Ensure BLUEPRINT.json requirements are specific, measurable, and testable. Use quantified metrics wherever possible (e.g., "99.9% uptime measured monthly" instead of "high availability"). Reference the BLUEPRINT template fields for non_functional_requirements.

### 7.2 Quantified Non-Functional Requirements

Performance, scalability, and availability disputes frequently arise when non_functional_requirements in BLUEPRINT.json contain qualitative descriptions rather than quantitative thresholds.

**Prevention action:** Define each non-functional requirement with:
- A measurable metric (latency in milliseconds, throughput in requests per second, uptime as a percentage)
- A measurement method (p50, p95, p99 percentiles; synthetic or real traffic)
- A threshold that distinguishes acceptable from unacceptable performance

When agents disagree about whether a design meets a requirement, the quantified definition provides an objective standard for resolution.

### 7.3 Explicit Security Policies

Security trade-off disputes (Scenario 1 in Section 6) arise when security_policies in BLUEPRINT.json are stated as principles without operational specifics. "Defense in depth" is a valuable principle, but without concrete application guidelines, agents may disagree about whether a specific control is mandatory or optional.

**Prevention action:** For each security policy in BLUEPRINT.json, include:
- The specific controls required (e.g., input validation, output encoding, rate limiting)
- The conditions under which controls may be deferred (e.g., "rate limiting may be deferred if the API is internal-only and behind a VPN")
- The severity threshold at which a security finding automatically becomes a blocker

### 7.4 Regular Cycle Reviews with System Owner

Many CONSENSUS_BLOCKED states could have been prevented if the System Owner had been consulted earlier, before positions hardened. Regular check-ins allow the System Owner to provide direction on ambiguous requirements before they become disputes.

**Prevention action:**
- The System Owner reviews BLUEPRINT.json during Thesis and provides clarifications before the Antithesis phase begins.
- The System Owner reviews high-severity findings in AUDIT_LOG.md before the Synthesis phase begins.
- The System Owner provides priority guidance when multiple blockers compete for limited cycle capacity.

### 7.5 Explicit Scope Boundaries

Scope disputes (Scenario 2 in Section 6) arise when the current cycle's scope is not clearly delineated. Without explicit scope boundaries, one agent may consider a finding in-scope while the other considers it out-of-scope.

**Prevention action:** During the Thesis phase, Claude Code should define explicit scope boundaries in BLUEPRINT.json:
- What is in-scope for this cycle (with specific deliverables)
- What is explicitly out-of-scope (with rationale for deferral)
- What conditions would cause an out-of-scope item to be pulled into the current cycle

---

## 8. Metrics and Tracking

Tracking conflict patterns across cycles provides diagnostic information about the health of the protocol and the clarity of the project's requirements.

### 8.1 What to Track

For each conflict that reaches Level 2 or above, record the following in the project's cycle history (maintained across cycles in Git history and STATE.yaml blocker archives):

| Metric | Description |
|--------|-------------|
| **Conflict ID** | The blocker ID (e.g., BLK-007) |
| **Cycle** | The cycle number in which the conflict occurred |
| **Phase** | The phase during which the conflict arose |
| **Escalation Level** | The highest level reached (2, 3, or 4) |
| **Area** | The domain area of the disagreement (Security, Performance, Scope, Architecture, Feasibility) |
| **Resolution Owner** | Who resolved it (Claude, Codex, or SystemOwner) |
| **Time to Resolution** | Duration from escalation to resolution |
| **Recurring** | Whether this conflict revisits a previously resolved issue |

### 8.2 What High Conflict Frequency Indicates

If the number of Level 2 or above conflicts per cycle consistently exceeds three, this is a signal that one or more upstream factors need attention:

| Pattern | Likely Root Cause | Recommended Action |
|---------|-------------------|--------------------|
| Conflicts concentrated in Security area | Security policies are ambiguous or incomplete | Revise BLUEPRINT security_policies with explicit controls and thresholds |
| Conflicts concentrated in Scope area | Cycle scope boundaries are unclear | Add explicit in-scope and out-of-scope definitions to BLUEPRINT |
| Recurring conflicts on the same topic | Previous resolution was a deferral without follow-through | Schedule deferred items for the next cycle's Thesis phase |
| Frequent escalation to Level 3 or 4 | Agents have fundamentally different interpretations of requirements | System Owner should conduct a requirements clarification session before the next cycle |
| Conflicts increasing across cycles | Requirements are evolving faster than the BLUEPRINT is updated | Increase the frequency of BLUEPRINT updates and System Owner reviews |

### 8.3 Improvement Actions

When conflict metrics indicate a systemic issue:

1. **Requirements audit.** Review BLUEPRINT.json for vague, qualitative, or missing requirements. Quantify everything that can be quantified.
2. **Scope calibration.** Review cycle scope against team capacity and project timeline. Overloaded cycles produce more disputes because agents compete for limited capacity.
3. **Retrospective.** At the end of each cycle, review all Level 2+ conflicts. For each one, ask: "Could this have been prevented with clearer requirements, better scope definition, or earlier System Owner input?"
4. **Prevention integration.** Incorporate retrospective findings into the next cycle's Thesis phase. Update BLUEPRINT, security_policies, and scope boundaries accordingly.

---

## 9. Quick Reference

### Escalation Ladder Summary

| Level | Name | Trigger | Who Decides | STATE.yaml Change |
|-------|------|---------|-------------|-------------------|
| 1 | Self-Resolve | Minor disagreement | Phase lead | None |
| 2 | Structured Argument | Significant disagreement | Phase lead | New blocker entry |
| 3 | CONSENSUS_BLOCKED | Unresolvable disagreement | System Owner | status: CONSENSUS_BLOCKED, blocking_model set |
| 4 | System Owner Decision | Level 3 triggered | System Owner | Blocker updated, status: OK restored |

### CONSENSUS_BLOCKED Checklist

- [ ] Confirm Level 2 Structured Argument has been attempted
- [ ] Set STATE.yaml `status` to `CONSENSUS_BLOCKED`
- [ ] Set STATE.yaml `blocking_model` to the triggering agent's name
- [ ] Add a new blocker entry with `owner: SystemOwner` and `status: OPEN`
- [ ] Document both positions in `handoff_notes` using the conflict summary template
- [ ] Notify the System Owner with blocker ID and summary

### Resolution Checklist

- [ ] System Owner decision documented with rationale and dissent acknowledgment
- [ ] STATE.yaml `status` set back to `OK`
- [ ] STATE.yaml `blocking_model` set to `null`
- [ ] Blocker entry updated: `status`, `owner`, `reason`, `updated_at`
- [ ] Affected artifacts updated (BLUEPRINT.json, specs/) if decision requires changes
- [ ] Resolution committed to Git with `resolve:` prefix

### System Owner Decision Criteria

1. Which position better serves security?
2. Which aligns with BLUEPRINT core_principles?
3. What is the risk/reward trade-off?
4. What is the implementation cost?
5. Which position is more reversible?
6. Is there a valid compromise?

### Key STATE.yaml Fields for Conflict Resolution

| Field | Normal Value | During CONSENSUS_BLOCKED |
|-------|-------------|--------------------------|
| `status` | `OK` | `CONSENSUS_BLOCKED` |
| `blocking_model` | `null` | `Claude` or `Codex` |
| `blockers[].owner` | `Claude` or `Codex` | `SystemOwner` |
| `blockers[].status` | Varies | `OPEN` (until resolved) |

---

**Cross-References:**
- PROTOCOL.md Section 2 -- Role definitions and constraints
- PROTOCOL.md Section 3 -- Execution loop and phase responsibilities
- PROTOCOL.md Section 4 -- Phase transition gates
- PROTOCOL.md Section 5 -- Handoff procedures
- PROTOCOL.md Section 6 -- Conflict resolution (this document expands Section 6)
- STATE.yaml template -- Field definitions and blocker schema
- GLOSSARY.md -- Term definitions for CONSENSUS_BLOCKED, Blocker, System Owner

---

*TITLE Protocol v3.0 -- Consensus Resolution Guide*
*Version 1.0.0 | 2026-02-15*
