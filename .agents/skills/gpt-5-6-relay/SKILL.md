---
name: gpt-5-6-relay
description: Route one task through model-specific GPT-5.6 child threads with explicit thinking efforts and checkable handoffs.
disable-model-invocation: true
---

# GPT-5.6 Relay

Turn the user's task into a **relay** of persistent Codex threads: a Sol coordinator plans, then creates Terra or Luna child threads for phases they can complete reliably. Each child receives a concrete artifact and returns evidence to its parent. The model names describe capability tiers, not fixed job titles.

Invocation of this skill explicitly authorizes the user-visible child threads required by the route, including their model and effort settings. The Sol coordinator owns the whole outcome, keeps the canonical task state, verifies every handoff, and reports the route actually used.

## Preflight

1. Restate the requested outcome, acceptance criteria, constraints, allowed mutations, and deployment authority. Infer ordinary implementation details, but do not infer permission for destructive or external actions.
2. Confirm the Codex app exposes project listing, thread creation, thread reading, and background thread messaging with model and effort controls. If it does not, show the proposed route and stop; do not replace persistent threads with hidden subprocesses.
3. For repository work, list projects and resolve the current workspace's project ID before creating a child. For a general task, use a projectless target.
4. Record the starting state. In a Git repository, capture the branch, revision, and dirty files so the relay can distinguish its work from pre-existing changes.
5. Classify the work by uncertainty, not apparent size:
   - **Clear**: the solution and finish line are already known.
   - **Judgment-heavy**: the implementation is tractable, but tradeoffs or failure modes need thought.
   - **Open-ended**: the problem, architecture, or safe path must be discovered.

Preflight is complete when the finish line is checkable and every proposed external side effect is authorized.

## The roster

| Model | Reach for it when | Relay starting effort | Escalate effort when |
| --- | --- | --- | --- |
| **Luna** | Fast reconnaissance, deterministic edits, formatting, focused checks, release mechanics, monitoring | Light | The operation has branching failure modes; raise through Medium, High, or Extra High, or hand the decision to Terra |
| **Terra** | Everyday implementation, tests, refactors, code review, bounded debugging, turning a settled plan into working artifacts | High | The change crosses ownership boundaries, has subtle invariants, or failed once for a non-obvious reason; raise to Extra High or Ultra |
| **Sol** | Ambiguous planning, architecture, hard diagnosis, high-risk review, resolving disagreement between child threads | Extra High | The decision is costly to reverse, evidence conflicts, or a failed route needs to be rebuilt; raise to Ultra |

Codex's platform default is Medium for all three models. The relay starting efforts above are deliberate task-specific overrides, not claims about the platform default.

Use the exact effort names exposed by each model:

| Model | Available efforts |
| --- | --- |
| **Luna** | Light, Medium, High, Extra High |
| **Terra** | Light, Medium, High, Extra High, Ultra |
| **Sol** | Light, Medium, High, Extra High, Ultra |

Never silently lower effort. Ultra on Sol and Terra enables automatic task delegation, so Extra High is the normal ceiling for a relay child. Use Ultra only when nested delegation is intentional, its write scopes are isolated, and the added threads cannot race the coordinator or another phase. Luna does not support Ultra; if a Luna phase genuinely needs Ultra, move the phase to Terra or Sol.

## Build the route

The Sol coordinator creates a short route before delegating:

```markdown
| Phase | Thread | Model | Effort | Deliverable | Gate |
| --- | --- | --- | --- | --- | --- |
| ... | parent/new child | Sol/Terra/Luna | Light/Medium/High/Extra High/Ultra | ... | ... |
```

Prefer these starting routes, then adapt them:

- **Mechanical task:** the Sol coordinator creates one Luna Light child to execute and run a focused check.
- **Normal feature or fix:** Sol settles the route; a Terra High child implements and tests; a Luna Light child runs mechanical closeout; a Luna Medium child releases when authorized.
- **Ambiguous or cross-system work:** Sol Extra High produces the plan and risks; a Terra High child implements and tests; the Sol parent reviews risky decisions; a Luna Medium child releases when authorized.
- **Production incident:** a Luna Light child gathers current evidence; Sol Extra High diagnoses; a Terra High child fixes and adds a regression test; a Luna Medium child releases and monitors.
- **Research or product design:** Sol Extra High resolves the hard questions; a Terra High child prototypes or produces the artifact; a Luna Light child packages and publishes authorized output.

Do not force all three models into every task. A phase earns a child thread only when its deliverable reduces uncertainty or performs necessary work. Do not use Sol for mechanical execution merely because Sol is stronger. Do not leave Luna to make an ambiguous production decision merely because deployment was assigned to Luna.

