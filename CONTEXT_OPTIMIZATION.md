# Context Window Optimization for TITLE Protocol v3.0

AI agent context management strategies for executing the TITLE Protocol in consumer projects with minimal token consumption.

**Version:** 1.0.0
**Date:** 2026-02-15
**Protocol:** TITLE Protocol v3.0
**Target Audience:** AI agents (Claude Code, GPT-5.3-Codex), System Owners configuring agent sessions

---

## 1. Purpose

### Why Context Optimization Matters

AI agents have limited context windows. Claude Code operates with approximately 200,000 tokens of context capacity. GPT-5.3-Codex has similar constraints. Loading all 6,379 lines of TITLE Protocol documentation would consume an estimated 80,000-95,000 tokens, nearly half of the available context budget, before any consumer project artifacts are read.

Context optimization enables AI agents to:
- Reserve context space for consumer project artifacts including BLUEPRINT.json, AUDIT_LOG.md, source code, and specifications
- Load only phase-relevant protocol documentation rather than the entire corpus
- Execute longer sessions without context exhaustion
- Maintain protocol compliance while minimizing documentation overhead

### Token Budget Allocation Per Phase

The TITLE Protocol operates in four phases. Each phase has different documentation needs and token budgets:

| Phase | Agent Lead | Consumer Project Budget | Protocol Docs Budget | Total Recommended |
|-------|-----------|------------------------|---------------------|------------------|
| Thesis | Claude Code | 20K tokens (requirements, previous cycle artifacts) | 10K tokens | 30K tokens |
| Antithesis | GPT-5.3-Codex | 15K tokens (BLUEPRINT, specs) | 15K tokens | 30K tokens |
| Synthesis | Claude Code | 25K tokens (BLUEPRINT, AUDIT_LOG, specs) | 15K tokens | 40K tokens |
| Execution | GPT-5.3-Codex | 35K tokens (BLUEPRINT, specs, implementation) | 15K tokens | 50K tokens |

These budgets assume progressive disclosure. Agents load REQUIRED documents at session start, then RECOMMENDED documents only when uncertainty arises, and OPTIONAL documents only for specific edge cases.

### Impact of Loading All vs. Selective Loading

Loading strategy comparison for a typical 5-cycle project:

**All Documentation Loaded (baseline):**
- Protocol documentation: approximately 95,000 tokens per session
- Consumer artifacts: approximately 30,000-50,000 tokens per session
- Total context consumption: 125,000-145,000 tokens (62-72% of available capacity)
- Remaining for work: 55,000-75,000 tokens (38-28%)
- Sessions before context exhaustion: 1-2 per phase

**Selective Phase-Relevant Loading (optimized):**
- Protocol documentation: approximately 12,000-22,000 tokens per session
- Consumer artifacts: approximately 30,000-50,000 tokens per session
- Total context consumption: 42,000-72,000 tokens (21-36% of available capacity)
- Remaining for work: 128,000-158,000 tokens (64-79%)
- Sessions before context exhaustion: 3-5 per phase

Token savings: 60-73,000 tokens per session, enabling 2-3x longer agent sessions.

### Target Audience

This document is for:
- **AI agents starting each phase:** Guidance on which protocol documents to load based on current phase from STATE.yaml
- **System Owners configuring agent sessions:** Session initialization scripts and prompt engineering for optimal context usage
- **Protocol adopters designing automation:** CI/CD pipelines and headless execution environments

---

## 2. Phase-Specific Document Matrix

Each phase has distinct documentation needs. Load documents in priority order: REQUIRED first, then RECOMMENDED only if needed, and OPTIONAL only for specific edge cases.

### Thesis Phase

**Agent:** Claude Code
**Budget:** 30K tokens (10K protocol documentation, 20K consumer artifacts)
**Primary Activities:** Analyze requirements, create BLUEPRINT.json, write interface specifications in specs directory, document design rationale

#### REQUIRED (Load at Session Start)

| Document | Lines | Est. Tokens | Why Required |
|----------|-------|-------------|--------------|
| PROTOCOL.md Section 3.1 | 95 lines | 1,200 | Thesis phase activities, responsibilities, exit criteria |
| INTEGRATION_CLAUDE_CODE.md Section 2.1 | 80 lines | 1,000 | Claude Code's Thesis responsibilities and boundaries |
| prompts/THESIS_PROMPT.md | 152 lines | 1,900 | Phase-specific prompt guidance and checklist |
| templates/BLUEPRINT.json | 50 lines | 600 | BLUEPRINT template structure and required fields |
| templates/specs/README.md | 40 lines | 500 | Interface specification formats and conventions |
| LANGUAGE_POLICY.md | 76 lines | 950 | Artifact language requirements (English only) |

**REQUIRED Subtotal:** 493 lines, approximately 6,150 tokens

#### RECOMMENDED (Load If Uncertain About Procedures)

| Document | Lines | Est. Tokens | When to Load |
|----------|-------|-------------|--------------|
| GETTING_STARTED.md Section "First Cycle - Thesis" | 50 lines | 625 | First cycle or new team onboarding |
| GLOSSARY.md | 24 lines | 300 | Unfamiliar terminology (BLUEPRINT, core_principles, io_contracts) |
| docs/EXAMPLES.md BLUEPRINT example | 100 lines | 1,250 | Reference for well-formed BLUEPRINT structure |
| PROTOCOL.md Section 4.1 | 25 lines | 310 | Detailed Thesis-to-Antithesis gate conditions |

**RECOMMENDED Subtotal:** 199 lines, approximately 2,485 tokens

#### OPTIONAL (Load Only If Specific Need Arises)

| Document | Lines | Est. Tokens | When to Load |
|----------|-------|-------------|--------------|
| DESIGN_RATIONALE.md | 500 lines | 6,250 | Understanding protocol design philosophy |
| REFERENCES.md | 78 lines | 975 | External standards or methodologies reference |
| CONSENSUS_RESOLUTION.md | 591 lines | 7,388 | Conflict resolution only if disagreement with System Owner occurs |

**OPTIONAL Subtotal:** 1,169 lines, approximately 14,613 tokens

**Thesis Phase Context Budget:**
- Minimal (REQUIRED only): 6,150 tokens
- Standard (REQUIRED + RECOMMENDED): 8,635 tokens
- Maximum (all documents): 23,248 tokens

---

### Antithesis Phase

**Agent:** GPT-5.3-Codex
**Budget:** 30K tokens (15K protocol documentation, 15K consumer artifacts including BLUEPRINT and specs)
**Primary Activities:** Independent red-team audit across 5 areas, document findings in AUDIT_LOG.md, optional PoC exploit code

#### REQUIRED (Load at Session Start)

