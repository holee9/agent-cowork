# Design Rationale

The reasoning behind every major design decision in the TITLE Protocol v3.0.

**Version:** 1.0.0
**Date:** 2026-02-15
**Status:** Living document -- updated as the protocol evolves

---

## Table of Contents

1. [Purpose](#1-purpose)
2. [The Dialectical Cycle](#2-the-dialectical-cycle)
3. [Role Separation](#3-role-separation)
4. [Quality Gates](#4-quality-gates)
5. [Stateless Architecture](#5-stateless-architecture)
6. [Artifact Ownership Matrix](#6-artifact-ownership-matrix)
7. [Conflict Resolution Model](#7-conflict-resolution-model)
8. [Rollback Paths](#8-rollback-paths)
9. [English-First Language Policy](#9-english-first-language-policy)
10. [The Human System Owner Role](#10-the-human-system-owner-role)
11. [Design Decisions Deferred](#11-design-decisions-deferred)
12. [Lessons from Previous Versions](#12-lessons-from-previous-versions)

---

## 1. Purpose

Design decisions without recorded rationale become mysterious constraints. A protocol that says "do X" without explaining "because Y" invites two kinds of failure: maintainers change the rule without understanding its purpose and break something, or maintainers preserve the rule long after its original justification has expired and accumulate unnecessary rigidity.

This document exists to prevent both failures. For every major design decision in the TITLE Protocol v3.0, it records:

- **The decision itself** -- what the protocol specifies.
- **The rationale** -- why this choice was made over alternatives.
- **Alternatives considered** -- what other approaches were evaluated and why they were not selected.
- **Trade-offs accepted** -- what costs or limitations the decision introduces, and why those costs are acceptable.

Future maintainers and adopters should read this document alongside PROTOCOL.md. Where PROTOCOL.md defines the "what," this document explains the "why." If a design decision no longer makes sense given new evidence, changed requirements, or improved agent capabilities, this document provides the context needed to revise it intelligently rather than blindly.

---

## 2. The Dialectical Cycle

### Decision

The TITLE Protocol structures work as a four-phase cycle: Thesis, Antithesis, Synthesis, Execution. This cycle repeats for each iteration of a project. The structure is defined in PROTOCOL.md Section 3.

### Rationale: Why Dialectical Reasoning

The cycle is modeled on the Hegelian dialectical method -- a philosophical framework where a proposition (thesis) is challenged by its contradiction (antithesis), and the tension between them is resolved through a higher-order integration (synthesis). This structure is not merely aesthetic. It encodes a specific epistemological claim: that the quality of a design improves when it is subjected to genuine adversarial scrutiny and the resulting conflicts are explicitly reconciled before implementation begins.

Software engineering has long recognized the value of adversarial review. Code review, security audits, and red teaming are established practices. What the dialectical cycle adds is a formal sequencing constraint: the challenge must occur before reconciliation, and reconciliation must occur before implementation. This prevents the common failure mode where implementation begins before critical feedback has been integrated, forcing expensive rework.

### Why Four Phases

The four-phase structure was chosen after evaluating simpler alternatives.

**Why not two phases (Design + Build)?** A two-phase model -- one agent designs, the other builds -- lacks adversarial review. The builder implements whatever the designer specifies without structured opportunity to challenge assumptions, identify security flaws, or question feasibility. This is the pattern of most single-developer workflows, and its weaknesses are well-documented: designs that look coherent in the abstract fail when confronted with implementation realities, security vulnerabilities, and edge cases that the designer did not consider.

**Why not three phases (Design + Review + Build)?** A three-phase model adds adversarial review but collapses two distinct activities into a single transition: the jump from "here are the problems" to "now build it." This omits explicit conflict resolution. When a reviewer identifies a critical flaw, someone must decide whether to fix it, defer it, or reject the finding. In a three-phase model, this decision is either made implicitly (the designer silently updates the design) or deferred to the builder (who must guess the designer's intent). Neither outcome produces the documented, deliberate reconciliation that complex systems require.

**Why Synthesis is separate from Antithesis.** The separation between Antithesis (identifying problems) and Synthesis (reconciling them) is perhaps the most distinctive feature of the protocol. This separation matters because the skills required to find problems are different from the skills required to resolve them. Finding a security vulnerability requires adversarial thinking -- the ability to imagine how a system can be misused. Resolving that vulnerability requires integrative thinking -- the ability to balance security against performance, scope, and implementation cost. Combining these activities in a single phase forces the auditor to simultaneously attack and defend, which weakens both functions. By separating them, the protocol allows the Antithesis phase to be purely adversarial (find everything that could go wrong) and the Synthesis phase to be purely integrative (decide what to do about each finding).

**Why Execution is separate from Synthesis.** Separating design decisions from implementation prevents a common failure mode: the implementer making architectural decisions on the fly because the design was not fully specified. When Synthesis produces a complete, agreed-upon design with all blockers resolved, the Execution phase can focus on faithful implementation rather than ad-hoc design. This separation also enables clear accountability: if the implementation diverges from the design, the divergence is visible and can be traced to a specific decision or mistake.

### Alternatives Considered

| Alternative | Weakness | Why Rejected |
|-------------|----------|-------------|
| 2 phases (Design + Build) | No adversarial review | Single point of failure in design quality |
| 3 phases (Design + Review + Build) | No explicit conflict resolution | Findings get lost or silently ignored |
| 5+ phases (adding separate testing, deployment phases) | Coordination overhead | Diminishing returns; testing is part of Execution |
| Unstructured collaboration | No phase discipline | Agents talk past each other without convergence |

### Trade-off

The four-phase cycle is slower than a two-phase approach. Each cycle requires four distinct handoffs and four sets of quality gates. This overhead is justified because the protocol targets safety-critical agent workflows where the cost of an undetected flaw exceeds the cost of a structured review process.

---

## 3. Role Separation

### Decision

Claude Code serves as the Architect, Coordinator, and Narrator. GPT-5.3-Codex serves as the Builder, Red Team, and Executor. Roles are fixed and non-overlapping per phase. These roles are defined in PROTOCOL.md Section 2.

### Why Claude Code as Architect

Claude Code's strengths -- as documented in the multi-agent architecture research referenced in REFERENCES.md Section 2 -- lie in design reasoning, synthesis, documentation, and coordination. The "Context Engineering for Multi-Agent LLM Code Assistants" research and the "Claude Code Sub-agents: The 90% Performance Gain" case study both demonstrate that Claude performs best in the Planner/Coordinator role: analyzing requirements, structuring architectures, maintaining documentation, and integrating feedback from multiple sources.

The Architect role requires exactly these capabilities. It demands the ability to hold a complete system design in context, reason about trade-offs across multiple dimensions (security, performance, maintainability, scope), and produce coherent documentation that both humans and other agents can follow. Claude Code's extended context handling and reasoning depth make it well-suited for this role.

### Why GPT-5.3-Codex as Builder and Red Team

GPT-5.3-Codex's strengths -- as documented in the system card and capability analyses referenced in REFERENCES.md Section 1 -- center on concrete coding, tool usage, execution, vulnerability detection, and test generation. Community feedback and comparative analysis consistently position Codex as stronger in implementation-focused tasks: writing production code, generating test suites, identifying specific vulnerabilities, and producing proof-of-concept exploits.

The Builder and Red Team roles require exactly these capabilities. The Red Team role demands the ability to find concrete vulnerabilities, produce evidence (PoC code, benchmark data), and classify findings with technical precision. The Builder role demands the ability to translate a design specification into working code, write comprehensive tests, and iterate until quality gates pass.

### Why the Red Team and Builder Are the Same Agent

This is a deliberate design choice. The agent that audits the design for vulnerabilities is the same agent that implements the design. This means the implementer has firsthand knowledge of every weakness identified during the audit. An agent that has spent the Antithesis phase finding security flaws, concurrency issues, and resource leaks carries that adversarial knowledge into the Execution phase. It knows where the design is most fragile and can write defensive code and targeted tests accordingly.

The alternative -- separate Red Team and Builder agents -- would require the Builder to learn about vulnerabilities secondhand through documentation. Documentation is never as effective as direct experience. The agent that found the timing side-channel understands its mechanics deeply; the agent that reads about it in a report may miss subtleties.

### Why the Architect and Red Team Are Different Agents

If the same agent designed the system and audited it, the audit would suffer from confirmation bias. The designer has a cognitive investment in the design's correctness. Even well-intentioned self-review tends to follow the designer's original reasoning rather than challenging it from an independent perspective. The research on "Architecting Resilient LLM Agents" referenced in REFERENCES.md Section 3 specifically recommends privilege separation and role independence in agent architectures for this reason.

By assigning the design role to one agent and the adversarial review role to a different agent, the protocol ensures that the review is genuinely independent. Codex has no attachment to Claude Code's design decisions and can evaluate them purely on technical merit.

### Alternatives Considered

**Single agent for everything.** A single agent would design, audit, and implement the system. This eliminates coordination overhead but also eliminates adversarial tension. A single agent reviewing its own design is a well-known anti-pattern in security engineering. There is no genuine challenge, no independent perspective, and a single point of failure in reasoning. The multi-agent research referenced in REFERENCES.md consistently shows that role separation improves output quality by 40% or more compared to single-agent approaches.

**Three or more specialized agents.** A more granular decomposition -- separate Designer, Reviewer, Synthesizer, Implementer, Tester agents -- was considered. This approach was rejected because it introduces coordination complexity that exceeds its benefits for a two-agent protocol. With three or more agents, the handoff surface area grows quadratically: who hands off to whom, who resolves disagreements between agents 2 and 3, who owns shared artifacts. The current two-agent design keeps coordination simple and responsibility clear. Future versions may revisit this decision if agent orchestration tooling matures (see Section 11).

### Trade-off

Fixing roles means the protocol cannot dynamically reallocate work based on real-time agent performance. If Claude Code is particularly weak on a specific design task, the protocol does not allow Codex to step in and design instead. This rigidity is accepted because role clarity is more valuable than role flexibility in a structured protocol: clear boundaries prevent conflicts, enable accountability, and simplify state tracking.

---

## 4. Quality Gates

### Decision

Every phase transition is gated by a set of testable conditions that must all be satisfied before the protocol advances. There are 37+ specific conditions across the four gates. The System Owner verifies and approves each transition. Quality gates are defined in PROTOCOL.md Section 4.

### Why Testable Conditions Instead of Subjective Approval

Subjective approval ("does this look good?") is unreliable across different System Owners, different cycles, and different project contexts. What "looks good" to one reviewer may be incomplete to another. Testable conditions eliminate this ambiguity. Each condition is either satisfied or not: `BLUEPRINT.json` either contains a `security_policies` field with `authentication`, `authorization`, `data_handling`, and `encryption`, or it does not. There is no room for interpretation.

This approach has a secondary benefit: it enables future automation. Conditions expressed as testable checks can eventually be verified by scripts rather than manual review (see Section 11 on deferred decisions).

### Why Gates at Every Phase Transition

The protocol gates transitions between all four phases rather than gating only the final Execution phase. This design prevents cascading errors. If the BLUEPRINT is incomplete when it enters the Antithesis phase, the red-team review will be incomplete (the auditor cannot find flaws in requirements that were never specified). An incomplete audit produces an incomplete Synthesis (missing findings cannot be classified). An incomplete Synthesis produces an underspecified Execution (the builder implements against a design with unresolved gaps).

By gating every transition, the protocol catches problems at the earliest possible point. A missing field in BLUEPRINT.json is caught at the Thesis-to-Antithesis gate, not after two more phases of compounding incompleteness.

### Why 37+ Specific Conditions

The number of conditions is a consequence of comprehensiveness. Each condition addresses a specific failure mode observed in earlier protocol versions or anticipated from the protocol's structure:

- The 11 Thesis-to-Antithesis conditions ensure the BLUEPRINT is complete enough to audit meaningfully.
- The 6 Antithesis-to-Synthesis conditions ensure the audit covers all five areas with structured evidence.
- The 8 Synthesis-to-Execution conditions ensure every blocker has been explicitly addressed (no finding left in an ambiguous state).
- The 8 Execution-to-Thesis conditions ensure implementation is complete, tested, reviewed, and documented.

Reducing the number of conditions would create gaps. For example, removing the condition that `security_policies` must contain `authentication` would allow a BLUEPRINT to pass the gate without specifying how users are authenticated -- a critical omission for any system handling user data.

### Why the System Owner Approves Transitions

Even with testable conditions, the System Owner serves as the final verification layer. Conditions check structural completeness, but they cannot verify semantic correctness. A BLUEPRINT may satisfy all conditions (all fields present, all values non-placeholder) while still containing a logically flawed architecture. The System Owner's domain knowledge and judgment catch errors that structural checks cannot.

### Alternatives Considered

**Automated gate checking.** Running a script that validates all conditions automatically was considered and is planned for a future version. It was deferred because not all conditions are currently automatable (some require judgment, such as "System Owner has reviewed and approved the BLUEPRINT"), and implementing the automation requires tooling that is outside the scope of a pure protocol specification. See Section 11.

**Fewer, broader conditions.** Reducing the gates to a handful of high-level checks ("BLUEPRINT is complete," "Audit is complete") was considered. This was rejected because broad conditions are subjective and invite inconsistent application. "Complete" means different things to different reviewers.

### Trade-off

Thoroughness versus speed. The 37+ conditions slow down phase transitions. Verifying each condition takes time, and the System Owner must review each transition. This overhead is justified for safety-critical agent workflows where the cost of an undetected problem (a security vulnerability, a missing requirement, an unresolved blocker) far exceeds the cost of careful verification.

---

## 5. Stateless Architecture

### Decision

All protocol state lives in explicit files: `BLUEPRINT.json`, `STATE.yaml`, `AUDIT_LOG.md`, and Git history. No protocol state depends on conversation history, agent memory, or session continuity. These files are defined in PROTOCOL.md Sections 3 through 5 and the templates directory.

### Why Explicit State Files

AI agents are fundamentally stateless between sessions. Conversation histories expire. Context windows have finite capacity. Sessions can be interrupted by network failures, token limits, or user decisions. Any protocol that relies on conversational context for state continuity is fragile by design: if the session ends, the state is lost, and the protocol cannot resume.

Explicit files solve this problem completely. `STATE.yaml` records the current phase, the leading agent, all open blockers, and artifact SHAs. `BLUEPRINT.json` records the complete system design. `AUDIT_LOG.md` records all red-team findings. Any agent, at any time, can read these files and understand the exact state of the protocol without access to any prior conversation.

This design also supports agent replaceability. If a specific Claude Code session is lost, a new session can read `STATE.yaml`, determine that the protocol is in the Synthesis phase, read `BLUEPRINT.json` and `AUDIT_LOG.md`, and resume work without any loss of protocol state. The agent is interchangeable; the state lives in the files.

### Why Not Rely on Conversation History

Conversation history has three fundamental limitations for protocol state management:

1. **Expiration.** Sessions end. When they do, the conversation history becomes inaccessible to the next session. The new session starts with no knowledge of what happened before unless that knowledge is persisted externally.

2. **Context window limits.** Even within a single session, conversation history consumes context window capacity. A multi-cycle protocol can easily generate hundreds of messages. If the protocol relies on conversational context, the agent must carry the full history of all prior decisions in its context window, which is neither practical nor reliable.

3. **Agent statelessness.** Both Claude Code and GPT-5.3-Codex are stateless between invocations. They do not "remember" prior sessions. Any design that assumes cross-session memory is building on a foundation that does not exist.

### Why Git as the State Persistence Layer

Git was chosen as the persistence layer because it provides three properties that the protocol requires:

- **Ubiquity.** Git is the standard version control system for software projects. Every project that might adopt the TITLE Protocol already uses Git. No additional infrastructure is needed.
- **Auditability.** Git history records every change to every file with timestamps, authors, and diffs. The protocol's audit trail is preserved automatically and immutably in the repository history.
- **Rollback support.** Git enables precise restoration of any prior state through SHA-addressed commits. When the protocol needs to roll back to a previous phase (see Section 8), Git provides the mechanism to restore the exact artifact versions from that phase.

### Why SHA Tracking in STATE.yaml

The `last_artifacts` field in `STATE.yaml` records the Git SHA of the most recently committed version of each key artifact (`blueprint_sha`, `audit_log_sha`, `src_sha`). This serves two purposes:

1. **Integrity verification.** When the incoming agent reads STATE.yaml during a handoff (PROTOCOL.md Section 5.3), it can verify that the artifacts on disk match the SHAs recorded in the state file. If they do not match, someone modified an artifact outside the protocol's normal flow, and the incoming agent can flag this as a handoff failure.

2. **Silent modification prevention.** Without SHA tracking, an agent could modify BLUEPRINT.json between phases without detection. SHA tracking makes all changes traceable: if the SHA in STATE.yaml does not match the file on disk, the change is visible and must be explained.

### Trade-off

Verbosity versus reliability. Maintaining explicit state files is more verbose than relying on implicit conversational context. Every phase transition requires updating STATE.yaml with the new phase, owner, status, and artifact SHAs. Handoff notes must be written explicitly rather than relying on "the agent remembers what happened." This verbosity is accepted because reliability is non-negotiable for a protocol designed to coordinate two independent AI agents across multiple phases and cycles.

---

## 6. Artifact Ownership Matrix

### Decision

Each artifact has a designated primary author for each phase. Ownership is strict: only the designated agent may write to an artifact during a given phase. Other agents may read or review but not modify. The ownership matrix is defined in PROTOCOL.md Section 8.

### Why Strict Ownership Per Phase

Strict ownership prevents two categories of problems:

1. **Conflicting edits.** If both agents could modify `BLUEPRINT.json` during the Synthesis phase, they could produce conflicting changes without a clear mechanism for resolving them. Strict ownership eliminates this possibility: during Synthesis, only Claude Code writes to the BLUEPRINT. There is no merge conflict because there is only one writer.

2. **Unclear accountability.** When an artifact contains an error, strict ownership makes it clear who introduced the error and when. If `BLUEPRINT.json` has a flawed security policy, the error was introduced by Claude Code during either Thesis or Synthesis. If `src/` has a bug, GPT-5.3-Codex introduced it during Execution. This clarity simplifies debugging and post-mortem analysis.

### Why Claude Code Owns BLUEPRINT but Not src/

This is a direct consequence of role separation (Section 3). Claude Code is the Architect: it designs the system. The BLUEPRINT is the design artifact. GPT-5.3-Codex is the Builder: it implements the system. The `src/` directory is the implementation artifact. If Claude Code wrote implementation code, it would be acting outside its designated role, and the clear boundary between design and implementation would erode.

This boundary also prevents a specific failure mode: the architect using implementation as a vehicle for design decisions. When the architect can write code, design decisions that should be documented in the BLUEPRINT may instead be embedded in implementation details, invisible to the auditor and the System Owner. By keeping Claude Code out of `src/`, the protocol ensures that all design decisions are expressed in the BLUEPRINT where they can be reviewed and challenged.

### Why Codex Owns AUDIT_LOG Exclusively

Audit independence requires that the audited party has no influence over the audit report. If Claude Code could modify the AUDIT_LOG, it could soften findings, reclassify severities, or remove inconvenient evidence. By making GPT-5.3-Codex the exclusive author of the AUDIT_LOG, the protocol ensures that the red-team report reflects the auditor's genuine assessment without interference from the party whose work is being audited.

Claude Code does interact with the AUDIT_LOG during Synthesis, but its interaction is limited to reading and classifying findings -- it cannot modify the findings themselves.

### Why STATE.yaml Is Shared

Unlike other artifacts, `STATE.yaml` is a coordination mechanism rather than a content artifact. Both agents need to update it at phase boundaries: the outgoing agent records the phase transition, and the incoming agent reads and verifies it. Making STATE.yaml exclusive to one agent would require a mediator for every phase transition, adding unnecessary complexity.

The shared nature of STATE.yaml is safe because its schema is well-defined (PROTOCOL.md Section 5.2) and its changes are tracked in Git history. If either agent makes an unauthorized modification, the Git diff reveals it.

### Alternatives Considered

**Shared editing of all artifacts.** Allowing both agents to edit any artifact was considered and rejected. Without strict ownership, the protocol would need a merge strategy for every artifact, a conflict resolution process for simultaneous edits, and a way to attribute changes to specific agents. These mechanisms would add significant complexity for little benefit, since the current ownership model already ensures that each artifact has exactly one authoritative voice per phase.

### Trade-off

Strict ownership reduces flexibility. Claude Code cannot make a quick fix to implementation code even if it spots an obvious bug during review. Codex cannot update the BLUEPRINT even if it discovers a design improvement during Execution. These limitations are accepted because the alternative -- blurred ownership boundaries -- creates more problems than it solves. Claude Code should document the bug it found and defer the fix to Codex; Codex should propose the design improvement as input to the next cycle's Thesis phase.

---

## 7. Conflict Resolution Model

### Decision

The protocol defines a four-level escalation ladder for resolving disagreements between agents: Self-Resolve, Structured Argument, CONSENSUS_BLOCKED, and System Owner Decision. The model is defined in PROTOCOL.md Section 6 and expanded in CONSENSUS_RESOLUTION.md.

### Why a Four-Level Escalation Ladder

A graduated response system is more efficient than a binary approach (either resolve it yourselves or escalate to the human). Most disagreements between agents are minor: naming conventions, severity classifications, parameter ordering. These should be resolved quickly by the phase-leading agent without involving the System Owner. Escalating every minor disagreement to a human would make the protocol unusable -- the System Owner would spend more time resolving trivial disputes than doing productive work.

At the same time, some disagreements are genuinely significant: security trade-offs, architectural choices, scope disputes. These require formal documentation of both positions, explicit decision-making, and sometimes human judgment. A flat escalation model that treats all disagreements the same fails to match the response to the severity.

The four levels provide proportional response:

- **Level 1 (Self-Resolve):** For minor issues. Quick resolution by the phase lead. Minimal documentation. No System Owner involvement.
- **Level 2 (Structured Argument):** For significant issues. Both positions documented with evidence. Phase lead decides. Decision recorded in STATE.yaml.
- **Level 3 (CONSENSUS_BLOCKED):** For unresolvable issues. Phase paused. Both positions documented. System Owner notified.
- **Level 4 (System Owner Decision):** The human makes a binding decision. Final for the current cycle.

### Why CONSENSUS_BLOCKED Exists

CONSENSUS_BLOCKED prevents silent capitulation. Without an explicit mechanism for an agent to say "I fundamentally disagree and I am not willing to proceed," the protocol would default to the phase-leading agent's decision in every case. This means a legitimate security concern raised by Codex during Synthesis could be overridden by Claude Code with no recourse -- the finding would be rejected, the concern would be buried in handoff notes, and the System Owner would never know a significant disagreement occurred.

CONSENSUS_BLOCKED makes disagreement visible. When an agent triggers it, the phase pauses, both positions are documented in a structured format, and the System Owner is explicitly notified. This ensures that genuine risks are surfaced rather than suppressed.

The CONSENSUS_RESOLUTION.md document provides the detailed procedure, including the information required in handoff_notes, the notification process, timeline expectations, and the resumption procedure.

### Why the System Owner Is the Final Arbiter

AI agents should not make binding decisions about their own disagreements for two reasons:

1. **No ground truth.** When two agents disagree about a trade-off (security versus performance, scope versus schedule), neither has access to the full business context that determines which trade-off is correct. The System Owner has domain knowledge, business constraints, and risk tolerance that agents cannot infer from the BLUEPRINT alone.

2. **Accountability.** Someone must be accountable for the decision. AI agents cannot be held accountable in the way that a human project owner can. When the System Owner makes a binding decision, there is a responsible party who accepted the consequences of that choice.

### Alternatives Considered

**Majority vote with three or more agents.** A voting mechanism was considered but rejected because the protocol uses two agents, making majority voting impossible. Even with three or more agents, voting has a fundamental limitation: the majority can be wrong, especially on technical issues where the minority position is better supported by evidence.

**Always escalate to System Owner.** Escalating every disagreement to the System Owner was considered and rejected because it would overwhelm the human with trivial disputes. Level 1 and Level 2 handle the vast majority of disagreements without human involvement. The System Owner is reserved for cases where agent-level resolution has genuinely failed.

### Trade-off

The escalation ladder adds process overhead. Level 2 requires formal position statements. Level 3 requires structured documentation and pauses the phase. Level 4 requires the System Owner to evaluate technical arguments. This overhead is accepted because the alternative -- unstructured disagreement resolution -- produces either deadlocks or silent capitulation, both of which are worse than a structured process.

---

## 8. Rollback Paths

### Decision

Rollback is supported from every phase except Thesis. Rollback goes to the previous phase only (not arbitrary). Rollback uses Git SHAs recorded in STATE.yaml for precise artifact restoration. Rollback paths are defined in PROTOCOL.md Section 7.

### Why Rollback from Every Phase Except Thesis

Thesis is the starting phase of each cycle. There is no prior phase to roll back to. Rolling back from Thesis would mean rolling back to the previous cycle's Execution phase, which is a fundamentally different operation (reverting a cycle, not a phase) and is handled separately through cycle abandonment (PROTOCOL.md Section 7.3).

Every other phase can encounter conditions that invalidate its inputs:

- **Antithesis to Thesis:** The audit reveals that the BLUEPRINT is fundamentally incomplete, requiring design rework rather than finding classification.
- **Synthesis to Antithesis:** The Synthesis review reveals that the audit missed an entire area, requiring additional red-team analysis.
- **Synthesis to Thesis:** The combined findings require a major design overhaul that goes beyond blocker resolution.
- **Execution to Synthesis:** Implementation reveals design flaws not caught during audit.
- **Execution to Thesis:** Requirements change significantly during implementation, invalidating the design.

Each of these rollback paths addresses a specific, realistic failure mode.

### Why Rollback Goes to the Previous Phase Only

Allowing arbitrary rollback (for example, from Execution directly to Thesis, skipping Antithesis and Synthesis) would break state machine integrity. The protocol is a sequential state machine with well-defined transitions. Each phase depends on the outputs of the preceding phase. Skipping phases during rollback would leave the protocol in an inconsistent state: the rolled-back-to phase might reference artifacts from a phase that has been skipped over.

The exception -- Execution to Thesis -- exists because requirements changes invalidate the entire design, making intermediate phases irrelevant. Even in this case, the rollback goes to Thesis (the beginning) rather than to an arbitrary intermediate point.

### Why Git-Based Rollback Using SHAs

Git SHAs provide precise, deterministic artifact restoration. When STATE.yaml records `blueprint_sha: abc123`, rolling back to that state means checking out exactly the version of BLUEPRINT.json that was committed at SHA `abc123`. There is no ambiguity, no manual reconstruction, and no risk of partial restoration. The artifacts are restored exactly as they were at the recorded point.

This is superior to manual rollback, where an agent would need to "undo" its changes by editing files back to a prior state -- a process that is error-prone and incomplete.

### Trade-off

Rollback flexibility versus state machine simplicity. The protocol does not support arbitrary rollback (for example, rolling back from Execution to Antithesis in a single step). This means that a complex recovery might require two sequential rollbacks (Execution to Synthesis, then Synthesis to Antithesis). This limitation is accepted because state machine simplicity is more important than rollback convenience: every rollback transition is well-defined, predictable, and safe.

---

## 9. English-First Language Policy

### Decision

All protocol artifacts, code, agent-to-agent communication, and documentation use English as the default language. The System Owner may communicate in any language. Agent-to-agent artifacts are English only. The language policy is defined in LANGUAGE_POLICY.md and referenced in PROTOCOL.md Section 2.3.

### Why All Artifacts in English

Three factors drive this decision, as documented in LANGUAGE_POLICY.md Section 2:

1. **Token efficiency.** Major LLMs (Claude, GPT) use English-centric tokenizers. English text consumes fewer tokens for equivalent content than most other languages. In a protocol where context window management is critical, this efficiency matters.

2. **Model accuracy.** Benchmarks consistently show higher accuracy on technical tasks -- code generation, specification writing, tool invocation -- when instructions and artifacts are in English. Both Claude Code and GPT-5.3-Codex perform best when operating in English. Requiring English artifacts maximizes the quality of both agents' work.

3. **Toolchain compatibility.** Static analysis tools, CI systems, search tools, and documentation generators are built around English conventions. Using non-English identifiers, comments, or documentation creates friction with standard tooling.

### Why the System Owner Can Communicate in Any Language

The System Owner is a human, not a toolchain component. Forcing the System Owner to communicate exclusively in English would create an unnecessary accessibility barrier. Both Claude Code and GPT-5.3-Codex can understand instructions in multiple languages and produce English artifacts in response. The protocol accommodates this by specifying that agents internally translate non-English instructions to English for processing, produce all artifacts in English regardless of instruction language, and respond to the System Owner in the language used for the instruction (LANGUAGE_POLICY.md Section 4).

### Why Agent-to-Agent Artifacts Are English Only

Agent-to-agent artifacts (BLUEPRINT.json, AUDIT_LOG.md, STATE.yaml, specs/) are consumed by both agents and by the quality gate verification process. If these artifacts were in different languages depending on which agent produced them, the consuming agent would need to translate before processing, introducing potential translation errors and token overhead. English-only artifacts ensure consistent, unambiguous communication between agents.

### Alternatives Considered

**Auto-detect and match System Owner language.** The protocol could have specified that all artifacts should be written in whatever language the System Owner uses. This was rejected for three reasons: it would create inconsistency across projects (some in Korean, some in English, some mixed), it would reduce model accuracy on technical tasks, and it would introduce tool incompatibility with English-centric CI/CD and analysis tools.

### Trade-off

The English-first policy may disadvantage System Owners whose primary language is not English. While the System Owner can communicate in any language, reading English-only artifacts requires English comprehension. This limitation is accepted because the alternative -- multilingual artifacts -- would degrade agent performance, complicate tooling integration, and introduce translation errors. The protocol mitigates this limitation by allowing agents to provide localized summaries when the System Owner requests them (LANGUAGE_POLICY.md Section 4).

---

## 10. The Human System Owner Role

### Decision

A human System Owner is required in the protocol loop. The System Owner approves phase transitions, resolves deadlocks, provides requirements, and makes binding decisions. The System Owner does not directly edit artifacts. The role is defined in PROTOCOL.md Section 2.3.

### Why a Human Is Required in the Loop

The TITLE Protocol coordinates two AI agents making design and implementation decisions about software systems. These decisions have real-world consequences: security vulnerabilities affect users, architectural choices affect long-term maintainability, scope decisions affect project timelines and budgets. A human in the loop provides three things that AI agents currently cannot:

1. **Safety.** A human can catch errors of judgment that both agents miss. AI agents, despite their capabilities, can agree on a flawed approach if both share the same blind spot. The System Owner provides an independent check from outside the agent system.

2. **Accountability.** Someone must be responsible for the decisions made during the protocol. AI agents cannot sign contracts, accept liability, or make business commitments. The System Owner bears this responsibility and therefore has the authority to override agent decisions.

3. **Domain knowledge.** The System Owner understands business context, user needs, regulatory constraints, and organizational priorities that are not captured in the BLUEPRINT. These factors influence design trade-offs in ways that agents cannot fully evaluate from technical specifications alone.

### Why the System Owner Does Not Directly Edit Artifacts

If the System Owner could directly edit BLUEPRINT.json, AUDIT_LOG.md, or source code, the clear ownership boundaries defined in the artifact ownership matrix (Section 6) would be violated. An edit by the System Owner would be outside the normal phase flow, potentially bypassing quality gates and audit processes. It would also create confusion about artifact authorship: was this change made by Claude Code during Synthesis, or by the System Owner outside the protocol?

Instead, the System Owner communicates requirements and decisions through the protocol's defined channels: providing requirements at the start of a cycle, approving or rejecting phase transitions, and making binding decisions during CONSENSUS_BLOCKED states. The responsible agent then incorporates the System Owner's input into the appropriate artifact through the normal phase process.

### Why the System Owner Only Approves or Rejects Transitions

This constraint implements the principle of minimal intervention with maximum oversight. The System Owner sees every phase transition and can block any transition that does not meet quality gate conditions. This provides comprehensive oversight. At the same time, the System Owner does not need to participate in every design discussion, every audit finding, or every implementation decision. This minimizes the human effort required to supervise the protocol.

The result is a supervisory role that is sustainable for a human operator. The System Owner checks in at four defined points per cycle (the four phase transitions) rather than continuously monitoring agent activity. Between transitions, the agents operate autonomously within their defined roles and constraints.

### Trade-off

Autonomy versus safety. A fully autonomous protocol (no human in the loop) would be faster and require less human attention. This was rejected for v3.0 because the protocol is new, the agent ecosystem is rapidly evolving, and the consequences of unchecked agent errors are significant. As the protocol matures and agents demonstrate consistent reliability, future versions may relax the human oversight requirements for lower-risk transitions (see Section 11).

---

## 11. Design Decisions Deferred

The following design decisions were evaluated and deliberately deferred to future protocol versions. Each deferral includes the rationale for deferring rather than deciding now.

### Automated Quality Gate Checking

**What:** A script or tool that automatically verifies all quality gate conditions before allowing a phase transition.

**Why deferred:** Not all conditions are currently automatable. Conditions like "System Owner has reviewed and approved the BLUEPRINT" require human judgment. Building partial automation (checking structural conditions but not semantic ones) could create a false sense of completeness. Additionally, the protocol is a pure specification -- introducing executable tooling changes its nature. Deferred to a future version where a companion tool can be developed alongside the protocol.

### Three or More Agent Support

**What:** Extending the protocol to support three or more specialized agents (for example, separate Red Team and Builder agents, or a dedicated Testing agent).

**Why deferred:** The current two-agent design achieves the essential goals (adversarial review, role separation, clear accountability) with minimal coordination complexity. Adding agents increases the handoff surface area quadratically. The benefits of finer-grained specialization do not yet outweigh the coordination costs. Deferred until agent orchestration tooling matures and practical experience with the two-agent model identifies specific capability gaps that a third agent would address.

### Real-Time Collaboration Mode

**What:** A mode where both agents work simultaneously on complementary aspects of the same phase, rather than the current sequential handoff model.

**Why deferred:** Real-time collaboration requires shared state management, conflict resolution for concurrent edits, and communication mechanisms that are more complex than the current async handoff model. The async model is simpler, more robust, and easier to reason about. Deferred until the benefits of real-time collaboration (speed) are demonstrated to outweigh its costs (complexity, conflict risk).

### Plugin and Extension System

**What:** A mechanism for consumer projects to define custom phases, custom quality gate conditions, or custom artifact types.

**Why deferred:** The protocol is in its first English-language version (v3.0). Introducing an extension system before the core protocol has been validated in practice would be premature abstraction. The core protocol needs to prove itself across multiple consumer projects before the right extension points can be identified. Building the wrong extension system now would constrain future evolution more than having no extension system at all.

### CI/CD Integration

**What:** Built-in integration with continuous integration and deployment pipelines (GitHub Actions, GitLab CI, Jenkins).

**Why deferred:** The protocol is tool-agnostic by design. It specifies what must happen (phase transitions, quality gates, artifact validation) without prescribing how it happens. CI/CD integration would bind the protocol to specific platforms and tooling choices. Consumer projects can implement CI/CD integration themselves using the protocol's well-defined quality gate conditions. Deferred to a companion project or future version that provides reference implementations without mandating them.

---

## 12. Lessons from Previous Versions

### v1.0 Lessons

The first version of the protocol was informal and underspecified.

**No quality gates.** Phase transitions were based on subjective judgment ("is the design good enough?"). This led to inconsistent quality across phases. Designs that were "good enough" for one reviewer were insufficient for another. The lack of testable conditions meant there was no objective standard for readiness.

**No state tracking.** Protocol state lived in conversation history and human memory. When sessions expired or participants changed, the protocol state was lost. Resuming work required reconstructing context from scattered artifacts and recollections.

**No formal roles.** Agents had loosely defined responsibilities that shifted based on convenience. This led to the architect writing implementation code, the builder making design decisions, and no clear accountability for any artifact.

**Lesson applied to v3.0:** Formal quality gates with testable conditions (Section 4). Explicit state tracking in STATE.yaml (Section 5). Fixed, non-overlapping roles with artifact ownership (Sections 3 and 6).

### v2.0 Lessons

The second version addressed some v1.0 weaknesses but introduced new limitations.

**Korean-only documentation limited adoption.** All protocol documents were written in Korean. This limited the pool of potential adopters to Korean-speaking teams and reduced the effectiveness of both AI agents, which perform technical tasks more accurately in English. International collaboration was effectively impossible.

**No conflict resolution mechanism.** When agents disagreed, there was no structured process for resolving the disagreement. This led to both failure modes described in CONSENSUS_RESOLUTION.md Section 1: deadlocks (agents refused to proceed) and silent capitulation (one agent yielded without documenting its concerns).

**No rollback support.** When a phase produced an unacceptable result, the only option was to continue forward or abandon the cycle entirely. There was no mechanism for returning to a previous phase with preserved context.

**Lesson applied to v3.0:** English-first language policy (Section 9, LANGUAGE_POLICY.md). Comprehensive conflict resolution with four-level escalation (Section 7, CONSENSUS_RESOLUTION.md). Rollback paths from every phase except Thesis with Git-based restoration (Section 8).

### v3.0 Improvements Summary

| Area | v1.0 | v2.0 | v3.0 |
|------|------|------|------|
| Quality gates | None | Informal | 37+ testable conditions |
| State tracking | Conversation history | Partial files | STATE.yaml with SHA tracking |
| Language | Mixed | Korean only | English-first with multilingual System Owner support |
| Conflict resolution | None | None | 4-level escalation ladder |
| Rollback | None | None | Phase-level rollback with Git SHAs |
| Role definition | Informal | Partial | Fixed roles with artifact ownership matrix |
| Audit structure | None | Informal | 5-area template with structured findings |
| Handoff procedure | None | Informal | Pre/post checklist with SHA verification |

---

**Cross-References:**

- [PROTOCOL.md](PROTOCOL.md) -- Complete protocol specification
- [REFERENCES.md](REFERENCES.md) -- Research references supporting design decisions
- [LANGUAGE_POLICY.md](LANGUAGE_POLICY.md) -- English-first language conventions
- [CONSENSUS_RESOLUTION.md](CONSENSUS_RESOLUTION.md) -- Detailed conflict resolution procedures
- [GLOSSARY.md](GLOSSARY.md) -- Protocol terminology
- [templates/STATE.yaml](templates/STATE.yaml) -- State tracking template with blocker schema
- [templates/BLUEPRINT.json](templates/BLUEPRINT.json) -- Design document template

---

*TITLE Protocol v3.0 -- Design Rationale*
*Version 1.0.0 | 2026-02-15*
