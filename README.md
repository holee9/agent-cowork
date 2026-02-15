# agent-cowork

A guidebook for orchestrating **Claude Code** and **GPT-5.3-Codex** in a closed-loop multi-agent development workflow.

> **This repository is a protocol specification, not a software project.** It contains documentation and templates that consumer projects read to coordinate dual-agent collaboration.

---

## Quick Start

1. **Read** [GETTING_STARTED.md](GETTING_STARTED.md) for a step-by-step onboarding guide
2. **Read** [PROTOCOL.md](PROTOCOL.md) to understand the four-phase execution loop
3. **Copy** the `templates/` directory into your project as `.aisync/`
4. **Start the Thesis phase** -- Claude Code fills out `BLUEPRINT.json`, then the cycle begins

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
| [GETTING_STARTED.md](GETTING_STARTED.md) | Step-by-step onboarding guide for consumer projects | First adoption. Before starting your first cycle. |
| [INTEGRATION_CLAUDE_CODE.md](INTEGRATION_CLAUDE_CODE.md) | Claude Code agent integration guide (Architect role) | Configuring Claude Code for the protocol |
| [INTEGRATION_GPT_CODEX.md](INTEGRATION_GPT_CODEX.md) | GPT-5.3-Codex agent integration guide (Builder/Red Team role) | Configuring Codex for the protocol |
| [CONSENSUS_RESOLUTION.md](CONSENSUS_RESOLUTION.md) | Detailed consensus deadlock resolution procedures | When agents disagree or CONSENSUS_BLOCKED occurs |
| [CONTEXT_OPTIMIZATION.md](CONTEXT_OPTIMIZATION.md) | Context window optimization strategies for AI agents | **FIRST READ for all agents**. Before starting any phase. |
| [VERSIONING.md](VERSIONING.md) | Versioning strategies for protocol, BLUEPRINT, artifacts, and cycles | When managing version increments or protocol upgrades |
| [TROUBLESHOOTING.md](TROUBLESHOOTING.md) | Protocol execution troubleshooting and error recovery | When encountering errors, BLOCKED states, or gate failures |
| [SCALING.md](SCALING.md) | Scaling guide for large projects and parallel workstreams | When managing multi-cycle or multi-feature projects |
| [DESIGN_RATIONALE.md](DESIGN_RATIONALE.md) | Design decision rationale for the TITLE Protocol | Understanding why the protocol works the way it does |
| [REFERENCES.md](REFERENCES.md) | Research references that informed protocol design | Understanding design rationale |
| [docs/EXAMPLES.md](docs/EXAMPLES.md) | Completed artifact examples (SecureNotes API) | When filling out templates for the first time |
| [templates/specs/README.md](templates/specs/README.md) | Guide for creating interface specifications | Thesis phase: before writing specs |
| [prompts/](prompts/) | Phase-specific prompt templates (4 files) | Each phase start: copy-paste to agent |
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
├── GETTING_STARTED.md         # Step-by-step onboarding guide
├── INTEGRATION_CLAUDE_CODE.md # Claude Code agent integration guide
├── INTEGRATION_GPT_CODEX.md   # GPT-5.3-Codex agent integration guide
├── CONSENSUS_RESOLUTION.md    # Consensus deadlock resolution procedures
├── CONTEXT_OPTIMIZATION.md    # Context window optimization for AI agents
├── VERSIONING.md              # Versioning strategies for protocol and artifacts
├── TROUBLESHOOTING.md         # Protocol execution troubleshooting guide
├── SCALING.md                 # Scaling guide for large projects
├── DESIGN_RATIONALE.md        # Design decision rationale
├── REFERENCES.md              # Research references and design rationale sources
├── LICENSE                    # MIT License
├── .gitignore                 # Git ignore rules
├── docs/                      # Supplementary documentation
│   └── EXAMPLES.md            # Completed artifact examples (SecureNotes API)
├── prompts/                   # Phase-specific prompt templates for AI agents
│   ├── THESIS_PROMPT.md       # Thesis phase prompt for Claude Code
│   ├── ANTITHESIS_PROMPT.md   # Antithesis phase prompt for GPT-5.3-Codex
│   ├── SYNTHESIS_PROMPT.md    # Synthesis phase prompt for Claude Code
│   └── EXECUTION_PROMPT.md    # Execution phase prompt for GPT-5.3-Codex
├── templates/                 # Ready-to-copy artifacts for consumer projects
│   ├── BLUEPRINT.json         # System design template (all required fields)
│   ├── AUDIT_LOG.md           # Red-team audit template (5-area checklist)
│   ├── STATE.yaml             # Initial protocol state
│   ├── CHECKLIST.md           # Per-phase completion checklist
│   └── specs/                 # Interface specification templates and guide
│       └── README.md          # How to create specs for your project
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

1. **FIRST**: Read **CONTEXT_OPTIMIZATION.md** to understand which documents to load for your current phase
2. Read **PROTOCOL.md** sections relevant to your current phase (not the entire document)
3. Read your integration guide: **INTEGRATION_CLAUDE_CODE.md** or **INTEGRATION_GPT_CODEX.md** (section matching your phase)
4. Read **GLOSSARY.md** if any terminology is unclear
5. Copy **templates/** to your project's `.aisync/` directory
6. Use the corresponding **prompts/** template at the start of each phase
7. Follow the phase you are assigned to in `STATE.yaml`
8. Use **templates/CHECKLIST.md** to verify gate conditions before requesting a phase transition
9. If conflicts arise, follow **CONSENSUS_RESOLUTION.md** escalation procedures

**Token Management**: Load only REQUIRED documents from CONTEXT_OPTIMIZATION.md. Load RECOMMENDED documents only when uncertain. Avoid loading all documentation at once.

---

## License

[MIT](LICENSE)
