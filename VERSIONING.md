# Versioning Strategy

Version: 1.0.0
Date: 2026-02-15
Protocol: TITLE Protocol v3.0

Versioning conventions for the TITLE Protocol, BLUEPRINT artifacts, and consumer project files.

---

## 1. Purpose

A multi-agent protocol produces several layers of evolving artifacts: the protocol specification itself, the system design document (BLUEPRINT.json), interface specifications, audit reports, implementation code, and tests. Each layer changes at a different cadence and for different reasons. Without clear versioning boundaries, agents and System Owners cannot determine whether an artifact is current, whether a protocol upgrade requires design rework, or whether two projects using different protocol editions remain compatible.

This document defines four distinct versioning dimensions:

| Dimension | What It Tracks | Example |
|-----------|---------------|---------|
| Protocol version | The TITLE Protocol specification itself (roles, phases, gates, handoffs) | 3.0.0 |
| BLUEPRINT version | A consumer project's system design document (`BLUEPRINT.json`) | 1.2.0 |
| Artifact versions | Individual protocol files (AUDIT_LOG, specs, docs) within a cycle | Per-cycle or per-commit |
| Cycle number | The monotonically increasing count of complete protocol passes | 5 |

These four dimensions are independent. A project at cycle 5 may have BLUEPRINT version 2.3.1 while following protocol version 3.0.0. Keeping them separate prevents conflation of design maturity with iteration count or protocol edition.

---

## 2. Protocol Versioning

Protocol versioning applies to the TITLE Protocol specification itself -- the contents of this `agent-cowork` repository.

### 2.1 Semantic Versioning Scheme

The protocol follows Semantic Versioning (MAJOR.MINOR.PATCH):

| Component | Incremented When | Examples |
|-----------|-----------------|----------|
| **MAJOR** | Breaking changes to phase structure, quality gate conditions, role definitions, artifact ownership matrix, handoff procedures, or conflict resolution escalation ladder | Removing a phase, changing gate conditions so that existing consumer checklists fail, redefining which agent owns an artifact |
| **MINOR** | New optional features, additional guidelines, expanded documentation, new templates, or additive changes that do not break existing workflows | Adding a new optional field to STATE.yaml, introducing a new prompt template, adding SCALING.md, adding VERSIONING.md |
| **PATCH** | Typo corrections, clarification of existing language, formatting improvements, broken link fixes | Fixing a typo in PROTOCOL.md, rewording a gate condition for clarity without changing its meaning |

### 2.2 Current Version

The current protocol version is **3.0.0**.

### 2.3 Where Protocol Version Is Declared

The protocol version appears in two authoritative locations:

1. **README.md** -- In the title and Protocol Summary section ("TITLE Protocol v3.0")
2. **PROTOCOL.md** -- In the document header ("TITLE Protocol v3.0 -- Protocol Specification")

All other documents that reference the protocol version (GETTING_STARTED.md, GLOSSARY.md, template headers, prompt templates) derive their version string from these two sources.

When the protocol version is incremented, both README.md and PROTOCOL.md must be updated first. All other references are then updated to match.

### 2.4 Backward Compatibility Policy

Consumer projects that adopt the TITLE Protocol should handle upgrades as follows:

**PATCH upgrades (e.g., 3.0.0 to 3.0.1):** Safe to adopt immediately. No changes to consumer project files, agent prompts, or workflows are required. Review the change for informational purposes.

**MINOR upgrades (e.g., 3.0.0 to 3.1.0):** Safe to adopt. New features are optional. Consumer projects may adopt new templates or guidelines at their discretion. Existing workflows continue to function without modification.

**MAJOR upgrades (e.g., 3.0.0 to 4.0.0):** Breaking changes are present. Consumer projects must review the CHANGELOG, update templates if structure changed, update agent prompts if role definitions changed, and test one complete cycle with the new protocol version before full adoption. See Section 7 for detailed upgrade procedures.

---

## 3. BLUEPRINT Versioning

BLUEPRINT versioning tracks the evolution of a consumer project's system design document (`BLUEPRINT.json`).

### 3.1 The Version Field

The `BLUEPRINT.json` template includes a `"version"` field (see [templates/BLUEPRINT.json](templates/BLUEPRINT.json)). This field records the semantic version of the design document itself, independent of the protocol version or cycle number.

