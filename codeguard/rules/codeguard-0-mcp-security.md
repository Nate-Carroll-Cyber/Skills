---
description: MCP (Model Context Protocol) server and client security, consolidated from the MCP Top 10, NSA CSI U/OO/6030316-26 (May 2026), and the CoSAI MCP Security white paper (January 2026)
languages:
- python
- javascript
- typescript
- go
- rust
- java
alwaysApply: false
---

rule_id: codeguard-0-mcp-security.md

# MCP Security Rules

NEVER ship an MCP server, client, tool, or agent integration that fails any MUST below. Each section maps to a category in the companion document *MCP Security Best Practices* (MCP01 through MCP11 plus Deployment Pattern Requirements), which holds the rationale, detection checks, and review checklist.

## Scope check before writing code

- Classify the deployment before choosing controls. All-local, single-tenant remote, or multi-tenant remote. See Deployment Patterns at the end.
- Score the agent on the trifecta. Private data access, untrusted content ingestion, external communication or consequential action. If all three are true, remove or constrain one leg before proceeding.
- Decide which tools are consequential (delete, send, publish, pay, modify production). Every control marked "consequential" below applies to those tools.

## Secrets and credentials (MCP01)

- NEVER place a credential in a prompt, system message, context window, model memory, vector store, `.env` file, configuration file, or log.
- MUST fetch secrets from a vault or secrets manager at runtime.
- MUST place credential-handling middleware between the model and the protected service. The model never sees the secret value.
- MUST issue tokens scoped to one session or task, expiring with that work.
- MUST redact token-like strings from every log sink before write.
- NEVER use a shared or static service account for an agent. One identity per agent or workflow.
- MUST route outbound connections through a filtering egress proxy or DLP pinned to approved URLs and methods.
- NEVER apply repository or project MCP configuration before the user has established trust in that project.

## Least privilege and scope (MCP02)

- MUST start every agent with the narrowest scope and add on demonstrated need. Never grant broad and trim later.
- MUST use just-in-time tokens. Issued when the action is needed, scoped to that action, valid for minutes, revocable on demand.
- MUST manage permissions as policy-as-code in CI/CD with review and audit on every change.
- NEVER let development credentials reach staging or production, and never reuse roles across environments.
- MUST enforce freezes, approvals, and other critical controls as hard authorization boundaries. A prompt instruction is not a control.
- MUST justify every write and delete capability explicitly. Remove any that is not strictly required.
- MUST segregate tools that touch sensitive or regulated data from tools that touch public data.
- SHOULD prefer locally deployed servers for private data the organization controls. For multi-tenant SaaS data, prefer the server the provider hosts directly over any third-party intermediary.

## Tool schemas and metadata (MCP03)

- MUST sign tool schemas and manifests. Reject any definition without a valid signature.
- MUST pin tool definitions at the host on first use and flag every later change.
- MUST scan the entire schema before it reaches the model. Names, descriptions, parameter definitions, and default values. Every field is instruction-grade untrusted content.
- MUST strip ANSI escape sequences and display-control characters from schema fields before review.
- MUST require RBAC on any writable schema registry and human review before promotion to production.
- MUST reject or quarantine runtime schema changes.
- MUST re-prompt for user consent when a connected server's capabilities or data access change.
- NEVER treat a user approval step as sufficient if untrusted metadata already reached the model. Validate first.
- MUST design each tool for one purpose with explicit boundaries. NEVER build a "do anything" tool.
- MUST isolate tools so one cannot direct another to access or export unrelated private data.

## Supply chain (MCP04)

- MUST require signed components with verifiable provenance, and MUST verify the signature in the client before loading the server, not after it is serving tools.
- MUST maintain a complete SBOM for every server and plugin.
- MUST pin every dependency, plugin, and server to a reviewed version. NEVER use `latest`, ranges, or floating versions.
- NEVER fetch dependencies dynamically at runtime.
- MUST scan dependencies in CI before release.
- MUST verify maintenance status before adoption. Prefer reference implementations and actively maintained projects. Apply the strictest code audit to newer integrations.
- MUST keep MCP development tooling patched (MCP Inspector RCE CVE-2025-49596, fixed in v0.14.1).
- MUST sandbox third-party plugins and restrict their network access to approved domains.
- MUST bind local servers to `127.0.0.1`, never `0.0.0.0`, with authentication and origin checks on any local web interface.
- MUST use `stdio` transport for local servers. It has no network listener, which removes DNS rebinding and cross-origin attack surface. Reserve HTTP streaming for servers that must be reached remotely.

## Command and code execution (MCP05)

