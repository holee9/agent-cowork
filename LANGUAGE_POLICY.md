# Language Policy

Language conventions for the TITLE Protocol v3.0 multi-agent system.

---

## 1. Core Rule

All protocol artifacts, code, and agent-to-agent communication use **English** as the default language. The System Owner may communicate in any language.

---

## 2. Rationale

- **Token efficiency:** Major LLMs (Claude, GPT) use English-centric tokenizers. English text consumes fewer tokens for equivalent content, leaving more context for reasoning.
- **Model accuracy:** Benchmarks consistently show higher accuracy on technical tasks (code generation, specification writing, tool invocation) when instructions and artifacts are in English.
- **Toolchain compatibility:** Static analysis, CI systems, search, and documentation tools are built around English conventions. Consistent English usage reduces integration friction.

---

## 3. Per-Artifact Language Rules

| Artifact / Scope | Language | Notes |
|---|---|---|
| `BLUEPRINT.json` -- keys and values | English | All fields including `core_principles`, `constraints`, `security_policies` |
| `AUDIT_LOG.md` -- body | English | Optional Korean summary section permitted at end (`## Owner Notes`) |
| `specs/` -- type names, fields, comments | English | JSDoc/TSDoc in English |
| `src/` -- code, variables, comments | English | Inline comments, log messages, error strings |
| `tests/` -- test names, assertions, comments | English | Test descriptions in English |
| `STATE.yaml` -- keys and values | English | Korean permitted only in YAML comments (`# ...`) |
| `docs/`, README, CHANGELOG | English | Korean docs maintained separately in `docs/ko/` if needed |
| CI/CD scripts and configuration | English | Pipeline names, job names, log output |
| Commit messages | English | Conventional Commits format recommended |
| Branch names | English | Lowercase with hyphens (e.g., `feature/add-auth-module`) |
| PR/MR titles and descriptions | English | Consistent with commit message language |
| Error messages in code | English | User-facing strings handled via i18n layer |
| CLI output (ai-sync) | English | Future: `--locale=ko` planned for localized output |
| Owner instructions | Any language | System Owner may use Korean, English, or any language freely |
| Agent responses to Owner | English primary | Optional Korean summary when Owner requests it |

---

## 4. Agent Behavior

When the System Owner gives instructions in a non-English language:

1. The agent internally translates the instruction to English for processing
2. All artifacts (BLUEPRINT, AUDIT_LOG, specs, code) are produced in English
3. The agent responds to the Owner in the same language used for the instruction
4. If the Owner requests, the agent may add a localized summary section

When the System Owner gives instructions in English:

- The agent processes and responds entirely in English

---

## 5. Internationalization (i18n)

For user-facing text in the implemented application:

- Store user-facing strings in a translation resource file (e.g., `locales/en.json`, `locales/ko.json`)
- Use the i18n layer of the chosen framework for runtime string resolution
- English is the source-of-truth locale; other locales are translations
- Do not embed user-facing Korean (or other non-English) strings directly in source code

---

## 6. Exceptions

The following are acceptable exceptions to the English-first rule:

- YAML comments in `STATE.yaml` may contain Korean annotations by the System Owner
- A dedicated `## Owner Notes (Korean Summary)` section at the end of `AUDIT_LOG.md`
- Files under `docs/ko/` are explicitly Korean-language documentation
- The System Owner's own notes, instructions, and comments in any format
