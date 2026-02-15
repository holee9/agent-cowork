# Getting Started with TITLE Protocol v3.0

Step-by-step guide for adopting the TITLE Protocol in your project.

---

## Prerequisites

Before starting, ensure your project has:

1. **Git Repository** -- Initialized and ready for commits. The protocol uses Git to track state transitions and artifact history.

2. **AI Agent Access** -- Both required:
   - **Claude Code** (Architect role for design and synthesis)
   - **GPT-5.3-Codex** (Builder/Red Team role for auditing and implementation)

3. **Human System Owner** -- One person responsible for:
   - Approving phase transitions after reviewing quality gates
   - Resolving consensus deadlocks when agents disagree
   - Providing initial requirements and priorities

4. **English Language** -- All protocol artifacts are produced in English, regardless of the System Owner's communication language (see LANGUAGE_POLICY.md).

---

## Setup (5 Steps)

### Step 1: Reference the Protocol

Add the agent-cowork repository as a reference source for your project:

Option A -- Clone as a reference:
```bash
git clone https://github.com/your-org/agent-cowork.git protocol-reference/
```

Option B -- Add as Git submodule:
```bash
git submodule add https://github.com/your-org/agent-cowork.git protocol-reference/
```

Option C -- Copy protocol files directly (if you prefer not to track updates).

### Step 2: Copy Templates

Copy the templates directory to your project as .aisync:
```bash
cp -r protocol-reference/templates/ .aisync/
```

Your project structure should now include:
```
your-project/
├── .aisync/
│   ├── BLUEPRINT.json       # System design template (to be filled by Claude Code)
│   ├── AUDIT_LOG.md         # Red-team audit template (to be filled by GPT-5.3-Codex)
│   ├── STATE.yaml           # Protocol state tracker (already initialized to THESIS)
│   └── CHECKLIST.md         # Per-phase quality gate checklist
├── specs/                   # Interface specifications (to be created)
├── src/                     # Implementation code (to be created)
└── tests/                   # Test suites (to be created)
```

### Step 3: Initialize Specs Directory

Create an empty specs directory for interface specifications:
```bash
mkdir -p specs/
```

Claude Code will populate this directory during the Thesis phase.

### Step 4: Verify STATE.yaml

Open .aisync/STATE.yaml and confirm the initial state:
- Phase: THESIS
- Owner Model: Claude
- Status: OK
- Cycle: 1

No changes needed. The template is ready to use.

### Step 5: Read the Protocol

Before starting the first cycle, all participants should read:
- PROTOCOL.md -- Complete protocol specification (roles, phases, gates)
- GLOSSARY.md -- Key terminology reference
- CHECKLIST.md -- Quality gate verification checklist

This ensures everyone understands their role and the execution flow.

---

## First Cycle Walkthrough

Each cycle consists of four phases. Here is what happens in each phase:

### Phase 1: Thesis (Claude Code)

**What Claude Code Does:**
1. Analyzes requirements, risks, and constraints provided by the System Owner
2. Creates BLUEPRINT.json with complete system design including project name, core principles, constraints, tech stack, IO contracts, security policies, and non-functional requirements
3. Writes interface specifications in specs directory
4. Documents design rationale and trade-offs
5. Updates STATE.yaml to reflect phase completion

**What You Get:**
- Complete BLUEPRINT.json with all required fields filled
- Interface specifications in specs directory
- Design rationale documented in BLUEPRINT

**Quality Gate Checklist:**
Before transitioning to Antithesis, verify using .aisync/CHECKLIST.md:
- [ ] BLUEPRINT.json is valid JSON with all required fields
- [ ] Core principles array has at least 1 entry
- [ ] Security policies cover authentication, authorization, data handling, encryption
- [ ] Interface specifications exist in specs directory
- [ ] System Owner has reviewed and approved the BLUEPRINT