| Document | Lines | Est. Tokens | Why Required |
|----------|-------|-------------|--------------|
| PROTOCOL.md Section 3.2 | 60 lines | 750 | Antithesis phase activities and audit independence principle |
| INTEGRATION_GPT_CODEX.md Section 2.2 | 90 lines | 1,125 | GPT-5.3-Codex Antithesis responsibilities |
| INTEGRATION_GPT_CODEX.md Section 3 | 200 lines | 2,500 | Red Team audit methodology for all 5 areas |
| prompts/ANTITHESIS_PROMPT.md | 196 lines | 2,450 | Phase-specific audit prompts and checklist |
| templates/AUDIT_LOG.md | 108 lines | 1,350 | Audit log template structure and finding format |
| BLUEPRINT.json (from consumer project) | Varies | Varies | Target of the audit (consumer artifact, not protocol doc) |
| specs/ directory (from consumer project) | Varies | Varies | Interface specifications to audit (consumer artifact) |

**REQUIRED Subtotal:** 654 lines protocol documentation, approximately 8,175 tokens (excluding consumer artifacts)

#### RECOMMENDED (Load If Uncertain About Procedures)

| Document | Lines | Est. Tokens | When to Load |
|----------|-------|-------------|--------------|
| GETTING_STARTED.md Section "First Cycle - Antithesis" | 60 lines | 750 | First cycle or uncertainty about handoff procedures |
| docs/EXAMPLES.md AUDIT_LOG example | 80 lines | 1,000 | Reference for well-formed audit report |
| PROTOCOL.md Section 4.2 | 25 lines | 310 | Detailed Antithesis-to-Synthesis gate conditions |
| GLOSSARY.md | 24 lines | 300 | Terminology clarification (blocker, severity, Critical vs Recommendation) |

**RECOMMENDED Subtotal:** 189 lines, approximately 2,360 tokens

#### OPTIONAL (Load Only If Specific Need Arises)

| Document | Lines | Est. Tokens | When to Load |
|----------|-------|-------------|--------------|
| TROUBLESHOOTING.md Section "Audit Issues" | 150 lines | 1,875 | Only if audit process errors occur |
| INTEGRATION_GPT_CODEX.md Section 9 | 100 lines | 1,250 | Common pitfalls reference (only if needed) |

**OPTIONAL Subtotal:** 250 lines, approximately 3,125 tokens

**Antithesis Phase Context Budget:**
- Minimal (REQUIRED only): 8,175 tokens
- Standard (REQUIRED + RECOMMENDED): 10,535 tokens
- Maximum (all documents): 13,660 tokens

---

### Synthesis Phase

**Agent:** Claude Code (lead), GPT-5.3-Codex (assistant)
**Budget:** 40K tokens (15K protocol documentation, 25K consumer artifacts including BLUEPRINT, AUDIT_LOG, specs)
**Primary Activities:** Review all audit findings, classify blockers as ACCEPTED/DEFERRED/REJECTED, update BLUEPRINT and specs for accepted changes, resolve conflicts

#### REQUIRED (Load at Session Start)

| Document | Lines | Est. Tokens | Why Required |
|----------|-------|-------------|--------------|
| PROTOCOL.md Section 3.3 | 80 lines | 1,000 | Synthesis phase activities and blocker classification rules |
| INTEGRATION_CLAUDE_CODE.md Section 2.3 | 100 lines | 1,250 | Claude Code Synthesis responsibilities and decision authority |
| prompts/SYNTHESIS_PROMPT.md | 180 lines | 2,250 | Phase-specific synthesis prompts and disposition framework |
| CONSENSUS_RESOLUTION.md (full document) | 591 lines | 7,388 | Critical for blocker classification, escalation ladder, and conflict resolution |
| AUDIT_LOG.md (from consumer project) | Varies | Varies | Findings to review and classify (consumer artifact) |
| BLUEPRINT.json (from consumer project) | Varies | Varies | Design to update based on accepted blockers (consumer artifact) |
| specs/ (from consumer project) | Varies | Varies | Specifications to update if interfaces change (consumer artifact) |

**REQUIRED Subtotal:** 951 lines protocol documentation, approximately 11,888 tokens (excluding consumer artifacts)

**Critical Note:** CONSENSUS_RESOLUTION.md is REQUIRED during Synthesis because blocker disposition is the central activity of this phase. All four escalation levels, CONSENSUS_BLOCKED procedures, and System Owner decision frameworks are needed for proper disposition.

#### RECOMMENDED (Load If Uncertain About Procedures)

| Document | Lines | Est. Tokens | When to Load |
|----------|-------|-------------|--------------|
| GETTING_STARTED.md Section "First Cycle - Synthesis" | 70 lines | 875 | First cycle or uncertainty about handoff procedures |
| GLOSSARY.md | 24 lines | 300 | Terminology clarification (ACCEPTED, DEFERRED, REJECTED, CONSENSUS_BLOCKED) |
| PROTOCOL.md Section 4.3 | 30 lines | 375 | Detailed Synthesis-to-Execution gate conditions |

**RECOMMENDED Subtotal:** 124 lines, approximately 1,550 tokens

#### OPTIONAL (Load Only If Specific Need Arises)

| Document | Lines | Est. Tokens | When to Load |
|----------|-------|-------------|--------------|
| TROUBLESHOOTING.md Section "Common Error Patterns" | 200 lines | 2,500 | Only if systematic issues detected in blocker dispositions |
| docs/EXAMPLES.md STATE.yaml example | 50 lines | 625 | Reference for blocker array structure |

**OPTIONAL Subtotal:** 250 lines, approximately 3,125 tokens

**Synthesis Phase Context Budget:**
- Minimal (REQUIRED only): 11,888 tokens
- Standard (REQUIRED + RECOMMENDED): 13,438 tokens
- Maximum (all documents): 16,563 tokens

---

### Execution Phase

**Agent:** GPT-5.3-Codex (lead), Claude Code (reviewer)
**Budget:** 50K tokens (15K protocol documentation, 35K consumer artifacts including BLUEPRINT, specs, implementation code)
**Primary Activities:** Implement production code in src, write tests in tests directory, create CI scripts, update documentation, self-correct until tests pass

#### REQUIRED (Load at Session Start)

