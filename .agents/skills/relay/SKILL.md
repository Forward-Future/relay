---
name: relay
description: Explicitly route one task through discovered models for frontier planning, workhorse execution, and independent frontier review.
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
save: project
```

Model names are configuration, never capability claims. The coordinator discovers what the active environment can actually delegate to, classifies those models from current evidence, suggests the route, verifies every handoff, and reports the route actually used.

Run the relay only when the user explicitly invokes it or explicitly requests this delegation pattern. Automatic skill loading is not authorization to create child work. Explicit invocation authorizes the native child tasks required by the route, including model and effort selection. It does not authorize destructive actions, external mutations, or deployment unless the user separately authorized them.

## Preflight

1. Restate the requested outcome, acceptance criteria, constraints, allowed mutations, and deployment authority.
2. Read [`ENVIRONMENTS.md`](ENVIRONMENTS.md) completely. Select the matching adapter and record whether the environment can discover child-selectable models, select a model, verify the actual model used, inspect results, and continue a child. If a required control is absent, report the precise limitation; do not imitate it with hidden subprocesses or direct provider calls.
3. Record the starting state. In a Git repository, capture the branch, revision, and dirty files so relay work remains distinguishable from pre-existing changes.
4. Classify the work:
   - **Settled:** the solution and finish line are already known; planning may stay in the coordinator.
   - **Judgment-heavy:** implementation is tractable, but tradeoffs or failure modes need a planner.
   - **Open-ended:** the problem, architecture, or safe path must be discovered before execution.
5. Read [`PREFERENCES.md`](PREFERENCES.md) completely. Load user and project preferences from its defined paths when present. Treat saved selectors as preferences, never as availability or tier evidence.
6. Read [`MODEL_SELECTION.md`](MODEL_SELECTION.md) completely. Use its discovery procedure to produce an evidence-backed inventory of every model selectable for child work, then classify each as **frontier**, **workhorse**, **utility**, or **unknown**. Availability discovery is complete only when every candidate has a native selector and evidence from the active environment—not merely a provider catalog, saved preference, or remembered model name.
7. Choose each phase using this precedence: an explicit choice for the current run, then a valid saved project preference, then a valid saved user preference, then automatic suggestion. Validate every selected model against the current inventory and phase gate before routing.

```markdown
| Role | Model | Effort | Why this model fits | Substitution |
| --- | --- | --- | --- | --- |
| Planning | <frontier model> | <effort> | <uncertainty it must resolve> | <allowed fallback or stop> |
| Execution | <workhorse model> | <effort> | <bounded work it can complete> | <allowed fallback or stop> |
| Review | <independent frontier model> | <effort> | <risks it must inspect independently> | <allowed fallback or stop> |
```

Show the inventory, classification confidence, and suggested map before launching children. Explicit invocation authorizes the skill to proceed with high-confidence selections. Pause when a required role has no qualifying model, a classification is low-confidence, or the suggestion conflicts with an explicit user choice.

When the user asks to save or update preferred models, follow [`PREFERENCES.md`](PREFERENCES.md). Persistence is a separate file mutation: never infer permission to save from a one-run model choice. Report the saved scope and path after writing, and show which preferences were applied or bypassed during preflight.

Use exact model IDs or documented native selectors and effort values exposed by the host. Never invent an identifier or silently substitute a model. Preflight passes when the finish line is checkable, every external side effect is authorized, and the planner and reviewer are genuine frontier models—not merely the strongest models in a weak inventory.

## Build the route

Create the shortest route that preserves independent review:

```markdown
| Phase | Child | Model | Effort | Deliverable | Gate |
| --- | --- | --- | --- | --- | --- |
| Planning | ... | ... | ... | settled plan | affected surfaces, decisions, risks, checks, integration path |
| Execution | ... | ... | ... | implementation artifact | requested behavior exists and focused checks pass |
| Review | ... | ... | ... | independent findings | every finding is resolved or rejected with evidence |
```

- Keep planning in the coordinator only when its verified model is frontier; otherwise create a frontier planning child even when the task appears settled.
- Give execution to the cheapest, fastest available workhorse that can reliably meet the gate. Promote execution to a frontier model when no workhorse satisfies the task's risk or capability requirements.
- Use a fresh reviewer thread with no executor context beyond the task, plan, actual artifact, and evidence. The reviewer must not review its own work.
- Prefer a reviewer with a different model ID and family from both executor and planner. If only one frontier model is available, use it in a fresh review context and disclose the reduced model diversity.
- Add reconnaissance, integration, release, or monitoring phases only when they produce a necessary artifact or reduce material uncertainty.
- Parallelize only independent slices with disjoint ownership or an explicit integration seam. Permit only one writing thread per checkout.

If deployment is in scope, read [`DEPLOYMENT.md`](DEPLOYMENT.md) completely before assigning the release phase. The route passes when every phase has one owner, one artifact, and one checkable gate, and no dependent phase starts before its inputs exist.

## Create and run child work

Use the coding environment's native visible delegation controls. Do not shell out to another model runner or provider API to manufacture capabilities the environment did not expose.

Before every delegation call, announce the assignment to the user. Name the phase, role, selected model or native selector, tier, effort, and reason for the choice. Do not create a child silently.

```text
Relay delegation — <phase>/<role>
Model: <exact model ID or native selector> (<tier>, effort: <effort>)
Why: <one concise reason this model fits>
```

When launching independent children in parallel, one compact table may announce the whole batch immediately before the calls, but it must include one row per child with its role, model, tier, effort, and assignment. After creation, report the returned child ID and verified model. If the verified model differs from the announcement, immediately announce the substitution, update the route, and reroute or stop according to the model gate.

For each phase:

1. Announce the configured role, native model selector, tier, effort, and assignment.
2. Create a child with that selector, effort, prompt, and target.
3. Record the returned child ID when the environment supplies one. Verify the actual model and effort through returned metadata or an authoritative native contract that guarantees an accepted exact selector is used and rejects unavailable values. A requested selector alone is not proof. Aliases and fallback-capable controls require resolved-model readback. Report the child ID and verified model. If the environment silently substitutes, announce the mismatch, update the inventory, and reroute; if it cannot verify actual model identity, stop rather than claim a model-specific relay.
4. Wait for any pending workspace setup before starting dependent work.
5. Give it a self-contained brief:

```markdown
Role: <planner, executor, reviewer, or added phase>
Outcome: <one concrete result>
Inputs: <canonical paths, commits, URLs, evidence, and prior artifacts>
Constraints: <scope, invariants, authority, and forbidden actions>
Acceptance: <checks that prove this phase is done>
Return: <artifact or concise handoff, including evidence and unresolved risks>
```

6. Wait for the terminal result and inspect the returned artifact. Child creation is not completion.
7. Correct incomplete work in the same child context so it retains role and context. Create a new child when responsibility moves to another role or model, and announce that new delegation too.
8. Update the canonical route with the actual child ID, verified model, effort, artifact, and status.

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
- the route actually used, including role, child ID, verified model, and effort per phase;
- artifacts produced and checks passed;
- substitutions, escalations, or skipped phases and why;
- review findings and their resolution;
- deployment revision and health evidence, when deployment was authorized;
- any remaining blocker or risk.

Never report the planned route as the route actually run.
