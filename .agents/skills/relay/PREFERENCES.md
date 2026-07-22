# Saved model preferences

Relay supports explicit model choices for one run plus optional project and user preferences for future runs.

## Precedence

Resolve each phase independently:

1. an explicit selector or effort supplied for the current run;
2. a valid saved preference from `.relay/models.yaml` at the project root;
3. a valid saved user preference;
4. Relay's evidence-backed automatic suggestion.

A higher-precedence choice does not bypass availability, tier, effort, tool-access, or reviewer-independence gates.

## File format

Use this schema and omit phases the user has not configured:

```yaml
version: 1
phases:
  planning:
    model: "frontier-planner-id"
    effort: "high"
    on_unavailable: "suggest"
  execution:
    model: "workhorse-executor-id"
    effort: "medium"
    on_unavailable: "suggest"
  review:
    model: "independent-frontier-reviewer-id"
    effort: "high"
    on_unavailable: "stop"
```

`model` is an exact ID or documented native selector. `effort` is optional. `on_unavailable` is `suggest` or `stop`; default to `suggest` when omitted. Store no credentials, provider keys, account identifiers, or other secrets.

## Saving

Persist preferences only when the user explicitly asks to save, remember, set a default, or update a preference. A model named for the current run is not permission to write configuration.

1. Resolve the requested scope. Default to `project` when the user asks to save without naming a scope and a project root is available; state that choice. Use:
   - `project`: `.relay/models.yaml` at the Git repository root or current workspace root;
   - `user`: `$RELAY_CONFIG_HOME/models.yaml` when set; otherwise `$XDG_CONFIG_HOME/relay/models.yaml` on Unix, `~/.config/relay/models.yaml` when `XDG_CONFIG_HOME` is unset, or `%APPDATA%\relay\models.yaml` on Windows.
2. Read the existing file at the resolved path before editing it.
3. Validate the requested phase names and values. The only phase keys are `planning`, `execution`, and `review`; the only setting keys are `model`, `effort`, and `on_unavailable`.
4. Create the parent directory when needed and patch only the requested settings. Preserve other phases and unrelated valid settings.
5. Parse the final YAML, show the saved path and resulting phase map, and state that availability will be revalidated on every run.

When the user asks to clear a preference, remove only that phase from the requested scope. Remove the file only when no phases remain, and report that removal.

## Applying preferences

Load user preferences first and overlay project preferences, then apply one-run choices. Match each resulting selector to the current inventory:

- Apply it when all phase gates pass and identify it as `saved preference` in the route and delegation announcement.
- With `on_unavailable: suggest`, explain why it failed and use the best valid automatic suggestion.
- With `on_unavailable: stop`, explain why it failed and stop before that phase.
- Never silently rewrite a saved preference because a model is temporarily unavailable.

One-run overrides do not modify either saved file unless the user separately asks to save them. Never copy project preferences into user scope, or user preferences into project scope, without an explicit request.
