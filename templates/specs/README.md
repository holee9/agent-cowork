# Interface Specifications Guide

This guide explains how to create interface specifications in your project's `specs/` directory when implementing the TITLE Protocol v3.0.

---

## 1. Purpose

The `specs/` directory contains formal interface specifications that define the contracts between system components, external APIs, and data structures. These specifications expand on the high-level `io_contracts` field in `BLUEPRINT.json` with complete technical details.

**Ownership:**
- **Created by:** Claude Code during the Thesis phase
- **Updated by:** Claude Code during the Synthesis phase (when accepted blockers require interface changes)
- **Referenced by:** GPT-5.3-Codex during the Antithesis phase (for red-team review) and Execution phase (for implementation)

**Purpose:**
- Provide implementation-ready interface definitions
- Serve as a single source of truth for contracts between components
- Enable early validation of design decisions before implementation
- Support automated code generation and validation tooling

---

## 2. Supported Specification Formats

The TITLE Protocol supports three common specification formats. Choose the format that best fits your technology stack and use case.

### 2.1 TypeScript Interface Definitions

**When to use:** TypeScript projects, Node.js backends, React/Vue frontends, type-safe contracts

**File extension:** `.ts`

**Expected structure:**
- Export TypeScript interfaces or types
- Include JSDoc comments for documentation
- Define request and response types matching `io_contracts` entries in BLUEPRINT.json
- Include validation constraints as JSDoc annotations

**Example naming:** `01_auth_interface.ts`, `02_user_api_types.ts`, `03_payment_contracts.ts`

**Content pattern:**
```
Interface export with name matching BLUEPRINT io_contract key
Request type defining all input fields with TypeScript types
Response type defining all output fields with TypeScript types
Error type union representing all possible error codes from BLUEPRINT
JSDoc comments describing field semantics and constraints
```

### 2.2 JSON Schema

**When to use:** REST APIs, data validation, cross-language contracts, OpenAPI integration

**File extension:** `.schema.json`

**Expected structure:**
- Valid JSON Schema draft 2020-12 or compatible version
- Schema title matching `io_contracts` entry in BLUEPRINT.json
- Definitions for request and response objects
- Enum constraints for error codes
- Required field declarations
- Type constraints and validation rules

**Example naming:** `01_auth_request.schema.json`, `02_user_response.schema.json`, `03_payment_contract.schema.json`

**Content pattern:**
```
JSON Schema object with schema version declaration
Title field matching BLUEPRINT io_contract entry name
Type object for request payload with properties and required fields
Type object for response payload with properties and required fields
Definitions section for reusable types
Validation constraints including pattern, minLength, maxLength, enum
Error schema with code and message properties matching BLUEPRINT errors array
```

### 2.3 OpenAPI / AsyncAPI

**When to use:** REST APIs, event-driven systems, API documentation, automated testing

**File extension:** `.yaml` or `.yml`

**Expected structure:**
- OpenAPI 3.1.x specification for REST/HTTP APIs
- AsyncAPI 2.x/3.x specification for event-driven/message-based APIs
- Complete path definitions matching BLUEPRINT `io_contracts` paths
- Request and response schemas with references to components
- Security scheme definitions matching BLUEPRINT `security_policies`
- Error response schemas matching BLUEPRINT error codes

**Example naming:** `01_rest_api.yaml`, `02_websocket_events.yaml`, `03_message_queue.yaml`

**Content pattern for OpenAPI:**
```
OpenAPI version declaration
Info section with title and version from BLUEPRINT
Paths section with each endpoint from BLUEPRINT io_contracts
Path operation with method, summary, and description
RequestBody with content type and schema reference
Responses section with success and error codes from BLUEPRINT
Components section with reusable schemas
SecuritySchemes matching BLUEPRINT security_policies
```

**Content pattern for AsyncAPI:**
```
AsyncAPI version declaration
Info section with title and version from BLUEPRINT
Channels section defining message topics or queues
Channel subscribe/publish operations
Message payload schemas
Security section matching BLUEPRINT security_policies
```

---

## 3. Required Content

