# Testing Suite: AI-BOM Architect (Regulatory Edition)

To ensure the `ai_bom_architect` skill meets the legal standards of the EU AI Act, CA AB 2013, and CO SB 24-205, execute these specific validation tests.

### 1. Regulatory "Gap" Test (CA AB 2013)
* **Scenario**: Provide a dataset name (e.g., "CustomerSupportLogs") but do not provide its size or whether it contains synthetic data.
* **Expected Result**: The agent must flag a "Compliance Gap" for AB 2013. It must explicitly state: `Synthetic Data Status: UNKNOWN (Required for AB 2013)` and `Dataset Size: UNKNOWN`.

### 2. High-Risk Integrity Test (EU AI Act)
* **Scenario**: Provide a model version (e.g., Nova Micro v1) but omit the SHA-256 hash or digital signature.
* **Expected Result**: The agent must mark the Integrity field as `UNVERIFIED` and generate a warning that cryptographic markers are missing for high-risk technical documentation.

### 3. Change History Persistence Test
* **Scenario**: Provide a version 1.0 BOM and ask to "Update the Python version from 3.11 to 3.12."
* **Expected Result**: The agent must increment the version to `1.1`, update the `Date of Creation`, and append a row to the `Change History` table detailing the specific library/image change.

### 4. Risk Disclosure Test (CO SB 24-205)
* **Scenario**: Ask the agent to generate a BOM for a "High-Risk" use case like automated hiring.
* **Expected Result**: The agent must include a section for "Foreseeable Risks of Algorithmic Discrimination" and "Known System Limitations," linking them back to the Colorado SB 24-205 requirement.

### 5. Format & Machine-Readability
* **Scenario**: Request a machine-readable export.
* **Expected Result**: The agent generates a JSON file that specifically includes the `externalReferences` and `evidence` fields found in the CycloneDX AI extension.
