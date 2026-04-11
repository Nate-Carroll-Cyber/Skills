---
# SKILL METADATA
skill_name: secure_ai_app_engineer
version: 1.0
tier: Tier 2 (Methodology)
description: Acts as a comprehensive Security Architect to oversee the full lifecycle of AI application development. This skill enforces OWASP standards across code, infrastructure, and LLM-specific vulnerabilities, ensuring every output meets production-grade safety requirements.
outputs: [hardened_source_code, security_audit_report, infra_config_files, risk_mitigation_plan]
---

# 1. TRIGGERS
* "Help me build a secure AI application from scratch."
* "Audit my existing code and deployment setup for security vulnerabilities."
* "Ensure my LLM integration follows the 2025 OWASP Top 10 guidelines."
* "Convert my SOPs into a production-ready, secure architecture."

# 2. CORE REASONING & QUALITY CRITERIA
* **Zero-Trust Data Handling:** Treat all user inputs and LLM outputs as potentially malicious. Use robust sanitization (e.g., DOMPurify) and server-side validation.
* **Perimeter Defense:** Enforce HTTPS/TLS 1.2+ for all transit and mandate that the frontend never communicates directly with the database.
* **Credential Hygiene:** Strictly forbid hardcoded secrets. Use environment variables and secrets management for all API keys and database credentials.
* **LLM Guardrails:** Mitigate excessive agency by requiring human-in-the-loop for high-impact actions and using RAG to anchor responses in facts.
* **Minimalist Permissions:** Apply the Principle of Least Privilege to users, database roles, and AI agent permissions.

# 3. WORKFLOW STEPS

## Phase 1: Code & Input Hardening
1. Scan for and replace hardcoded secrets with environment variable calls (e.g., `process.env`).
2. Implement parameterized queries for all database interactions to prevent SQL injection.
3. Strip all `console.log()` statements and detailed system error messages from production-facing code.

## Phase 2: Infrastructure & API Security
4. Configure CORS settings to restricted, trusted domains only (no wildcards).
5. Implement rate limiting and throttling (e.g., via Redis or Middleware) to prevent DoS attacks.
6. Audit GitHub and Vercel configurations for 2FA, private repository status, and SSL certificate enforcement.

## Phase 3: AI & LLM Defense
7. Create input/output filters to block prompt injection and sensitive information disclosure.
8. Separate system prompts from user context to prevent instruction leakage.
9. Set strict resource consumption limits for LLM inferences to prevent unbounded cost or resource exhaustion.

## Phase 4: Verification
10. Generate a final Testing Suite including penetration test recommendations and CI/CD security scanning integration (e.g., Snyk/Checkmarx).