| Document | Lines | Est. Tokens | Why Required |
|----------|-------|-------------|--------------|
| PROTOCOL.md Section 3.4 | 70 lines | 875 | Execution phase activities and completion criteria |
| INTEGRATION_GPT_CODEX.md Section 2.4 | 80 lines | 1,000 | GPT-5.3-Codex Execution responsibilities |
| INTEGRATION_GPT_CODEX.md Section 7 | 150 lines | 1,875 | Implementation standards (tech stack, IO contracts, security policies, tests) |
| prompts/EXECUTION_PROMPT.md | 187 lines | 2,338 | Phase-specific implementation prompts and quality checklist |
| BLUEPRINT.json (from consumer project) | Varies | Varies | Design to implement (consumer artifact) |
| specs/ (from consumer project) | Varies | Varies | Interface specifications to implement (consumer artifact) |
| AUDIT_LOG.md (from consumer project) | Varies | Varies | Context on accepted blockers and mitigations (consumer artifact) |

**REQUIRED Subtotal:** 487 lines protocol documentation, approximately 6,088 tokens (excluding consumer artifacts)

#### RECOMMENDED (Load If Uncertain About Procedures)

| Document | Lines | Est. Tokens | When to Load |
|----------|-------|-------------|--------------|
| GETTING_STARTED.md Section "First Cycle - Execution" | 60 lines | 750 | First cycle or uncertainty about handoff procedures |
| LANGUAGE_POLICY.md Section 3 | 25 lines | 310 | Artifact language requirements for code comments and documentation |
| PROTOCOL.md Section 4.4 | 30 lines | 375 | Detailed Execution-to-Thesis gate conditions |

**RECOMMENDED Subtotal:** 115 lines, approximately 1,435 tokens

#### OPTIONAL (Load Only If Specific Need Arises)

| Document | Lines | Est. Tokens | When to Load |
|----------|-------|-------------|--------------|
| TROUBLESHOOTING.md | 879 lines | 10,988 | Only if implementation errors occur |
| SCALING.md | 643 lines | 8,038 | Only for large projects (more than 10 services or more than 50K LOC) |
| INTEGRATION_GPT_CODEX.md Section 9 | 100 lines | 1,250 | Common pitfalls reference (only if needed) |

**OPTIONAL Subtotal:** 1,622 lines, approximately 20,276 tokens

**Execution Phase Context Budget:**
- Minimal (REQUIRED only): 6,088 tokens
- Standard (REQUIRED + RECOMMENDED): 7,523 tokens
- Maximum (all documents): 27,799 tokens

---

## 3. Document Size Reference Table

Complete protocol documentation inventory with line counts and estimated token costs. Token estimates use 12.5 tokens per line average for technical documentation.

### Core Protocol Documents

| Document | Lines | Est. Tokens | Content Type |
|----------|-------|-------------|--------------|
| PROTOCOL.md | 402 | 5,025 | Complete protocol specification |
| INTEGRATION_CLAUDE_CODE.md | 507 | 6,338 | Claude Code integration guide |
| INTEGRATION_GPT_CODEX.md | 638 | 7,975 | GPT-5.3-Codex integration guide |
| CONSENSUS_RESOLUTION.md | 591 | 7,388 | Conflict resolution procedures |
| GETTING_STARTED.md | 364 | 4,550 | Adoption guide and walkthrough |
| GLOSSARY.md | 24 | 300 | Terminology quick reference |
| LANGUAGE_POLICY.md | 76 | 950 | Artifact language conventions |

**Core Documents Subtotal:** 2,602 lines, approximately 32,526 tokens

### Extended Documentation

| Document | Lines | Est. Tokens | Content Type |
|----------|-------|-------------|--------------|
| TROUBLESHOOTING.md | 879 | 10,988 | Error recovery and debugging |
| SCALING.md | 643 | 8,038 | Large project optimization strategies |
| VERSIONING.md | 400 | 5,000 | Protocol version management |
| DESIGN_RATIONALE.md | 500 | 6,250 | Protocol design philosophy |
| REFERENCES.md | 78 | 975 | External standards and methodologies |

**Extended Documentation Subtotal:** 2,500 lines, approximately 31,251 tokens

### Examples and Templates

| Document | Lines | Est. Tokens | Content Type |
|----------|-------|-------------|--------------|
| docs/EXAMPLES.md | 386 | 4,825 | Complete artifact examples (BLUEPRINT, AUDIT_LOG, STATE) |
| prompts/THESIS_PROMPT.md | 152 | 1,900 | Thesis phase agent prompt |
| prompts/ANTITHESIS_PROMPT.md | 196 | 2,450 | Antithesis phase agent prompt |
| prompts/SYNTHESIS_PROMPT.md | 180 | 2,250 | Synthesis phase agent prompt |
| prompts/EXECUTION_PROMPT.md | 187 | 2,338 | Execution phase agent prompt |
| templates/BLUEPRINT.json | 50 | 600 | BLUEPRINT template |
| templates/AUDIT_LOG.md | 108 | 1,350 | Audit log template |
| templates/STATE.yaml | 60 | 750 | Protocol state tracker template |
| templates/CHECKLIST.md | 68 | 850 | Quality gate verification checklist |

**Examples and Templates Subtotal:** 1,387 lines, approximately 17,313 tokens

### Grand Total

**All Protocol Documentation:** 6,489 lines, approximately 81,090 tokens

**Context Impact:** Loading all documentation consumes 40% of a 200K token context window before any consumer project artifacts are loaded.

---

## 4. Token Budget Management

### Phase Budget Allocation

From Section 1, each phase has a recommended token budget:

| Phase | Total Budget | Protocol Docs | Consumer Artifacts | Work Reserve |
|-------|-------------|---------------|-------------------|--------------|
| Thesis | 30K | 10K (REQUIRED + RECOMMENDED) | 20K | Reserve 170K for iterative design |
| Antithesis | 30K | 15K (REQUIRED + RECOMMENDED + audit methodology) | 15K | Reserve 170K for audit work |
| Synthesis | 40K | 15K (REQUIRED + RECOMMENDED + conflict resolution) | 25K | Reserve 160K for disposition work |
| Execution | 50K | 15K (REQUIRED + RECOMMENDED + implementation standards) | 35K | Reserve 150K for implementation |

These budgets assume agents unload OPTIONAL documentation unless specifically needed.

### Calculating Current Context Usage

AI agents should track token consumption throughout a session:

**Session Start Baseline:**
- Protocol documentation loaded: sum of REQUIRED and RECOMMENDED document tokens
- Consumer artifacts loaded: BLUEPRINT, specs, AUDIT_LOG, STATE.yaml, relevant source files
- Total baseline consumption: protocol + consumer artifacts

**Progressive Consumption:**
- Each user interaction adds tokens (user message, agent response)
- Each additional document loaded adds to consumption
- Each code generation or artifact update adds to consumption