- NEVER build a shell command string from user input, model output, remote responses, OAuth metadata, URLs, or file content.
- NEVER pass untrusted strings to `exec`, `eval`, `os.system`, `child_process.exec`, or any API with `shell: true`.
- MUST use structured process APIs (`execFile`, `spawn`, subprocess with argument arrays) with executable and arguments separated.
- MUST terminate option parsing with `--` where the command supports it.
- NEVER execute model-generated code automatically. Gate it behind validation and authorization.
- MUST canonicalize and constrain file paths to approved roots. Reject traversal.
- MUST parameterize SQL. NEVER interpolate values into query strings.
- MUST apply context-aware output encoding for every sink, including SQL, shell, and HTML or other markup rendered to a user or another agent.
- MUST allowlist permitted commands, verbs, and operations.
- MUST validate every deserialized object against a strict schema and keep code and data isolated during deserialization.
- MUST block parameter forwarding when the data origin is ambiguous, so input meant for one component cannot reach another.
- MUST run servers and agents as non-root with no unrestricted `sudo`.
- MUST confine each tool with OS-level isolation (seccomp, AppArmor, SELinux, AppContainers) and deny access to filesystems, model files, and internal networks the tool does not require.
- MUST apply every injection defense above to protocol fields such as OAuth authorization URLs.

## Intent integrity (MCP06)

- MUST anchor the user's original objective in trusted system instructions as a fixed reference, and re-anchor it in long-running sessions.
- MUST treat every retrieved page, document, issue, file, code comment, and tool output as data, never as instructions.
- MUST tag or fence retrieved content as untrusted before it reaches the planning model, and keep it structurally separate from system instructions.
- MUST validate that every planned action aligns with the original intent before execution.
- MUST gate the transition from reading context to any consequential action. No blind planning paths.
- MUST implement consequential tools as a two-stage commit. Stage one returns a draft or preview with a draft ID and no side effects. Stage two commits only on explicit confirmation of that draft ID. Bind the commit to a time window where the target supports it.
- MUST provide a rollback or undo path for every consequential tool (snapshot, reversible operation, retained draft). A tool with no undo carries the highest gate.
- MAY use an independent guardrail model outside the primary context to detect intent drift. The guardrail advises. It NEVER authorizes. No authorization decision rests on the judgment of any model.
- NEVER rely on human approval alone. Pair confirmation prompts with hard authorization boundaries, keep the prompt rate low enough that each is read, and state the consequence plainly in every security-relevant prompt.
- SHOULD request confirmation of risky actions from the server side through MCP elicitation, so the check does not depend on the client's implementation.
- MUST constrain how much prior context can influence consequential decisions. Seeded context history is an attack vector.
- MUST inspect and filter each tool's output before it passes to the next component in a chain. Length checks, disallowed-keyword scanning, injected-instruction detection.

## Authentication and authorization (MCP07)

- MUST treat MCP as an API and apply API security controls to every request.
- MUST use OAuth 2.1 with short-lived, session-scoped tokens and per-user delegation.
- MUST require mutual authentication between clients, agents, tools, and servers. Use mTLS where appropriate. For workload-to-workload identity, use SPIFFE/SPIRE so short-lived SVIDs replace long-lived certificates in configuration.
- MUST validate every token server-side on every request for authenticity, expiry, intended resource, and authorization for the specific action.
- NEVER derive an authorization decision from client-supplied identity, roles, or scopes.
- MUST bind tokens to their target server with OAuth resource indicators.
- NEVER pass a client token through to a downstream service. Use token exchange (RFC 8693) or an on-behalf-of flow so the agent receives its own narrowly scoped downstream token.
- MUST separate the OAuth client and authorization-server roles.
- NEVER share client IDs or reuse tokens across agents.
- MUST sign sensitive MCP messages within the JSON payload with expiry and replay-protection metadata.
- MUST enforce idempotency keys on consequential operations so a replayed or duplicated message cannot re-execute.
- MUST correlate every logged action with a verified user and agent identity.

## Audit and telemetry (MCP08)

- MUST record agent activity as structured, queryable, tamper-evident logs.
- MUST capture every tool call with verified identity, tool name, parameters or safe parameter metadata, and timestamp. SHOULD record cryptographic hashes of tool results.
- MUST capture the prompt and context changes and the network events needed to reconstruct intent changes and external communication, with masking applied.
- MUST alert natively on protocol anomalies. Repeated or malformed requests, authorization failures, RBAC violations.
- MUST ship telemetry to the organization's SIEM or XDR.
- NEVER allow the monitored process or its operators to disable, delete, or rewrite the audit trail.
- MUST mask PII and credentials in logs so the log store does not become a sensitive-data store.
- MUST establish behavioral baselines and alert on unusual volume, new tools or endpoints, identity changes, atypical call sequences, and goal drift.

