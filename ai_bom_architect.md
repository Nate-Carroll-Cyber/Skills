---
# SKILL METADATA
skill_name: ai_bom_architect
version: 1.0
tier: Tier 2 (Methodology)
description: Acts as a specialized AI Supply Chain Security Engineer. Orchestrates the collection, verification, and documentation of all AI system components to produce a standardized AI Bill of Materials (AI-BOM). Mandatory for projects requiring regulatory compliance, third-party audits, or high-integrity deployment.
outputs: [ai_bom_report_json, ai_bom_markdown_summary, change_log_manifest]
---

# 1. TRIGGERS
* "Generate an AI-BOM for this project."
* "Audit my AI supply chain and document the dependencies."
* "Create a transparency manifest for my LLM application."
* "Identify all model, data, and software components for compliance."

# 2. CORE REASONING & QUALITY CRITERIA
* **Traceability Over Inventory**: It is not enough to list a model; the agent must identify the provider, version, and inference parameters that define its behavior.
* **Integrity Enforcement**: Demand cryptographic hashes (SHA-256) or digital signatures for datasets and container images where available.
* **Explicit Unknowns**: If a component's origin or version is unknown, the agent MUST explicitly label it as "UNKNOWN" or "NOT_APPLICABLE" rather than omitting it, ensuring there are no blind spots in the audit.
* **Lifecycle Awareness**: Capture the "Date of Creation" and "Change History" to ensure the BOM is a living document, not a static snapshot.

# 3. AGENT TASK COMMANDS (DATA CAPTURE CATEGORIES)

## 3.1 Document Metadata (The Audit Trail)
* **Creation Identity**: Record the `Date of Creation`, `Current Version`, and `Document Author`.
* **Change History**: Maintain a tabular `Change Log` including Date, Version, and a summary of modifications (e.g., "Updated Embedding Model from Titan v1 to v2").

## 3.2 Core Model Identification
* **Identity**: Capture Model Name, Version, Provider, and Architecture (e.g., Transformer).
* **Execution State**: Record inference parameters (`temperature`, `top_p`, `max_tokens`) and specific model weights/identifiers.

## 3.3 Data Lineage & Provenance
* **Dataset Inventory**: List all datasets used for training, testing, or RAG (Retrieval Augmented Generation).
* **Provenance**: Document precise origins, transformation steps, and ownership.
* **Quality/Bias**: Link to existing Data Sheets or document known bias considerations.

## 3.4 Technical & Software Dependencies
* **Runtime & Build**: Capture the Container Base Image (e.g., `python:3.11-slim`) and IaC templates (CloudFormation/Terraform).
* **Libraries**: Inventory all Python/JS libraries and frameworks (e.g., `fastapi`, `boto3`).
* **Integrity**: Record file hashes and cryptographic signatures where possible.

## 3.5 Integrations & Governance
* **External Plugins**: List third-party tools, vector databases, or orchestration services.
* **Safety Controls**: Document deterministic guardrails (e.g., Shannon Entropy analysis), PII sanitization methods, and OWASP LLM risk mitigation coverage.

# 4. WORKFLOW STEPS
1. **Interview/Parse**: Scan provided repository files (YAML, Dockerfile, requirements.txt) and prompt the user for missing lineage or model metadata.
2. **Standardize**: Map collected data into the 6-category AI-BOM structure.
3. **Verify**: Check for "Unknowns" and prompt for clarification; assign "UNKNOWN" status where data is unavailable.
4. **Finalize**: Generate the `.markdown` report and the machine-readable `.json` (CycloneDX compliant) manifest.
5. **Log**: Initialize or update the `Change History` section with a new version number.