Every specification file must contain the following elements to satisfy the Thesis-to-Antithesis quality gate.

### 3.1 Endpoint or Interface Identification

**Requirement:** Clear mapping to a corresponding entry in `BLUEPRINT.json` `io_contracts`

**How to achieve:**
- Use the same name as the io_contract key in BLUEPRINT
- Include the endpoint path or interface identifier
- Document the HTTP method or operation type

### 3.2 Request and Response Schemas

**Requirement:** Complete data structure definitions for inputs and outputs

**How to achieve:**
- Define all request parameters, headers, and body fields
- Define all response fields and nested structures
- Specify data types for every field
- Mark required versus optional fields
- Include field-level constraints such as string length, numeric ranges, and allowed values
- Reference BLUEPRINT io_contracts to ensure alignment

### 3.3 Error Codes and Descriptions

**Requirement:** Exhaustive error scenarios matching BLUEPRINT

**How to achieve:**
- Include every error code listed in the corresponding BLUEPRINT io_contract errors array
- Provide human-readable descriptions for each error
- Define error response payload structure
- Document when each error is triggered
- Include error mitigation guidance for client developers

### 3.4 Authentication Requirements

**Requirement:** Security constraints aligned with BLUEPRINT security_policies

**How to achieve:**
- Reference the authentication method from BLUEPRINT security_policies authentication field
- Specify authentication headers or parameters such as Authorization header, API key location, or mTLS certificate requirements
- Document token formats or credential schemas
- Define authorization checks and required permissions referencing BLUEPRINT security_policies authorization field
- Include examples of valid authentication credentials for development use only

---

## 4. Naming Conventions

Use consistent file naming to improve discoverability and maintain ordering.

### Pattern

```
<sequence_number>_<domain>_<type>.<extension>
```

**Components:**
- `sequence_number`: Two-digit prefix establishing logical ordering such as 01, 02, 03
- `domain`: Functional area or component name such as auth, user, payment, or inventory
- `type`: Specification category such as interface, api, types, contract, request, response, or events
- `extension`: Format-specific extension such as .ts, .schema.json, or .yaml

### Examples

| File Name | Purpose |
|-----------|---------|
| `01_auth_interface.ts` | TypeScript types for authentication endpoints |
| `02_user_api.yaml` | OpenAPI spec for user management REST API |
| `03_payment_request.schema.json` | JSON Schema for payment request validation |
| `04_inventory_events.yaml` | AsyncAPI spec for inventory event messages |
| `05_admin_types.ts` | TypeScript types for admin panel interfaces |

### Sequencing Strategy

**Group by domain:** Assign sequence ranges to related components such as 01-09 for authentication, 10-19 for user management, and 20-29 for payment processing

**Order by dependency:** Lower numbers for foundational interfaces, higher numbers for interfaces that depend on earlier definitions

**Reserve gaps:** Use increments of 5 or 10 to allow insertion of related specs later such as 10, 20, 30 instead of 01, 02, 03

---

## 5. Relationship to BLUEPRINT.json

The `specs/` directory expands on the `io_contracts` field in BLUEPRINT with full technical details.

### BLUEPRINT io_contracts Role

**Purpose:** High-level contract summary for architectural review

**Contents:**
- Endpoint or interface name
- HTTP method or operation type
- Request and response content types
- Brief schema description or reference
- Error code list

**Audience:** Architects, system owners, security reviewers

### specs Directory Role

**Purpose:** Complete implementation-ready specification

**Contents:**
- Full field-level schemas with types and constraints
- Validation rules and format specifications
- Authentication and authorization details
- Error handling and edge case documentation
- Code generation targets

**Audience:** Implementation agents, developers, automated tooling

### Consistency Requirements

**Critical:** Every `io_contract` entry in BLUEPRINT must have a corresponding specification file in `specs/`

**Validation checklist:**
- [ ] Each BLUEPRINT io_contract key has a matching spec file
- [ ] Endpoint paths and methods are identical
- [ ] Error codes in spec match BLUEPRINT errors array
- [ ] Authentication requirements align with BLUEPRINT security_policies
- [ ] Response schemas match BLUEPRINT io_contract response content_type

