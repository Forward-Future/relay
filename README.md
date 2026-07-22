# Relay

A model-agnostic coding-environment skill for discovering available models and routing one task through planning, execution, and independent review.

The relay first inventories the models that the active environment can actually delegate to, classifies them from current evidence, and then assigns each phase by capability and cost:

```text
frontier planner
    -> faster, cheaper executor
    -> separate frontier reviewer
```

For example: Fable plans, GPT-5.5 executes, and GPT-5.6 Sol reviews. Those names are configuration, not requirements. The route can use any models exposed by the host, and deployment is always a separate authorized phase.

## Install

Copy `.agents/skills/relay/` into a project that supports Codex project skills. Then invoke it explicitly:

```text
Invoke /relay with this task: <task>
```

The skill uses the coding environment's native visible delegation controls with explicit model and reasoning settings. It does not use hidden subprocesses or direct provider calls to manufacture unavailable capabilities.

## Discovery

`relay` inspects the active environment's child-task or agent-spawn capabilities for exact selectable model IDs. Environment-native model lists and validated configuration can add evidence. Public provider catalogs and remembered model names never prove availability for the current account or surface.

Each discovered model is classified as:

| Tier | Meaning |
| --- | --- |
| Frontier | Highest-capability available tier for difficult planning, reasoning, diagnosis, and review |
| Workhorse | Cost- or latency-balanced agentic coding tier for bounded implementation and tests |
| Utility | Fast narrow-task tier for mechanical, checkable work |
| Unknown | Selectable, but insufficient current evidence for automatic routing |

The skill shows the inventory, evidence, confidence, and suggested route before launching child work. Low-confidence classifications require confirmation.

## Model routing

| Role | Selection rule |
| --- | --- |
| Planning | A frontier model suited to ambiguity, architecture, and hard diagnosis |
| Execution | The cheapest, fastest available workhorse that can reliably implement the settled plan and pass its checks |
| Review | A fresh frontier model, preferably different from both executor and planner |

The user may provide all three choices. Otherwise the relay maps currently available model IDs and effort settings to these roles and discloses any substitution.

## What the skill guarantees

- Each child thread has one role, one deliverable, and one checkable gate.
- A dependent child is not started until its required artifact exists.
- Corrections stay in the same child thread; changing model responsibility creates a new child.
- Review happens in a fresh thread that did not produce the implementation.
- Only one thread writes to a checkout at a time.
- Deployment follows the repository's clean integrated-main rules in `DEPLOYMENT.md`.
- The final report names the discovered inventory, actual child IDs, models, efforts, artifacts, checks, and deployment evidence.

## Requirements

The host must expose native controls for creating, inspecting, and continuing child work with selectable models. If it cannot enumerate models, the user must supply exact IDs. If it cannot select a model for child work, the skill stops instead of pretending that model routing happened.

## License

MIT. See [LICENSE](LICENSE).
