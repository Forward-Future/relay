---
name: relay
description: Discover available models and route one task through frontier planning, workhorse execution, and independent frontier review.
disable-model-invocation: true
---

# Relay

Route the user's task through the coding environment's native child tasks or agents, chosen by role rather than model family:

1. a frontier **planner** resolves ambiguity and defines the finish line;
2. a cheaper, faster **workhorse executor** implements the settled plan;
3. a separate frontier **reviewer** independently checks the result.

The user may supply any model map, for example:

```text
planning: Fable
execution: GPT-5.5
review: GPT-5.6 Sol
```

Model names are configuration, never capability claims. The coordinator discovers what the active environment can actually delegate to, classifies those models from current evidence, suggests the route, verifies every handoff, and reports the route actually used.

Invocation explicitly authorizes the user-visible child tasks required by the route, including model and effort selection through the environment's native delegation controls. It does not authorize destructive actions, external mutations, or deployment unless the user separately authorized them.

## Preflight

1. Restate the requested outcome, acceptance criteria, constraints, allowed mutations, and deployment authority.
2. Identify the active coding environment, authentication boundary, and native delegation controls. Confirm it can create, inspect, and continue child work with explicit model selection. If it cannot, report the missing control and stop; do not imitate the relay with hidden subprocesses or direct provider calls.
3. Record the starting state. In a Git repository, capture the branch, revision, and dirty files so relay work remains distinguishable from pre-existing changes.
4. Classify the work:
   - **Settled:** the solution and finish line are already known; planning may stay in the coordinator.
   - **Judgment-heavy:** implementation is tractable, but tradeoffs or failure modes need a planner.
   - **Open-ended:** the problem, architecture, or safe path must be discovered before execution.
5. Read [`MODEL_SELECTION.md`](MODEL_SELECTION.md) completely. Use its discovery procedure to produce an evidence-backed inventory of every model selectable for child work in this environment, then classify each as **frontier**, **workhorse**, **utility**, or **unknown**. Availability discovery is complete only when every candidate has an exact selectable ID and evidence from the active environment—not merely a provider catalog or remembered model name.
6. Respect explicit user choices when they are available. Otherwise auto-suggest the best role map from the inventory:

```markdown
| Role | Model | Effort | Why this model fits | Substitution |
| --- | --- | --- | --- | --- |
| Planning | <frontier model> | <effort> | <uncertainty it must resolve> | <allowed fallback or stop> |
| Execution | <workhorse model> | <effort> | <bounded work it can complete> | <allowed fallback or stop> |
| Review | <independent frontier model> | <effort> | <risks it must inspect independently> | <allowed fallback or stop> |
```

Show the inventory, classification confidence, and suggested map before launching children. The suggestion is informational; invocation already authorizes the skill to proceed with high-confidence selections. Pause only when a required role has no available model, a classification is low-confidence, or the suggestion conflicts with an explicit user choice.

Use exact model IDs and effort values exposed by the host. Never invent an identifier or silently substitute a model. Preflight passes when the finish line is checkable, every external side effect is authorized, and each selected model can actually be created through the native delegation control.

## Build the route

Create the shortest route that preserves independent review:

```markdown
| Phase | Thread | Model | Effort | Deliverable | Gate |
| --- | --- | --- | --- | --- | --- |
| Planning | ... | ... | ... | settled plan | affected surfaces, decisions, risks, checks, integration path |
| Execution | ... | ... | ... | implementation artifact | requested behavior exists and focused checks pass |
| Review | ... | ... | ... | independent findings | every finding is resolved or rejected with evidence |
```

- Keep planning in the coordinator when the task is already settled.
- Give execution to the cheapest, fastest available workhorse that can reliably meet the gate. Promote execution to a frontier model when no workhorse satisfies the task's risk or capability requirements.
- Use a fresh reviewer thread with no executor context beyond the task, plan, actual artifact, and evidence. The reviewer must not review its own work.
- Prefer a reviewer with a different model ID and family from both executor and planner. If only one frontier model is available, use it in a fresh review context and disclose the reduced model diversity.
- Add reconnaissance, integration, release, or monitoring phases only when they produce a necessary artifact or reduce material uncertainty.
- Parallelize only independent slices with disjoint ownership or an explicit integration seam. Permit only one writing thread per checkout.

If deployment is in scope, read [`DEPLOYMENT.md`](DEPLOYMENT.md) completely before assigning the release phase. The route passes when every phase has one owner, one artifact, and one checkable gate, and no dependent phase starts before its inputs exist.

## Create and run child work

Use the coding environment's native visible delegation controls. Do not shell out to another model runner or provider API to manufacture capabilities the environment did not expose.

For each phase:

1. Create a child with the exact configured model ID, effort, prompt, and target.
2. Record the returned child ID when the environment supplies one. Wait for any pending workspace setup before starting dependent work.
3. Give it a self-contained brief:

```markdown
Role: <planner, executor, reviewer, or added phase>
Outcome: <one concrete result>
Inputs: <canonical paths, commits, URLs, evidence, and prior artifacts>
Constraints: <scope, invariants, authority, and forbidden actions>
Acceptance: <checks that prove this phase is done>
Return: <artifact or concise handoff, including evidence and unresolved risks>
```

4. Wait for the terminal result and inspect the returned artifact. Child creation is not completion.
5. Correct incomplete work in the same child context so it retains role and context. Create a new child when responsibility moves to another role or model.
6. Update the canonical route with the actual child ID, model, effort, artifact, and status.

Do not pass a summary forward as if it were the artifact. The next phase receives the actual plan, diff, test output, commit, or production evidence.

Do not archive or discard child records automatically. They are user-owned relay evidence.

## Gates and escalation

- **Planning passes** only when it identifies affected surfaces, decisions, risks, acceptance checks, and a safe integration path.
- **Execution passes** only when the requested behavior exists, focused checks pass, and unrelated user changes remain untouched.
- **Review passes** only when it inspects the actual artifact independently and every actionable finding is fixed and rechecked or explicitly rejected with evidence.

Retry the same model only for a transient failure or an underspecified brief. Escalate to a more capable model when failure reveals a capability gap or new uncertainty. Return to planning when evidence invalidates the route rather than accumulating patches against a broken plan.

Substitute only according to the recorded model map. If a required model is unavailable and no substitution was authorized, stop at that gate and return completed artifacts. Never describe a fallback as equivalent without evidence.

## Final report

Lead with the finished outcome. Then report:

- the discovered inventory and classification evidence;
- the route actually used, including role, child ID, model, and effort per phase;
- artifacts produced and checks passed;
- substitutions, escalations, or skipped phases and why;
- review findings and their resolution;
- deployment revision and health evidence, when deployment was authorized;
- any remaining blocker or risk.

Never report the planned route as the route actually run.
