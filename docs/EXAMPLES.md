# Protocol Artifact Examples

This document provides completed examples of TITLE Protocol v3.0 artifacts using a fictional project called **SecureNotes API** to illustrate what properly filled protocol documents look like.

## Purpose

The examples demonstrate:

- **Completeness** - All required fields filled with realistic values
- **Specificity** - Quantified targets rather than vague descriptions
- **Evidence-based** - Concrete findings with reproducible scenarios
- **Clarity** - Clear blocker dispositions with documented reasoning

These examples complement the templates in `templates/` by showing the expected end state for each artifact type.

---

## Example 1: Completed BLUEPRINT.json

This example shows a fully populated BLUEPRINT for SecureNotes API, a note-taking service with authentication, end-to-end encryption, and sharing capabilities.

```json
{
  "_comment": "TITLE Protocol v3.0 -- BLUEPRINT Template. Claude Code owns this artifact. All fields below are REQUIRED for the Thesis-to-Antithesis quality gate.",

  "project_name": "SecureNotes API",
  "version": "0.2.1",
  "cycle": 2,
  "last_updated": "2026-02-15",

  "core_principles": [
    "Security First -- security and stability are first-class requirements",
    "Stateless Design -- all state persisted in explicit files, agents restartable at any time",
    "Zero-Knowledge Architecture -- server cannot decrypt user notes; encryption keys derived from user credentials"
  ],
  "_comment_core_principles": "Array of guiding principles. Minimum 1 entry required. Include Security First and Stateless Design by default.",

  "constraints": {
    "network": "All external communication must use TLS 1.3. No outbound connections except to approved third-party services (email, push notifications). API rate limit: 1000 requests/hour per user.",
    "deployment": "Single-region deployment (us-east-1). Containers orchestrated via Kubernetes. Database must support point-in-time recovery with 7-day retention.",
    "compliance": "GDPR-compliant data handling. User data export within 30 days of request. Right to erasure within 14 days."
  },
  "_comment_constraints": "Object with at least 1 key. Document hard boundaries the design must respect (network, deployment, compliance, budget, etc.).",

  "tech_stack": {
    "language": "Python 3.12",
    "framework": "FastAPI 0.109",
    "runtime": "uvicorn with asyncio",
    "database": "PostgreSQL 16 with pgcrypto extension",
    "cache": "Redis 7.2 for session tokens and rate limiting",
    "infrastructure": "Kubernetes 1.29 on AWS EKS"
  },
  "_comment_tech_stack": "Required: language and framework. Add runtime, database, infrastructure keys as needed.",

  "io_contracts": {
    "POST /auth/login": {
      "method": "POST",
      "path": "/api/v1/auth/login",
      "request": {
        "content_type": "application/json",
        "schema": "{ \"email\": \"string\", \"password\": \"string\" }"
      },
      "response": {
        "content_type": "application/json",
        "schema": "{ \"access_token\": \"string (JWT)\", \"refresh_token\": \"string\", \"expires_in\": \"number (seconds)\" }"
      },
      "errors": [
        { "code": 400, "description": "Bad Request -- invalid email format" },
        { "code": 401, "description": "Unauthorized -- invalid credentials" },
        { "code": 429, "description": "Too Many Requests -- rate limit exceeded" }
      ]
    },
    "GET /notes": {
      "method": "GET",
      "path": "/api/v1/notes",
      "request": {
        "content_type": "N/A",
        "schema": "Query params: ?page=number&limit=number (max 100)"
      },
      "response": {
        "content_type": "application/json",
        "schema": "{ \"notes\": [{ \"id\": \"uuid\", \"title\": \"string (encrypted)\", \"created_at\": \"iso8601\", \"updated_at\": \"iso8601\" }], \"total\": \"number\", \"page\": \"number\" }"
      },
      "errors": [
        { "code": 401, "description": "Unauthorized -- missing or invalid token" },
        { "code": 400, "description": "Bad Request -- invalid pagination parameters" }
      ]
    },
    "POST /notes": {
      "method": "POST",
      "path": "/api/v1/notes",
      "request": {
        "content_type": "application/json",
        "schema": "{ \"title\": \"string (client-encrypted)\", \"content\": \"string (client-encrypted)\", \"tags\": \"array of strings (optional)\" }"
      },
      "response": {
        "content_type": "application/json",
        "schema": "{ \"id\": \"uuid\", \"created_at\": \"iso8601\" }"
      },
      "errors": [
        { "code": 401, "description": "Unauthorized -- missing or invalid token" },
        { "code": 400, "description": "Bad Request -- missing required fields or content exceeds 1MB limit" },
        { "code": 507, "description": "Insufficient Storage -- user storage quota exceeded" }
      ]
    }
  },
  "_comment_io_contracts": "Define every external-facing endpoint or interface contract. Each entry must specify method, path, request/response schema, and error codes.",

  "security_policies": {
    "authentication": "JWT (HS256) with 15-minute access token lifetime, 7-day refresh token lifetime. Tokens signed with server secret rotated every 90 days.",
    "authorization": "RBAC with roles: user (default), premium_user, admin. Premium features gated by role checks. Admins can only access aggregate anonymized metrics, not user content.",
    "data_handling": "User notes encrypted client-side (AES-256-GCM) before transmission. Server stores only ciphertext. Encryption keys derived from user password via Argon2id, never transmitted or stored on server. Metadata (timestamps, note IDs) stored in plaintext for indexing.",
    "encryption": "TLS 1.3 for all network traffic. Database connections encrypted via SSL. Secrets (JWT signing key, DB credentials) stored in AWS Secrets Manager with automatic rotation."
  },
  "_comment_security_policies": "Required fields: authentication, authorization, data_handling, encryption. Add audit_logging, secrets_management, etc. as needed.",

  "non_functional_requirements": {
    "performance": "p95 latency < 200ms for GET /notes, < 500ms for POST /notes. Support 500 concurrent users per instance.",
    "scalability": "Horizontal pod autoscaling from 3 to 20 replicas based on CPU (70% threshold). Database connection pooling with max 100 connections per instance. Stateless API design to support seamless scaling.",
    "availability": "99.9% uptime SLA (< 8.76 hours downtime/year). Multi-AZ database deployment with automatic failover. Rolling deployment strategy with zero-downtime updates.",
    "observability": "Structured JSON logging with request_id tracing. Prometheus metrics for request rate, error rate, latency histograms. OpenTelemetry distributed tracing. Alerts on error rate > 1%, p95 latency > 1s, or pod restart > 3/hour."
  },
  "_comment_non_functional_requirements": "Required fields: performance, scalability, availability, observability. Quantify targets wherever possible."
}
```

