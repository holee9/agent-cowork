# Glossary

Quick-reference terminology for the TITLE Protocol v3.0.

---

| Term | Definition | Context |
|------|-----------|---------|
| **Thesis** | Phase 1 of the execution loop. Claude Code analyzes requirements and produces the BLUEPRINT and interface specifications. | Phase; BLUEPRINT.json, specs/ |
| **Antithesis** | Phase 2 of the execution loop. GPT-5.3-Codex performs an independent red-team review of the BLUEPRINT and specifications. | Phase; AUDIT_LOG.md |
| **Synthesis** | Phase 3 of the execution loop. Claude Code reviews red-team findings and classifies each blocker as ACCEPTED, DEFERRED, or REJECTED. | Phase; BLUEPRINT.json, AUDIT_LOG.md |
| **Execution** | Phase 4 of the execution loop. GPT-5.3-Codex implements the agreed design, writes tests, and runs CI. | Phase; src/, tests/ |
| **Cycle** | One complete pass through all four phases (Thesis, Antithesis, Synthesis, Execution). Each cycle produces a full audit trail. | All phases |
| **BLUEPRINT** | `BLUEPRINT.json` -- the system design document authored by Claude Code. Contains project principles, constraints, tech stack, IO contracts, security policies, and non-functional requirements. | Artifact; Thesis, Synthesis |
| **AUDIT_LOG** | `AUDIT_LOG.md` -- the red-team findings document authored by GPT-5.3-Codex. Covers 5 audit areas with evidence, impact, and mitigations. | Artifact; Antithesis |
| **STATE.yaml** | The protocol state file that tracks the current phase, leading model, blockers, and artifact SHAs. Updated at every phase transition. | Artifact; all phases |
| **System Owner** | The human operator who approves phase transitions, resolves consensus failures, and provides requirements. | Role; all phases |
| **Blocker** | A critical finding from the red-team audit that must be resolved before the protocol can advance to Execution. Each blocker has an ID, severity, owner, and status. | Antithesis, Synthesis |
| **ACCEPTED** | Blocker disposition: the finding is valid and the design will be changed to address it. | Synthesis |
| **DEFERRED** | Blocker disposition: the finding is valid but will be addressed in a future cycle, with documented reasoning. | Synthesis |
| **REJECTED** | Blocker disposition: the finding is not applicable or the risk is acceptable, with documented reasoning. | Synthesis |
| **CONSENSUS_BLOCKED** | A STATE.yaml status indicating that the two agents cannot reach agreement. Triggers System Owner intervention. | Conflict resolution |
| **Quality Gate** | A set of testable conditions that must be satisfied before a phase transition is permitted. Defined in PROTOCOL.md Section 4. | Phase transitions |
| **Handoff** | The procedure by which one agent transfers control to another at a phase boundary. Includes STATE.yaml update, context transfer, and pre-handoff checklist. | Phase transitions |