**Context Health Thresholds:**
- Green (0-50% consumed): Normal operation, load RECOMMENDED documents freely
- Yellow (50-75% consumed): Conservative mode, defer OPTIONAL documents until needed
- Orange (75-85% consumed): Critical mode, unload OPTIONAL documents, use external memory files
- Red (85%+ consumed): Emergency mode, start new session or execute clear command

**Calculation Example for Synthesis Phase:**

Session start:
- REQUIRED protocol documentation: 11,888 tokens
- Consumer BLUEPRINT: 3,000 tokens
- Consumer AUDIT_LOG: 5,000 tokens
- Consumer specs: 2,000 tokens
- Total baseline: 21,888 tokens (10.9% of 200K context)

After 10 message exchanges (average 500 tokens per exchange):
- Baseline: 21,888 tokens
- Conversation history: 5,000 tokens
- Total consumption: 26,888 tokens (13.4% of 200K context)
- Status: Green, continue normally

After loading RECOMMENDED documents:
- Baseline: 21,888 tokens
- RECOMMENDED documents: 1,550 tokens
- Conversation history: 5,000 tokens
- Total consumption: 28,438 tokens (14.2% of 200K context)
- Status: Green, still healthy

### When to Start a New Session vs. Continue

**Continue Current Session When:**
- Context consumption is below 75% of available capacity
- Current phase work is nearly complete (approaching quality gate)
- No complex multi-turn conversations or large artifact generations are planned
- Agent has clear memory of recent decisions and context

**Start New Session When:**
- Context consumption exceeds 85% of available capacity
- Phase transition is about to occur (always start fresh at phase boundaries)
- Agent needs to load OPTIONAL documentation that would push consumption to Orange or Red threshold
- Complex refactoring or large-scale code generation is planned
- Agent has lost context or needs to re-read foundational artifacts

**Session Transition Best Practices:**
- Commit all work to Git before ending session
- Update STATE.yaml with current phase status and handoff_notes
- Document in-progress work in handoff_notes for next session
- Next session begins by reading STATE.yaml and recent Git commits for context recovery

### Progressive Loading Strategy

Load documentation in three tiers based on need:

**Tier 1 - REQUIRED (Load at Session Start):**
- Always loaded at the beginning of each phase
- Contains phase-critical procedures, responsibilities, and exit criteria
- Minimal token cost, maximum value
- Never unload during the phase

**Tier 2 - RECOMMENDED (Load Only If Needed):**
- Load when agent is uncertain about procedures or terminology
- Examples: first cycle execution, unfamiliar edge case, complex conflict resolution
- Moderate token cost, high value when uncertainty exists
- Unload after uncertainty is resolved if context budget becomes constrained

**Tier 3 - OPTIONAL (Load Only for Specific Edge Cases):**
- Load only for specific scenarios: troubleshooting errors, scaling large projects, understanding design philosophy
- High token cost, low frequency of use
- Always unload after immediate need is satisfied

**Progressive Loading Example for Antithesis Phase:**

Session initialization:
1. Read STATE.yaml to confirm phase is ANTITHESIS and owner_model is Codex
2. Load REQUIRED documents: PROTOCOL Section 3.2, INTEGRATION_GPT_CODEX Sections 2.2 and 3, ANTITHESIS_PROMPT, AUDIT_LOG template
3. Load consumer artifacts: BLUEPRINT.json, specs directory
4. Begin audit work

During audit (if uncertainty arises):
5. Agent encounters unfamiliar terminology (e.g., "io_contracts") → Load GLOSSARY.md (300 tokens)
6. Agent is unsure about audit report format → Load EXAMPLES.md AUDIT_LOG example (1,000 tokens)

At audit completion:
7. Verify gate conditions → Load PROTOCOL Section 4.2 (310 tokens) for detailed gate checklist
8. No errors encountered → TROUBLESHOOTING.md remains unloaded (saving 10,988 tokens)

Total protocol documentation loaded: 8,175 (REQUIRED) + 300 + 1,000 + 310 = 9,785 tokens (vs. 81,090 tokens if all documentation loaded)

**Token Savings:** 71,305 tokens (87.9% reduction in protocol documentation overhead)

---

## 5. Selective Section Loading

Some protocol documents are organized into sections that map directly to phases. Load only the relevant sections instead of the entire document.

### PROTOCOL.md Section-by-Section Loading Guide

PROTOCOL.md is 402 lines total, but agents typically need only specific sections per phase.

**Section 1 - Purpose and Principles (30 lines, 375 tokens):**
- Load for: All phases if unfamiliar with protocol
- Skip if: Agent has executed multiple cycles and understands core principles

**Section 2 - Roles (90 lines, 1,125 tokens):**
- Load for: All phases (defines agent responsibilities and boundaries)
- Section 2.1 (Claude Code role): Thesis, Synthesis
- Section 2.2 (GPT-5.3-Codex role): Antithesis, Execution
- Section 2.3 (System Owner role): All phases (for escalation context)

**Section 3 - Execution Loop (180 lines total, 2,250 tokens):**
- Section 3.1 Thesis: Thesis phase only (60 lines, 750 tokens)
- Section 3.2 Antithesis: Antithesis phase only (50 lines, 625 tokens)
- Section 3.3 Synthesis: Synthesis phase only (40 lines, 500 tokens)
- Section 3.4 Execution: Execution phase only (30 lines, 375 tokens)

Load only the current phase subsection, not all four subsections.

**Section 4 - Phase Transition Gates (70 lines total, 875 tokens):**
- Section 4.1 Thesis to Antithesis: Thesis phase only (at exit)
- Section 4.2 Antithesis to Synthesis: Antithesis phase only (at exit)
- Section 4.3 Synthesis to Execution: Synthesis phase only (at exit)
- Section 4.4 Execution to Thesis: Execution phase only (at exit)

Load only the outgoing gate for the current phase. Incoming agent loads relevant gate during post-handoff checklist if needed.

**Section 5 - Handoff Procedures (40 lines, 500 tokens):**
- Load for: Outgoing phase only (when preparing handoff)
- Incoming agent loads during post-handoff checklist if uncertainty exists

**Section 6 - Conflict Resolution (30 lines, 375 tokens):**
- Load for: Synthesis phase primarily (blocker disposition work)
- All phases if CONSENSUS_BLOCKED status is detected in STATE.yaml

**Section 7 - Rollback and Recovery (25 lines, 310 tokens):**
- Load for: Only when rollback is needed (triggered by BLOCKED status or design flaw discovery)
- Skip during normal phase execution

**Section 8 - Artifact Ownership Matrix (20 lines, 250 tokens):**
- Load for: All phases (quick reference for artifact permissions)
- Alternative: Use phase-specific integration guides which include ownership details

**Selective Loading Example:**