---

## Example 2: Completed AUDIT_LOG.md

This example shows a realistic red-team audit report from GPT-Codex during the Antithesis phase, identifying two critical security vulnerabilities and one improvement recommendation.

```markdown
# Technical Audit Report -- TITLE Protocol v3.0

> **Reviewer:** GPT-5.3-Codex
> **Date:** 2026-02-15
> **Target Spec:** specs/api_endpoints.ts, specs/auth_flow.ts
> **BLUEPRINT Version:** 0.2.1
> **Cycle:** 2

---

## Audit Checklist

### 1. Memory / Resource Management

- [x] Identify potential memory leaks in long-running processes
- [x] Check for unbounded buffer/queue growth
- [x] Verify resource cleanup on error paths (connections, file handles, locks)
- [x] Assess garbage collection pressure under load
- [x] Review resource pooling and lifecycle management

**Findings:**

No critical issues found. Connection pooling properly configured with max_connections=100 and connection timeout=30s. Error paths correctly release database connections via context managers.

### 2. Concurrency / Race Conditions

- [x] Identify shared mutable state across threads/processes
- [x] Check for TOCTOU vulnerabilities
- [x] Verify lock ordering to prevent deadlocks
- [x] Assess async/await correctness
- [x] Review concurrent data structure usage

**Findings:**

No issues found. Stateless API design eliminates shared mutable state. All async database operations use proper await patterns. Redis rate limiting uses atomic INCR/EXPIRE commands preventing race conditions.

### 3. Hardware / Infrastructure Lifecycle

- [x] Assess single points of failure in deployment topology
- [x] Check graceful degradation under partial infrastructure failure
- [x] Verify connection retry and circuit breaker strategies
- [x] Review infrastructure scaling triggers and limits
- [x] Assess disaster recovery and backup procedures

**Findings:**

See BLK-002 for missing JWT secret rotation mechanism during infrastructure failures.

### 4. Security Vulnerabilities

- [x] Review authentication and authorization enforcement
- [x] Check for injection vulnerabilities
- [x] Verify input validation and output encoding
- [x] Assess secrets management
- [x] Review TLS/encryption configuration and certificate management

**Findings:**

See BLK-001 for SQL injection vulnerability in note search endpoint.

### 5. AI Misuse Scenarios

- [x] Assess prompt injection and jailbreak resistance
- [x] Check for data exfiltration paths through AI interactions
- [x] Verify rate limiting and abuse prevention for AI endpoints
- [x] Review AI output validation and sanitization
- [x] Assess model behavior boundaries and guardrails

**Findings:**

No AI-powered features in current scope. Reserved for future cycles.

---

## Critical Issues / Blockers

### BLK-001: SQL Injection in Note Search Endpoint

- **Area:** Security
- **Severity:** CRITICAL
- **Description:** The note search functionality at `GET /api/v1/notes/search?query={term}` constructs a SQL query using string interpolation without parameterization. User-supplied search terms are directly embedded in the SQL statement, allowing SQL injection attacks.
- **Evidence / PoC:**
  - Request: `GET /api/v1/notes/search?query=' OR 1=1--`
  - Current code: `query = f"SELECT * FROM notes WHERE title LIKE '%{search_term}%'"`
  - Result: Returns all notes in the database, bypassing authorization checks.
