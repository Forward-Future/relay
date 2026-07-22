# Model discovery and selection

This file is the single source of truth for deciding which models are available and which count as frontier, workhorse, utility, or unknown. Recompute the inventory for every relay run; model access depends on the active coding environment, authentication boundary, account, policy, and delegation tool.

## What “available” means

A model is **available for delegation** only when the active environment exposes an exact model ID that can be selected for a new child task or agent in the current session. The current coordinator model may also be inventoried as **current-only** when its exact identity is exposed, but current-only availability does not make it eligible for execution or independent review. A model appearing in provider documentation, memory, or configuration does not prove it can be delegated to.

Discover candidates in this order:

1. Record the current coordinator's exact model ID and effort when the environment exposes them; mark its scope `current-only` unless the same ID is independently selectable for child work.
2. Inspect the active child-task, agent-spawn, or thread-creation tool schema and capability metadata. Enumerated model IDs and effort values are direct evidence of `child-selectable` scope.
3. Use an environment-native model-list capability when it is callable by the agent. A human-visible picker or interactive command counts only when its choices are exposed to the current agent or supplied by the user.
4. Read current project or user agent configuration for candidate IDs and role descriptions. Treat configuration as a hint until the native delegation control confirms selection.
5. Accept exact IDs supplied by the user as candidates, then validate them through the native delegation control before assigning dependent work.

Do not use a public provider model catalog as availability evidence; catalogs do not reflect the current surface, account entitlement, region, policy, or authentication boundary. Do not probe availability by calling provider APIs or shelling out to a different coding agent. If the environment exposes no enumerable model list, report that limitation and request the available IDs rather than pretending to auto-discover them.

Record the result before routing:

```markdown
| Exact model ID | Scope | Availability evidence | Supported effort | Tier | Tier evidence | Confidence |
| --- | --- | --- | --- | --- | --- | --- |
| ... | current-only / child-selectable | tool schema / native list / validated user ID | ... | frontier / workhorse / utility / unknown | ... | high / medium / low |
```

## Tier definitions

Classify relative to the models actually available in this environment. A newer name, larger version number, or higher price is not sufficient evidence.

### Frontier

The environment or provider's highest-capability available tier for autonomous coding, difficult reasoning, architecture, diagnosis, and adversarial review. A frontier model must support the tools and context needed for the phase. Use current environment metadata or current provider guidance as evidence; use remembered reputation only as low-confidence evidence.

### Workhorse

A general-purpose agentic coding model optimized for a better cost or latency profile than the frontier tier while remaining reliable at bounded implementation, test writing, refactoring, and ordinary debugging. A cheap model that cannot use the required tools or reliably edit and test is not a workhorse for this task.

### Utility

A speed- or cost-optimized model suited to narrow, checkable operations such as reconnaissance, extraction, formatting, mechanical edits, or focused verification. Utility models do not own ambiguous planning or high-risk review.

### Unknown

A selectable model whose relative capability, coding/tool support, or cost/latency position cannot be established from current evidence. Do not auto-assign an unknown model to a required phase.

## Evidence hierarchy

Use the strongest available evidence:

1. tier or role metadata exposed by the active environment;
2. current provider documentation for the exact model ID;
3. current environment configuration that describes the model's intended role;
4. model-family naming conventions;
5. remembered model reputation.

Levels 1–2 support high-confidence classification. Level 3 supports medium confidence when native selection is confirmed. Levels 4–5 are low confidence and require user confirmation before the relay launches that role.

When evidence conflicts, prefer the higher-ranked evidence and record the conflict. Availability evidence and tier evidence are separate: a tool schema can prove that `model-x` is selectable without proving whether it is frontier or workhorse.

## Auto-suggestion

Select models only after the inventory is complete:

1. **Planning:** choose the strongest coding-suitable frontier model for judgment-heavy or open-ended work. A current-only frontier model may plan in the coordinator. Keep settled planning there when it already meets the gate.
2. **Execution:** choose the lowest-cost or lowest-latency workhorse that satisfies required tools, context, risk, and acceptance checks. Use a utility model only for a narrow deterministic slice. Use frontier execution when no workhorse qualifies.
3. **Review:** choose the strongest coding-suitable frontier model that did not execute the work. Prefer a different model ID and family from the planner as well, because diversity reduces correlated blind spots. If only one frontier model exists, use a fresh context and disclose reduced diversity.

Rank cost and latency only when the environment supplies credible metadata or the relative position is established by current provider guidance. Otherwise state that the workhorse suggestion optimizes for the provider's declared balanced tier, not a verified price comparison.

The suggestion passes when every role has an available exact ID, a supported effort, sufficient tool access, and medium- or high-confidence tier evidence. If no map passes, show the partial inventory and stop at preflight.
