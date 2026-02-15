# Troubleshooting Guide

Comprehensive troubleshooting reference for problems encountered during TITLE Protocol v3.0 execution.

**Version:** 1.0.0
**Date:** 2026-02-15
**Audience:** System Owners and AI agents (Claude Code, GPT-5.3-Codex)

---

## Table of Contents

1. [Purpose](#1-purpose)
2. [Quick Diagnosis](#2-quick-diagnosis)
3. [STATE.yaml Error States](#3-stateyaml-error-states)
4. [Quality Gate Failures](#4-quality-gate-failures)
5. [Handoff Failures](#5-handoff-failures)
6. [Role Violation Recovery](#6-role-violation-recovery)
7. [Session Recovery](#7-session-recovery)
8. [Rollback Procedures](#8-rollback-procedures)
9. [Common Error Patterns](#9-common-error-patterns)
10. [Escalation Guide](#10-escalation-guide)
11. [Prevention Checklist](#11-prevention-checklist)

---

## 1. Purpose

Use this guide when the TITLE Protocol's four-phase dialectical cycle (Thesis, Antithesis, Synthesis, Execution) encounters a problem that prevents normal progress. Problems range from stuck STATE.yaml statuses to failed quality gates, broken handoffs, role violations, and session interruptions.

**Who should use this guide:**

- **System Owners** who need to diagnose why a cycle has stalled and determine the correct recovery action.
- **Claude Code** (Architect) when it detects inconsistencies in STATE.yaml, receives a corrupted handoff, or needs to determine whether a rollback is warranted.
- **GPT-5.3-Codex** (Builder/Red Team) when it encounters missing artifacts, incomplete gate conditions, or needs to signal a BLOCKED state.

**How to use this guide:**

1. Start with Section 2 (Quick Diagnosis) to identify the problem category.
2. Follow the cross-reference to the detailed section for that category.
3. Execute the resolution steps in order.
4. If the problem persists after following the resolution steps, consult Section 10 (Escalation Guide).

**Cross-references:** This guide references PROTOCOL.md (phase definitions, quality gates, rollback paths, conflict resolution), CONSENSUS_RESOLUTION.md (detailed deadlock resolution procedures), and GETTING_STARTED.md (setup and first-cycle walkthrough).

---

## 2. Quick Diagnosis

Use the following decision tree to identify which section of this guide addresses your problem. Work through the questions in order; the first "Yes" answer directs you to the relevant section.

| Step | Question | If Yes | If No |
|------|----------|--------|-------|
| 1 | Is `STATE.yaml` status set to `BLOCKED`? | Go to Section 3, "BLOCKED" | Continue to Step 2 |
| 2 | Is `STATE.yaml` status set to `CONSENSUS_BLOCKED`? | Go to Section 3, "CONSENSUS_BLOCKED" | Continue to Step 3 |
| 3 | Is a phase transition failing because a quality gate condition is not met? | Go to Section 4 | Continue to Step 4 |
| 4 | Did the incoming agent receive incomplete, missing, or corrupted handoff information? | Go to Section 5 | Continue to Step 5 |
| 5 | Is an agent performing actions outside its designated role for the current phase? | Go to Section 6 | Continue to Step 6 |
| 6 | Was the session interrupted mid-phase (crash, context exhaustion, timeout)? | Go to Section 7 | Continue to Step 7 |
| 7 | Does the situation require returning to a previous phase? | Go to Section 8 | Continue to Step 8 |
| 8 | Does the problem match one of the named error patterns? | Go to Section 9 | Go to Section 10 (Escalation) |

**Additional diagnostic checks:**

- Verify that `STATE.yaml` exists and is valid YAML. If the file is missing or unparseable, see Section 3, "Stale state."
- Verify that `BLUEPRINT.json` exists and is valid JSON. If it is missing or unparseable, this indicates the Thesis phase was not completed; see Section 4, "Thesis-to-Antithesis gate failure."
- Check Git log for the most recent commits. If the last commit does not correspond to the current phase in STATE.yaml, the state may be stale; see Section 3, "Stale state."

---

## 3. STATE.yaml Error States

STATE.yaml is the single source of truth for protocol progress. Its `status` field indicates whether the cycle is progressing normally or has encountered a problem. This section covers the four categories of STATE.yaml errors.

### 3.1 BLOCKED

**Definition:** The `status` field is set to `BLOCKED`, indicating that a concrete obstacle prevents phase transition. Unlike CONSENSUS_BLOCKED, this does not involve agent disagreement; it indicates a technical or procedural failure.

**Common Causes:**

| Cause | Description |
|-------|-------------|
| Missing artifact | A required artifact (BLUEPRINT.json, AUDIT_LOG.md, specs/) does not exist or was not committed to Git |
| SHA mismatch | The SHA recorded in `last_artifacts` does not match the current committed version of the artifact |
| Incomplete quality gate | One or more conditions for the current phase transition are not met (see Section 4) |
| Handoff failure | The incoming agent detected a problem during post-handoff verification (see Section 5) |

**Diagnosis Steps:**

1. Read `STATE.yaml` completely. Note the values of `phase`, `owner_model`, `handoff_notes`, and `last_artifacts`.
2. Read `handoff_notes` for a description of what caused the block. The agent that set the BLOCKED status should have documented the reason here.
3. Verify that all artifacts referenced in `last_artifacts` exist in the repository and that their Git SHAs match.
4. Review the quality gate checklist for the current phase transition (PROTOCOL.md Section 4) to identify any unmet conditions.

**Resolution Procedure:**

1. Identify the specific cause from `handoff_notes` or by checking the quality gate conditions.
2. Address the root cause:
   - If an artifact is missing, the responsible agent must create it and commit it to Git.
   - If a SHA mismatch exists, determine which version is authoritative (the committed file or the recorded SHA), resolve the discrepancy, and update `last_artifacts`.
   - If a quality gate condition is unmet, the responsible agent must complete the required work.
3. Once the root cause is resolved, update `STATE.yaml`:
   - Set `status` back to `OK`.
   - Update `handoff_notes` to describe the resolution.
   - Update `last_artifacts` with current Git SHAs.
4. Commit the STATE.yaml update and resume normal phase activity.

### 3.2 CONSENSUS_BLOCKED

**Definition:** The `status` field is set to `CONSENSUS_BLOCKED` and `blocking_model` identifies which agent triggered the block. This indicates that the two agents cannot reach agreement on a design decision, blocker disposition, or implementation approach, and the System Owner must intervene.

**Common Causes:**

| Cause | Description |
|-------|-------------|
| Blocker disposition dispute | Claude Code and GPT-5.3-Codex disagree on whether a finding should be ACCEPTED, DEFERRED, or REJECTED |
| Architectural disagreement | The agents propose incompatible design approaches and neither will yield |
| Severity classification conflict | The agents disagree on the severity rating of a finding |
| Scope boundary dispute | The agents disagree on whether an issue is in-scope for the current cycle |
| Feasibility disagreement | One agent considers a requirement infeasible with the current tech_stack |

**Diagnosis Steps:**

1. Read `STATE.yaml` to identify the `blocking_model` and the current `phase`.
2. Read `handoff_notes` for the structured conflict summary. Per CONSENSUS_RESOLUTION.md Section 3.3, this should contain: Dispute Title, Phase and Context, Position A, Position B, Level 2 Decision, Impact of Inaction, and Suggested Resolution Paths.
3. Check the `blockers` array for the entry with `status: OPEN` and `owner: SystemOwner`. This is the formal record of the dispute.
4. If the conflict summary is incomplete, request both agents to provide their position statements before proceeding with resolution.

**Resolution Procedure:**

The System Owner must resolve CONSENSUS_BLOCKED states. The full resolution process is documented in CONSENSUS_RESOLUTION.md. In summary:

1. The System Owner reviews both agents' positions using the evaluation criteria in CONSENSUS_RESOLUTION.md Section 4.1 (Security Impact, Alignment with Core Principles, Risk and Reward, Implementation Cost, Reversibility, Precedent and Consistency).
2. The System Owner makes a binding decision and documents it using the Decision Documentation Template (CONSENSUS_RESOLUTION.md Section 4.2).
3. Update `STATE.yaml`:
   - Set `status` back to `OK`.
   - Set `blocking_model` to `null`.
   - Update the relevant blocker entry: set `status` to ACCEPTED, DEFERRED, or REJECTED; set `owner` to `SystemOwner`; populate `reason` with the decision rationale; update `updated_at`.
   - Update `handoff_notes` with the resolution summary.
4. The phase-leading agent incorporates the decision and resumes phase activities.

### 3.3 Stale State

**Definition:** STATE.yaml has not been updated after a phase's activities were completed. The file shows an earlier phase or status while artifacts in the repository reflect later progress.

**Common Causes:**

| Cause | Description |
|-------|-------------|
| Agent forgot to update | The outgoing agent completed work and committed artifacts but did not update STATE.yaml |
| Session interruption | The session ended between artifact commit and STATE.yaml update |
| Manual editing error | STATE.yaml was edited incorrectly or partially |

**Diagnosis Steps:**

1. Compare `STATE.yaml` phase and cycle values against Git log. Look at the timestamps and content of recent commits to determine the actual progress.
2. Check whether the artifacts listed in `last_artifacts` have SHAs that match recent commits or are empty/outdated.
3. Review the most recent commit messages for evidence of phase completion (e.g., "Complete Thesis: BLUEPRINT.json and specs created").

**Recovery Steps:**

1. Determine the true current phase by examining:
   - Which artifacts exist and are complete (BLUEPRINT.json, AUDIT_LOG.md, implementation in src/).
   - The quality gate conditions for each phase transition (PROTOCOL.md Section 4).
   - The most recent Git commits and their content.
2. Update `STATE.yaml` to reflect the true current state:
   - Set `phase` to the correct value.
   - Set `owner_model` to the agent that should be leading the current phase.
   - Set `status` to `OK` (or `BLOCKED` if conditions are not met).
   - Update `last_artifacts` with current Git SHAs.
   - Document the recovery in `handoff_notes` (e.g., "STATE.yaml recovered from stale state. True phase determined from Git history.").
3. Commit the corrected STATE.yaml.

### 3.4 Invalid Phase/Owner_Model Combination

**Definition:** The `phase` and `owner_model` fields in STATE.yaml contain a combination that violates the protocol's role assignments.

**Valid Combinations (per PROTOCOL.md Section 3):**

| Phase | Valid owner_model |
|-------|-------------------|
| THESIS | Claude |
| ANTITHESIS | Codex |
| SYNTHESIS | Claude |
| EXECUTION | Codex |

**Detection:** Any combination not in the table above is invalid. For example, `phase: THESIS` with `owner_model: Codex` would be invalid.

**Common Causes:**

- Manual editing error in STATE.yaml.
- Incorrect handoff update by the outgoing agent.
- Copy-paste mistake when updating phase fields.

**Correction Procedure:**

1. Determine the true current phase from Git history and artifact state (same method as Section 3.3).
2. Set `owner_model` to the correct value for the determined phase per the table above.
3. Document the correction in `handoff_notes`.
4. Commit the corrected STATE.yaml.

---

## 4. Quality Gate Failures

Each phase transition requires all quality gate conditions to be met (PROTOCOL.md Section 4). When a gate check fails, the transition cannot proceed. This section provides diagnosis and resolution for each gate.

### 4.1 Thesis-to-Antithesis Gate Failure

**Gate Reference:** PROTOCOL.md Section 4.1

**Diagnosis Checklist:**

| Condition | How to Verify | Common Failure Cause |
|-----------|---------------|----------------------|
| BLUEPRINT.json exists and is valid JSON | Parse the file; check for syntax errors | File not created, or contains trailing commas or unquoted keys |
| `project_name` is set (not a placeholder) | Check that the value is not "REPLACE_WITH_PROJECT_NAME" | Template not filled in |
| `core_principles` has at least 1 entry | Check array length | Empty array or missing field |
| `constraints` has at least 1 key | Check object keys | Empty object or missing field |
| `tech_stack` contains `language` and `framework` | Check for both keys with non-placeholder values | One or both keys missing or still contain placeholder text |
| `io_contracts` has at least 1 endpoint | Check for at least one defined endpoint | Empty object or placeholder-only entries |
| `security_policies` contains required fields | Verify `authentication`, `authorization`, `data_handling`, `encryption` all exist with non-placeholder values | One or more required policy fields missing |
| `non_functional_requirements` contains required fields | Verify `performance`, `scalability`, `availability`, `observability` all exist | One or more required NFR fields missing |
| Specs exist in `specs/` | Check that the directory contains at least one specification file | Directory empty or not created |
| STATE.yaml updated | Verify `phase: THESIS` and `owner_model: Claude` | STATE.yaml not updated after BLUEPRINT creation |
| System Owner has reviewed and approved | Confirm explicit approval | Approval not obtained or not documented |

**Resolution:** Return to the Thesis phase. Claude Code must complete all missing BLUEPRINT fields, create any missing specification files in `specs/`, and obtain System Owner approval before attempting the transition again.

### 4.2 Antithesis-to-Synthesis Gate Failure

**Gate Reference:** PROTOCOL.md Section 4.2

**Diagnosis Checklist:**

| Condition | How to Verify | Common Failure Cause |
|-----------|---------------|----------------------|
| AUDIT_LOG.md exists and follows template | Check file exists and matches structure in templates/AUDIT_LOG.md | File not created, or structure deviates from template |
| All 5 audit areas evaluated | Check for sections: Memory/Resource Management, Concurrency/Race Conditions, Hardware/Infrastructure Lifecycle, Security Vulnerabilities, AI Misuse Scenarios | One or more areas omitted or marked only with placeholder text |
| Each finding has required fields | Verify each finding has: ID, Title, Area, Severity, Description, Evidence/PoC, Impact, Suggested Mitigation | Missing severity ratings, no evidence provided, or incomplete finding entries |
| Critical issues separated from recommendations | Check that blockers (BLK-XXX entries) are in the "Critical Issues / Blockers" section and non-blocking items are in "Recommendations" | Findings mixed together without classification |
| Blockers have severity ratings | Check each BLK-XXX entry for a Severity field with a valid value (CRITICAL, HIGH, MEDIUM, LOW) | Severity field missing or contains invalid value |
| STATE.yaml updated | Verify `phase: ANTITHESIS` and `owner_model: Codex` | STATE.yaml not updated after AUDIT_LOG completion |

**Resolution:** Return to the Antithesis phase. GPT-5.3-Codex must complete the AUDIT_LOG.md by evaluating all 5 audit areas, providing evidence for each finding, assigning severity ratings, and separating critical issues from recommendations.

### 4.3 Synthesis-to-Execution Gate Failure

**Gate Reference:** PROTOCOL.md Section 4.3

**Diagnosis Checklist:**

| Condition | How to Verify | Common Failure Cause |
|-----------|---------------|----------------------|
| Every blocker has a disposition | Check that each BLK-XXX entry in the AUDIT_LOG has been classified as ACCEPTED, DEFERRED, or REJECTED | Blockers left without disposition; some findings overlooked |
| DEFERRED blockers have documented reason and planned cycle | Check each DEFERRED entry for a reason and a target cycle number | Deferred without specifying when the issue will be addressed |
| REJECTED blockers have documented reason with evidence | Check each REJECTED entry for a rationale explaining why the risk is acceptable | Rejected without justification |
| ACCEPTED blockers reflected in updated BLUEPRINT and/or specs | Cross-check accepted findings against BLUEPRINT.json and specs/ for corresponding design changes | Design changes not incorporated; BLUEPRINT unchanged despite accepted findings |
| STATE.yaml blockers array updated | Check that the `blockers` array contains entries matching all findings with their dispositions | Blockers array empty, incomplete, or missing entries |
| No blocker has status OPEN | Search the blockers array for any entry with `status: OPEN` | One or more blockers not yet classified |
| GPT-5.3-Codex has reviewed feasibility | Confirm that Codex provided feasibility assessment for accepted design changes | Feasibility review not performed or not documented |
| System Owner has approved final design | Confirm explicit approval | Approval not obtained |

**Resolution:** Return to the Synthesis phase. Claude Code must classify all remaining OPEN blockers, document rationale for each DEFERRED and REJECTED disposition, update BLUEPRINT.json and specs/ to reflect ACCEPTED changes, and obtain System Owner approval.

### 4.4 Execution-to-Thesis Gate Failure

**Gate Reference:** PROTOCOL.md Section 4.4

**Diagnosis Checklist:**

| Condition | How to Verify | Common Failure Cause |
|-----------|---------------|----------------------|
| Implementation complete per BLUEPRINT | Cross-check src/ against io_contracts and specs/ | Endpoints or interfaces missing from implementation |
| Unit tests pass | Run test suite; check results | Failing tests; tests not written for new functionality |
| Integration tests pass (if applicable) | Run integration test suite | Integration tests failing or not created when applicable |
| Security tests pass (if applicable) | Run security test suite | Security tests failing or not addressing accepted blockers |
| Claude Code high-level review completed | Confirm Claude Code has reviewed against BLUEPRINT, security_policies, and io_contracts | Review not performed or review findings not addressed |
| Documentation updated | Check README, CHANGELOG, architecture docs | Documentation not reflecting implementation changes |
| STATE.yaml updated | Verify cycle increment, `phase: THESIS`, `owner_model: Claude` | STATE.yaml not updated after Execution completion |
| System Owner approved | Confirm explicit approval for release or next-cycle transition | Approval not obtained |

**Resolution:** Remain in the Execution phase. GPT-5.3-Codex must fix failing tests, complete missing implementation, and update documentation. Claude Code must complete its high-level review. System Owner must provide approval once all conditions are met.

---

## 5. Handoff Failures

Handoffs occur at every phase boundary (PROTOCOL.md Section 5). When a handoff fails, the incoming agent cannot reliably begin work. This section covers the four main handoff failure modes.

### 5.1 Missing handoff_notes

**Impact:** The incoming agent lacks context about the outgoing agent's work, open questions, areas of concern, and specific guidance. This can lead to misaligned priorities, missed critical context, and wasted effort.

**Detection:** The `handoff_notes` field in STATE.yaml is empty, contains only whitespace, or contains placeholder text.

**Recovery:**

1. The incoming agent sets `STATE.yaml` status to `BLOCKED` and documents the issue: "Handoff notes missing. Cannot determine context from previous phase."
2. The outgoing agent (or the System Owner, if the outgoing agent is unavailable) must provide the handoff context by updating `handoff_notes` with:
   - Summary of work completed in the previous phase.
   - Any open questions or areas of concern.
   - Specific guidance for the incoming agent.
3. Once handoff_notes are populated, the incoming agent sets `status` back to `OK` and proceeds.

**If the outgoing agent is unavailable:** The incoming agent and System Owner can reconstruct context from Git history and artifact content. The incoming agent should document any assumptions made due to missing handoff context.

### 5.2 SHA Mismatch

**Definition:** The SHA values recorded in `STATE.yaml` `last_artifacts` do not match the current Git SHAs of the corresponding files. This means an artifact was modified after the handoff was recorded.

**Detection:** The incoming agent compares the SHA values in `last_artifacts` against the actual Git SHAs of BLUEPRINT.json, AUDIT_LOG.md, and src/ using Git commands.

**Common Causes:**

| Cause | Description |
|-------|-------------|
| Post-handoff edit | An artifact was edited and committed after the outgoing agent updated STATE.yaml |
| Unauthorized modification | An agent modified an artifact it does not own for the current phase |
| Rebase or merge | Git operations changed commit SHAs without updating STATE.yaml |

**Resolution:**

1. Identify which artifact has a mismatched SHA and determine what changed by comparing the two versions in Git.
2. Evaluate whether the change is legitimate:
   - If the change was an authorized correction (e.g., fixing a typo in BLUEPRINT.json before the handoff was received), update `last_artifacts` with the correct SHA and proceed.
   - If the change was unauthorized (e.g., an agent modified an artifact it does not own), see Section 6 (Role Violation Recovery) and revert the unauthorized change.
3. After resolving the mismatch, update `last_artifacts` in STATE.yaml and commit.

### 5.3 Wrong owner_model in STATE.yaml

**Definition:** The `owner_model` field does not match the agent that should be leading the current phase (see the valid combinations table in Section 3.4).

**Detection:** The incoming agent reads STATE.yaml and finds that `owner_model` points to the wrong agent for the current `phase`.

**Correction Procedure:**

1. Verify the current phase is correct by examining Git history and artifact state.
2. Set `owner_model` to the correct agent for the current phase:
   - THESIS or SYNTHESIS: `owner_model: Claude`
   - ANTITHESIS or EXECUTION: `owner_model: Codex`
3. Document the correction in `handoff_notes` (e.g., "Corrected owner_model from Codex to Claude for SYNTHESIS phase.").
4. Commit the corrected STATE.yaml.

### 5.4 Missing Artifact Commits

**Definition:** The outgoing agent completed work on artifacts but did not commit them to Git before updating STATE.yaml.

**Detection:** Artifacts referenced in handoff_notes or implied by the phase exist as local changes but are not in the Git history. The `last_artifacts` SHA fields are empty or point to outdated versions.

**Recovery:**

1. Identify the uncommitted artifacts. Check working directory for modified or new files that should have been committed during the previous phase.
2. If uncommitted work is present:
   - Review the content for completeness and correctness.
   - Commit the artifacts with a descriptive message referencing the phase (e.g., "Thesis: commit BLUEPRINT.json and specs created during Thesis phase -- recovered from missed commit").
   - Update `last_artifacts` in STATE.yaml with the new commit SHAs.
3. If uncommitted work is lost (e.g., session ended without saving):
   - The responsible agent must redo the work.
   - If the agent is unavailable, the System Owner determines whether to redo the work or roll back to a previous phase (see Section 8).

---

## 6. Role Violation Recovery

The TITLE Protocol assigns fixed, non-overlapping responsibilities to each agent per phase (PROTOCOL.md Section 2, Section 8). When an agent acts outside its designated role, the violation must be detected, undone, and documented.

### 6.1 Claude Code Wrote Implementation Code

**Violation:** Claude Code authored implementation-level code (function bodies, test implementations, CI scripts) in `src/` or `tests/`. Per PROTOCOL.md Section 2.1, Claude Code must not write implementation-level code.

**How to Identify:**

- Check Git log for commits to `src/` or `tests/` where the author or committer is Claude Code.
- Review commit messages for implementation-related descriptions attributed to Claude Code.
- During the Execution phase, the Artifact Ownership Matrix (PROTOCOL.md Section 8) assigns `src/` and `tests/` write access exclusively to Codex.

**Undo and Reassign:**

1. Identify the specific commits containing Claude Code's implementation code.
2. Revert those commits using Git (e.g., `git revert <commit-sha>`).
3. Document the violation and revert in `handoff_notes`: "Reverted implementation code authored by Claude Code in commits [SHAs]. Code must be re-implemented by GPT-5.3-Codex."
4. GPT-5.3-Codex re-implements the affected code during its designated phase.
5. If the implementation was technically correct, Codex may use the reverted code as a reference, but Codex must author the committed version.

### 6.2 GPT-5.3-Codex Modified BLUEPRINT.json

**Violation:** GPT-5.3-Codex directly modified BLUEPRINT.json fields. Per PROTOCOL.md Section 2.2, Codex must not modify architectural decisions or BLUEPRINT fields directly; it should submit proposals as comments or issues.

**How to Identify:**

- Check Git log for commits to BLUEPRINT.json where the author is GPT-5.3-Codex.
- Compare the current BLUEPRINT.json against the last version authored by Claude Code to identify unauthorized changes.

**Revert and Document:**

1. Revert the unauthorized BLUEPRINT.json changes to the last Claude Code-authored version.
2. Preserve Codex's proposed changes as a separate document or note (e.g., in handoff_notes or a comments file) so that Claude Code can evaluate them.
3. Document the violation in `handoff_notes`: "Reverted unauthorized BLUEPRINT.json modification by GPT-5.3-Codex. Proposed changes preserved for Claude Code review."
4. Claude Code evaluates the proposed changes during its next leading phase (Thesis or Synthesis) and incorporates any that are appropriate.

### 6.3 Agent Acted During Wrong Phase

**Violation:** An agent performed phase-specific activities outside its designated phase. For example, GPT-5.3-Codex performing red-team review during the Thesis phase, or Claude Code making synthesis decisions during the Antithesis phase.

**Detection:**

- Check `STATE.yaml` `phase` against the activities performed in recent commits.
- Compare the Artifact Ownership Matrix (PROTOCOL.md Section 8) against actual file modifications.

**Correction:**

1. Determine the extent of the out-of-phase work.
2. If the work does not conflict with the current phase's activities, it may be preserved but must be re-validated during its proper phase. Document this in `handoff_notes`.
3. If the work conflicts with or pre-empts the designated phase's activities, revert the changes and document the violation.
4. Update `STATE.yaml` to reflect the correct current phase and owner_model.

### 6.4 Agent Skipped Quality Gate

**Violation:** A phase transition occurred without all quality gate conditions being verified (PROTOCOL.md Section 4). The `phase` field in STATE.yaml advanced despite unmet conditions.

**Detection:**

- Review the quality gate checklist for the transition that was skipped.
- Check whether the System Owner approved the transition.
- Examine whether required artifacts are complete.

**Rollback Procedure:**

1. Roll back STATE.yaml to the previous phase:
   - Set `phase` to the phase before the skipped gate.
   - Set `owner_model` to the appropriate agent for that phase.
   - Set `handoff_notes` to document the rollback: "ROLLBACK: Phase transition occurred without meeting quality gate conditions. Conditions [list unmet conditions] were not verified."
2. The cycle counter does not increment on rollback (PROTOCOL.md Section 7.2).
3. The responsible agent completes the unmet gate conditions.
4. Once all conditions are satisfied, the System Owner re-approves the transition.

---

## 7. Session Recovery

AI agent sessions can be interrupted by crashes, context window exhaustion, timeouts, or platform issues. Because the TITLE Protocol uses stateless design with all state persisted in explicit files, sessions can be recovered.

### 7.1 Session Interrupted Mid-Phase

**Situation:** A session ended unexpectedly while an agent was in the middle of phase activities. Work may be partially complete.

**How to Resume:**

1. Read `STATE.yaml` to determine the current phase, owner_model, and status.
2. Check Git log for the most recent commits to understand what work was completed before the interruption.
3. Review any uncommitted changes in the working directory. If there are uncommitted artifacts:
   - Evaluate whether the work is complete enough to commit.
   - If complete, commit with a descriptive message.
   - If incomplete, determine what remains and continue the work.
4. Read `handoff_notes` for context from the previous phase.
5. Resume phase activities from the point of interruption based on the current state of artifacts.

### 7.2 Context Window Exhausted

**Situation:** An AI agent's context window has been fully consumed, preventing further meaningful output. A new session must be started.

**How to Start a New Session with Proper State:**

1. The new session must begin by reading `STATE.yaml` to establish the current protocol state.
2. Read `handoff_notes` for context carried over from the previous phase or session.
3. Read the artifacts relevant to the current phase:
   - Thesis/Synthesis: Read BLUEPRINT.json, specs/, and any existing AUDIT_LOG.md.
   - Antithesis: Read BLUEPRINT.json and specs/.
   - Execution: Read BLUEPRINT.json, specs/, AUDIT_LOG.md with dispositions, and existing src/ and tests/.
4. Check the `blockers` array for any OPEN items that need attention.
5. Resume work based on the established state. The agent does not need to re-derive context from the full conversation history; the artifacts and STATE.yaml provide sufficient state.

### 7.3 Agent Confusion (Lost Track of Phase)

**Situation:** An agent has lost track of which phase it is in, what work has been completed, or what is expected next.

**Reset Procedure:**

1. Read `STATE.yaml`. The `phase` and `owner_model` fields are the authoritative indicators of the current state.
2. Read `handoff_notes` for context from the previous transition.
3. Review the quality gate checklist (PROTOCOL.md Section 4) for the current phase to understand what exit criteria must be met.
4. Review the phase activities description (PROTOCOL.md Section 3) for the current phase to understand what work is expected.
5. Check Git log for recent commits to understand what has already been done in the current phase.
6. Resume work based on the gap between what has been done and what the quality gate requires.

### 7.4 Multi-Session Continuity Best Practices

To maintain continuity across multiple sessions:

- **Always update STATE.yaml before ending a session.** If work is in progress, update `handoff_notes` with a summary of progress and remaining tasks, even if the phase is not complete.
- **Commit frequently.** Small, descriptive commits create a reliable audit trail that a new session can use to understand progress.
- **Use handoff_notes as a self-handoff mechanism.** When a session is ending mid-phase, treat the handoff_notes as a message to the next session of the same agent, not just to the next agent.
- **Record the last completed step.** In `handoff_notes`, note specifically which step of the phase activities (PROTOCOL.md Section 3) was last completed.
- **Do not rely on conversation history.** A new session has no access to the previous session's conversation. All state must be in files.

---

## 8. Rollback Procedures

Rollbacks return the protocol to a previous phase when the current phase cannot proceed due to fundamental issues. This section expands on PROTOCOL.md Section 7.

### 8.1 Rollback from Antithesis to Thesis

**When to roll back:** The BLUEPRINT is fundamentally incomplete or incorrect. GPT-5.3-Codex discovers during red-team review that the BLUEPRINT cannot be meaningfully audited because critical sections are missing, internally contradictory, or based on invalid assumptions.

**How to roll back:**

1. GPT-5.3-Codex (or the System Owner) identifies the specific deficiencies in the BLUEPRINT that prevent a meaningful audit.
2. Update `STATE.yaml`:
   - Set `phase` to `THESIS`.
   - Set `owner_model` to `Claude`.
   - Set `status` to `OK`.
   - Set `handoff_notes` to: "ROLLBACK from ANTITHESIS: [specific reason, e.g., 'BLUEPRINT security_policies are incomplete; authentication method is unspecified, preventing security audit.']"
3. The `cycle` counter does not increment (PROTOCOL.md Section 7.2).
4. Claude Code reads the rollback reason, addresses the deficiencies in BLUEPRINT.json and specs/, and re-attempts the Thesis-to-Antithesis gate.

### 8.2 Rollback from Synthesis to Antithesis

**When to roll back:** The audit was incomplete. During Synthesis review, Claude Code determines that the AUDIT_LOG does not cover all 5 audit areas, lacks evidence for critical findings, or missed significant risk areas that were visible in the BLUEPRINT.

**How to roll back:**

1. Claude Code identifies the specific gaps in the AUDIT_LOG.
2. Update `STATE.yaml`:
   - Set `phase` to `ANTITHESIS`.
   - Set `owner_model` to `Codex`.
   - Set `status` to `OK`.
   - Set `handoff_notes` to: "ROLLBACK from SYNTHESIS: [specific reason, e.g., 'AUDIT_LOG omits Hardware/Infrastructure Lifecycle area. AI Misuse Scenarios section contains placeholder text only.']"
3. The `cycle` counter does not increment.
4. GPT-5.3-Codex completes the missing audit areas and re-submits the AUDIT_LOG.

### 8.3 Rollback from Synthesis to Thesis

**When to roll back:** The design requires major rework beyond what blocker dispositions can address. Accepted findings reveal that the BLUEPRINT's fundamental architecture is flawed and incremental fixes are insufficient.

**How to roll back:**

1. Claude Code (or the System Owner) determines that the required changes are architectural rather than incremental.
2. Update `STATE.yaml`:
   - Set `phase` to `THESIS`.
   - Set `owner_model` to `Claude`.
   - Set `status` to `OK`.
   - Set `handoff_notes` to: "ROLLBACK from SYNTHESIS to THESIS: [specific reason, e.g., 'Red-team findings reveal that the authentication architecture requires a fundamental redesign. Patching the existing BLUEPRINT is insufficient.']"
3. The `cycle` counter does not increment.
4. Claude Code revises the BLUEPRINT using insights from both the original Thesis and the Antithesis audit.

### 8.4 Rollback from Execution to Synthesis

**When to roll back:** Implementation reveals design flaws not caught during the audit. GPT-5.3-Codex encounters issues during coding that indicate the BLUEPRINT's design decisions are impractical, contradictory, or incomplete.

**How to roll back:**

1. GPT-5.3-Codex documents the specific implementation issues that trace back to design flaws.
2. Update `STATE.yaml`:
   - Set `phase` to `SYNTHESIS`.
   - Set `owner_model` to `Claude`.
   - Set `status` to `OK`.
   - Set `handoff_notes` to: "ROLLBACK from EXECUTION: [specific reason, e.g., 'io_contracts specify a response schema that is incompatible with the database model defined in specs/. Synthesis must reconcile the conflict.']"
3. The `cycle` counter does not increment.
4. Implementation progress in `src/` and `tests/` is preserved (not deleted). Synthesis builds upon the existing artifacts while addressing the design flaw.

**Note:** A rollback from Execution directly to Thesis is also permitted when requirements changed significantly during implementation. The procedure is the same but with `phase` set to `THESIS` and an appropriate reason in `handoff_notes`.

### 8.5 No Rollback from Thesis

Thesis is the first phase of each cycle. There is no earlier phase to roll back to within the same cycle. If the Thesis phase itself is producing unworkable results:

- The System Owner may abandon the current cycle entirely (PROTOCOL.md Section 7.3).
- The System Owner sets `status` to `BLOCKED` with `handoff_notes`: "CYCLE ABANDONED: [reason]".
- All artifacts are preserved in Git history.
- A new cycle begins from Thesis with `cycle` incremented.

### 8.6 Git-Based Rollback Using Artifact SHAs

The `last_artifacts` field in STATE.yaml records the Git SHAs of key artifacts at each handoff point. These SHAs enable precise file-level rollback.

**Procedure:**

1. Identify the target state by reading `last_artifacts` from the STATE.yaml version at the desired rollback point. Use Git history to find the correct STATE.yaml version.
2. For each artifact that needs rollback, retrieve the version at the recorded SHA:
   - Use `git show <sha>:<filepath>` to view the artifact at the recorded SHA.
   - Use `git checkout <sha> -- <filepath>` to restore the artifact to that version.
3. Update `STATE.yaml` with the rolled-back phase and owner_model.
4. Commit all restored artifacts and the updated STATE.yaml with a descriptive message (e.g., "Rollback to Thesis: restore BLUEPRINT.json to pre-Antithesis version [SHA]").

---

## 9. Common Error Patterns

This section documents ten recurring problems with their symptoms, root causes, resolutions, and prevention strategies.

### Pattern 1: Infinite Synthesis Loop

**Symptoms:** The Synthesis phase repeats multiple times within the same cycle. Agents keep re-classifying blockers, updating dispositions, and returning to Synthesis without progressing to Execution. The `cycle` counter does not advance.

**Root Cause:** Agents are unable to reach stable dispositions for all blockers. Common triggers include: ambiguous BLUEPRINT requirements that make disposition justification difficult, new findings being added during Synthesis that reset the classification process, or agents re-opening previously resolved blockers.

**Resolution:**

1. The System Owner reviews all current blockers and their disposition history.
2. For each blocker that has changed disposition more than once, the System Owner makes a binding decision.
3. Establish a rule: once a blocker has a disposition and the System Owner has approved, it cannot be re-opened within the same cycle. It may be revisited in a future cycle's Thesis phase.
4. Clear the Synthesis-to-Execution gate and proceed.

**Prevention:** Define clear disposition criteria in BLUEPRINT.json. Quantify thresholds for severity ratings and blocker classification. Ensure BLUEPRINT requirements are specific enough that disposition decisions are defensible.

### Pattern 2: Ghost Blockers

**Symptoms:** The `STATE.yaml` blockers array references blocker IDs (e.g., BLK-005) that do not appear in AUDIT_LOG.md. Or, Synthesis dispositions reference findings that cannot be located in the audit.

**Root Cause:** Blocker IDs were assigned inconsistently. Findings were added to STATE.yaml without corresponding AUDIT_LOG entries, or AUDIT_LOG entries were deleted or renumbered after blockers were created.

**Resolution:**

1. Cross-reference every entry in the `blockers` array against AUDIT_LOG.md.
2. For each ghost blocker (no corresponding AUDIT_LOG entry):
   - If the blocker represents a real finding that was documented elsewhere (handoff_notes, comments), create the corresponding AUDIT_LOG entry.
   - If the blocker is orphaned and no one can identify its source, mark it as REJECTED with reason: "No corresponding AUDIT_LOG entry found. Blocker ID is orphaned."
3. Re-synchronize blocker IDs between AUDIT_LOG.md and STATE.yaml.

**Prevention:** Assign blocker IDs exclusively within AUDIT_LOG.md and copy them to STATE.yaml. Never create blocker entries in STATE.yaml without a corresponding AUDIT_LOG finding.

### Pattern 3: Stale BLUEPRINT

**Symptoms:** The implementation in `src/` does not match the current version of BLUEPRINT.json. IO contracts define endpoints that differ from what was implemented. Security policies in the BLUEPRINT have been updated but the implementation reflects an earlier version.

**Root Cause:** BLUEPRINT.json was updated during Synthesis (to reflect accepted blockers) but the implementation was based on the pre-Synthesis version. Or, multiple rollbacks and updates caused the BLUEPRINT to diverge from what Codex last read.

**Resolution:**

1. Compare the current BLUEPRINT.json (particularly `io_contracts`, `security_policies`, and `non_functional_requirements`) against the implementation in `src/`.
2. Identify the specific discrepancies.
3. Determine which version is authoritative: the BLUEPRINT (design intent) or the implementation (working code).
4. If the BLUEPRINT is authoritative: update the implementation to match.
5. If the implementation has valid reasons for diverging (e.g., feasibility issues discovered during Execution): update the BLUEPRINT to match, with documentation of why the change was necessary. This may require a rollback to Synthesis.

**Prevention:** At the start of the Execution phase, GPT-5.3-Codex should verify the BLUEPRINT SHA in `last_artifacts` matches the version it is reading. Any discrepancy should be flagged immediately.

### Pattern 4: Missing Specs

**Symptoms:** The `io_contracts` section of BLUEPRINT.json references endpoints or interfaces, but the `specs/` directory does not contain corresponding specification files. Codex cannot perform a meaningful audit or implementation because the interface details are incomplete.

**Root Cause:** Claude Code defined IO contracts at a high level in BLUEPRINT.json but did not create the detailed specification files in `specs/`. Or, spec files were created but not committed to Git.

**Resolution:**

1. List all endpoints in `io_contracts` and compare against files in `specs/`.
2. For each endpoint without a corresponding spec:
   - If still in Thesis: Claude Code creates the missing spec.
   - If past Thesis: roll back to Thesis (Section 8) so that Claude Code can complete the specs.
3. Verify that all spec files are committed to Git.

**Prevention:** Include a spec file count verification in the Thesis-to-Antithesis gate check. The number of spec files should correspond to the number of IO contract entries.

### Pattern 5: Audit Blind Spots

**Symptoms:** AUDIT_LOG.md has findings in some audit areas but completely omits one or more of the 5 required areas (Memory/Resource Management, Concurrency/Race Conditions, Hardware/Infrastructure Lifecycle, Security Vulnerabilities, AI Misuse Scenarios). The omitted areas have no findings and no "No issues found" notation.

**Root Cause:** GPT-5.3-Codex focused on areas where issues were most obvious and skipped areas that appeared clean without documenting that assessment. Or, the session was interrupted before all areas were evaluated.

**Resolution:**

1. Identify which of the 5 areas are missing from the AUDIT_LOG.
2. If the Antithesis-to-Synthesis gate has not been passed: GPT-5.3-Codex completes the missing areas.
3. If the gate was improperly passed: roll back to Antithesis (Section 8.2) and complete the audit.
4. Even if an area has no issues, the AUDIT_LOG must document that the area was evaluated and state "No issues found" with a brief explanation of what was checked.

**Prevention:** Use the AUDIT_LOG template (templates/AUDIT_LOG.md) which lists all 5 areas as mandatory sections. The Antithesis-to-Synthesis gate explicitly requires all 5 areas to be evaluated.

### Pattern 6: Handoff Amnesia

**Symptoms:** The incoming agent begins phase activities without incorporating the context and guidance provided in `handoff_notes`. Critical warnings are ignored. Priorities from the outgoing agent are not reflected in the incoming agent's work.

**Root Cause:** The incoming agent did not read `handoff_notes` during its post-handoff checklist (PROTOCOL.md Section 5.3). Or, the agent read the notes but lost them due to context window limitations.

**Resolution:**

1. The System Owner or the other agent identifies that handoff context was ignored.
2. The incoming agent re-reads `handoff_notes` and identifies any guidance that was missed.
3. The agent adjusts its current work to incorporate the missed context.
4. If significant work was done in the wrong direction due to missed handoff notes, evaluate whether a rollback is necessary.

**Prevention:** The first action of any incoming agent at a phase boundary must be to read STATE.yaml, including `handoff_notes`, and acknowledge the context in its initial output. Agents should explicitly reference handoff notes in their first artifact or communication.

### Pattern 7: Scope Creep in Execution

**Symptoms:** GPT-5.3-Codex implements features, endpoints, or functionality that are not defined in BLUEPRINT.json or specs/. The implementation includes components that were never designed, audited, or approved.

**Root Cause:** Codex extrapolated beyond the BLUEPRINT scope, adding features it deemed useful or necessary without formal approval. Or, ambiguous BLUEPRINT descriptions were interpreted more broadly than intended.

**Resolution:**

1. Claude Code identifies the out-of-scope additions during its high-level review.
2. For each out-of-scope addition:
   - If the addition is valuable and does not conflict with the design: document it as a proposal for the next cycle's Thesis phase. It may be kept in the code but must be formally incorporated into the BLUEPRINT in the next cycle.
   - If the addition conflicts with the design or introduces risk: revert the changes. The functionality can be proposed for a future cycle.
3. Document all scope decisions in `handoff_notes`.

**Prevention:** During the Execution phase, Codex should verify each implementation task against BLUEPRINT.json before beginning work. Claude Code's high-level review should explicitly check for scope adherence.

### Pattern 8: Silent Gate Bypass

**Symptoms:** The `phase` field in STATE.yaml advanced to the next phase, but one or more quality gate conditions for the transition were not met. The transition happened without System Owner verification.

**Root Cause:** The agent updated STATE.yaml phase without completing the quality gate checklist. Or, the System Owner approval was assumed rather than explicitly obtained.

**Resolution:**

1. Identify which gate conditions were not met by reviewing the checklist for the bypassed transition.
2. Roll back to the previous phase (Section 8).
3. Complete all unmet gate conditions.
4. The System Owner explicitly verifies and approves the transition before STATE.yaml is updated.

**Prevention:** Treat the quality gate checklist (PROTOCOL.md Section 4) as mandatory. The System Owner should verify all conditions before approving any phase transition. Consider adding a "gate_verified_by: SystemOwner" field to STATE.yaml for audit purposes.

### Pattern 9: Conflicting Artifact Versions

**Symptoms:** BLUEPRINT.json and spec files in `specs/` contain contradictory information. For example, BLUEPRINT.json defines an endpoint's response schema one way while the corresponding spec file defines it differently. Or, BLUEPRINT `security_policies` specify one authentication method while specs reference a different one.

**Root Cause:** BLUEPRINT.json and specs/ were updated at different times without cross-verification. Synthesis phase updates to BLUEPRINT.json did not include corresponding spec updates, or vice versa.

**Resolution:**

1. Identify all contradictions between BLUEPRINT.json and specs/.
2. Determine which artifact represents the intended design:
   - If BLUEPRINT.json is authoritative: update specs/ to match.
   - If specs/ contain more detailed or recent decisions: update BLUEPRINT.json to match.
3. Commit the reconciled artifacts together in a single commit.
4. If the contradiction is discovered during Execution, a rollback to Synthesis may be necessary to properly reconcile the artifacts.

**Prevention:** When updating BLUEPRINT.json during Synthesis, always review and update the corresponding spec files in the same commit. Cross-reference io_contracts entries against spec files as a gate check.

### Pattern 10: Cycle Number Drift

**Symptoms:** The `cycle` field in STATE.yaml does not match the actual number of complete Thesis-Antithesis-Synthesis-Execution iterations that have occurred. The cycle number is higher or lower than expected based on Git history.

**Root Cause:** The cycle counter was incremented during a rollback (it should not be). Or, the cycle counter was not incremented after a successful Execution-to-Thesis transition. Or, the cycle was incremented multiple times due to a stale state correction.

**Resolution:**

1. Count the actual number of complete cycles by reviewing Git history for Execution-to-Thesis transitions.
2. Set `cycle` in STATE.yaml to the correct value.
3. Document the correction in `handoff_notes`.
4. Commit the corrected STATE.yaml.

**Prevention:**

- The cycle counter increments only at the Execution-to-Thesis transition (PROTOCOL.md Section 4.4).
- The cycle counter never increments on rollback (PROTOCOL.md Section 7.2).
- The cycle counter increments by 1 on cycle abandonment (PROTOCOL.md Section 7.3), even though the cycle did not complete normally.

---

## 10. Escalation Guide

When the resolution procedures in this guide do not resolve the problem, or when the problem falls outside the documented patterns, use this escalation framework.

### When to Involve the System Owner

The System Owner should be involved when:

- `STATE.yaml` status is `CONSENSUS_BLOCKED` (this always requires System Owner intervention).
- `STATE.yaml` status is `BLOCKED` and neither agent can identify or resolve the root cause.
- A role violation has occurred and the agents disagree on the correct remediation.
- Multiple rollbacks within the same cycle suggest a systemic issue that agents cannot resolve on their own.
- Requirements have changed in a way that invalidates the current cycle's assumptions.

### When to Restart the Current Phase

Restart the current phase (rollback to the beginning of the same phase) when:

- Work completed during the phase is fundamentally flawed or based on incorrect assumptions.
- The session interruption was severe enough that reconstructing the mid-phase state is impractical.
- A role violation corrupted the phase's artifacts beyond incremental repair.

**Procedure:** Update STATE.yaml with the same phase value but refresh `handoff_notes` to describe the restart reason. Artifacts from the failed attempt are preserved in Git history.

### When to Restart the Entire Cycle

Restart the entire cycle (return to Thesis with cycle incremented) when:

- The System Owner determines that the requirements captured in the current cycle's BLUEPRINT are no longer valid.
- Multiple rollbacks within the same cycle indicate that the foundational design is unworkable.
- External factors (new security requirements, technology changes, project pivot) invalidate the current cycle.

**Procedure:** Follow the Cycle Abandonment process in PROTOCOL.md Section 7.3. Set `status: BLOCKED` and `handoff_notes: "CYCLE ABANDONED: [reason]"`. Increment the cycle counter and begin a new Thesis phase.

### When to Archive and Start Fresh

Archive the current project state and start fresh when:

- The cumulative complexity of the STATE.yaml blockers array, rollback history, and artifact versions makes it impractical to determine the true current state.
- The project has pivoted significantly enough that existing artifacts provide no value for the new direction.
- Both agents and the System Owner agree that continuing from the current state would be less efficient than starting over.

**Procedure:**

1. Create a Git tag marking the current state (e.g., `v0.x-archived-YYYY-MM-DD`).
2. Document the archive decision and reasons in a commit message.
3. Reset STATE.yaml to its initial template state: `phase: THESIS`, `owner_model: Claude`, `status: OK`, `cycle: 1`, empty `blockers` and `last_artifacts`.
4. Optionally preserve the old BLUEPRINT.json and AUDIT_LOG.md as reference files (renamed or moved to an `archive/` directory).
5. Begin a fresh Thesis phase.

---

## 11. Prevention Checklist

A summary of best practices that prevent the problems documented in this guide. Review this checklist at the start of each cycle.

**STATE.yaml Management:**

- [ ] Update STATE.yaml at every phase transition, not just at the end of work.
- [ ] Verify that `phase` and `owner_model` are a valid combination before committing.
- [ ] Populate `handoff_notes` with actionable context at every transition.
- [ ] Update `last_artifacts` with current Git SHAs at every transition.
- [ ] Never increment the `cycle` counter during a rollback.

**Quality Gate Discipline:**

- [ ] Use the quality gate checklist (PROTOCOL.md Section 4) before every phase transition.
- [ ] Obtain explicit System Owner approval before advancing to the next phase.
- [ ] Never skip a gate condition, even if it appears trivial.

**Artifact Integrity:**

- [ ] Commit all artifacts to Git before updating STATE.yaml.
- [ ] Cross-reference BLUEPRINT.json `io_contracts` against spec files in `specs/` for consistency.
- [ ] Verify artifact SHAs match during post-handoff checks.
- [ ] Keep BLUEPRINT.json and specs/ in sync when either is updated.

**Role Adherence:**

- [ ] Review the Artifact Ownership Matrix (PROTOCOL.md Section 8) at the start of each phase.
- [ ] Claude Code: do not write implementation code in `src/` or `tests/`.
- [ ] GPT-5.3-Codex: do not modify BLUEPRINT.json or `specs/` directly.
- [ ] Verify that the current agent matches `owner_model` before beginning phase work.

**Handoff Quality:**

- [ ] The outgoing agent writes handoff_notes before updating the phase.
- [ ] The incoming agent reads handoff_notes as its first action.
- [ ] Include: summary of completed work, open questions, specific guidance for the next agent.

**Session Continuity:**

- [ ] Commit work frequently with descriptive messages.
- [ ] Update handoff_notes when ending a session mid-phase.
- [ ] Do not rely on conversation history; all state belongs in files.
- [ ] Record the last completed step in handoff_notes before session end.

**Audit Completeness:**

- [ ] AUDIT_LOG.md must evaluate all 5 audit areas, even if the finding is "No issues found."
- [ ] Every finding must have an ID, severity, evidence, impact, and suggested mitigation.
- [ ] Blocker IDs in STATE.yaml must correspond to entries in AUDIT_LOG.md.

**Conflict Resolution:**

- [ ] Follow the escalation ladder (CONSENSUS_RESOLUTION.md Section 2): attempt Level 1 before Level 2, Level 2 before Level 3.
- [ ] Document all resolutions in the STATE.yaml blockers array.
- [ ] Record dissenting positions for future reference.

---

**Cross-References:**

- PROTOCOL.md -- Phase definitions (Section 3), quality gates (Section 4), handoff procedures (Section 5), conflict resolution (Section 6), rollback paths (Section 7), artifact ownership (Section 8)
- CONSENSUS_RESOLUTION.md -- Detailed deadlock resolution procedures
- GETTING_STARTED.md -- Setup, first-cycle walkthrough, common mistakes
- GLOSSARY.md -- Protocol terminology definitions
- templates/STATE.yaml -- STATE.yaml field schema and valid values
- templates/AUDIT_LOG.md -- Audit report structure and finding template
- templates/BLUEPRINT.json -- System design document template

---

*TITLE Protocol v3.0 -- Troubleshooting Guide*
*Version 1.0.0 | 2026-02-15*
