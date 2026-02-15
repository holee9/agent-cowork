# Phase Completion Checklist

Use this checklist to verify that all quality gate conditions are met before requesting a phase transition. Each item corresponds to a testable condition in PROTOCOL.md Section 4.

---

## Phase 1: Thesis (Claude Code)

Before transitioning to Antithesis:

- [ ] `BLUEPRINT.json` exists and is valid JSON
- [ ] `project_name` is set to the actual project name (not a placeholder)
- [ ] `core_principles` array has at least 1 entry
- [ ] `constraints` object has at least 1 key
- [ ] `tech_stack` contains `language` and `framework` fields
- [ ] `io_contracts` has at least 1 endpoint or interface defined
- [ ] `security_policies` contains: `authentication`, `authorization`, `data_handling`, `encryption`
- [ ] `non_functional_requirements` contains: `performance`, `scalability`, `availability`, `observability`
- [ ] Interface specifications exist in `specs/`
- [ ] `STATE.yaml` is updated: `phase: THESIS`, `owner_model: Claude`
- [ ] System Owner has reviewed and approved the BLUEPRINT

---

## Phase 2: Antithesis (GPT-5.3-Codex)

Before transitioning to Synthesis:

- [ ] `AUDIT_LOG.md` exists and follows the template structure
- [ ] Memory/Resource Management area has been evaluated
- [ ] Concurrency/Race Conditions area has been evaluated
- [ ] Hardware/Infrastructure Lifecycle area has been evaluated
- [ ] Security Vulnerabilities area has been evaluated
- [ ] AI Misuse Scenarios area has been evaluated
- [ ] Each critical finding has: ID, Title, Area, Severity, Description, Evidence/PoC, Impact, Suggested Mitigation
- [ ] Critical issues are separated from recommendations
- [ ] Blockers have severity ratings assigned
- [ ] `STATE.yaml` is updated: `phase: ANTITHESIS`, `owner_model: Codex`

---

## Phase 3: Synthesis (Claude Code leads, GPT-5.3-Codex assists)

Before transitioning to Execution:

- [ ] Every blocker from `AUDIT_LOG.md` has a disposition: ACCEPTED, DEFERRED, or REJECTED
- [ ] Each DEFERRED blocker has a documented reason and planned cycle for resolution
- [ ] Each REJECTED blocker has a documented reason with evidence or rationale
- [ ] ACCEPTED blockers are reflected in updated `BLUEPRINT.json` and/or `specs/`
- [ ] `STATE.yaml` blockers array is updated with all dispositions
- [ ] No blocker remains with status: OPEN
- [ ] Codex has reviewed feasibility of accepted design changes
- [ ] System Owner has approved the final design

---

## Phase 4: Execution (GPT-5.3-Codex leads, Claude Code reviews)

Before transitioning to next Thesis cycle:

- [ ] All planned implementation is complete per the BLUEPRINT
- [ ] Unit tests pass
- [ ] Integration tests pass (if applicable)
- [ ] Security tests pass (if applicable)
- [ ] Claude Code has performed high-level review against BLUEPRINT and security policies
- [ ] Documentation is updated: README, CHANGELOG, architecture docs
- [ ] `STATE.yaml` is updated: `cycle` incremented, `phase: THESIS`, `owner_model: Claude`
- [ ] System Owner has approved the release or next-cycle transition