### 3.2 Versioning Scheme

BLUEPRINT versions follow Semantic Versioning (MAJOR.MINOR.PATCH):

| Component | Incremented When | Examples |
|-----------|-----------------|----------|
| **MAJOR** | Breaking interface changes: `io_contracts` changed incompatibly (endpoints removed, request/response schemas restructured, error codes redefined), `security_policies` changed in ways that invalidate existing implementations | Removing an API endpoint, changing authentication from JWT to mTLS |
| **MINOR** | New `io_contracts` added (existing ones unchanged), non-breaking additions to `security_policies`, new `non_functional_requirements` targets, new keys added to `constraints` or `tech_stack` | Adding a new API endpoint, adding rate limiting to security policies |
| **PATCH** | Constraint clarifications, typo corrections, documentation updates within BLUEPRINT fields, adjusting NFR targets without structural change | Correcting a description string, updating a performance target from "p99 < 200ms" to "p99 < 150ms" |

### 3.3 When to Increment

BLUEPRINT version changes occur during two phases of the protocol cycle:

**During Thesis (Phase 1):**
- Initial creation sets version to `0.1.0` (as in the template default)
- Subsequent cycles where Claude Code revises the BLUEPRINT increment the version according to the change scope

**During Synthesis (Phase 3):**
- Accepted blockers that result in design changes trigger a version increment
- The version component incremented depends on the nature of the accepted change (see table above)

A single cycle may produce multiple version increments. For example, Thesis might add new endpoints (MINOR bump) and Synthesis might accept a blocker that restructures an existing endpoint (MAJOR bump). In this case, only the highest-severity bump applies: the version increments by MAJOR.

### 3.4 Relationship to Cycle Number

The BLUEPRINT version and cycle number are independent values:

| Cycle | BLUEPRINT Version | What Happened |
|-------|------------------|---------------|
| 1 | 0.1.0 | Initial design created |
| 2 | 0.2.0 | New endpoints added, no breaking changes |
| 3 | 0.2.1 | Constraint clarifications only |
| 4 | 1.0.0 | Breaking IO contract change from accepted blocker |
| 5 | 1.1.0 | New endpoint added |

### 3.5 Tracking BLUEPRINT Versions in STATE.yaml

`STATE.yaml` tracks the current committed state of the BLUEPRINT through the `last_artifacts.blueprint_sha` field. This SHA serves as the authoritative reference for which BLUEPRINT version is active. At each phase transition, the outgoing agent updates `blueprint_sha` to the current Git commit SHA of `BLUEPRINT.json`.

To reconstruct version history, use Git log filtered to the BLUEPRINT file:

```
git log --oneline -- .aisync/BLUEPRINT.json
```

---

## 4. Artifact Versioning

Different protocol artifacts follow different versioning strategies based on their lifecycle and purpose.

### 4.1 specs/ Files

Interface specifications in `specs/` are versioned per specification file. Each spec file should include a version header or comment indicating its current version. Spec versions follow the same MAJOR.MINOR.PATCH scheme as BLUEPRINT versions:

- MAJOR: Breaking interface contract changes
- MINOR: Additive interface changes
- PATCH: Clarifications and documentation

Spec versions should align with the BLUEPRINT version when the spec change is driven by a BLUEPRINT change. Independent spec refinements may increment independently.

### 4.2 AUDIT_LOG.md

The AUDIT_LOG is created fresh each cycle during the Antithesis phase. It is not versioned across cycles -- each cycle produces a distinct AUDIT_LOG that is preserved in Git history. The AUDIT_LOG for a given cycle is identified by:

- The cycle number recorded in STATE.yaml at the time of creation
- The Git commit SHA recorded in `last_artifacts.audit_log_sha`

To retrieve a previous cycle's AUDIT_LOG, use Git history:

```
git log --all --oneline -- .aisync/AUDIT_LOG.md
```

### 4.3 STATE.yaml

STATE.yaml is not versioned because it IS the version tracker. It records the current protocol state including phase, cycle number, blocker dispositions, and artifact SHAs. Its own history is tracked through Git commits. STATE.yaml should never carry a version field -- its role is to track the versions and states of other artifacts.

