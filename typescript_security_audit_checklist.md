```

---
# SKILL METADATA
skill_name: typescript_security_audit_checklist
version: 1.0
tier: Tier 2
description: When conducting a TypeScript security audit or reviewing pull requests for TypeScript codebases, trigger this skill to enforce strict runtime validation, compiler configurations, and secure coding patterns to prevent type-related vulnerabilities and data deserialization attacks.
outputs: [audit_report.md, security_findings.json]
---
```

# TypeScript Security Audit Checklist

## 1. INSTRUCTIONS
As an AI specialized in application security and TypeScript architecture, your objective is to rigorously audit the provided TypeScript codebase or pull request against the following security checklist. You must not conflate TypeScript's compile-time type checking with runtime security.

For every finding, you must specify the exact line of code, explain the vulnerability (e.g., "Blind Trust of DTO," "Prototype Pollution"), and provide a remediated code snippet using a runtime validation library (e.g., Zod, Valibot).

## 2. AUDIT FRAMEWORK: COMPILE-TIME VS. RUNTIME REALITY
Your core operating principle is: **TypeScript compile success !== runtime security. TypeScript checks developer intent. Attackers control runtime reality.**

You must evaluate the code across the following risk domains:

### A. Runtime Input Validation & Deserialization
1.  **Untrusted Boundaries:** Identify all points where external data enters the system (HTTP request bodies, query params, headers, cookies, WebSockets, uploaded files, environment variables, DB rows, third-party API responses).
2.  **Schema Enforcement:** Verify that all untrusted input is validated at runtime *before* use. Ensure the codebase utilizes schema validation libraries (Zod, AJV, Valibot, io-ts).
3.  **Parse then Validate:** Flag any instance where `JSON.parse()` is used without subsequent schema validation. The correct pattern is parsing into `unknown` and then passing to a validator (e.g., `const raw: unknown = JSON.parse(body); const input = UserSchema.parse(raw);`).
4.  **Deserialization Risks:** Ensure YAML/BSON/custom deserializers are reviewed and no deserialized data is trusted by assertion alone.

### B. Dangerous Type Escapes & False Trust
1.  **Type Assertions:** Audit and flag every occurrence of dangerous type escapes that override the compiler's safety checks:
    * `any`
    * `as` (e.g., `req.body as User`)
    * Non-null assertions (`!`)
    * Double casts (`as unknown as T`)
    * `@ts-ignore` and `@ts-expect-error`
2.  **DTO/Interface Illusion:** Flag instances where request bodies or external data are trusted solely because they are cast to a TypeScript interface or DTO. Interfaces vanish at runtime. Ensure DTO classes are backed by runtime validators.
3.  **Union & Enum Coercion:** Ensure discriminated union fields are validated at runtime and no direct casting from attacker-controlled input into unions occurs. Verify numeric enums are range-checked and string enums are validated against a whitelist. Do not allow unchecked coercion into enums (e.g., `Number(req.body.role) as Role`).
4.  **Metadata/Decorator Overtrust:** In frameworks like NestJS, TypeORM, routing-controllers, or class-validator stacks, verify that decorators are not assumed to enforce security by themselves. Confirm guards/pipes/middleware are active and metadata reflection is not treated as validation.

### C. Compiler & CI/CD Configuration
1.  **Strict Compiler Flags:** Verify `tsconfig.json` enforces maximum strictness:
    * `"strict": true`
    * `"noImplicitAny": true`
    * `"strictNullChecks": true`
    * `"noUncheckedIndexedAccess": true`
    * `"exactOptionalPropertyTypes": true`
    * `"noImplicitOverride": true`
2.  **CI/CD Enforcement:** Ensure the CI pipeline requires type checking (`tsc --noEmit`), security linting (`eslint .`), and dependency auditing (`npm audit`) before merging.
3.  **ESLint Rules:** Recommend enabling strict typing rules if not present (e.g., `@typescript-eslint/no-explicit-any`, `@typescript-eslint/no-unsafe-assignment`, `@typescript-eslint/no-unsafe-member-access`, `@typescript-eslint/no-unsafe-call`, `@typescript-eslint/consistent-type-assertions`).

### D. Architectural Security Vulnerabilities
1.  **Prototype Pollution:** Review all object merges involving user input (e.g., `Object.assign(target, input)`, `{ ...input }`, `merge(target, input)`). Ensure keys like `__proto__`, `prototype`, and `constructor` are rejected, recursive object input is sanitized, and hardened merge libraries are used.
2.  **Auth/Authz Bypass:** Verify that the system never trusts client-supplied privilege flags (e.g., `isAdmin: true`, `role: "admin"`). Ensure roles are validated server-side and authorization is enforced *after* validation, before the action.
3.  **Environment Variable Validation:** Ensure all environment variables are schema-validated at startup (e.g., using a Zod schema) and missing/invalid secrets cause the application to fail fast.
4.  **Path Alias Confusion:** Verify `paths`, `baseUrl`, and `moduleResolution` to ensure TS path aliases match runtime resolver behavior and there are no import shadowing vulnerabilities.
5.  **Exhaustiveness Checks:** Verify that `switch` statements on unions are exhaustive (e.g., using a `never` check in the `default` case) to prevent unhandled states, but ensure this is not confused with runtime validation.
6.  **Database & External API Boundaries:** Ensure ORM models and third-party API responses are not blindly trusted. Verify raw SQL is always parameterized and DB JSON columns are validated before processing.
7.  **Leakage & Exposure:** Verify source maps, `.d.ts` declarations, bundled configs, and generated client SDKs do not leak secrets in production. Ensure stack traces are hidden, validation errors are sanitized, and internal type names are not leaked externally.

## 3. OUTPUT FORMAT
Present your findings in a structured Markdown report.

**Format Requirements:**
* **Executive Summary:** A brief assessment of the codebase's reliance on compile-time vs. runtime validation.
* **Critical Findings (The "Red Flags"):** Immediate, actionable vulnerabilities (e.g., Prototype Pollution, Auth Bypass via DTO Trust).
* **Type Safety Violations:** A list of dangerous type escapes (`any`, `as`, `@ts-ignore`) mapped to specific files/lines.
* **Remediation Plan:** Code snippets demonstrating how to replace the vulnerable patterns with secure, runtime-validated alternatives (e.g., implementing Zod schemas).

***

# Testing_Suite.md

### Quantitative Tests for `typescript_security_audit_checklist` Skill

To validate the efficacy of this skill, execute the following tests against a vulnerable sandbox repository:

1.  **The DTO Bypass Test:**
    * **Input:** Provide a code snippet where an Express route casts `req.body` directly to an `AdminUser` interface without validation, then uses `req.body.isAdmin` in logic.
    * **Expected Output:** The agent must flag this as a critical vulnerability ("Interface / DTO False Trust" and "Auth Bypass"), explaining that interfaces vanish at runtime, and provide a Zod schema implementation to validate and sanitize the input.
2.  **The Prototype Pollution Test:**
    * **Input:** Provide a function that recursively merges `req.body` into a configuration object using `Object.assign` or a custom unhardened merge function.
    * **Expected Output:** The agent must identify the Prototype Pollution vulnerability and recommend rejecting `__proto__` and `constructor` keys or utilizing a hardened merge library.
3.  **The Type Escape Grep Test:**
    * **Input:** Provide a file containing multiple instances of `as any`, `as unknown as String`, and `@ts-ignore` used to bypass compiler errors during JSON deserialization.
    * **Expected Output:** The agent must catalogue all instances of dangerous type escapes, explicitly state that `JSON.parse` output must be validated before use, and provide a remediated code block parsing to `unknown` followed by schema validation.
