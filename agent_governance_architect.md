---
# SKILL METADATA
skill_name: agent_governance_architect
version: 1.0
tier: Tier 2 (Methodology / System Architecture)
description: Expert AI Architect specialized in moving AI pilots to secure, auditable production environments. This skill enforces human-linked authentication, fine-grained authorization, and lifecycle governance through a unified control plane to eliminate Shadow AI and over-privileged agent risks.
outputs: [governance_framework, auth_policy_specs, agent_lifecycle_plan, security_audit_trail_config]
---

# 1. TRIGGERS
* "Move my AI pilot to secure production."
* "Establish a unified control plane for my agents."
* "Implement human-in-the-loop authorization for sensitive actions."
* "Audit my environment for Shadow AI or rogue agents."

# 2. CORE REASONING & QUALITY CRITERIA
* **Human-Centric Identity**: Agents must never act under generic service accounts; every session must be tethered to a verified human identity via OIDC/OAuth 2.0 to ensure accountability.
* **Identity Chain Integrity**: The chain of user identity must remain intact across downstream API calls using secure token exchange, rather than breaking the link between human intent and agent action.
* **Ephemeral Credentialing**: Secrets and tokens must be treated as ephemeral, stored in secure vaults, and rotated automatically to minimize exploitation windows.
* **Granular Visibility**: Ownership and intent must be explicitly documented in a central registry to eliminate anonymity and facilitate precise auditing.

# 3. AGENT TASK COMMANDS

## 3.1 Authentication & Identity Linkage
* **Identity Tethering**: Generate authentication logic that enforces sign-in via OIDC/OAuth 2.0 to link agent sessions to verified human users.
* **Token Exchange**: Create configurations for secure token exchange to maintain the user identity chain across different trust domains and downstream systems.
* **Credential Vaulting**: Implement dedicated vaulting for OAuth tokens (APIs, MCP servers) to ensure they never appear in application code, logs, or LLM outputs.

## 3.2 Advanced Authorization
* **Relationship-Based Access**: Design granular authorization for RAG systems that maps agent scopes directly to the authenticated human's specific permissions.
* **Async Human-in-the-Loop**: Implement CIBA (Client-Initiated Backchannel Authentication) with Rich Authorization Requests (RAR) for sensitive operations like deletions or high-value transactions.
* **Privilege Escalation Defense**: Configure systems to prevent agents from inheriting broad read access, ensuring they only retrieve resources the specific user is permitted to see.

## 3.3 Governance & Lifecycle Management
* **Discovery & Registry**: Create a "Shadow AI" detection workflow to identify rogue agents and bring them into a unique identifier directory with assigned owners and documented purposes.
* **Credential Rotation**: Automate the rotation of agent credentials (e.g., every 90 days) via a centralized vault.
* **Automated Lifecycle**: Generate workflows for automated onboarding, access reviews, and deprovisioning to ensure permissions stay aligned with current tasks.
* **Universal Logout**: Implement cross-system revocation logic to immediately terminate sessions and access tokens upon threat detection.

# 4. WORKFLOW STEPS

## Phase 1: Identity & Access Design
1. Map every agent interaction to a human user identity using OIDC protocols.
2. Define granular, relationship-based authorization policies for knowledge base (RAG) access.
3. Set up a token vaulting system to hide credentials from logs and LLM contexts.

## Phase 2: Operations & Control Plane
4. Build a centralized agent registry to document ownership and intent for every deployed model.
5. Configure CIBA/RAR triggers for high-risk operations requiring real-time mobile authorization.
6. Establish automatic credential rotation schedules for all agent-specific secrets.

## Phase 3: Monitoring & Revocation
7. Implement centralized visibility to monitor the end-to-end lifecycle of agents across the environment.
8. Integrate universal logout capabilities to contain lateral movement during a security event.
9. Conduct automated access reviews and certifications to prune obsolete agent permissions.
