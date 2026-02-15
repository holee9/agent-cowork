# agent-cowork

A guidebook for orchestrating **Claude Code** and **GPT-5.3-Codex** in a closed-loop multi-agent development workflow.

> **This repository is a protocol specification, not a software project.** It contains documentation and templates that consumer projects read to coordinate dual-agent collaboration.

---

## Quick Start

1. **Read** [PROTOCOL.md](PROTOCOL.md) to understand the four-phase execution loop
2. **Copy** the `templates/` directory into your project as `.aisync/`
3. **Start the Thesis phase** -- Claude Code fills out `BLUEPRINT.json`, then the cycle begins

---

## Protocol Summary

The TITLE Protocol v3.0 defines a four-phase dialectical cycle -- **Thesis, Antithesis, Synthesis, Execution** -- that repeats for each feature or iteration. Claude Code acts as the Architect (design, synthesis, documentation) while GPT-5.3-Codex acts as the Builder and Red Team (audit, implementation, testing).

Every phase transition is gated by testable quality conditions. All protocol state lives in explicit files (`BLUEPRINT.json`, `STATE.yaml`, `AUDIT_LOG.md`), making the process stateless and auditable. A human System Owner approves transitions and resolves deadlocks.

---

## Document Map

| File | Purpose | When to Read |
|------|---------|-------------|
| [PROTOCOL.md](PROTOCOL.md) | Complete protocol specification: roles, phases, gates, handoffs, conflict resolution, rollback | First. This is the primary reference. |
| [GLOSSARY.md](GLOSSARY.md) | Protocol terminology quick reference | When encountering unfamiliar terms |
| [LANGUAGE_POLICY.md](LANGUAGE_POLICY.md) | English-first language conventions for all artifacts | When producing any artifact |
| [templates/BLUEPRINT.json](templates/BLUEPRINT.json) | System design document template with all required fields | Thesis phase: copy and fill out |
| [templates/AUDIT_LOG.md](templates/AUDIT_LOG.md) | Red-team audit template with 5-area checklist | Antithesis phase: copy and fill out |
| [templates/STATE.yaml](templates/STATE.yaml) | Protocol state tracking template | Project setup: copy to initialize state |
| [templates/CHECKLIST.md](templates/CHECKLIST.md) | Per-phase completion checklist | Before every phase transition |
| [archive/plan_rev2.md](archive/plan_rev2.md) | Original Korean protocol plan (superseded) | Historical reference only |
| [LICENSE](LICENSE) | MIT License | As needed |
| [.gitignore](.gitignore) | Git ignore rules | Project setup |

---

## Repository Structure

```
agent-cowork/
├── README.md                  # This file -- entry point and document map
├── PROTOCOL.md                # Consolidated protocol specification
├── GLOSSARY.md                # Protocol terminology quick reference
├── LANGUAGE_POLICY.md         # English-first language conventions
├── LICENSE                    # MIT License
├── .gitignore                 # Git ignore rules
├── templates/                 # Ready-to-copy artifacts for consumer projects
│   ├── BLUEPRINT.json         # System design template (all required fields)
│   ├── AUDIT_LOG.md           # Red-team audit template (5-area checklist)
│   ├── STATE.yaml             # Initial protocol state
│   └── CHECKLIST.md           # Per-phase completion checklist
└── archive/                   # Superseded documents (historical reference only)
    ├── plan_rev2.md           # Original Korean protocol plan
    ├── README.original.md     # Original English README (pre-transformation)
    ├── LANGUAGE_POLICY.original.md  # Original Korean language policy
    └── TITLE_Protocol_v3_Project_Proposal.docx  # Original project proposal
```

---

## For System Owners

As the human supervisor, you:

- **Approve phase transitions** after verifying quality gate conditions (see PROTOCOL.md Section 4)
- **Resolve deadlocks** when agents reach CONSENSUS_BLOCKED status (see PROTOCOL.md Section 6)
- **Provide requirements** at the start of each cycle
- **Communicate in any language** -- agents will produce artifacts in English regardless

---

## For AI Agents

When adopting this protocol in a consumer project:

1. Read **PROTOCOL.md** -- it contains everything you need: roles, phase activities, gate conditions, handoff procedures, and conflict resolution
2. Read **GLOSSARY.md** if any terminology is unclear
3. Copy **templates/** to your project's `.aisync/` directory
4. Follow the phase you are assigned to in `STATE.yaml`
5. Use **templates/CHECKLIST.md** to verify gate conditions before requesting a phase transition

---

## License

[MIT](LICENSE)