Thesis Phase Agent (Claude Code):
- Load Section 2.1 (Claude Code role): 30 lines, 375 tokens
- Load Section 3.1 (Thesis activities): 60 lines, 750 tokens
- Load Section 4.1 (Thesis to Antithesis gate): 18 lines, 225 tokens
- Load Section 8 (Artifact ownership): 20 lines, 250 tokens
- **Total:** 128 lines, 1,600 tokens (vs. 402 lines, 5,025 tokens for full document)
- **Savings:** 274 lines, 3,425 tokens (68% reduction)

### Integration Guide Selective Loading

INTEGRATION_CLAUDE_CODE.md (507 lines, 6,338 tokens) and INTEGRATION_GPT_CODEX.md (638 lines, 7,975 tokens) follow similar section-by-section organization.

**INTEGRATION_CLAUDE_CODE.md Section Loading:**

- Section 1 Overview: All phases (30 lines, 375 tokens) - responsibilities summary
- Section 2 Phase-by-Phase Responsibilities:
  - Section 2.1 Thesis: Thesis phase only (80 lines, 1,000 tokens)
  - Section 2.2 Antithesis: Never load (Claude Code is passive during Antithesis)
  - Section 2.3 Synthesis: Synthesis phase only (100 lines, 1,250 tokens)
  - Section 2.4 Execution: Execution phase only (50 lines, 625 tokens)
- Section 3 STATE.yaml Operations: All phases (60 lines, 750 tokens)
- Section 4 Artifact Ownership: All phases (40 lines, 500 tokens)
- Section 5 Handoff Procedures: When preparing handoff (60 lines, 750 tokens)
- Section 6 Conflict Resolution: Synthesis primarily (40 lines, 500 tokens)
- Section 7 Quality Gate Verification: Thesis and Synthesis exits (50 lines, 625 tokens)
- Section 8 Session Management: All phases (30 lines, 375 tokens)
- Section 9 Common Pitfalls: OPTIONAL, load only if needed (60 lines, 750 tokens)

**Thesis Phase Claude Code Selective Load:**
- Section 1 + Section 2.1 + Section 3 + Section 4 + Section 7 = 260 lines, 3,250 tokens
- vs. full document 507 lines, 6,338 tokens
- **Savings:** 247 lines, 3,088 tokens (49% reduction)

**INTEGRATION_GPT_CODEX.md Section Loading:**

- Section 1 Overview: All phases (40 lines, 500 tokens)
- Section 2 Phase-by-Phase Responsibilities:
  - Section 2.1 Thesis: Never load (GPT-5.3-Codex is passive)
  - Section 2.2 Antithesis: Antithesis phase only (90 lines, 1,125 tokens)
  - Section 2.3 Synthesis: Synthesis phase only (60 lines, 750 tokens)
  - Section 2.4 Execution: Execution phase only (80 lines, 1,000 tokens)
- Section 3 Red Team Audit Methodology: Antithesis phase only (200 lines, 2,500 tokens)
- Section 4 STATE.yaml Operations: All phases (70 lines, 875 tokens)
- Section 5 Artifact Ownership Matrix: All phases (50 lines, 625 tokens)
- Section 6 Handoff Procedures: When preparing handoff (80 lines, 1,000 tokens)
- Section 7 Implementation Standards: Execution phase only (150 lines, 1,875 tokens)
- Section 8 Session Management: All phases (40 lines, 500 tokens)
- Section 9 Common Pitfalls: OPTIONAL, load only if needed (100 lines, 1,250 tokens)

**Execution Phase GPT-5.3-Codex Selective Load:**
- Section 1 + Section 2.4 + Section 4 + Section 5 + Section 7 + Section 8 = 430 lines, 5,375 tokens
- vs. full document 638 lines, 7,975 tokens
- **Savings:** 208 lines, 2,600 tokens (33% reduction)

---

## 6. Multi-Session Strategy

Complex phases may require multiple agent sessions due to context exhaustion, session timeouts, or multi-day workflows.

### When to Use Clear Command and Start Fresh Session

**Mandatory Clear Points:**
- After Thesis phase completion (before Antithesis begins)
- After Antithesis phase completion (before Synthesis begins)
- After Synthesis phase completion (before Execution begins)
- After Execution phase completion (before next cycle Thesis begins)

Phase boundaries are natural session breaks because:
1. Handoff procedures explicitly preserve all state in STATE.yaml and Git commits
2. Incoming agent needs different protocol documentation than outgoing agent
3. Consumer artifacts may have changed (BLUEPRINT updated, AUDIT_LOG created)
4. Fresh context window enables full documentation loading without legacy conversation overhead

**Recommended Clear Points:**
- When context consumption exceeds 85% during active work
- When switching between major activities within a phase (e.g., during Execution, after tests pass and before documentation updates)
- When agent encounters errors requiring troubleshooting documentation (clear after issue resolved)
- After complex multi-turn discussions with System Owner

**Avoid Clearing When:**
- In the middle of active work on a single artifact (e.g., mid-BLUEPRINT authoring)
- During conflict resolution escalation (preserve conversation context until resolution)
- When less than 10 message exchanges have occurred (unnecessary overhead)

### State Preservation Across Sessions

The TITLE Protocol is designed for stateless agent operation. All critical state lives in explicit files, not in conversation history.

**What Persists Across Sessions (No Action Required):**
- STATE.yaml with phase, owner_model, status, cycle, blockers, handoff_notes
- Git commit history with all artifact changes and commit messages
- Consumer artifacts: BLUEPRINT.json, AUDIT_LOG.md, specs directory, src directory, tests directory
- Protocol documentation (re-load selectively at next session start)

**What Does NOT Persist (Conversation-Only Context):**
- Agent's memory of previous message exchanges
- Uncommitted artifact changes
- Informal discussions or clarifications not recorded in artifacts
- System Owner instructions not reflected in STATE.yaml or Git

**State Preservation Best Practices:**

Before ending session:
1. Commit all work to Git with descriptive messages
2. Update STATE.yaml handoff_notes with:
   - Summary of work completed this session
   - In-progress work and next steps
   - Open questions or areas of concern
3. Update STATE.yaml last_artifacts with current Git SHAs
4. If mid-phase, document progress toward quality gate in handoff_notes

At next session start:
1. Read STATE.yaml to recover phase, status, and handoff_notes
2. Read recent Git commits (last 3-5 commits) to understand recent changes
3. Review blockers array for any CONSENSUS_BLOCKED or OPEN statuses
4. Load phase-appropriate REQUIRED documentation
5. Resume work from last checkpoint documented in handoff_notes

**Session Resumption Example:**