**Handoff:** Claude Code commits all artifacts, updates STATE.yaml to phase ANTITHESIS and owner_model Codex, and adds handoff notes summarizing the design.

---

### Phase 2: Antithesis (GPT-5.3-Codex)

**What GPT-5.3-Codex Does:**
1. Reads BLUEPRINT.json and specs directory for system design
2. Performs independent red-team security audit covering all 5 areas: Memory/Resource Management, Concurrency/Race Conditions, Hardware/Infrastructure Lifecycle, Security Vulnerabilities, and AI Misuse Scenarios
3. Documents each finding in AUDIT_LOG.md with ID, title, area, severity, description, evidence or proof-of-concept, impact, and suggested mitigation
4. Classifies findings as Critical Issues or Blockers versus Recommendations
5. Optionally writes proof-of-concept exploit code in tests directory
6. Updates STATE.yaml to reflect phase completion

**What You Get:**
- Complete AUDIT_LOG.md with all 5 audit areas evaluated
- Critical findings separated from recommendations
- Optional proof-of-concept code demonstrating vulnerabilities

**Quality Gate Checklist:**
Before transitioning to Synthesis, verify using .aisync/CHECKLIST.md:
- [ ] AUDIT_LOG.md exists and follows template structure
- [ ] All 5 audit areas have been evaluated
- [ ] Each critical finding has complete details: ID, severity, evidence, impact, mitigation
- [ ] Blockers have severity ratings assigned
- [ ] STATE.yaml is updated to phase ANTITHESIS and owner_model Codex

**Handoff:** GPT-5.3-Codex commits AUDIT_LOG.md, updates STATE.yaml to phase SYNTHESIS and owner_model Claude, and adds handoff notes highlighting critical blockers.

**Important:** Claude Code does not defend or rebut findings during this phase. The audit is independent.

---

### Phase 3: Synthesis (Claude Code leads, GPT-5.3-Codex assists)

**What Claude Code Does:**
1. Reviews every finding in AUDIT_LOG.md
2. Classifies each blocker with a disposition: ACCEPTED (valid, will fix in design), DEFERRED (valid, will fix in future cycle with documented reason), or REJECTED (not applicable or risk acceptable with documented reason)
3. Updates BLUEPRINT.json and specs directory to reflect accepted changes
4. Documents rationale for each disposition decision
5. GPT-5.3-Codex evaluates feasibility, migration risks, and test strategies for proposed changes
6. Resolves disagreements or escalates to System Owner for CONSENSUS_BLOCKED status
7. Updates STATE.yaml blockers array with all dispositions

**What You Get:**
- Updated BLUEPRINT.json reflecting design changes from accepted blockers
- Updated interface specifications if needed
- Complete blocker dispositions recorded in STATE.yaml
- Design decision log with rationale

**Quality Gate Checklist:**
Before transitioning to Execution, verify using .aisync/CHECKLIST.md:
- [ ] Every blocker has a disposition: ACCEPTED, DEFERRED, or REJECTED
- [ ] Each DEFERRED blocker has documented reason and planned cycle
- [ ] Each REJECTED blocker has documented reason with evidence
- [ ] ACCEPTED blockers are reflected in updated BLUEPRINT.json or specs
- [ ] No blocker remains with status OPEN
- [ ] GPT-5.3-Codex has reviewed feasibility of design changes
- [ ] System Owner has approved the final design

**Handoff:** Claude Code commits updated BLUEPRINT.json and STATE.yaml with blocker dispositions, then updates STATE.yaml to phase EXECUTION and owner_model Codex.

**Conflict Resolution:** If agents disagree on a blocker disposition, either agent may set STATE.yaml status to CONSENSUS_BLOCKED. The System Owner then reviews both positions and makes a binding decision.

---

### Phase 4: Execution (GPT-5.3-Codex leads, Claude Code reviews)