- **Impact:**
  - Unauthorized access to all notes across all users (confidentiality breach)
  - Potential data deletion via `'; DROP TABLE notes--` injection
  - GDPR compliance violation due to unauthorized data access
  - Complete compromise of zero-knowledge architecture principle
- **Suggested Mitigation:**
  - Replace string interpolation with parameterized queries: `cursor.execute("SELECT * FROM notes WHERE title LIKE %s", (f"%{search_term}%",))`
  - Add input validation to reject special characters in search terms
  - Implement Content Security Policy to prevent script injection in responses
  - Add integration test suite specifically for injection attack scenarios

### BLK-002: JWT Secret Rotation Incomplete During Infrastructure Failures

- **Area:** Hardware
- **Severity:** CRITICAL
- **Description:** BLUEPRINT specifies JWT signing key rotation every 90 days via AWS Secrets Manager. However, the rotation mechanism does not handle pod restart scenarios during secret rotation windows. If a pod restarts while the secret is being rotated, it may load a stale signing key, causing signature verification failures for tokens signed with the new key.
- **Evidence / PoC:**
  - Simulate: Rotate secret in AWS Secrets Manager, then trigger pod restart within 60 seconds
  - Current behavior: New pod loads cached secret from previous rotation (stale key)
  - Result: 401 Unauthorized errors for 50% of authenticated requests until cache expires (15 minutes)
- **Impact:**
  - Service disruption affecting 50% of authenticated users during rotation windows
  - Violates 99.9% availability SLA (rotation occurs 4x/year, each causing ~15 min outage = 1 hour/year)
  - Forces manual intervention (restart all pods) during rotation, increasing operational complexity
- **Suggested Mitigation:**
  - Implement dual-key verification: accept tokens signed with either current OR previous secret during rotation window (grace period of 30 minutes)
  - Add secret version tracking to pods: check Secrets Manager for version updates every 60 seconds
  - Gracefully handle verification failures: retry with alternative signing key before returning 401
  - Add end-to-end test for secret rotation with zero-downtime requirement

---

## Recommendations

- **Rate Limiting Granularity:** Current rate limiting is per-user (1000 req/hour). Consider adding per-endpoint limits to prevent abuse of expensive operations. For example, `POST /notes` could have a stricter limit (100/hour) than `GET /notes` (900/hour) to prevent storage quota exhaustion attacks.

---

## Summary Statistics

| Metric | Count |
|---|---|
| Areas Audited | 5/5 |
| Critical Issues (Blockers) | 2 |
| High Severity | 0 |
| Medium Severity | 0 |
| Low Severity | 0 |
| Recommendations (Non-blocking) | 1 |
```

---

## Example 3: STATE.yaml During Synthesis Phase

This example shows STATE.yaml during the Synthesis phase (cycle 2) with two blockers: one accepted and incorporated into the implementation plan, and one deferred to the next cycle with documented reasoning.

```yaml
# TITLE Protocol v3.0 -- STATE.yaml Template
# This file tracks protocol state across all four phases.
# Updated at every phase transition and blocker status change.

# Current protocol phase.
# Valid values: THESIS, ANTITHESIS, SYNTHESIS, EXECUTION
phase: SYNTHESIS

# The model currently leading the phase.
# Valid values: Claude, Codex
# Thesis/Synthesis -> Claude; Antithesis/Execution -> Codex
owner_model: Claude

# The model blocking consensus, if any.
# Set to Claude or Codex when one side blocks progress.
# null when no consensus issue exists.
blocking_model: null

# Overall protocol status.
# OK              -- normal operation
# BLOCKED         -- a blocker prevents phase transition
# CONSENSUS_BLOCKED -- agents cannot agree; System Owner intervention required
status: OK

# Current cycle number. Increments after each complete
# Thesis -> Antithesis -> Synthesis -> Execution pass.
cycle: 2