Synthesis phase, Session 1 ends:
- Claude Code reviewed 8 of 12 blockers from AUDIT_LOG
- Classified 5 as ACCEPTED, 2 as DEFERRED, 1 as REJECTED
- 4 blockers remain with status OPEN
- Committed BLUEPRINT updates for 5 accepted blockers
- Updated STATE.yaml handoff_notes: "Synthesis in progress. 8 of 12 blockers resolved. Remaining blockers: BLK-009 through BLK-012. BLK-009 requires System Owner clarification on performance NFR threshold. Resume with BLK-009 next session."

Synthesis phase, Session 2 begins:
- Read STATE.yaml: phase is SYNTHESIS, owner_model is Claude, status is OK, handoff_notes indicates 4 blockers remain
- Read Git commits: last commit updated BLUEPRINT with 5 accepted blocker mitigations
- Read blockers array: BLK-001 through BLK-008 are resolved, BLK-009 through BLK-012 are OPEN
- Load REQUIRED documentation: PROTOCOL Section 3.3, INTEGRATION_CLAUDE_CODE Section 2.3, SYNTHESIS_PROMPT, CONSENSUS_RESOLUTION
- Resume work: Begin reviewing BLK-009, consult System Owner as noted in handoff_notes

**Context Recovery:** Session 2 agent has complete state recovery without conversation history from Session 1.

### Session Handoff Best Practices

**For Outgoing Agent (Ending Session):**

1. **Commit Frequently:** Small, focused commits with clear messages enable fine-grained recovery
2. **Detailed Handoff Notes:** Write as if the incoming agent has no memory of previous sessions (because it does not)
3. **Explicit Next Steps:** State exactly what work remains and where to begin next session
4. **Document Decisions:** All decisions and rationale must be in artifacts, not just in conversation history
5. **Update Artifact SHAs:** Ensure last_artifacts in STATE.yaml reflects committed changes

**For Incoming Agent (Starting Session):**

1. **STATE.yaml First:** Always read STATE.yaml before loading any other documentation
2. **Git History Review:** Read last 3-5 commits to understand recent work trajectory
3. **Selective Documentation Load:** Load only phase-relevant REQUIRED documentation, defer RECOMMENDED until needed
4. **Verify Artifact Integrity:** Check that artifact SHAs in STATE.yaml match actual committed versions
5. **Resume from Checkpoint:** Use handoff_notes to identify exact resumption point

### Context-Aware Session Planning for Long Cycles

Large projects with complex implementations may span 5-10 sessions per phase. Plan session boundaries proactively.

**Session Planning Heuristics:**

Thesis Phase (typical: 2-3 sessions):
- Session 1: Requirements analysis, initial BLUEPRINT structure, core_principles and constraints
- Session 2: Complete BLUEPRINT, write interface specifications in specs directory
- Session 3 (if needed): Design rationale documentation, quality gate verification

Antithesis Phase (typical: 2-4 sessions):
- Session 1: Audit areas 1-2 (Memory/Resource, Concurrency)
- Session 2: Audit areas 3-4 (Hardware/Infrastructure, Security)
- Session 3: Audit area 5 (AI Misuse), PoC exploit code if needed
- Session 4 (if needed): Finalize AUDIT_LOG, verify gate conditions

Synthesis Phase (typical: 2-3 sessions):
- Session 1: Review and classify half of blockers
- Session 2: Review and classify remaining blockers, update BLUEPRINT for accepted changes
- Session 3 (if needed): Conflict resolution, System Owner consultations, gate verification

Execution Phase (typical: 3-6 sessions):
- Session 1: Implement core functionality in src directory
- Session 2: Write unit and integration tests
- Session 3: Self-correction until tests pass
- Session 4: Write security tests, implement accepted blocker mitigations
- Session 5: Update documentation (README, CHANGELOG, architecture docs)
- Session 6 (if needed): Claude Code high-level review, final refinements

**Proactive Session Boundaries:**

Set session boundaries at natural checkpoints:
- After completing a major section of work (all io_contracts implemented, all tests written)
- Before switching to a different artifact (from BLUEPRINT to specs, from src to tests)
- When approaching 75% context consumption
- At the end of each working day for multi-day phases

**Session Boundary Commits:**

Use Git commit messages to mark session boundaries:
```
feat(thesis): complete BLUEPRINT core structure [session 1 end]
audit(antithesis): complete Memory and Concurrency review [session 1 end]
fix(synthesis): classify all blockers [session 2 end]
impl(execution): all io_contracts implemented, tests passing [session 3 end]
```

These markers enable clear session recovery and progress tracking.

---

## 7. Common Optimization Mistakes

Understanding what NOT to do prevents inefficient context usage and workflow disruptions.

### Mistake 1: Loading All Documentation at Session Start

**Error Pattern:**
Agent loads PROTOCOL.md, both integration guides, CONSENSUS_RESOLUTION.md, TROUBLESHOOTING.md, SCALING.md, and all examples at the beginning of every session regardless of phase or need.

**Impact:**
- Protocol documentation consumes 60,000-80,000 tokens
- Leaves only 120,000-140,000 tokens for consumer artifacts and work
- Forces early session termination due to context exhaustion
- Reduces agent productivity by 40-50%

**Correct Approach:**
Load only REQUIRED documents for the current phase at session start. Defer RECOMMENDED documents until uncertainty arises. Never preload OPTIONAL documents.

**Example:**
Antithesis phase session start:
- Load: PROTOCOL Section 3.2, INTEGRATION_GPT_CODEX Sections 2.2 and 3, ANTITHESIS_PROMPT, AUDIT_LOG template (8,175 tokens)
- Skip: TROUBLESHOOTING, SCALING, DESIGN_RATIONALE, CONSENSUS_RESOLUTION (saving 32,663 tokens)

### Mistake 2: Not Using Clear Command After Phase Completion

**Error Pattern:**
Agent completes Thesis phase, updates STATE.yaml to ANTITHESIS, but does not execute clear command. Next agent (GPT-5.3-Codex) continues in the same session with Claude Code's Thesis conversation history still loaded.

**Impact:**
- Thesis conversation history (design discussions, iterations) consumes 20,000-40,000 tokens unnecessarily
- Antithesis agent has wrong protocol documentation loaded (Thesis docs vs. Antithesis docs)
- Context pollution: Antithesis agent may be influenced by Thesis design discussions, compromising audit independence
- Earlier context exhaustion during Antithesis work

**Correct Approach:**
Always execute clear command at phase boundaries. Fresh session for each phase ensures:
- Clean context window
- Correct protocol documentation loaded
- No cross-contamination between phases
- Maximum context available for phase work