**Update protocol:**
- When Synthesis accepts a blocker requiring interface changes, Claude Code updates both BLUEPRINT io_contracts and the corresponding specs file
- Git commits should modify both files atomically to maintain consistency

---

## 6. Quality Gate Checklist

Before approving the Thesis-to-Antithesis transition, the System Owner should verify the following conditions.

### Completeness

- [ ] Every `io_contracts` entry in BLUEPRINT has a corresponding specification file
- [ ] All required content elements are present in each spec file
- [ ] No placeholder values remain in specification files
- [ ] Request and response schemas are fully defined with all fields documented

### Accuracy

- [ ] Endpoint names and paths match BLUEPRINT exactly
- [ ] Error codes in specs align with BLUEPRINT errors arrays
- [ ] Authentication requirements reference BLUEPRINT security_policies correctly
- [ ] Data types and constraints are realistic and implementable

### Consistency

- [ ] Naming conventions are applied uniformly across all spec files
- [ ] Shared types are defined once and referenced elsewhere
- [ ] Security constraints are consistent across related endpoints
- [ ] Version numbers in specs match BLUEPRINT version field

### Clarity

- [ ] Field descriptions are clear and unambiguous
- [ ] Edge cases and optional behaviors are documented
- [ ] Examples are provided for complex data structures
- [ ] References to external standards or RFCs are included where applicable

### Reviewability

- [ ] Specifications are formatted correctly and validate against their schema standards
- [ ] File structure is logical and easy to navigate
- [ ] Comments and documentation are in English per LANGUAGE_POLICY.md
- [ ] Spec files are committed to Git with descriptive commit messages

---

## 7. Examples by Format

### TypeScript Interface Example Structure

File contains exported interfaces for request and response types. Request interface defines all input fields with TypeScript primitive types, object types, or union types. Response interface defines all output fields with the same type system. Error type is a union of string literals or objects representing each error code from BLUEPRINT. JSDoc comments above each interface describe purpose, usage, and any constraints not expressible in TypeScript type syntax.

### JSON Schema Example Structure

File contains a JSON object with schema keyword set to the JSON Schema draft version. Type keyword set to object. Properties object containing field definitions where each field has a type keyword and optional validation keywords such as minLength, maxLength, pattern, enum, minimum, or maximum. Required array listing mandatory fields. Definitions section containing reusable schema fragments. Error schema defining code and message properties with enum constraints for valid error codes from BLUEPRINT.

### OpenAPI YAML Example Structure

File begins with openapi version keyword. Info section with title matching BLUEPRINT project_name and version matching BLUEPRINT version. Paths section where each key is an endpoint path from BLUEPRINT io_contracts. Each path contains an operation object for the HTTP method such as get, post, put, or delete. Operation contains summary, description, requestBody with content and schema, responses with status codes and schemas, and security array referencing security schemes. Components section defining schemas as reusable schema objects and securitySchemes matching BLUEPRINT security_policies.

---

## 8. Revision History

Track specification evolution to maintain protocol integrity.

### Version Tracking

- Increment BLUEPRINT version field when io_contracts change
- Update corresponding spec file headers with version and last_updated date
- Document breaking versus non-breaking changes in spec file comments

### Change Log

Consider maintaining a CHANGELOG.md in the specs directory for projects with frequent interface changes. Include version number, date, list of added endpoints or interfaces, list of modified schemas, list of deprecated or removed elements, and migration notes for breaking changes.

### Git Workflow

- Commit spec changes with descriptive messages following Conventional Commits format
- Tag commits that represent stable interface versions
- Use pull requests for spec changes to enable review and discussion
- Preserve spec history to support rollback and audit requirements

---

## 9. Tooling Integration

Specifications in this directory can drive automated workflows.

### Code Generation

Use TypeScript compiler API to generate runtime types from TypeScript spec files. Use JSON Schema to OpenAPI converters to generate OpenAPI specs from JSON Schema files. Use OpenAPI Generator to generate server stubs and client SDKs from OpenAPI YAML files. Use AsyncAPI Generator to generate event handlers and message validators from AsyncAPI specs.

