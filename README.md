# Relay

A model-agnostic coding-environment skill for discovering available models and routing one task through frontier planning, workhorse execution, and independent frontier review.

The relay first inventories the models that the active environment can actually delegate to, classifies them from current evidence, and then assigns each phase by capability and cost:

```text
frontier planner
    -> faster, cheaper executor
    -> separate frontier reviewer
```

For example: Fable plans, GPT-5.5 executes, and GPT-5.6 Sol reviews. Those names are configuration, not requirements. The route can use any models exposed by the host, and deployment is always a separate authorized phase.

## Install and invoke

| Environment | Project installation | Invocation |
| --- | --- | --- |
| Codex | `.agents/skills/relay/` | `$relay` |
| Cursor | `.agents/skills/relay/` | `/relay` |
| Claude Code | `.agents/skills/relay/` plus `.claude/skills/relay/` | `/relay` |

Keep the repository layout intact when installing. Codex and Cursor read the canonical skill directly; the Claude Code entrypoint loads that same implementation with host-compatible explicit-invocation metadata. Restart an environment if it does not detect the newly installed skill.

The skill uses the coding environment's native visible delegation controls with explicit model and reasoning settings. It does not use hidden subprocesses or direct provider calls to manufacture unavailable capabilities.

## Discovery

`relay` inspects native child-task capabilities, distinguishes parent availability from child selectability, and verifies the actual model assigned to each child. Public provider catalogs and remembered names never prove current availability. See [`ENVIRONMENTS.md`](.agents/skills/relay/ENVIRONMENTS.md) for adapters and [`MODEL_SELECTION.md`](.agents/skills/relay/MODEL_SELECTION.md) for tier evidence and routing gates.

## Requirements

The host must expose native controls for selecting and verifying child models. Verification can be returned model metadata or a native contract that guarantees an accepted exact ID is used and rejects unsupported values. Aliases and fallback-capable selectors require resolved-model readback. When the host cannot verify the actual model, Relay stops instead of pretending that model routing happened.

## License

MIT. See [LICENSE](LICENSE).