**Phase Transition Pattern:**
```
Thesis completion:
1. Commit BLUEPRINT and specs to Git
2. Update STATE.yaml to phase ANTITHESIS, owner_model Codex
3. Write handoff_notes
4. Execute clear command
5. End session

Antithesis start:
1. New session begins
2. Read STATE.yaml
3. Load Antithesis REQUIRED documentation
4. Begin audit work
```

### Mistake 3: Reading TROUBLESHOOTING.md Preemptively

**Error Pattern:**
Agent loads TROUBLESHOOTING.md (879 lines, 10,988 tokens) at session start "just in case" errors occur during the phase.

**Impact:**
- 10,988 tokens consumed preemptively
- TROUBLESHOOTING content is irrelevant unless errors actually occur
- Reduces available context by 5% for speculative benefit
- Most phases complete without errors, making the preload waste permanent

**Correct Approach:**
Load TROUBLESHOOTING.md only when errors actually occur. Mark as OPTIONAL in all phase documentation matrices.

**Error-Driven Loading Pattern:**
```
Normal execution:
1. Phase work proceeds without errors
2. TROUBLESHOOTING.md never loaded
3. Tokens saved: 10,988

Error encountered:
1. Agent detects error condition (e.g., STATE.yaml validation fails)
2. Load relevant TROUBLESHOOTING section (e.g., "STATE.yaml Issues" - 150 lines, 1,875 tokens)
3. Apply troubleshooting guidance
4. Unload TROUBLESHOOTING after issue resolved
5. Tokens used: 1,875 (83% reduction vs. preloading full document)
```

### Mistake 4: Loading SCALING.md in Single-Feature Projects

**Error Pattern:**
Agent loads SCALING.md (643 lines, 8,038 tokens) for a project with a single microservice, 5 endpoints, and 2,000 lines of code.

**Impact:**
- SCALING guidance is irrelevant for small projects
- 8,038 tokens consumed for content addressing 50+ service architectures, multi-region deployments, and 100K+ LOC codebases
- No actionable benefit for the current project scale
- Context waste reduces capacity for actual implementation work

**Correct Approach:**
Load SCALING.md only for large projects. Use objective thresholds:
- More than 10 microservices or distinct deployable units
- More than 50,000 lines of code in src directory
- Multi-region or multi-cloud deployment topology
- More than 5 development teams or 20 contributors

**Scale-Appropriate Loading:**
```
Small project (single service, fewer than 10K LOC):
- Skip SCALING.md entirely
- Use standard phase documentation
- Tokens saved: 8,038

Large project (10+ services, 50K+ LOC):
- Load SCALING.md during Thesis for architecture planning
- Load SCALING.md Section 4 during Execution for implementation partitioning strategies
- Tokens used: approximately 3,000 (relevant sections only)
```

### Mistake 5: Reading DESIGN_RATIONALE.md During Active Work

**Error Pattern:**
Agent loads DESIGN_RATIONALE.md (500 lines, 6,250 tokens) during active implementation work to "understand the protocol philosophy."

**Impact:**
- DESIGN_RATIONALE is background knowledge, not operational guidance
- 6,250 tokens consumed for philosophical context that does not change phase procedures
- Time spent reading rationale delays actual deliverable work
- No improvement in artifact quality or protocol compliance

**Correct Approach:**
DESIGN_RATIONALE.md is educational material for humans and for agents during initial onboarding. Never load during active cycle work.

**Appropriate DESIGN_RATIONALE Usage:**
- System Owner education before first cycle
- Agent training or onboarding before first protocol execution
- Protocol customization or extension design (outside of normal cycle work)
- Never during Thesis, Antithesis, Synthesis, or Execution phases

**Active Work Documentation Priority:**
1. REQUIRED: Phase procedures, responsibilities, quality gates
2. RECOMMENDED: Examples, terminology, handoff procedures
3. OPTIONAL: Troubleshooting, scaling
4. Never during active work: Design rationale, references, version history

---

## 8. Quick Reference: Minimal Document Sets

One-page summary for rapid session initialization. Use this table to load the absolute minimum protocol documentation for each phase.

### Thesis Phase Minimal Set

| Document | Lines | Tokens | Purpose |
|----------|-------|--------|---------|
| PROTOCOL.md Section 3.1 | 95 | 1,200 | Thesis activities and exit criteria |
| INTEGRATION_CLAUDE_CODE.md Section 2.1 | 80 | 1,000 | Claude Code Thesis responsibilities |
| prompts/THESIS_PROMPT.md | 152 | 1,900 | Phase-specific guidance |
| templates/BLUEPRINT.json | 50 | 600 | BLUEPRINT template structure |
| templates/specs/README.md | 40 | 500 | Interface specification formats |
| LANGUAGE_POLICY.md | 76 | 950 | Artifact language requirements |

**Total:** 493 lines, approximately 6,150 tokens

**Enables:** Complete BLUEPRINT creation, interface specification authoring, quality gate verification

**Context Reserve:** 193,850 tokens remaining (96.9% of 200K window available for consumer artifacts and design work)

---

### Antithesis Phase Minimal Set

| Document | Lines | Tokens | Purpose |
|----------|-------|--------|---------|
| PROTOCOL.md Section 3.2 | 60 | 750 | Antithesis activities and audit independence |
| INTEGRATION_GPT_CODEX.md Section 2.2 | 90 | 1,125 | GPT-5.3-Codex Antithesis responsibilities |
| INTEGRATION_GPT_CODEX.md Section 3 | 200 | 2,500 | Red Team audit methodology (5 areas) |
| prompts/ANTITHESIS_PROMPT.md | 196 | 2,450 | Phase-specific audit prompts |
| templates/AUDIT_LOG.md | 108 | 1,350 | Audit log template and finding format |

**Total:** 654 lines, approximately 8,175 tokens

**Enables:** Complete red-team audit across all 5 areas, AUDIT_LOG authoring, blocker identification, quality gate verification

**Context Reserve:** 191,825 tokens remaining (95.9% of 200K window available for BLUEPRINT review, audit work, and PoC development)

---

### Synthesis Phase Minimal Set

| Document | Lines | Tokens | Purpose |
|----------|-------|--------|---------|
| PROTOCOL.md Section 3.3 | 80 | 1,000 | Synthesis activities and blocker classification |
| INTEGRATION_CLAUDE_CODE.md Section 2.3 | 100 | 1,250 | Claude Code Synthesis responsibilities |
| prompts/SYNTHESIS_PROMPT.md | 180 | 2,250 | Phase-specific disposition framework |
| CONSENSUS_RESOLUTION.md (full) | 591 | 7,388 | Escalation ladder and conflict resolution procedures |

**Total:** 951 lines, approximately 11,888 tokens

