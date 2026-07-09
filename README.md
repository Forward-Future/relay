# GPT-5.6 Relay

A public Codex skill for routing one task through model-specific GPT-5.6 child threads.

The relay gives each phase to the model and reasoning effort that fit it best, while preserving visible thread history and concrete handoffs:

```text
Sol coordinator
    -> Terra implementation
    -> Sol review
    -> Luna release
```

The route adapts to the task. Small mechanical work may use only Luna. Ambiguous architecture may stay with Sol longer. Deployment is always a separate, authorized Luna phase.

## Install

Copy `.agents/skills/gpt-5-6-relay/` into a project that supports Codex project skills. Then invoke it explicitly:

```text
Invoke /gpt-5-6-relay with this task: <task>
```

The skill uses Codex app thread tools to create visible child threads with explicit model and reasoning settings. It does not use hidden subprocesses.

## Model routing

| Model | Best fit | Relay starting effort |
| --- | --- | --- |
| Sol | Planning, architecture, ambiguity, hard diagnosis | Extra High |
| Terra | Implementation, tests, bounded debugging | High |
| Luna | Reconnaissance, mechanical checks, release execution | Light or Medium |

The user-facing effort levels are Light, Medium, High, Extra High, and Ultra. Sol and Terra support all five; Luna supports all except Ultra. Ultra enables automatic delegation on Sol and Terra, so the relay normally stops at Extra High and uses Ultra only when nested delegation is intentional.

## What the skill guarantees

- Each child thread has one role, one deliverable, and one checkable gate.
- A dependent child is not started until its required artifact exists.
- Corrections stay in the same child thread; changing model responsibility creates a new child.
- Only one thread writes to a checkout at a time.
- Deployment follows the repository's clean integrated-main rules in `DEPLOYMENT.md`.
- The final report names the actual thread IDs, models, efforts, artifacts, checks, and deployment evidence.

## Requirements

The host must expose Codex app tools for listing projects, creating threads, reading threads, and sending background messages with model and effort controls. If those controls are unavailable, the skill reports the proposed route instead of pretending that model routing happened.

## License

MIT. See [LICENSE](LICENSE).