**What GPT-5.3-Codex Does:**
1. Implements production code in src directory following BLUEPRINT.json
2. Writes unit, integration, load, and security tests in tests directory
3. Creates or updates CI and deployment scripts
4. Runs all tests and self-corrects until tests pass
5. Claude Code performs high-level review against BLUEPRINT, security policies, and IO contracts
6. GPT-5.3-Codex updates documentation: README, CHANGELOG, architecture docs
7. Updates STATE.yaml to reflect cycle completion

**What You Get:**
- Complete implementation in src directory
- Comprehensive test suites in tests directory
- CI and deployment scripts ready for production
- Updated documentation including README and CHANGELOG
- Test results and quality metrics

**Quality Gate Checklist:**
Before transitioning to next Thesis cycle, verify using .aisync/CHECKLIST.md:
- [ ] All planned implementation is complete per BLUEPRINT
- [ ] Unit tests pass
- [ ] Integration tests pass (if applicable)
- [ ] Security tests pass (if applicable)
- [ ] Claude Code has reviewed implementation against BLUEPRINT and security policies
- [ ] Documentation is updated: README, CHANGELOG, architecture docs
- [ ] STATE.yaml is updated: cycle incremented, phase THESIS, owner_model Claude
- [ ] System Owner has approved release or next-cycle transition

**Handoff:** GPT-5.3-Codex commits all implementation code, tests, and documentation. Updates STATE.yaml to phase THESIS, owner_model Claude, and increments cycle number.

**Next Cycle:** After System Owner approval, the protocol returns to Thesis phase for the next feature or iteration.

---

## Agent Interaction Patterns

### How the System Owner Communicates

**With Claude Code:**
- Provide initial requirements at the start of each cycle in any language
- Review and approve BLUEPRINT.json after Thesis phase
- Review and approve final design after Synthesis phase
- Resolve deadlocks when STATE.yaml status is CONSENSUS_BLOCKED

**With GPT-5.3-Codex:**
- Review AUDIT_LOG.md after Antithesis phase
- Approve implementation and documentation after Execution phase
- Provide clarification on technical constraints if requested

### How Agents Discover Their Current Role

**Both agents must:**
1. Read STATE.yaml at the start of every interaction
2. Check phase field to determine current protocol phase
3. Check owner_model field to confirm which agent is leading
4. Check status field to detect BLOCKED or CONSENSUS_BLOCKED states
5. Read handoff_notes from the previous agent for context

**Example STATE.yaml read pattern:**
```yaml
phase: SYNTHESIS
owner_model: Claude
status: OK
handoff_notes: "Red-team audit identified 3 critical blockers in security area. Focus on authentication flow design."
```

In this example, Claude Code is leading Synthesis phase and should prioritize reviewing security-related blockers.

### How Handoffs Work in Practice

**Pre-Handoff (Outgoing Agent):**
1. Verify all quality gate conditions are met using CHECKLIST.md
2. Commit all modified artifacts to Git with descriptive messages
3. Write handoff_notes in STATE.yaml summarizing work completed and guidance for incoming agent
4. Update STATE.yaml with next phase, next owner_model, and artifact Git SHAs

**Post-Handoff (Incoming Agent):**
1. Read STATE.yaml to confirm phase and owner_model
2. Read handoff_notes to understand context from previous agent
3. Verify artifact integrity by checking Git SHAs match committed versions
4. Review relevant artifacts: BLUEPRINT for Antithesis and Execution, AUDIT_LOG for Synthesis
5. Begin phase activities as defined in PROTOCOL.md Section 3

**Handoff Failure Recovery:**
If the incoming agent detects missing artifacts, SHA mismatch, or incomplete gate conditions:
1. Set STATE.yaml status to BLOCKED
2. Document the issue in handoff_notes
3. Notify System Owner for resolution

---

## Common Mistakes (5 Pitfalls to Avoid)

### 1. Skipping Quality Gate Verification

**Mistake:** Transitioning to the next phase without verifying all checklist items in CHECKLIST.md.

