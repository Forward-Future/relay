# Model discovery and selection

This file is the single source of truth for deciding which models are available and which count as frontier, workhorse, utility, or unknown. Recompute the inventory for every relay run; model access depends on the active coding environment, authentication boundary, account, policy, and delegation tool.

## What “available” means

A model is **available for delegation** only when the active environment exposes a native selector that can be used for a new child task or agent in the current session. Prefer an exact model ID. An authoritative native contract may verify an accepted exact ID when it guarantees that ID is used and rejects unavailable values; aliases and fallback-capable selectors require resolved-model readback. The current coordinator may be inventoried as **current-only** when its exact identity is exposed, but current-only availability does not make it eligible for execution or independent review. A model appearing in provider documentation, memory, or configuration does not prove it can be delegated to.

Discover candidates in this order:

1. Use the selected environment adapter in [`ENVIRONMENTS.md`](ENVIRONMENTS.md) to collect the current coordinator and every child-selectable model.
2. Record the current coordinator's exact model ID and effort when exposed; mark it `current-only` unless the same selector is independently available for child work.
3. Inspect native delegation schemas and model-list capabilities. Enumerated selectors and effort values are direct availability evidence.
4. Read environment configuration for candidates and role descriptions. Treat configuration as a hint until native delegation verifies it.
5. Accept selectors supplied by the user as candidates, then validate them through native delegation before assigning dependent work.

Saved project and user preferences follow the same rule as selectors supplied for the current run: they enter the candidate set but do not prove availability, tier, effort support, or tool access.

Do not use a public provider catalog as availability evidence; catalogs do not reflect the current surface, entitlement, region, policy, or authentication boundary. Provider documentation may establish tier after native availability is proven. Do not probe by calling provider APIs or shelling out to another coding agent. If the environment exposes no enumerable list, use its adapter's safe sources; if those still cannot prove availability, request the picker choices rather than pretending to auto-discover them.

Record the result before routing:

```markdown
| Native selector | Verified model | Scope | Availability evidence | Supported effort | Tier | Tier evidence | Confidence |
| --- | --- | --- | --- | --- | --- | --- | --- |
| ... | exact ID from readback or guaranteed exact-selection contract | current-only / child-selectable | tool schema / native list / validated selector | ... | frontier / workhorse / utility / unknown | ... | high / medium / low |
```

## Tier definitions

Classify each model against current provider or environment tier evidence, then filter by availability. Never promote the best model in a weak inventory to frontier merely because nothing stronger is available. A newer name, larger version number, or higher price is not sufficient evidence.

### Frontier

The environment or provider's current highest-capability tier for autonomous coding, difficult reasoning, architecture, diagnosis, and adversarial review. A frontier model must have affirmative tier evidence and support the tools and context needed for the phase. If no available model qualifies, planning or review is blocked. Use current environment metadata or provider guidance as evidence; remembered reputation alone is insufficient.

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

Levels 1–2 support high-confidence classification. Level 3 supports medium confidence when native selection is confirmed. Levels 4–5 are low confidence and cannot establish frontier status; they require user confirmation for workhorse or utility assignment.

When evidence conflicts, prefer the higher-ranked evidence and record the conflict. Availability evidence and tier evidence are separate: a tool schema can prove that `model-x` is selectable without proving whether it is frontier or workhorse.

## Auto-suggestion

Select models only after the inventory is complete:

1. **Planning:** always use a coding-suitable frontier model. A current-only frontier model may plan in the coordinator; otherwise create a frontier planning child.
2. **Execution:** choose the lowest-cost or lowest-latency workhorse that satisfies required tools, context, risk, and acceptance checks. Use a utility model only for a narrow deterministic slice. Use frontier execution when no workhorse qualifies.
3. **Review:** choose the strongest coding-suitable frontier model that did not execute the work. Prefer a different model ID and family from the planner as well, because diversity reduces correlated blind spots. If only one frontier model exists, use a fresh context and disclose reduced diversity.

Apply the preference precedence from [`PREFERENCES.md`](PREFERENCES.md) independently for each phase. A preference wins only when it passes that phase's availability, tier, effort, tool-access, and independence gates. Follow its configured `on_unavailable` behavior when it fails.

Rank cost and latency only when the environment supplies credible metadata or the relative position is established by current provider guidance. Otherwise state that the workhorse suggestion optimizes for the provider's declared balanced tier, not a verified price comparison.

The suggestion passes when every role has an available selector, supported effort, sufficient tool access, verified model identity, and medium- or high-confidence tier evidence. Frontier assignments require affirmative frontier evidence. If no map passes, show the partial inventory and stop at preflight.

After each child starts, compare its verified model and effort with the route. Silent fallback invalidates the handoff gate. Reroute using the actual inventory, or stop when neither resolved-model readback nor a guaranteed exact-selection contract can verify the assignment.
