---
# 1. SKILL METADATA & REGISTRY (Items 1, 2, 5, 12)
# This section acts as the Tool Registry data structure before any code executes.
skill_name: enterprise_data_auditor
version: 2.0
agent_type: verify_and_plan  # [Item 12] Constrains the agent's behavior and system prompt
permission_tier: high_trust_internal  # [Item 2] Defines the risk level and approval gates
token_budget_max: 150000  # [Item 5] Hard limit to prevent runaway loops
outputs: [audit_report.json, telemetry_log.csv]
---

# 2. TOOL POOL ASSEMBLY (Items 1, 9)
**Dynamic Context:** This skill does not load all enterprise tools. It dynamically requests only the following subset based on execution context.
* **Allowed Tools:** * `sql_read_only` (No write permissions allowed)
  * `read_confluence_wiki`
  * `human_approval_gate`

# 3. TRIGGERS & WORKFLOW STATE (Item 4)
**Primary Directive:** Audit the previous week's database transactions for compliance anomalies.
**State Checkpoints:** The agent must update the orchestrator's explicit workflow state at these specific checkpoints. Do not proceed to the next step if a crash or halt occurs:
1. `STATE: PLANNED` - Query logic generated.
2. `STATE: EXECUTING_READS` - Gathering data via SQL tool.
3. `STATE: AWAITING_APPROVAL` - Paused for human review if anomalies > 5%.
4. `STATE: REPORTING` - Compiling final markdown.

# 4. METHODOLOGY & DUAL-LEVEL VERIFICATION (Item 8)
**Execution Reasoning:**
* Focus on identifying null values in required compliance fields.
* **Verification Level 1 (Task):** Before finalizing the output, self-verify that the generated JSON matches the strict enterprise schema.
* **Verification Level 2 (System Safety):** Ensure no "DROP", "UPDATE", or "ALTER" commands were attempted in the SQL logic. If detected, trigger an immediate system fault.

# 5. PERMISSION AUDIT TRAILS & TELEMETRY (Items 6, 7, 11)
**Event Streaming:** During execution, emit typed streaming events to the UI. Do not just output text.
* Stream `tool_consideration` before invoking SQL.
* Stream `token_consumption_warning` if usage exceeds 80% of `token_budget_max`.
**Audit Logging:** Every tool invocation, routing decision, and permission denial must be written to the separate `system_event_log.json` and tagged with this skill's ID.
**Permission Handlers:** If `human_approval_gate` is triggered, log the specific user ID that granted the approval. 

# 6. SESSION PERSISTENCE & RECOVERY (Item 3)
**Crash Survival:** At each Workflow State change (Section 3), write the current variables, token counts, and retrieved SQL data to `session_state_[run_id].json`.
* **Recovery Protocol:** If the agent crashes mid-execution, upon reboot, it must read the latest `session_state.json` and resume from the exact token and state checkpoint, rather than restarting the query.

# 7. TRANSCRIPT COMPACTION RULES (Item 10)
**Memory Management:** Because database schemas take up massive context windows, apply the following compaction threshold:
* Once context reaches 100k tokens, drop raw SQL query outputs from the active conversational transcript, retaining only the summary statistics and the system state.
---