**Impact:** Downstream phases fail due to incomplete artifacts. Causes rollback to previous phase and wasted effort.

**Solution:** Always use CHECKLIST.md before updating STATE.yaml to signal phase completion. The System Owner should verify gates before approving transitions.

### 2. Mixing Agent Roles

**Mistake:** Claude Code writing implementation code during Execution phase, or GPT-5.3-Codex modifying BLUEPRINT.json during Thesis phase.

**Impact:** Violates role separation principle. Creates confusion about artifact ownership and accountability.

**Solution:** Strictly follow the Artifact Ownership Matrix in PROTOCOL.md Section 8. Each artifact has one primary author per phase.

### 3. Ignoring Handoff Notes

**Mistake:** The incoming agent begins work without reading handoff_notes from the outgoing agent.

**Impact:** Loss of context and continuity. Critical guidance is missed, leading to rework or misalignment.

**Solution:** Always read handoff_notes in STATE.yaml immediately after taking control of a phase. Treat handoff notes as required reading.

### 4. Leaving Blockers Unresolved

**Mistake:** Transitioning from Synthesis to Execution with blockers still having status OPEN.

**Impact:** Blocks Synthesis to Execution gate. Execution phase cannot proceed without all blockers resolved to ACCEPTED, DEFERRED, or REJECTED.

**Solution:** During Synthesis phase, Claude Code must classify every blocker from AUDIT_LOG.md. GPT-5.3-Codex should verify all blockers have dispositions before accepting handoff.

### 5. Not Recording Conflict Resolutions

**Mistake:** Agents resolve disagreements verbally or in chat but do not document the decision in STATE.yaml blockers array.

**Impact:** Loss of decision rationale. Future cycles may repeat the same debate without historical context.

**Solution:** Follow the Conflict Resolution procedure in PROTOCOL.md Section 6. Record all resolutions in STATE.yaml blockers array with owner, status, severity, area, and reason fields.

---

## Next Steps

### For Your First Cycle

1. **Prepare Requirements:** Define the feature or system component for the first cycle. Provide these to Claude Code.

2. **Initiate Thesis Phase:** Claude Code reads requirements and begins creating BLUEPRINT.json.

3. **Monitor Progress:** Check STATE.yaml periodically to track phase transitions and status.

4. **Approve Transitions:** Review quality gates using CHECKLIST.md before approving each phase transition.

5. **Complete the Cycle:** Follow all four phases through to Execution. Review the complete audit trail in Git history.

### Deep Dive Documentation

- **PROTOCOL.md** -- Complete specification including role definitions, phase activities, quality gates, handoff procedures, conflict resolution, rollback paths, and artifact ownership matrix.

- **GLOSSARY.md** -- Quick reference for protocol terminology including Thesis, Antithesis, Synthesis, Execution, Blocker, ACCEPTED, DEFERRED, REJECTED, CONSENSUS_BLOCKED, and Quality Gate.

- **LANGUAGE_POLICY.md** -- English-first conventions for all artifacts regardless of System Owner communication language.

- **templates/BLUEPRINT.json** -- System design document template with all required fields and examples.

- **templates/AUDIT_LOG.md** -- Red-team audit template with 5-area checklist and finding documentation structure.

### Getting Help

If you encounter issues during adoption:

1. Review PROTOCOL.md Section 6 for conflict resolution procedures
2. Review PROTOCOL.md Section 7 for rollback and recovery options
3. Check STATE.yaml status field for BLOCKED or CONSENSUS_BLOCKED states
4. Consult GLOSSARY.md for terminology clarification

The protocol is designed to be self-documenting and recoverable. All state lives in explicit files, and Git history provides a complete audit trail.

---

**Protocol Version:** TITLE Protocol v3.0
**Document Status:** Production Ready
**Maintained by:** agent-cowork Protocol Authors
**License:** MIT