## Server inventory and discovery (MCP09)

- MUST register every server in a central registry with a named owner and lifecycle state. Unregistered servers fail deployment or access.
- MUST enroll approved local servers in a registry service so vetted implementations are reachable through a controlled path.
- MUST scan networks, repositories, developer machines, and CI/CD continuously for unregistered servers, including differential scans for relocated ports.
- MUST alert when an agent connects to an endpoint absent from the registry.
- MUST require SSO or OIDC on every server.
- NEVER ship default credentials or permissive development configuration to production.
- MUST segment networks so an unknown server cannot reach production systems or sensitive data.
- MUST revalidate registered servers continuously.

## Context and tenant isolation (MCP10)

- MUST use ephemeral per-session context destroyed at session end.
- MUST isolate context by tenant, user, agent, and workflow with infrastructure-enforced namespaces. One tenant's namespace is technically incapable of answering another's queries.
- MUST revalidate session and tenant ownership on every request. NEVER trust a cached session-to-user mapping.
- NEVER share singleton state, context buffers, or vector stores across tenants without enforced and tested isolation.
- MUST classify and tag sensitive data at ingestion and apply block, mask, or minimize policy before storage.
- MUST minimize tool output. Return only the fields the task needs, and redact PII and sensitive values inside the tool before the result reaches the model.
- MUST give every context item a TTL with automatic purge.
- MUST govern persistent context as a data store with access control, classification, retention, and deletion.
- NEVER let injected content in shared memory become an instruction in a later session.
- MUST run a tenant-isolation test in CI on every build.

## Availability (MCP11)

- MUST rate-limit and quota per verified identity. Requests, token volume, tool calls, concurrent tasks.
- MUST cap recursion depth, chain length, fan-out, and per-task compute and time budgets, with timeouts and circuit breakers.
- MUST reject malformed, missing-field, and oversized inputs at the MCP boundary before model or tool execution.
- MUST apply backpressure and queue limits so prompt storms degrade gracefully.
- MUST set per-session resource ceilings against legitimate-looking but exhausting tasks.

## Deployment Patterns

Classify the deployment. The pattern decides which of the above are mandatory versus not applicable.

### All-local (one host, one user)
- Security rests entirely on host posture.
- MUST use `stdio` transport.
- MUST sandbox the server and run it as non-root.
- Appropriate for development and personal use only. NEVER for another person's data.

### Single-tenant remote (one organization, reached over the network)
- MUST authenticate client to server and encrypt all traffic.
- MUST store client credentials in the OS keychain or a secrets manager, never in configuration files.
- MUST enforce authenticated server discovery against an explicit allowlist. A server not on the list is unreachable.
- HTTP streaming transport MUST implement all of the following. Payload size limits and recursive-payload rejection, rate limiting on tool calls and transport requests, client and server authentication, mutual TLS, CORS protection, CSRF protection, integrity checks against replay, spoofing, and poisoned responses.

### Multi-tenant remote (several organizations share the server)
- Everything in single-tenant remote, plus:
- MUST enforce tenant isolation in infrastructure with per-tenant encryption and RBAC, tested in CI.
- SHOULD prefer the server hosted directly by the provider that already holds the tenant data over any third-party intermediary.
- SHOULD provide remote attestation where the platform supports it, so a client can verify the server runs the expected code. Target state, not baseline.

## Pre-merge checks

Before merging any MCP-related change, confirm:

1. No secret in prompt, config, env, or log. `grep` for token-like strings.
2. No `exec`, `eval`, `os.system`, `child_process.exec`, or `shell: true` receiving untrusted content.
3. No client token forwarded downstream unchanged.
4. Every consequential tool has a preview stage, a bound confirmation, and a rollback path.
5. No authorization decision depends on model output.
6. Tool results return named fields, not whole records.
7. Every schema is signed and pinned. Every dependency is pinned.
8. Deployment pattern declared and its mandatory controls present.
9. Tool-call logging emits identity, tool, safe parameter metadata, and time to the SIEM.
10. Trifecta scorecard updated for the agent.

## References

1. Consolidated MCP Top 10 security practices (MCP01 through MCP10).
2. NSA. *Model Context Protocol (MCP): Security Design Considerations for AI-Driven Automation*. U/OO/6030316-26, May 2026.
3. Coalition for Secure AI. *Model Context Protocol (MCP) Security*. OASIS Open Project, Workstream 4, approved 8 January 2026. https://www.coalitionforsecureai.org/wp-content/uploads/2026/03/model-context-protocol-security-1.pdf
