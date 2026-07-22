# Coding environment adapters

Select one adapter during preflight. Product names are routing hints; the native tools visible in the current session are authoritative.

First record this capability matrix:

```markdown
| Capability | Native control | Evidence |
| --- | --- | --- |
| Discover child-selectable models | ... | ... |
| Select model and effort | ... | ... |
| Verify actual model and effort | ... | ... |
| Inspect terminal result | ... | ... |
| Continue or correct the same child | ... | ... |
```

A full relay requires every row. If continuation is unavailable, corrections may use a new child only when it receives the actual prior artifact and the final report discloses the lost context. Model verification must come from resolved metadata or an authoritative native contract that guarantees an accepted exact selector is used and rejects unavailable values. Aliases and fallback-capable selectors always require resolved-model readback.

## Codex

- Install project skills at `.agents/skills/relay/`; invoke explicitly as `$relay` or by naming the skill in the prompt.
- Discover child models from the callable spawn/thread tool schema and any native model-list capability. Treat `config.toml` agent defaults as candidates, not entitlement proof.
- Use Codex's native agent or task controls. Explicit spawn values override configured child defaults; omitted values may inherit from the parent.
- Verify the actual model and effort from returned child/task metadata when available. Otherwise an accepted exact ID may be verified by an authoritative spawn contract only when it guarantees that exact selection and rejects unsupported values; record that contract as evidence.
- `agents/openai.yaml` disables implicit invocation for Codex; do not remove it while this skill treats explicit invocation as delegation authority.

## Cursor

- Install the canonical implementation at `.agents/skills/relay/`; invoke explicitly as `/relay`. Automatic skill loading is not authority to create child work.
- The main model picker proves main-session availability, not child selectability. Prefer the native subagent/task schema and subagent settings for the current IDE or CLI version.
- Use Cursor's native subagents. Only selectors accepted by the current child tool count; do not assume every chat model can be assigned to a child.
- Verify the actual child model through native task metadata or usage detail. If Cursor exposes only a generic selector such as `auto`, `fast`, or `inherit`, record that selector and the resolved model separately.
- When model selection or verification is unavailable, report Cursor as a single/unverified-model environment and do not promise the three-model route.

## Claude Code

- Keep the canonical implementation at `.agents/skills/relay/` and install the host entrypoint at `.claude/skills/relay/`; invoke explicitly as `/relay`. Claude Code does not discover project skills from `.agents/skills` directly.
- Discover candidates from the Agent tool's current model parameter, the `/model` choices when supplied by the user, and organization `availableModels` policy when readable.
- Use Claude Code's native Agent/subagent control. Model resolution can be affected by environment variables, per-invocation selection, agent frontmatter, and parent inheritance; configuration is not proof of the model ultimately used.
- Verify the resolved model after creation. If an excluded selector falls back to the inherited model or actual model metadata is unavailable, stop or reroute and disclose the substitution.
- Keep `disable-model-invocation: true` in the Claude Code entrypoint; it makes `/relay` explicit rather than model-triggered without adding unsupported metadata to the Codex skill.

## Other environments

Use the generic adapter only when the environment exposes native child work with the five capabilities in the matrix. Infer no model IDs from brand names. If it cannot enumerate models, accept a user-supplied native list; if it cannot select and verify the actual child model, explain that it cannot run a verified relay.