### 4.4 src/ and tests/

Implementation code in `src/` and test suites in `tests/` are versioned via standard software versioning practices:

- Git commits with descriptive messages (per PROTOCOL.md Section 5 handoff procedures)
- Semantic version tags on releases (see Section 4.5)
- CHANGELOG entries documenting user-facing changes

### 4.5 When to Tag Git Commits

Git tags should be applied at the following points:

| Event | Tag Format | Example |
|-------|-----------|---------|
| Cycle completion (Execution phase exit) | `cycle-N` | `cycle-3` |
| BLUEPRINT MAJOR version change | `blueprint-vMAJOR.MINOR.PATCH` | `blueprint-v1.0.0` |
| Software release from Execution phase | `vMAJOR.MINOR.PATCH` | `v1.2.0` |
| Protocol version adopted by consumer project | `protocol-vMAJOR.MINOR.PATCH` | `protocol-v3.0.0` |

---

## 5. Cycle Number vs. Version

The cycle number and BLUEPRINT version serve different purposes and must not be conflated.

### 5.1 Cycle Number

- A monotonically increasing integer starting at 1
- Increments by 1 after each complete pass through all four phases (Thesis, Antithesis, Synthesis, Execution)
- Tracks how many times the protocol loop has executed
- Recorded in `STATE.yaml` `cycle` field and `BLUEPRINT.json` `cycle` field
- Does not reset in normal operation (see Section 5.4)

### 5.2 BLUEPRINT Version

- A semantic version (MAJOR.MINOR.PATCH) tracking design maturity
- Increments based on the nature of changes, not the passage of cycles
- A cycle with no design changes does not increment the BLUEPRINT version
- A cycle with multiple categories of changes increments by the highest severity

### 5.3 Independence

These values are fully independent:

| Scenario | Cycle | BLUEPRINT Version | Explanation |
|----------|-------|------------------|-------------|
| First design | 1 | 0.1.0 | Initial creation |
| Major redesign in cycle 2 | 2 | 1.0.0 | Breaking changes accepted |
| No design changes in cycles 3-4 | 4 | 1.0.0 | Implementation and testing only |
| New feature in cycle 5 | 5 | 1.1.0 | Additive endpoint |
| Blocker fixes in cycle 5 | 5 | 1.1.1 | Patch from Synthesis |

### 5.4 Cycle Resets

Cycle numbers do not reset in normal operation. The only scenario where a cycle counter might reset is project reinitialization -- creating a fresh `.aisync/` directory from templates. Even in cases of cycle abandonment (see PROTOCOL.md Section 7.3), the cycle counter increments rather than resets.

### 5.5 Correlating Cycles and Versions Using Git Tags

To maintain traceability between cycles and versions, apply Git tags at cycle boundaries:

```
git tag -a cycle-3 -m "Cycle 3 complete. BLUEPRINT v1.1.0"
git tag -a blueprint-v1.1.0 -m "New endpoint added in cycle 3"
```

This allows reconstruction of the relationship between any cycle and the BLUEPRINT version that was active at that point:

```
git tag --list 'cycle-*'
git log --oneline cycle-3..cycle-4 -- .aisync/BLUEPRINT.json
```

---

## 6. Version Compatibility Matrix

### 6.1 Protocol-to-BLUEPRINT Compatibility

| Protocol Version | BLUEPRINT Format | Notes |
|-----------------|-----------------|-------|
| 3.0.x | Template with required fields: `project_name`, `version`, `cycle`, `last_updated`, `core_principles`, `constraints`, `tech_stack`, `io_contracts`, `security_policies`, `non_functional_requirements` | Current format. All fields required for Thesis-to-Antithesis gate. |
| 2.x (archived) | Korean-language plan format | Superseded. Not compatible with v3.0 templates. See `archive/plan_rev2.md`. |
| 1.x (archived) | Original plan format | Superseded. Not compatible with v3.0 templates. |

### 6.2 Detecting Version Mismatches

A version mismatch occurs when a consumer project's `.aisync/` templates do not align with the protocol version being followed. Detection methods:

1. **Template header check:** Compare the `_comment` field in BLUEPRINT.json against the expected protocol version string
2. **Required field check:** Verify that all fields required by the current protocol version exist in the BLUEPRINT
3. **STATE.yaml structure check:** Confirm that STATE.yaml contains all fields defined in the current protocol's template
4. **Quality gate check:** Verify that the consumer project's CHECKLIST.md matches the current protocol's gate conditions

### 6.3 Migration Procedures for Breaking Changes

When a protocol MAJOR version upgrade introduces breaking changes:

1. **Identify gaps:** Compare the old and new template structures field by field
2. **Map existing data:** Determine which existing BLUEPRINT fields map to new fields
3. **Create migration plan:** Document field additions, removals, renames, and restructures
4. **Execute migration:** Update `.aisync/` files to match the new template structure
5. **Validate:** Run the new protocol's quality gate checklist against the migrated artifacts
6. **Test:** Execute one complete cycle to verify the migration

---

## 7. Upgrade Procedures

### 7.1 Upgrading Protocol Version

When the `agent-cowork` repository releases a new protocol version:

**Step 1 -- Review changes:**
- Read the CHANGELOG (or Git commit history) for the new version
- Identify whether the release is MAJOR, MINOR, or PATCH
- Note any breaking changes, new features, or deprecations

**Step 2 -- Update templates (if structure changed):**
- Compare your `.aisync/` templates against the new `templates/` directory
- Add new required fields to BLUEPRINT.json
- Update STATE.yaml if new fields were introduced
- Update CHECKLIST.md if gate conditions changed

**Step 3 -- Update agent prompts (if role definitions changed):**
- Review PROTOCOL.md Section 2 (Roles) for any changes to agent responsibilities
- Review PROTOCOL.md Section 8 (Artifact Ownership Matrix) for ownership changes
- Update the prompt templates used for each phase (`prompts/` directory)
- Ensure agent integration guides (INTEGRATION_CLAUDE_CODE.md, INTEGRATION_GPT_CODEX.md) reflect changes

**Step 4 -- Test before full adoption:**
- Run one complete cycle (Thesis through Execution) using the new protocol version
- Verify all quality gates pass with the updated templates
- Confirm handoff procedures work correctly with updated STATE.yaml structure
- Only after a successful test cycle, adopt the new protocol version for production work

**Step 5 -- Record the upgrade:**
- Tag the Git commit with the new protocol version: `protocol-v3.1.0`
- Document the upgrade in the consumer project's CHANGELOG or decision log

### 7.2 Upgrading BLUEPRINT Version

When a BLUEPRINT version increment occurs within a consumer project:

**Step 1 -- Document the change:**
- Record what changed in BLUEPRINT.json and why
- Note whether the change originated in Thesis (new design) or Synthesis (accepted blocker)
- Update the `"version"` field in BLUEPRINT.json
- Update the `"last_updated"` field

**Step 2 -- Update corresponding artifacts:**
- If `io_contracts` changed: update the corresponding `specs/` files
- If `security_policies` changed: verify implementation alignment
- If `tech_stack` changed: assess implementation impact

**Step 3 -- Update STATE.yaml:**
- Commit the updated BLUEPRINT.json
- Ensure `last_artifacts.blueprint_sha` reflects the new commit SHA after the next phase transition

**Step 4 -- Communicate the change:**
- Include version change details in `handoff_notes` during the next phase transition
- If the change is MAJOR, flag it prominently for the incoming agent

---

## 8. Deprecation Policy

### 8.1 Protocol Feature Deprecation

Protocol features (phases, gate conditions, template fields, procedures) follow a two-step deprecation process:

1. **Announce in MINOR release:** The feature is marked as deprecated in the protocol documentation. A deprecation notice explains what replaces it and when removal will occur. Consumer projects are encouraged to migrate but existing usage continues to work.

2. **Remove in next MAJOR release:** The deprecated feature is removed. Consumer projects that have not migrated must update their workflows as part of the MAJOR version upgrade procedure (Section 7.1).

Example timeline:

| Version | Action |
|---------|--------|
| 3.1.0 | Feature X marked as deprecated with migration guidance |
| 3.2.0 | Feature X still functional but deprecated warnings added |
| 4.0.0 | Feature X removed; migration required |

### 8.2 BLUEPRINT Field Deprecation

When a BLUEPRINT field becomes unnecessary or is replaced:

