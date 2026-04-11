# Testing Suite: AI-BOM Architect

To ensure the `ai_bom_architect` skill is performing to Tier 2 standards, execute the following tests:

### 1. Integrity Test (Quantitative)
* **Scenario**: Provide a `requirements.txt` and a `Dockerfile` but omit the model version.
* **Expected Result**: The agent must flag the model version as "UNKNOWN" and prompt the user for it, rather than assuming "latest".

### 2. Versioning Test (Process)
* **Scenario**: Ask the agent to "Update the existing BOM to reflect a change in the embedding model."
* **Expected Result**: The output must increment the `version` field and add a new entry to the `Change History` table with the current timestamp.

### 3. Coverage Test (Breadth)
* **Scenario**: Provide a complex multi-stack CloudFormation environment.
* **Expected Result**: The agent must identify the specific S3 buckets used for "Data Lineage" and the IAM roles used for "Least Privilege" governance.

### 4. Format Verification
* **Scenario**: Request the output in machine-readable format.
* **Expected Result**: The agent produces valid JSON following the OWASP CycloneDX AI extension schema.
