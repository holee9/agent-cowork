# REFERENCES

This document contains research references and source materials that informed the design decisions of the TITLE Protocol v3.0. These references establish the technical foundation for role separation, multi-agent architecture, and security-focused agent design patterns.

---

## Section 1: GPT-5.3-Codex Capabilities

**GPT-5.3-Codex System Card** [Role Design]
The official OpenAI system card documenting Codex's strengths and limitations across coding, tool usage, vulnerability detection, and test generation. This reference establishes Codex's technical capabilities baseline.

**GPT-5.3-Codex Performance Analysis** [Role Design]
Comparative analysis of GPT-5.3-Codex against Claude models, highlighting Codex's positioning as particularly strong in concrete coding, execution, and tool usage scenarios. This comparison informed our role allocation strategy.

**GPT-5.3-Codex Security and Vulnerability Detection Evaluation** [Security]
Security team assessment of Codex's vulnerability detection capabilities, proof-of-concept generation abilities, and documented session stall issues. This evaluation shaped the Red Team Executor role requirements.

**GPT-5.3-Codex User Feedback and Community Discussion** [Role Design]
Community feedback from platforms like Hacker News evaluating Codex as stronger in specific implementation, testing, and exploit generation rather than high-level architectural design. This real-world usage data validated the Executor role assignment.

---

## Section 2: Claude Code and Multi-Agent Architecture

**Context Engineering for Multi-Agent LLM Code Assistants** [Architecture]
Academic research analyzing the effectiveness of separating Planner (design and coordination) from Executor (implementation) roles in multi-agent coding systems. This study provides empirical support for the protocol's role separation model.

**Multi-Agent Systems: Architecture and Use Cases** [Architecture]
Best practices documentation explaining role separation patterns including Planner versus Worker and Coordinator versus Executor architectures in multi-agent systems. This guide influenced the protocol's structural design.

**Designing a State-of-the-Art Multi-Agent System** [Architecture]
Architectural guide covering state management, role separation, and failure recovery strategies for large-scale multi-agent systems. These patterns informed the protocol's resilience mechanisms.

**Claude Code Sub-agents: The 90% Performance Gain** [Role Design]
Case study demonstrating performance improvements from a design agent plus sub-agent execution structure compared to single-agent approaches using Claude Code. This data supported the protocol's architectural choices.

**When to Use Multi-Agent Systems** [Methodology]
Official Anthropic blog post explaining when multi-agent approaches are effective and outlining strategies for agent responsibility division. This guidance shaped the protocol's decision criteria.

**Community Multi-Agent Architecture Examples for Claude Code** [Architecture]
Community-contributed patterns demonstrating how simple multi-agent structures improve context quality and success rates through effective role separation. These practical examples validated our approach.

**Claude Code Multi-Agent Orchestration Examples** [Architecture]
GitHub repository examples showing Claude as central Planner or Coordinator with execution agents handling implementation tasks. These working implementations provided reference patterns.

---

## Section 3: Security, Red Team, and Resilient Agent Design

**AI Red Teaming and Security Checklists** [Security]
Methodology and checklists for red teaming AI systems, covering threat modeling, proof-of-concept generation, and vulnerability management. This framework informed the Red Team Executor security validation role.

**Architecting Resilient LLM Agents** [Security]
Research paper on designing safe Plan-Act-Observe loops with privilege separation and architectures resilient to failures and attacks. These resilience patterns were integrated into the protocol's safety mechanisms.

---

## Section 4: Protocol Design Sources

**User-Provided plan.md** [Methodology]
The foundational document defining Protocol v3.0 AI architecture, the Thesis-Antithesis-Synthesis-Execution loop, directory structure including .aisync/BLUEPRINT.json and .aisync/AUDIT_LOG.md, STATE.yaml, specs/ and src/ organization, and basic role definitions. This document served as the protocol's primary requirements source.

**Thesis-Antithesis-Synthesis Dialectical Reasoning** [Methodology]
The classical dialectical reasoning tradition that underpins the protocol's three-phase deliberation cycle, ensuring balanced consideration of proposals, counterarguments, and synthesized resolutions before execution.

---

## Notes on Reference Usage

These references collectively support the protocol's core design decision: assigning Claude to the Architect and Coordinator roles while assigning GPT-5.3-Codex to the Red Team and Executor roles. This division aligns with published performance characteristics, multi-agent best practices, and security-focused agent design principles.

All references informed design decisions made during protocol revision. This document does not include URLs as the original planning materials referenced these works by title and description only.

---

**Document Status**: Reference archive for TITLE Protocol v3.0
**Last Updated**: 2026-02-15
**Maintained By**: Protocol documentation team