# SHA hashes of the latest committed versions of key artifacts.
# Empty strings indicate the artifact has not been created yet.
last_artifacts:
  blueprint_sha: "a3f2e1d9c8b7a6f5e4d3c2b1a0f9e8d7c6b5a4f3"
  audit_log_sha: "b8e7d6c5b4a3f2e1d0c9b8a7f6e5d4c3b2a1f0e9"
  src_sha: "c1d2e3f4a5b6c7d8e9f0a1b2c3d4e5f6a7b8c9d0"

# Free-text notes from the outgoing agent to the incoming agent.
# Used during handoff to transfer context that is not captured in artifacts.
handoff_notes: |
  Codex -> Claude handoff (2026-02-15):

  Audit complete. Two critical blockers identified (BLK-001, BLK-002).

  BLK-001 (SQL injection) is straightforward - parameterized queries + input validation.
  Estimated remediation: 2 hours development + 1 hour testing.

  BLK-002 (JWT rotation) requires dual-key verification strategy. Implementation
  complexity is moderate but testing complexity is high (requires simulating
  AWS Secrets Manager rotation + pod restarts). Estimated: 6 hours development
  + 4 hours integration testing.

  Recommendation on rate limiting granularity is valid but non-blocking.
  Can be deferred to cycle 3 as it is an enhancement, not a stability issue.

  All other audit areas passed. Memory management, concurrency, and existing
  security controls are solid. Good work on the parameterized queries for
  the primary endpoints - just missed the search endpoint.

  Suggested priority: Fix BLK-001 immediately (security breach), then BLK-002
  (availability impact). Both must be resolved before Execution phase per protocol.

# List of open blockers. Each blocker tracks a critical issue
# that must be resolved before the next phase transition.
blockers:
  - id: BLK-001
    title: SQL Injection in Note Search Endpoint
    owner: Claude
    status: ACCEPTED
    severity: CRITICAL
    area: Security
    reason: "Accepted for immediate remediation. Implementation plan: (1) Replace string interpolation with parameterized query using psycopg2 parameter binding. (2) Add regex input validation to search_term allowing only alphanumeric and basic punctuation. (3) Create integration test suite with 10 injection attack scenarios including quote escaping, comment injection, and UNION attacks. Estimated completion: 3 hours. Will verify fix with Codex before phase transition."
    updated_at: "2026-02-15T14:23:00Z"

  - id: BLK-002
    title: JWT Secret Rotation Incomplete During Infrastructure Failures
    owner: Claude
    status: DEFERRED
    severity: CRITICAL
    area: Hardware
    reason: "Deferred to cycle 3. Rationale: (1) Current rotation frequency is 90 days, next rotation not due until 2026-04-15 (60 days away). (2) Implementing dual-key verification requires changes to core authentication middleware affecting all endpoints - high regression risk. (3) Proper testing requires AWS Secrets Manager test harness not yet available in CI/CD pipeline. (4) Workaround exists: manual coordination of secret rotation during maintenance windows avoids pod restart collision. Trade-off: Accept temporary SLA violation (15 min/quarter) to avoid destabilizing current cycle. Will prioritize as BLK-001 in cycle 3 with proper test infrastructure."
    updated_at: "2026-02-15T14:45:00Z"
```

---

## Key Observations

Reviewing these examples reveals what distinguishes complete, high-quality protocol artifacts:

**Completeness:**
- All required fields populated with realistic values, not placeholders
- BLUEPRINT includes quantified non-functional requirements with specific numbers (p95 < 200ms, 99.9% uptime)
- AUDIT_LOG provides evidence and proof-of-concept for each finding, not just descriptions
- STATE.yaml documents reasoning for blocker dispositions with timestamps and ownership

**Specificity:**
- BLUEPRINT constraints include exact versions (TLS 1.3, PostgreSQL 16) and quantified limits (1000 req/hour, 7-day retention)
- AUDIT_LOG demonstrates vulnerabilities with actual attack payloads and expected vs. actual behavior
- Blocker mitigation steps are concrete and actionable, not generic recommendations

**Evidence-Based:**
- Each critical issue includes reproducible steps to demonstrate the vulnerability
- Impact statements quantify consequences (confidentiality breach, SLA violation hours/year)
- Deferred blockers document the risk analysis and trade-offs, not just postponement

**Clear Dispositions:**
- ACCEPTED blockers include implementation plans with time estimates
- DEFERRED blockers justify the decision with specific rationale, workarounds, and future commitment
- No blockers left in ambiguous states - every issue has a clear next action and owner

These examples should serve as reference implementations when creating protocol artifacts for real projects. The templates provide structure; these examples demonstrate quality.