If the invoking thread is confirmed to be Sol at High or above, it is the coordinator. Otherwise create a Sol Extra High child with `Relay role: coordinator` in its prompt; that child plans and creates its own Terra and Luna children. The role marker prevents it from creating another Sol coordinator recursively.

If the route includes deployment, read [`DEPLOYMENT.md`](DEPLOYMENT.md) completely before assigning that phase. The route is complete when every phase has one owning thread, one artifact, and one checkable gate, and no child is started before its required inputs exist.

## Create child threads

Use the Codex app's thread tools, not in-process subagents or `codex exec`:

1. Call the project-listing tool and select the exact project ID for repository work.
2. Create the child with an explicit model ID, effort, prompt, and target.
3. Record the returned thread ID. If worktree creation is pending, do not start a dependent phase until a thread ID exists.
4. Read the child thread to verify its terminal result and obtain its handoff. If it is still running, keep the parent active and check at sensible intervals; creation is not completion.
5. Send corrections to the same child thread without model or effort overrides so it retains its role and context.
6. Create a new child when responsibility moves to another model. Do not change a Terra implementation thread into Luna deployment merely because background messaging supports a model override.

| Model | Thread model ID |
| --- | --- |
| **Luna** | `gpt-5.6-luna` |
| **Terra** | `gpt-5.6-terra` |
| **Sol** | `gpt-5.6-sol` |

| UI effort | Thread `thinking` value |
| --- | --- |
| **Light** | `low` |
| **Medium** | `medium` |
| **High** | `high` |
| **Extra High** | `xhigh` |
| **Ultra** | `ultra` |

For sequential phases, use the project's local environment and permit only one writing thread at a time. Use a worktree when the user requests isolation or independent phases can run safely in parallel. Do not include the current checkout's uncommitted changes in a new worktree unless the user explicitly asks for a working-tree starting state. A worktree route must include an integration gate before release.

Do not archive child threads automatically. They are user-owned records of the relay. Include their thread IDs in the final report.

## Run the relay

Send each child thread a self-contained brief containing:

```markdown
Role: <phase role, not merely the model name>
Outcome: <one concrete result>
Inputs: <paths, commits, URLs, evidence, and prior artifacts>
Constraints: <scope, invariants, authority, and forbidden actions>
Acceptance: <checks that prove this phase is done>
Return: <artifact or concise handoff, including unresolved risks>
```

Give children only the context needed for their phase. Point to canonical files and evidence instead of pasting a long conversation. Require evidence for completion: a written plan with decisions, a patch plus tests, a review with line-level findings, or a deployment receipt plus health result.

After each handoff, the coordinator must:

1. Check the deliverable against its gate.
2. Update the canonical route with the child thread ID, actual model, effort, artifact, and status.
3. Retry at the same tier only when the failure was transient or the brief lacked concrete evidence.
4. Escalate when the failure reveals a capability gap or new uncertainty: Luna to Terra, Terra to Sol, or the current model to the next effort tier.
5. Re-plan with Sol when new evidence invalidates the route rather than piling patches onto a broken plan.

Do not pass a summary forward as if it were the artifact. The next child must receive the actual plan, diff, test output, commit, or production evidence.

## Implementation and review gates

- A plan passes only when it identifies affected surfaces, decisions, risks, acceptance checks, and a safe integration path.
- Implementation passes only when the requested behavior exists, focused tests pass, and unrelated user changes remain untouched.
- Review passes only when every actionable finding is either fixed and rechecked or explicitly rejected with evidence.
- Parallelize independent implementation slices only when they have disjoint ownership or a declared integration seam. The coordinator integrates and tests the combined result before release.

## Availability and substitution

- If Luna is unavailable, Terra may substitute. If Terra is unavailable, Sol may substitute. Do not substitute downward.
- If Sol is unavailable for a phase that genuinely requires Sol, stop and report the blocked phase; do not disguise a weaker plan as equivalent.
- Use only the effort levels listed for the selected model. Move a Luna phase to Terra or Sol when it requires Ultra.
- Do not substitute the hidden `max` effort for the user-facing Ultra route; this relay uses the five effort levels listed above.
- If cost, latency, quota, or policy prevents an upward substitution, stop at the affected gate and return the completed artifacts.

## Final report

Lead with the finished outcome. Then report:

- the route actually used, including thread ID, model, and effort per phase;
- artifacts produced and checks passed;
- substitutions, escalations, or skipped phases and why;
- deployment revision and health evidence, when deployment was authorized;
- any remaining blocker or risk.

Never report the planned route as the route actually run.