### Validation

Integrate JSON Schema validators in CI pipelines to verify request and response payloads. Use OpenAPI validators to test API implementation compliance. Use TypeScript compiler to check implementation alignment with spec interfaces.

### Documentation

Generate API documentation sites from OpenAPI or AsyncAPI specs using tools like Redoc, Swagger UI, or AsyncAPI Studio. Export TypeScript interfaces to Markdown using typedoc for inclusion in project documentation.

---

## 10. Migration from Older Versions

If upgrading from TITLE Protocol v2.x or earlier that did not include a specs directory, follow this migration procedure.

### Step 1: Extract Contracts

Review BLUEPRINT.json io_contracts section. Identify all endpoint or interface entries. For each entry, determine the most appropriate specification format based on technology stack and current documentation.

### Step 2: Create Initial Specs

Create a new file in specs directory for each io_contract entry using the naming convention from Section 4. Populate the file with request, response, error, and authentication details from BLUEPRINT. Expand terse BLUEPRINT entries into full schemas with types and validation constraints. Add JSDoc or schema descriptions for all fields.

### Step 3: Validate Consistency

Compare each spec file against the corresponding BLUEPRINT io_contract. Verify endpoint names, paths, methods, error codes, and security requirements match exactly. Run validation tools appropriate to the format such as TypeScript compiler, JSON Schema validator, or OpenAPI linter.

### Step 4: Update STATE.yaml

Increment the BLUEPRINT version field. Set last_updated to the current date. Commit all spec files and updated BLUEPRINT in a single Git commit. Document the migration in the commit message.

---

## 11. Frequently Asked Questions

**Q: Can I mix specification formats in the same project?**

A: Yes. Use the format that best fits each interface. For example, use OpenAPI for REST endpoints and TypeScript interfaces for internal module contracts.

**Q: What if my io_contract does not map to a traditional request-response API?**

A: Use the format closest to your use case. For message queues or event streams, use AsyncAPI. For function signatures or in-process interfaces, use TypeScript or language-specific IDL.

**Q: Do I need to version every spec file individually?**

A: Not required. The BLUEPRINT version field tracks overall system version. Individual spec file versioning is optional but recommended for large projects with independent interface lifecycles.

**Q: How detailed should validation constraints be?**

A: Be as specific as the implementation requires. If the system enforces a maximum string length, document it. If certain field combinations are invalid, use JSON Schema dependencies or TypeScript utility types to express the constraint.

**Q: Can GPT-5.3-Codex modify spec files during Execution?**

A: No. Per the Artifact Ownership Matrix in PROTOCOL.md Section 8, Codex reads and implements specs during Execution but does not write to them. Only Claude Code modifies specs during Thesis and Synthesis phases.

**Q: What if a red-team finding during Antithesis identifies a missing interface?**

A: The blocker should be classified in Synthesis. If ACCEPTED, Claude Code adds the new io_contract to BLUEPRINT and creates the corresponding spec file during the Synthesis phase before Execution begins.

---

## 12. Additional Resources

- PROTOCOL.md Section 3.1 for Thesis phase responsibilities and exit criteria
- PROTOCOL.md Section 4.1 for complete Thesis-to-Antithesis quality gate checklist
- PROTOCOL.md Section 8 for Artifact Ownership Matrix showing who writes specs in each phase
- LANGUAGE_POLICY.md Section 3 for language requirements for spec files and comments
- BLUEPRINT.json template for io_contracts field structure and required keys
- GLOSSARY.md for definitions of BLUEPRINT, io_contracts, and quality gate terms

For JSON Schema reference, see json-schema.org. For OpenAPI specification, see spec.openapis.org. For AsyncAPI specification, see asyncapi.com. For TypeScript handbook, see typescriptlang.org/docs.

---

**Document Version:** 1.0.0
**Last Updated:** 2026-02-15
**Maintained By:** TITLE Protocol v3.0 Project
**License:** Same as agent-cowork repository