1. **Mark as deprecated:** Add a `_comment_deprecated_FIELDNAME` entry in BLUEPRINT.json explaining the deprecation reason and replacement
2. **Grace period:** The deprecated field remains valid for 2 complete cycles, giving agents time to adapt
3. **Remove:** After the grace period, the field is removed from the BLUEPRINT template and quality gates no longer check for it

### 8.3 Handling Deprecated but Present Fields

When an agent encounters a deprecated field in BLUEPRINT.json or STATE.yaml:

- **Read it:** The field may still contain useful information during the grace period
- **Do not rely on it:** Use the replacement field or mechanism instead
- **Do not remove it:** Only the BLUEPRINT author (Claude Code) should remove deprecated fields, and only after the grace period
- **Flag it:** Include a note in `handoff_notes` that a deprecated field is still present, recommending removal in the next appropriate phase

---

## 9. Version History

Version history of the TITLE Protocol itself:

| Version | Date | Description |
|---------|------|-------------|
| **v1.0** | 2026-02 | Initial protocol concept. Single planning document (`plan.md`) outlining the dual-agent collaboration idea. No formal phases, gates, or templates. |
| **v2.0** | 2026-02-14 | Korean-language protocol plan (`plan_rev2.md`). Introduced the four-phase dialectical cycle (Thesis, Antithesis, Synthesis, Execution), role definitions, and the BLUEPRINT concept. Written in Korean. See `archive/plan_rev2.md`. |
| **v3.0** | 2026-02-15 | Current version. English-first consolidated protocol with formal quality gates, conflict resolution escalation ladder, rollback paths, handoff procedures, artifact ownership matrix, STATE.yaml tracking, and complete template set. Supersedes all previous versions. See `PROTOCOL.md`. |

---

## 10. Best Practices

**Always update BLUEPRINT version when io_contracts change.** Interface contract changes affect both the implementation agent (GPT-5.3-Codex) and any external consumers. An unversioned contract change creates ambiguity about which contract is authoritative.

**Tag Git commits at cycle completion.** Apply `cycle-N` tags at every Execution phase exit. This creates reliable reference points for correlating cycles with code state, BLUEPRINT versions, and audit reports.

**Keep a CHANGELOG in consumer projects.** The protocol specification does not mandate a specific CHANGELOG format, but consumer projects should maintain one. Record user-facing changes, BLUEPRINT version increments, and notable blocker resolutions.

**Document breaking changes in handoff_notes during Synthesis.** When Synthesis produces a MAJOR BLUEPRINT version increment, Claude Code should explicitly flag this in `handoff_notes` so that GPT-5.3-Codex understands the scope of implementation changes required in the Execution phase.

**Use Git tags for version correlation.** When multiple version dimensions change in the same cycle, apply multiple tags to the same commit. This enables future reconstruction of the complete version state at any point in project history.

**Review protocol version before starting a new project.** Before copying `templates/` to a new consumer project, verify that you are using the latest protocol version from the `agent-cowork` repository. Record the adopted protocol version with a `protocol-vX.Y.Z` tag.

**Do not embed protocol version in BLUEPRINT.json.** The BLUEPRINT tracks its own version and the cycle number. The protocol version is a property of the `agent-cowork` repository, not of individual consumer projects. Consumer projects reference the protocol version through Git submodule, clone, or tag -- not through BLUEPRINT fields.

---

## Cross-References

- [PROTOCOL.md](PROTOCOL.md) -- Complete protocol specification including phase definitions, quality gates, and artifact ownership
- [templates/BLUEPRINT.json](templates/BLUEPRINT.json) -- BLUEPRINT template with the `version` and `cycle` fields
- [templates/STATE.yaml](templates/STATE.yaml) -- State tracking template with `cycle`, `last_artifacts`, and blocker fields
- [GLOSSARY.md](GLOSSARY.md) -- Protocol terminology definitions
- [GETTING_STARTED.md](GETTING_STARTED.md) -- Onboarding guide for consumer projects
- SCALING.md -- Scaling strategies for multi-team and multi-project protocol adoption (planned)

---

**Document Version:** 1.0.0
**Protocol Version:** TITLE Protocol v3.0.0
**License:** MIT
