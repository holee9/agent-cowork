# Scaling the TITLE Protocol

Guidance for scaling the TITLE Protocol v3.0 to larger projects, longer timelines, multiple parallel workstreams, and team coordination.

**Version:** 1.0.0
**Date:** 2026-02-15
**Expands:** PROTOCOL.md (execution loop, artifacts, and phase gates)

---

## Table of Contents

1. [Purpose](#1-purpose)
2. [Single-Stream Scaling](#2-single-stream-scaling)
3. [Multi-Feature Scaling](#3-multi-feature-scaling)
4. [Team Scaling](#4-team-scaling)
5. [Artifact Management at Scale](#5-artifact-management-at-scale)
6. [Quality Assurance at Scale](#6-quality-assurance-at-scale)
7. [Performance Considerations](#7-performance-considerations)
8. [Scaling Patterns](#8-scaling-patterns)
9. [Migration Guide](#9-migration-guide)
10. [Limitations and Boundaries](#10-limitations-and-boundaries)

---

## 1. Purpose

The TITLE Protocol, as defined in PROTOCOL.md, describes a single dialectical cycle: Thesis, Antithesis, Synthesis, Execution. GETTING_STARTED.md walks through the first cycle of a single feature. For many projects, this is sufficient.

This guide is for situations that go beyond that baseline:

- **Long-running projects** that accumulate ten, twenty, or more cycles on a single feature stream and need strategies for managing growing artifact history.
- **Multi-feature projects** where two or more features are developed in parallel, each following its own cycle cadence, and must eventually integrate into a shared codebase.
- **Team projects** where multiple System Owners, multiple agent pairs, or both are involved, and coordination conventions are needed to prevent conflicts.
- **Large codebases** where the volume of specifications, audit logs, and implementation code begins to strain the protocol's artifact model.

If your project is a single feature built in one to three cycles with one System Owner and one agent pair, you do not need this document. Start with GETTING_STARTED.md and PROTOCOL.md instead.

---

## 2. Single-Stream Scaling

Single-stream scaling addresses projects that develop one feature at a time, in strict sequential order, but accumulate many cycles over the life of the project. The core challenge is managing artifact growth while preserving the auditability that the protocol provides.

### 2.1 Managing BLUEPRINT.json Evolution

BLUEPRINT.json is the central design artifact, authored by Claude Code during the Thesis phase and updated during Synthesis. As cycles accumulate, the BLUEPRINT grows to reflect an increasingly complex system.

**Incremental updates versus full rewrites.** In most cycles, Claude Code should update the existing BLUEPRINT incrementally. Each cycle typically adds new io_contracts, extends security_policies, or adjusts non_functional_requirements. A full rewrite of BLUEPRINT.json is warranted only when the system's architecture changes fundamentally, such as a migration from monolith to microservices, a change of primary language or framework, or a complete redesign of the security model.

**Version tracking within BLUEPRINT.json.** The BLUEPRINT template includes a `version` field and a `cycle` field. Claude Code should increment the version following semantic conventions: increment the minor version for additive changes within a cycle and the major version for breaking changes to io_contracts or security_policies. The cycle field should always match the current cycle number in STATE.yaml.

**Preserving design rationale.** Over many cycles, the reasoning behind early design decisions may become unclear. Claude Code should maintain a design rationale section or document within the BLUEPRINT or in a companion file such as `docs/design-rationale.md`. Each significant decision should reference the cycle in which it was made and the blocker (if any) that motivated it.

### 2.2 AUDIT_LOG.md History

Each Antithesis phase produces a new AUDIT_LOG.md. Over many cycles, a single AUDIT_LOG.md file becomes unwieldy if prior audits are simply appended.

**Archiving completed audits.** At the start of each new cycle's Antithesis phase, rename the previous cycle's AUDIT_LOG.md to include the cycle number. The recommended naming convention is:

- `AUDIT_LOG_cycle1.md`
- `AUDIT_LOG_cycle2.md`
- `AUDIT_LOG_cycle3.md`

The current cycle's audit is always named `AUDIT_LOG.md` without a suffix. This convention ensures that the standard quality gate references in PROTOCOL.md Section 4 and CHECKLIST.md continue to work without modification.

**Storage location.** Archived audit logs should be moved to an `archive/audits/` directory within the `.aisync/` directory. The current AUDIT_LOG.md remains at its standard location.

**Cross-referencing findings across cycles.** Findings sometimes recur or evolve across cycles. When GPT-5.3-Codex documents a finding that relates to a previous cycle's finding, it should reference the original finding ID and cycle number. For example, a finding might note: "This finding revisits AUD-003 from Cycle 2, which was classified as DEFERRED in Synthesis."

### 2.3 STATE.yaml Cycle Progression

STATE.yaml tracks the protocol's current state. Understanding what persists and what resets across cycle boundaries is essential for long-running projects.

**What persists across cycles:**

- The `cycle` counter, which increments after each complete four-phase pass.
- The `blockers` array, which accumulates all blocker entries from all cycles. DEFERRED blockers from previous cycles remain visible as a backlog.
- The `last_artifacts` SHA references, which reflect the most recent committed versions.

**What resets at cycle boundaries:**

- The `phase` field resets to THESIS.
- The `owner_model` field resets to Claude.
- The `status` field resets to OK (assuming the previous cycle completed successfully).
- The `handoff_notes` field is overwritten with context for the new cycle.
- The `blocking_model` field resets to null.

**Blocker array management.** Over many cycles, the blockers array grows. Completed blockers (those with status ACCEPTED, DEFERRED with resolution, or REJECTED) should remain in the array for traceability but may be moved to a `resolved_blockers` section or a companion archive file if the array becomes unwieldy. The key constraint is that DEFERRED blockers must remain visible until they are addressed.

### 2.4 specs/ Directory Growth

The specs/ directory contains interface and contract definitions authored by Claude Code. As the system's interface surface expands across cycles, this directory requires organization.

**Organizing by domain or module.** When the specs/ directory grows beyond ten to fifteen files, introduce subdirectories organized by domain area. For example:

- `specs/auth/` for authentication and authorization contracts
- `specs/api/` for external API endpoint specifications
- `specs/events/` for event schemas and message contracts
- `specs/data/` for data model specifications

**Spec versioning.** Each spec file should include a header or metadata section indicating the cycle in which it was created and the cycle in which it was last modified. This supports traceability when reviewing how an interface evolved over time.

**Deprecation handling.** When a spec is superseded by a new version, mark it as deprecated rather than deleting it. Move deprecated specs to a `specs/deprecated/` directory. This preserves the audit trail and allows GPT-5.3-Codex to reference historical contracts during red-team reviews.

### 2.5 Deferred Blocker Management

DEFERRED blockers represent acknowledged issues that the team has chosen not to address in the current cycle. Without active tracking, deferred blockers accumulate silently and become technical debt.

**Tracking deferred blockers.** Maintain a deferred blocker register, either within STATE.yaml (as entries in the blockers array with status DEFERRED) or as a companion document. Each deferred blocker entry must include:

- The original finding ID and cycle of origin
- The documented reason for deferral (as required by PROTOCOL.md Section 4.3)
- The planned cycle for resolution
- The current status (still deferred, being addressed, resolved)

**Review cadence.** At the start of each new cycle's Thesis phase, Claude Code should review all outstanding DEFERRED blockers and determine which ones should be addressed in the current cycle. This review should be documented in the handoff notes.

**Escalation of aging deferrals.** If a DEFERRED blocker has been deferred for more than three consecutive cycles without resolution, it should be escalated to the System Owner during the next Thesis phase. The System Owner decides whether to prioritize it, continue deferring with explicit justification, or reject it with documented rationale.

### 2.6 Git Branching Strategy

The protocol relies on Git for artifact versioning and state tracking. As cycles accumulate, a clear branching strategy prevents confusion.

**Option A: One branch per cycle.** Create a branch for each cycle (e.g., `cycle/3`, `cycle/4`). Each branch is merged to the main branch upon cycle completion. This approach provides clean isolation between cycles and makes it easy to review the changes introduced by a specific cycle. It works well for projects where cycles are relatively self-contained.

**Option B: Feature branches within cycles.** Use a single long-lived development branch with feature branches for significant changes within a cycle. This approach is more familiar to teams accustomed to Git Flow or trunk-based development. The trade-off is that cycle boundaries are less visible in the branch structure.

**Recommended approach.** For most projects, Option A (one branch per cycle) aligns naturally with the protocol's cycle model. The branch name encodes the cycle number, making it easy to correlate Git history with STATE.yaml cycle progression. Merge to main at the end of each Execution phase, after the System Owner approves the cycle.

---

## 3. Multi-Feature Scaling

Multi-feature scaling addresses projects where two or more features are developed in parallel, each following its own cycle of Thesis, Antithesis, Synthesis, and Execution. This introduces coordination challenges that do not exist in single-stream development.

### 3.1 Feature Isolation Strategy

Each feature stream needs its own protocol state to track phases, blockers, and handoffs independently.

**Separate .aisync/ directories per feature.** The recommended approach is to create a feature-scoped .aisync/ directory for each parallel feature stream. For example:

- `.aisync/feature-auth/` containing BLUEPRINT.json, AUDIT_LOG.md, STATE.yaml, and CHECKLIST.md for the authentication feature
- `.aisync/feature-payments/` containing the same set of artifacts for the payments feature
- `.aisync/shared/` containing shared artifacts such as GLOSSARY, common security policies, and shared tech stack definitions

Each feature directory is self-contained and follows the standard protocol structure. The shared directory provides common ground that all feature streams reference.

**Shared state versus isolated state.** Feature-specific artifacts (BLUEPRINT, AUDIT_LOG, feature-scoped specs) should be isolated per feature. Cross-cutting concerns (security policies that apply to all features, shared tech stack constraints, project-wide non-functional requirements) should be maintained in the shared directory and referenced by each feature's BLUEPRINT.

### 3.2 BLUEPRINT.json Partitioning

When multiple features are developed in parallel, the question arises whether to maintain one monolithic BLUEPRINT or feature-scoped BLUEPRINTs.

**Feature-scoped BLUEPRINTs.** Each feature maintains its own BLUEPRINT.json within its .aisync/ directory. This BLUEPRINT contains only the io_contracts, security_policies extensions, and non_functional_requirements specific to that feature. It references the shared BLUEPRINT for project-wide settings.

**Shared BLUEPRINT.** A project-level BLUEPRINT.json in `.aisync/shared/` defines the common foundation: project_name, core_principles, base constraints, tech_stack, shared security_policies, and project-wide non_functional_requirements. Feature BLUEPRINTs inherit from and extend this shared foundation.

**Referencing convention.** Feature BLUEPRINTs should include a field such as `extends` or a comment referencing the shared BLUEPRINT path. This makes the inheritance relationship explicit and allows agents to load the shared context when working on a feature.

### 3.3 Conflict Prevention

When two features modify overlapping areas, conflicts can arise at the artifact level, the specification level, or the implementation level.

**IO contract conflicts.** If two features define or modify the same endpoint or interface, a conflict exists. To prevent this, maintain a registry of io_contracts in the shared BLUEPRINT. Before a feature's Thesis phase begins, Claude Code should check the shared contract registry to identify overlapping endpoints. If overlap exists, the System Owner must decide which feature owns the endpoint and how the other feature integrates with it.

**Security policy conflicts.** Parallel features may propose incompatible security policies. For example, one feature may require API key authentication while another requires OAuth 2.0. The shared security_policies in the project-level BLUEPRINT serve as the authoritative standard. Feature-level security extensions must be compatible with the shared baseline.

**Specification conflicts.** When two features modify the same spec file, use a lock-based or ownership-based approach. Each spec file has one owning feature at a time. If both features need to modify the same spec, one feature must complete its Synthesis phase and commit the updated spec before the other feature begins modifying it. Alternatively, the spec can be split into feature-scoped components that are composed at integration time.

### 3.4 Merge Strategy

Parallel feature cycles must eventually integrate into a single codebase. The merge strategy determines when and how this integration happens.

**Periodic sync points.** Define regular sync points where all active feature streams merge their completed cycles into the shared main branch. A sync point involves:

- Verifying that all feature BLUEPRINTs are compatible with the shared BLUEPRINT
- Resolving any specification conflicts between features
- Running integration tests that exercise cross-feature interactions
- Updating the shared BLUEPRINT to reflect the combined system state

**Integration cycle.** After a sync point, run a dedicated integration cycle (Thesis through Execution) focused solely on cross-feature concerns. Claude Code reviews the combined BLUEPRINT for consistency, GPT-5.3-Codex audits cross-feature interactions for security and concurrency issues, and the integration is validated end-to-end during Execution.

**Merge order.** When merging multiple features, merge them in order of dependency. Features with fewer dependencies on other features should merge first. This reduces the complexity of conflict resolution during the merge process.

### 3.5 System Owner Coordination

With multiple active feature cycles, the System Owner's coordination burden increases.

**Transition approval scheduling.** The System Owner must approve phase transitions for each feature independently. To prevent bottlenecks, establish a regular review cadence (e.g., transition reviews twice per week) rather than reviewing each transition ad hoc.

**Cross-feature priority decisions.** When two features compete for the same resource (e.g., both want to modify a shared component), the System Owner arbitrates priority. The decision should be recorded in the shared directory's state tracking.

**Delegation to domain owners.** For projects with more than three parallel features, the System Owner may delegate transition approval authority to domain owners (see Section 4, Team Scaling). The overall System Owner retains authority over cross-feature decisions and CONSENSUS_BLOCKED resolution.

---

## 4. Team Scaling

Team scaling addresses projects where multiple humans, multiple agent pairs, or both participate in the protocol. This extends the single System Owner and single agent pair model defined in PROTOCOL.md Section 2.

### 4.1 Multi-System-Owner Model

As projects grow, a single System Owner becomes a bottleneck. The protocol supports a hierarchical ownership model.

**Domain owners.** Each major domain (e.g., authentication, payments, user interface, data pipeline) has a designated domain owner. The domain owner has authority to:

- Approve phase transitions for cycles within their domain
- Resolve Level 2 Structured Arguments within their domain
- Provide requirements and priority guidance for their domain's features

**Overall project owner.** One person retains overall project authority. The project owner:

- Resolves CONSENSUS_BLOCKED states that span multiple domains
- Approves integration cycles and sync points
- Makes binding decisions when domain owners disagree
- Maintains the shared BLUEPRINT and project-wide policies

**Authority boundaries.** Domain owners may not modify shared security_policies, project-wide non_functional_requirements, or core_principles without project owner approval. Domain owners have full authority over their domain's feature-scoped artifacts.

### 4.2 Agent Pair Isolation

Each feature stream should operate with its own dedicated agent pair (one Claude Code instance and one GPT-5.3-Codex instance). This isolation provides several benefits.

**Context separation.** Each agent pair maintains its own conversation context, loaded with only the artifacts relevant to its feature. This prevents context pollution where one feature's design decisions inadvertently influence another feature's audit or implementation.

**Independent phase progression.** Each agent pair progresses through the four phases at its own pace. One pair may be in Synthesis while another is in Execution. The pairs do not need to synchronize their phase progression except at designated sync points.

**Failure isolation.** If one agent pair encounters a CONSENSUS_BLOCKED state or a rollback, it does not affect other pairs. Each pair's protocol state is tracked in its own STATE.yaml.

### 4.3 Cross-Pair Communication

While agent pairs operate independently, findings from one pair may be relevant to others. Cross-pair communication should be structured and intentional.

**Shared findings register.** Maintain a `shared-findings.md` document in the `.aisync/shared/` directory. When an agent pair discovers a finding that affects other features (e.g., a security vulnerability in a shared component), the finding is recorded in the shared register with a reference to the originating feature, cycle, and blocker ID.

**Broadcast protocol.** When a finding is added to the shared register, all active agent pairs should be notified at their next phase transition. The incoming agent reads the shared findings register as part of its post-handoff checklist (PROTOCOL.md Section 5.3) and evaluates whether any shared findings affect its feature.

**Design decision propagation.** When the shared BLUEPRINT is updated (e.g., a new security policy is added), all active feature BLUEPRINTs must be reviewed for compatibility. This review happens at the next Thesis phase for each feature or at a sync point, whichever comes first.

### 4.4 Shared Artifact Conventions

When multiple agent pairs and System Owners work on the same project, shared artifacts require explicit conventions.

**Common GLOSSARY.** Maintain a single GLOSSARY.md at the project level. All agent pairs use the same terminology. Domain-specific terms are added to the glossary as they are introduced, with the originating domain noted.

**Shared security_policies.** Project-wide security policies are defined in the shared BLUEPRINT and are binding on all features. Feature-specific security extensions (e.g., additional access controls for a payments feature) are defined in the feature BLUEPRINT but must not contradict the shared baseline.

**Unified tech_stack.** The shared BLUEPRINT defines the project's technology stack. Individual features may not introduce new languages, frameworks, or infrastructure components without project owner approval. This prevents technology sprawl and ensures integration compatibility.

**Artifact naming conventions.** All artifacts follow the naming conventions defined in LANGUAGE_POLICY.md. Branch names, commit messages, and artifact file names use English. Feature-scoped artifacts include the feature name in the directory path, not in the file name. The file names within each feature directory match the standard template names (BLUEPRINT.json, AUDIT_LOG.md, STATE.yaml, CHECKLIST.md).

### 4.5 Decision Authority

With multiple System Owners, clear decision authority prevents deadlocks.

**Within a single domain.** The domain owner has final authority. CONSENSUS_BLOCKED states are resolved by the domain owner following the procedure in CONSENSUS_RESOLUTION.md.

**Across domains.** When a decision affects multiple domains, the project owner decides. Domain owners present their positions, and the project owner makes a binding decision following the System Owner Decision Framework in CONSENSUS_RESOLUTION.md Section 4.

**When domain owners disagree.** If two domain owners disagree on a cross-domain issue, the project owner resolves the dispute. The resolution is documented in the shared directory's state tracking with both positions and the project owner's rationale.

---

## 5. Artifact Management at Scale

As the number of cycles, features, and team members grows, artifact management becomes a first-class concern.

### 5.1 Directory Structure for Large Projects

For projects with multiple features and many cycles, the following directory structure provides clear organization:

A project-level `.aisync/` directory contains a `shared/` subdirectory for the project-wide BLUEPRINT.json, shared security policies, and shared GLOSSARY. Each feature has its own subdirectory under `.aisync/`, such as `.aisync/feature-auth/` or `.aisync/feature-payments/`, containing its own BLUEPRINT.json, AUDIT_LOG.md, STATE.yaml, and CHECKLIST.md. An `archive/` subdirectory under each feature directory stores completed cycle artifacts.

The `specs/` directory at the project root is organized by domain, with subdirectories such as `specs/auth/`, `specs/payments/`, and `specs/shared/`. The `src/` and `tests/` directories follow the same domain-based organization.

A `docs/` directory at the project root contains a `cycles/` subdirectory for cycle summary documents (one per completed cycle), a `decisions/` subdirectory for design decision records, and an `architecture/` subdirectory for system-wide architecture documentation.

### 5.2 Naming Conventions for Multi-Cycle Artifacts

Consistent naming prevents confusion as artifacts accumulate.

**Audit logs.** Current cycle: `AUDIT_LOG.md`. Archived: `AUDIT_LOG_cycle{N}.md` where N is the cycle number.

**Cycle summaries.** `docs/cycles/cycle-{N}-summary.md` where N is the cycle number, zero-padded to two digits for sorting (e.g., `cycle-01-summary.md`, `cycle-12-summary.md`).

**Decision records.** `docs/decisions/DR-{NNN}-{short-title}.md` where NNN is a sequential number and short-title is a lowercase hyphenated description.

**Feature branches.** `feature/{feature-name}/cycle-{N}` for feature-scoped cycle branches, or `cycle/{N}` for single-stream projects.

### 5.3 Archiving Completed Cycles

Not all artifacts from completed cycles need to remain in the active project directory. Archiving reduces clutter while preserving the audit trail.

**What to keep in the active directory:**

- The current BLUEPRINT.json (always the latest version)
- The current STATE.yaml (always the latest state)
- Active specs/ (current versions of all interface specifications)
- The current AUDIT_LOG.md (for the active cycle)

**What to archive:**

- Previous cycles' AUDIT_LOG files (move to archive/audits/)
- Previous cycles' BLUEPRINT snapshots (tag in Git rather than maintaining separate files)
- Resolved blocker entries older than five cycles (move to an archive file, keeping a summary reference)
- Deprecated specs (move to specs/deprecated/)

**What to rely on Git for:**

- Full artifact history (Git log and diff provide complete traceability)
- BLUEPRINT evolution (Git blame shows when each field was added or modified)
- Point-in-time snapshots (Git tags at cycle boundaries provide exact state reconstruction)

### 5.4 Git Repository Structure

**Monorepo approach.** For most projects, a single repository containing all features, artifacts, and code is the simplest approach. The directory structure provides isolation between features. Git tags mark cycle boundaries. This approach works well for up to approximately ten parallel features.

**Multi-repo approach.** For very large projects with independently deployable components, each component may have its own repository with its own .aisync/ directory. A coordination repository contains the shared BLUEPRINT, shared security policies, and cross-component specifications. This approach introduces synchronization complexity but provides stronger isolation and independent release cadences.

**Recommended approach.** Start with a monorepo. Migrate to multi-repo only when the repository size, build times, or release cadence requirements make it necessary. See Section 9 for migration guidance.

### 5.5 Documentation Indexing

As the project grows, maintaining a master document map becomes essential for discoverability.

**Master index file.** Create a `docs/INDEX.md` file that lists all documentation artifacts organized by category: protocol documents, cycle summaries, decision records, architecture documents, and archived materials. Update this index at the end of each cycle's Execution phase.

**Cross-reference conventions.** When one document references another, use relative paths from the project root. This ensures references remain valid as the project structure evolves. For cross-cycle references, include the cycle number in the reference (e.g., "as decided in DR-012 during Cycle 5").

---

## 6. Quality Assurance at Scale

Quality assurance in a single cycle is governed by the phase transition gates defined in PROTOCOL.md Section 4. At scale, additional quality concerns emerge that span multiple cycles and features.

### 6.1 Regression Testing Across Cycles

Each cycle's Execution phase produces tests. As cycles accumulate, the cumulative test suite must be maintained.

**Test suite growth management.** All tests from previous cycles remain in the test suite unless explicitly deprecated. The full test suite runs during each Execution phase to ensure new changes do not break previous implementations.

**Test categorization.** Categorize tests by the cycle and feature that introduced them. This allows targeted test runs during development (running only the current feature's tests for rapid feedback) and full regression runs before cycle completion.

**Regression test ownership.** When a new cycle modifies code that existing tests cover, Claude Code's review during Execution should verify that:

- All pre-existing tests still pass
- New tests cover the modified behavior
- No test was deleted or weakened without documented justification

### 6.2 Cumulative Security Audit

Security posture should improve or at least remain stable across cycles. Without tracking, security regressions can occur as new features introduce new attack surface.

**Security posture tracking.** Maintain a `docs/security-posture.md` document that tracks:

- The total number of security findings across all cycles
- The disposition of each finding (ACCEPTED, DEFERRED, REJECTED)
- The number of open DEFERRED security findings
- New attack surface introduced by each cycle (new endpoints, new authentication flows, new data stores)

**Cumulative audit review.** Every five cycles (or at the System Owner's discretion), conduct a cumulative security review. This is a dedicated Antithesis phase where GPT-5.3-Codex audits the entire system rather than just the most recent cycle's changes. The cumulative audit focuses on:

- Interactions between components built in different cycles
- Security assumptions that may have been invalidated by later changes
- DEFERRED security blockers that remain unaddressed

### 6.3 BLUEPRINT Consistency Verification

As the BLUEPRINT evolves across cycles, internal inconsistencies can emerge. A new io_contract may conflict with an existing security_policy. A change to the tech_stack may invalidate assumptions in earlier non_functional_requirements.

**Consistency check during Thesis.** At the start of each Thesis phase, Claude Code should perform a consistency review of the entire BLUEPRINT, checking:

- All io_contracts reference valid authentication and authorization mechanisms defined in security_policies
- All non_functional_requirements are achievable with the current tech_stack
- Core_principles are not contradicted by any specific policy or contract
- Constraints are still accurate and have not been superseded by implementation decisions

**Cross-feature consistency.** When multiple feature BLUEPRINTs exist, the consistency check extends to verifying that feature BLUEPRINTs do not contradict the shared BLUEPRINT or each other.

### 6.4 Technical Debt Tracking

DEFERRED blockers are the protocol's built-in mechanism for tracking technical debt. Managing this debt intentionally prevents it from accumulating to unmanageable levels.

**Using DEFERRED blockers as a tech debt register.** Each DEFERRED blocker represents a known issue that was consciously postponed. The blockers array in STATE.yaml serves as the project's technical debt register. Supplement it with a `docs/tech-debt.md` summary that provides a human-readable overview of outstanding debt, organized by severity and domain.

**Debt reduction targets.** Set a target for the maximum number of open DEFERRED blockers. When the count exceeds this threshold, the next cycle should include debt reduction as an explicit goal. A reasonable starting threshold is five to ten open DEFERRED blockers, adjusted based on project size.

**Maintenance cycles.** Periodically (every three to five feature cycles), run a maintenance cycle focused entirely on addressing DEFERRED blockers, improving test coverage, updating documentation, and paying down technical debt. This cycle follows the standard four-phase structure but with requirements focused on debt reduction rather than new features.

### 6.5 Periodic Full-Cycle Reviews

Beyond individual cycle quality gates, periodic reviews assess the overall health of the protocol and project.

**When to run a full-cycle review:**

- After every five completed cycles
- When a new team member (human or agent pair) joins the project
- When the project undergoes a significant architecture change
- When the System Owner observes degrading quality metrics (increasing defects, growing debt, frequent rollbacks)

**What a full-cycle review covers:**

- BLUEPRINT coherence and completeness
- Cumulative test coverage and regression test health
- DEFERRED blocker backlog and aging analysis
- Conflict resolution patterns (per the metrics in CONSENSUS_RESOLUTION.md Section 8)
- Documentation currency and accuracy
- Git history cleanliness and tag consistency

---

## 7. Performance Considerations

The TITLE Protocol relies on AI agents processing artifacts within their context windows. As projects scale, artifact size and quantity can affect agent performance.

### 7.1 Context Window Limits

Both Claude Code and GPT-5.3-Codex have finite context windows. Large BLUEPRINTs, extensive spec directories, and multi-cycle audit histories can exceed these limits.

**BLUEPRINT.json size.** A BLUEPRINT with fifty or more io_contracts and detailed security_policies can reach tens of thousands of tokens. At this scale, agents may not be able to hold the entire BLUEPRINT in context alongside other relevant artifacts.

**Cumulative specs.** A specs/ directory with dozens of specification files may exceed what an agent can process in a single session.

**Audit log history.** Loading multiple cycles' audit logs simultaneously is typically not necessary, but cross-referencing findings from previous cycles requires selective loading.

### 7.2 Selective Artifact Loading

Not every artifact is needed at every phase. Agents should load artifacts selectively based on the current phase and task.

**Thesis phase loading:**

- Full BLUEPRINT.json (required for updates)
- Previous cycle's STATE.yaml (for deferred blocker review)
- Relevant specs/ files (those being modified or extended)
- Previous cycle's summary document (for context)

**Antithesis phase loading:**

- Full BLUEPRINT.json (read-only, for audit)
- All specs/ files (for comprehensive review)
- Previous audit logs are not required unless cross-referencing deferred findings

**Synthesis phase loading:**

- Full BLUEPRINT.json (required for updates)
- Current AUDIT_LOG.md (required for blocker classification)
- Relevant specs/ files (those affected by accepted changes)
- STATE.yaml blockers array (for deferred blocker context)

**Execution phase loading:**

- BLUEPRINT.json io_contracts and security_policies sections (not necessarily the full document)
- Relevant specs/ files for the features being implemented
- Current test suite files for the affected modules

### 7.3 Summary Documents

When artifacts become too large for a single context window, summary documents provide a compressed view.

**Cycle summary documents.** At the end of each cycle's Execution phase, create a `docs/cycles/cycle-{N}-summary.md` document containing:

- A one-paragraph description of what was accomplished
- The number of blockers identified, accepted, deferred, and rejected
- A list of major design decisions made during Synthesis
- A list of new or modified io_contracts
- Key test results and coverage metrics

These summaries allow agents in future cycles to understand the project's history without loading every previous artifact.

**BLUEPRINT summary.** For very large BLUEPRINTs, maintain a `BLUEPRINT_SUMMARY.md` that provides an overview of the system architecture, a list of all io_contracts with one-line descriptions, and a summary of security_policies. Agents can load the summary first and then selectively load the full BLUEPRINT sections they need.

### 7.4 Agent Session Management

AI agent sessions have practical limits in length and context retention. Managing sessions effectively is important for protocol execution.

**When to start a fresh session:**

- At every phase boundary. Each phase transition is a natural session boundary. The incoming agent starts a fresh session, reads STATE.yaml, and loads the relevant artifacts.
- When the current session's context window is approaching capacity.
- When the agent's responses indicate context degradation (repetition, contradiction, or loss of earlier reasoning).

**When to continue an existing session:**

- Within a single phase, when the work is progressing smoothly and the context remains manageable.
- When iterating on a specific artifact (e.g., Claude Code refining BLUEPRINT.json during Thesis).
- When GPT-5.3-Codex is running and debugging tests during Execution.

**Session handoff documentation.** If a session must be restarted mid-phase, update STATE.yaml handoff_notes with a summary of work completed in the current session and guidance for the next session. This is analogous to the inter-phase handoff procedure in PROTOCOL.md Section 5 but applied within a single phase.

---

## 8. Scaling Patterns

The following three reference architectures describe common scaling configurations. Each pattern is appropriate for different team sizes, feature coupling levels, and project structures.

### 8.1 Pattern A: Sequential Cycles

**Description.** One feature is developed at a time, in strict cycle order. Cycle 1 completes all four phases before Cycle 2 begins. There is one agent pair and one System Owner.

**How it works.** The project follows the standard protocol flow described in PROTOCOL.md and GETTING_STARTED.md without modification. Each cycle builds on the previous cycle's output. BLUEPRINT.json is updated incrementally. The specs/ directory grows linearly. All decisions are sequential and there are no parallel coordination challenges.

**Artifact flow.** The BLUEPRINT moves from Cycle 1 to Cycle 2 to Cycle 3, each cycle adding to or modifying the design. Audit logs are archived after each cycle. STATE.yaml tracks a single linear progression of cycle numbers.

**Best for:** Small teams (one System Owner, one agent pair). Tightly coupled features where each feature depends on the previous one. Projects where the order of feature development matters. Early-stage projects establishing their architecture.

**Advantages.** Simplest coordination model. No merge conflicts. Complete audit trail in linear Git history. Every decision is made with full context from previous cycles.

**Disadvantages.** Throughput is limited to one feature at a time. Idle agent capacity when waiting for System Owner approval. Not suitable for projects with independent feature tracks that could proceed in parallel.

### 8.2 Pattern B: Parallel Feature Tracks

**Description.** Multiple features are developed in parallel, each with its own cycle cadence, with periodic sync points to integrate. There are multiple agent pairs (one per feature) and one or more System Owners.

**How it works.** Each feature operates as an independent stream with its own .aisync/ directory, agent pair, and cycle progression (see Section 3.1). Features progress through Thesis, Antithesis, Synthesis, and Execution at their own pace. At defined sync points (e.g., every two weeks or after each feature completes a cycle), all completed feature cycles are integrated into the shared codebase.

**Artifact flow.** Multiple parallel tracks each produce their own BLUEPRINT, AUDIT_LOG, and STATE.yaml. A shared BLUEPRINT at the project level defines the common foundation. Integration cycles run after sync points to verify cross-feature compatibility. The overall project state is a composite of all feature states plus the shared state.

**Best for:** Medium teams (two to five agent pairs). Loosely coupled features that can be developed independently. Projects where throughput is more important than strict sequential ordering. Projects with a clear domain decomposition.

**Advantages.** Higher throughput through parallelism. Features can progress independently. Natural alignment with domain-driven organization. Failures in one feature do not block others.

**Disadvantages.** Requires coordination overhead for sync points. Merge conflicts possible when features touch shared components. More complex System Owner review process. Requires clear ownership boundaries for shared artifacts.

### 8.3 Pattern C: Domain-Driven Streams

**Description.** The project is organized by domain (e.g., authentication, payments, user interface, data pipeline), each with its own cycle cadence, domain owner, and agent pair. Domains are loosely coupled and may release independently.

**How it works.** Each domain operates as a semi-autonomous unit with its own .aisync/ directory, domain owner, agent pair, and release cycle. The project-level shared BLUEPRINT defines the overall architecture, shared interfaces, and project-wide policies. Each domain follows the four-phase cycle independently. Cross-domain integration is managed through shared interface specifications and periodic integration testing rather than synchronized cycles.

**Artifact flow.** Each domain maintains its own complete artifact set. The shared BLUEPRINT and shared specs/ directory define the contracts between domains. When a domain needs to modify a shared interface, it proposes the change through its Thesis phase and coordinates with affected domains before implementation. Integration testing runs continuously against the combined codebase, independent of individual domain cycle progression.

**Best for:** Large teams (five or more agent pairs). Microservices architectures where domains map to services. Projects with independent deployment and release cadences per domain. Organizations with established domain ownership.

**Advantages.** Maximum autonomy per domain. Independent release cadences. Natural alignment with microservices architecture. Scales to large teams. Minimal cross-domain coordination overhead for well-defined interfaces.

**Disadvantages.** Most complex coordination model. Requires well-defined shared interfaces from the start. Cross-domain changes are expensive (require multi-domain coordination). Risk of interface drift if shared specs are not actively maintained. Requires a strong project owner to maintain coherence.

---

## 9. Migration Guide

This section provides guidance for transitioning from a simpler scaling pattern to a more complex one as the project grows.

### 9.1 When to Transition

**From single-stream (Pattern A) to multi-feature (Pattern B).** Consider transitioning when:

- Two or more features have no dependencies on each other and could proceed in parallel
- The System Owner is becoming a bottleneck, waiting for one feature's cycle to complete before starting the next
- The team has grown to include additional people who could serve as domain owners
- Cycle completion time is extending beyond the project's desired delivery cadence

**From multi-feature (Pattern B) to domain-driven (Pattern C).** Consider transitioning when:

- The project has more than five parallel feature tracks
- Features naturally cluster into domains with well-defined interfaces
- The team wants independent release cadences for different parts of the system
- The monorepo is becoming unwieldy (long build times, complex merge conflicts)

### 9.2 Step-by-Step Migration: Single-Stream to Multi-Feature

The following procedure preserves existing cycle history while establishing the multi-feature structure.

**Step 1: Complete the current cycle.** Finish the active cycle through all four phases before restructuring. This provides a clean baseline.

**Step 2: Create the shared directory.** Create `.aisync/shared/` and copy the current BLUEPRINT.json into it as the project-level shared BLUEPRINT. Identify which fields are project-wide (core_principles, constraints, tech_stack, shared security_policies, shared non_functional_requirements) and which are feature-specific (specific io_contracts, feature-scoped requirements).

**Step 3: Create feature directories.** For each planned feature stream, create `.aisync/feature-{name}/`. Create a feature-scoped BLUEPRINT.json containing only the feature-specific fields, with a reference to the shared BLUEPRINT. Initialize a fresh STATE.yaml with cycle set to 1 for each feature. Copy the standard CHECKLIST.md and AUDIT_LOG.md template into each feature directory.

**Step 4: Archive existing cycle artifacts.** Move the existing AUDIT_LOG archives and resolved blockers to an `.aisync/archive/` directory. The existing Git history preserves the complete single-stream history.

**Step 5: Assign agent pairs.** Designate one agent pair per feature stream. Establish communication conventions per Section 4.3.

**Step 6: Define sync points.** Establish a sync point cadence and document it in the project's operational guidelines.

**Step 7: Update documentation.** Update the project README and docs/INDEX.md to reflect the new structure. Add a section explaining the multi-feature organization and referencing this document.

### 9.3 Preserving Cycle History

During migration, the existing cycle history must be preserved for auditability.

**Git history.** The Git commit history is the primary record of cycle progression. Do not rewrite Git history during migration. The restructuring commits should clearly document the migration (e.g., "migrate: restructure .aisync/ for multi-feature scaling per SCALING.md Section 9.2").

**Blocker history.** Copy all existing blocker entries from the original STATE.yaml into the shared directory's state tracking. DEFERRED blockers should be assigned to the appropriate feature stream during migration.

**Audit log history.** Archived audit logs from the single-stream era are moved to `.aisync/archive/audits/` with their original cycle-numbered names preserved. These logs remain accessible for cross-referencing in future cycles.

---

## 10. Limitations and Boundaries

The TITLE Protocol is designed for a specific type of multi-agent development workflow. Understanding its boundaries prevents misapplication and sets appropriate expectations.

### 10.1 What the TITLE Protocol Is NOT Designed For

**Real-time collaboration.** The protocol is a sequential, phase-gated process. It is not designed for real-time pair programming between agents or simultaneous editing of the same artifact. Each artifact has one primary author per phase (see PROTOCOL.md Section 8, Artifact Ownership Matrix). Attempting to use the protocol for real-time collaboration undermines its role separation and quality gate guarantees.

**CI/CD orchestration.** The protocol defines what agents produce and how they hand off, but it does not replace a CI/CD pipeline. The Execution phase produces code and tests, but the actual build, test, and deployment automation is the project's responsibility, not the protocol's. The protocol works alongside CI/CD systems; it does not replace them.

**Fully autonomous operation.** The protocol requires a human System Owner to approve phase transitions and resolve CONSENSUS_BLOCKED states. It is not designed for fully autonomous agent operation where agents run cycles without human oversight. Removing the System Owner from the loop eliminates the safety valve that prevents agent disagreements from stalling progress or producing compromised designs.

**Sub-task parallelism within a phase.** The protocol defines four sequential phases within a cycle. It does not define sub-task parallelism within a single phase. If an Execution phase involves implementing ten endpoints, the protocol does not specify how those ten endpoints should be divided among agents or executed in parallel. That level of task management is left to the implementation team.

### 10.2 Maximum Practical Scale

The protocol does not impose hard limits on scale, but practical limits emerge from the coordination overhead and artifact management complexity.

**Parallel feature streams.** Beyond ten parallel feature streams, the coordination overhead for sync points, shared BLUEPRINT management, and cross-feature conflict resolution may outweigh the benefits of parallelism. At this scale, consider whether the project should be decomposed into truly independent projects rather than parallel streams within a single protocol instance.

**Cycle count.** Beyond fifty cycles on a single feature stream, the accumulated blocker history, specification archive, and cross-cycle references may become difficult to navigate. At this point, consider resetting the cycle counter (with appropriate documentation) or splitting the feature stream into sub-streams.

**Team size.** Beyond ten System Owners (domain owners plus project owner), the decision-making hierarchy requires formalization beyond what this document provides. Consider adopting a formal governance framework for decision authority.

**BLUEPRINT size.** Beyond two hundred io_contracts in a single BLUEPRINT, the document becomes unwieldy for agent processing. Consider partitioning the BLUEPRINT into domain-scoped documents with a lightweight shared index.

### 10.3 When to Consider Alternative Approaches

The TITLE Protocol is one approach to multi-agent development coordination. Consider alternatives when:

- The project requires real-time agent collaboration rather than phase-gated handoffs
- The development workflow does not follow a design-audit-resolve-implement pattern
- The team size exceeds what the protocol's coordination model can support
- The project's release cadence is faster than one cycle per feature can accommodate (e.g., continuous deployment with multiple releases per day)
- The project does not require the audit and traceability overhead that the protocol provides

In these cases, lighter-weight coordination mechanisms (shared prompt libraries, code review conventions, or CI/CD-integrated agent workflows) may be more appropriate.

---

**Cross-References:**
- PROTOCOL.md -- Complete protocol specification (roles, phases, gates, handoffs, conflict resolution, rollback)
- GETTING_STARTED.md -- Step-by-step onboarding for single-cycle, single-feature projects
- CONSENSUS_RESOLUTION.md -- Detailed consensus deadlock resolution procedures
- GLOSSARY.md -- Protocol terminology quick reference
- LANGUAGE_POLICY.md -- English-first language conventions for all artifacts
- templates/STATE.yaml -- STATE.yaml field definitions and blocker schema
- templates/BLUEPRINT.json -- BLUEPRINT template with required fields

---

*TITLE Protocol v3.0 -- Scaling Guide*
*Version 1.0.0 | 2026-02-15*