**Enables:** Blocker disposition (ACCEPTED/DEFERRED/REJECTED), conflict resolution, BLUEPRINT updates, quality gate verification

**Critical Note:** CONSENSUS_RESOLUTION.md is REQUIRED (not RECOMMENDED) for Synthesis because blocker disposition is the central activity and conflict resolution procedures are frequently needed.

**Context Reserve:** 188,112 tokens remaining (94.1% of 200K window available for AUDIT_LOG review, BLUEPRINT updates, and disposition work)

---

### Execution Phase Minimal Set

| Document | Lines | Tokens | Purpose |
|----------|-------|--------|---------|
| PROTOCOL.md Section 3.4 | 70 | 875 | Execution activities and completion criteria |
| INTEGRATION_GPT_CODEX.md Section 2.4 | 80 | 1,000 | GPT-5.3-Codex Execution responsibilities |
| INTEGRATION_GPT_CODEX.md Section 7 | 150 | 1,875 | Implementation standards (tech stack, IO contracts, security policies, tests) |
| prompts/EXECUTION_PROMPT.md | 187 | 2,338 | Phase-specific implementation prompts and quality checklist |

**Total:** 487 lines, approximately 6,088 tokens

**Enables:** Production code implementation in src, test suite development in tests, CI script creation, documentation updates, quality gate verification

**Context Reserve:** 193,912 tokens remaining (96.9% of 200K window available for BLUEPRINT implementation, test development, and documentation work)

---

### Minimal vs. Standard vs. Maximum Comparison

| Phase | Minimal (REQUIRED only) | Standard (REQUIRED + RECOMMENDED) | Maximum (All Documents) |
|-------|------------------------|----------------------------------|------------------------|
| Thesis | 6,150 tokens | 8,635 tokens | 23,248 tokens |
| Antithesis | 8,175 tokens | 10,535 tokens | 13,660 tokens |
| Synthesis | 11,888 tokens | 13,438 tokens | 16,563 tokens |
| Execution | 6,088 tokens | 7,523 tokens | 27,799 tokens |

**Recommendation:**
- Use Minimal for experienced agents on cycles 2+
- Use Standard for first cycle or when uncertainty exists
- Never use Maximum; load OPTIONAL documents only when specific need arises

**Token Efficiency:**
- Minimal averages 8,075 tokens per phase (95.9% context window free)
- Standard averages 10,033 tokens per phase (95.0% context window free)
- Maximum averages 20,318 tokens per phase (89.8% context window free)

**Optimal Strategy:** Start with Minimal, add RECOMMENDED documents as needed during the session, never preload OPTIONAL documents.

---

## 9. Conclusion

### Summary of Key Optimization Principles

**Progressive Disclosure:**
Load documentation in three tiers (REQUIRED, RECOMMENDED, OPTIONAL) based on actual need, not anticipated need.

**Phase-Specific Loading:**
Each phase requires different documentation. Load only the current phase's procedures, responsibilities, and quality gates.

**Section-Level Granularity:**
Large documents like PROTOCOL.md and integration guides are organized by section. Load only relevant sections instead of entire documents.

**Session Boundaries:**
Always execute clear command at phase boundaries. Fresh session for each phase ensures clean context and correct documentation.

**Error-Driven Loading:**
Load troubleshooting and scaling documentation only when errors or scale issues actually occur, never preemptively.

**State Persistence:**
All critical state lives in STATE.yaml and Git commits, not conversation history. Sessions can be started, stopped, and resumed without data loss.

**Token Budget Awareness:**
Monitor context consumption throughout the session. Defer OPTIONAL documentation when approaching 75% threshold. Start new session when exceeding 85%.

### Expected Outcomes

Projects following these optimization strategies should achieve:

- **60-70% reduction** in protocol documentation overhead per session
- **2-3x longer** agent sessions before context exhaustion
- **30-40% more** consumer artifacts loadable in each session
- **Zero protocol compliance degradation** despite reduced documentation loading
- **Faster phase transitions** due to optimized session boundaries

### Implementation Checklist for AI Agents

At session initialization:
- [ ] Read STATE.yaml to determine current phase and status
- [ ] Load REQUIRED documents for current phase only (6,000-12,000 tokens)
- [ ] Load consumer artifacts: BLUEPRINT, AUDIT_LOG, specs, relevant source files
- [ ] Calculate baseline context consumption and remaining budget
- [ ] Begin phase work

During active work:
- [ ] Monitor context consumption at every 10-15 message exchanges
- [ ] Load RECOMMENDED documents only when uncertainty arises
- [ ] Never preload OPTIONAL documents; load only when specific need detected
- [ ] Commit work frequently to Git with descriptive messages
- [ ] Update STATE.yaml handoff_notes when approaching session end

Before session end or phase transition:
- [ ] Commit all work to Git
- [ ] Update STATE.yaml with current status and detailed handoff_notes
- [ ] Update last_artifacts with current Git SHAs
- [ ] Execute clear command if at phase boundary
- [ ] Document in-progress work for next session resumption

### Implementation Checklist for System Owners

Session configuration:
- [ ] Initialize agent sessions with phase-specific REQUIRED documentation only
- [ ] Configure session prompts to include STATE.yaml reading as first action
- [ ] Set context monitoring alerts at 75% and 85% thresholds
- [ ] Enable automatic session restart at phase boundaries

Workflow automation:
- [ ] CI/CD pipelines load minimal documentation sets per phase
- [ ] Automated session scripts execute clear command at phase transitions
- [ ] Session initialization scripts verify STATE.yaml before loading documentation
- [ ] Logging captures context consumption metrics for optimization tracking

Quality assurance:
- [ ] Review handoff_notes completeness before phase transitions
- [ ] Verify agent sessions use minimal documentation sets (audit session logs)
- [ ] Track average context consumption per phase across cycles
- [ ] Identify documentation gaps that force excessive OPTIONAL loading

---

**Document Version:** 1.0.0
**Protocol Version:** TITLE Protocol v3.0
**Last Updated:** 2026-02-15
**Maintained by:** agent-cowork Protocol Authors
**License:** MIT

**Cross-References:**
- PROTOCOL.md -- Complete protocol specification with phase definitions and quality gates
- INTEGRATION_CLAUDE_CODE.md -- Claude Code phase-specific integration guide
- INTEGRATION_GPT_CODEX.md -- GPT-5.3-Codex phase-specific integration guide
- GETTING_STARTED.md -- First cycle walkthrough and adoption guidance
- STATE.yaml template -- Protocol state file structure and field definitions

---

*This document optimizes AI agent context window usage for TITLE Protocol execution in consumer projects. All recommendations are based on 200K token context window limits common to Claude Code and GPT-5.3-Codex as of February 2026.*
