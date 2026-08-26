---
name: setup-pstack
description: Configure pstack's provider-qualified models, per-family requested effort, and parent-owned routes per role. Verifies native and external Claude, Codex, and Grok lanes before writing the override sheet. Use for /setup-pstack, "configure pstack models", or changing pstack's model choices.
---

# Setup pstack

Configure one portable model sheet for the current parent harness. Read [`provider-dispatch.md`](../poteto-mode/references/provider-dispatch.md) before probing or writing anything. Its model matrix, descriptor grammar, and route table are the contract. Choose one requested effort per matrix family. Do not add a second configuration file, a runtime resolver, or a weaker-model fallback.

Claude Code writes `~/.claude/pstack-models.md` and loads it from `~/.claude/CLAUDE.md` with:

```text
@~/.claude/pstack-models.md
```

Codex writes `~/.codex/pstack-models.md`. Codex has no `@` include, so add the sheet's routing block to `~/.codex/AGENTS.md` and retain the sheet as the editable source of truth.

## Steps

### 1. Establish the parent

Use the harness and tool surface running this skill: Claude Code or Codex. Environment markers may corroborate that top-level answer, but do not launch a child and ask it to detect where it came from. Record the parent because the same descriptor takes a different route in each harness.

### 2. Load current state

Read the current parent-specific sheet when it exists. Treat its values as current role-to-family assignments. A bare host-native slug from an older sheet is invalid here because it does not say which provider owns it. If the sheet is missing, the current assignments are the first-run role rows in step 7 and the current efforts are the matrix Default effort cells: Fable `max`, Sol `max`, Grok `xhigh`, Opus `xhigh`.

### 3. Parse per-family efforts

Read the model matrix. Map every non-alias descriptor on the sheet to a matrix family by `(provider, model)` and collect its `@effort`. `inherit-parent` and `auto` rows carry no family effort. One distinct effort per family is the current value.

If one family has mixed efforts, stop. Show the conflicting role rows verbatim and ask for one normalized effort from that family's selectable cell. Do not invent a precedence rule. Do not probe or write while the mix is unresolved. After the operator picks, that value is the family's current effort.

### 4. Collect one requested effort per family

Ask exactly four effort questions, one each for Fable, Sol, Grok, and Opus. The domain is that family's selectable efforts. The keep-current default is the current value from step 3. Empty input keeps current.

On a first run the four current values are the matrix defaults. Name them:

> Use Fable 5 at max, GPT-5.6 Sol at max, Grok 4.6 at xhigh, and Opus 5 at xhigh for the multi-model panels, with the upstream single-role assignments below?

On a rerun, name the four current values the same way. Do not offer to reset customized role lanes to the first-run assignments.

### 5. Probe the four requested pairs

Probe only the four selected `provider:model@effort` pairs. Do not enumerate or offer older models as substitutes. A failed probe writes nothing: the active sheet and the mirrored Codex routing block keep their current bytes, and a failed first run creates neither artifact.

| Provider | Claude parent route | Codex parent route | Availability proof |
|---|---|---|---|
| `claude` | native Agent `pstack-<stem>-<effort>` | Claude CLI | native schema/one-turn probe or `claude auth status --json` plus one-turn probe |
| `codex` | `codex exec` | native `spawn_agent` | `codex login status` plus one-turn probe or native schema/one-turn probe |
| `grok` | Grok CLI | Grok CLI | `grok models` must list the requested model; one-turn probe |

Use a tiny read-only probe that returns a unique marker. A login-status command alone proves credentials, not that the requested model and effort flags run. Record native and external results separately. Never call the external launcher for the parent's own provider. On a Claude parent, the Fable and Opus probes are one-turn runs of the mapped `pstack-<stem>-<effort>` agent. On a Codex parent, the Sol probe is native `spawn_agent` with the selected `reasoning_effort`. Every other pair uses the external runner with the selected effort flag.

Receipts and native transcripts prove the requested effort and the route. They do not prove a provider's hidden applied reasoning depth. There is no implicit timeout, weaker-model fallback, same-provider external fallback, or second mutable configuration source.

### 6. Render, preserving role families

Build the new sheet in memory. Do not write it yet.

- First run: start from the documented role assignments in step 7.
- Rerun: start from the loaded sheet's role names, lane order, and family (or alias) per lane.

After effort selection, ask whether to keep those role-to-family assignments or change named roles. Keeping them is the default. Apply only role changes the operator names; never offer a reset of a customized sheet to the first-run assignments. A changed role may use one of the four probed matrix families, `inherit-parent`, or `auto`.

Rewrite every matrix-family descriptor to `provider:model@<requested effort for that family>`. Leave `inherit-parent` and `auto` unchanged. An effort-only rerun cannot change a role's family. Changing Grok's effort updates every Grok occurrence and does not move a Sol role onto Grok. Refuse an unqualified slug, an unavailable route, a model other than the four matrix families, or a provider/model mismatch.

### 7. Confirm and commit

Show the route table for this parent, then show every rendered role and descriptor. Ask for confirmation before writing.

Why and Reflect require the parent's live MCP surface. Keep their investigator, reviewer, and synthesizer roles on `inherit-parent` or `auto`; the bounded external runner deliberately omits ambient MCPs. `inherit-parent` and `auto` always validate, but say when they reduce a panel's provider diversity. For panel roles, one lane runs per entry. The list length is the fan-out count. `arena cross-judge pool` is a list from which Arena chooses a provider different from the parent and base candidate when possible. `swarm workers` is the default for every worker unless a race explicitly assigns another descriptor.

Every non-alias value must match `<provider>:<model>@<effort>` and must have passed step 5.

After the operator confirms, overwrite the whole parent-specific sheet so reruns are idempotent. The first-run sheet, when the operator keeps the matrix defaults, is:

```markdown
# pstack model configuration

Provider-qualified per-role choices. Read the installed pstack provider-dispatch reference before dispatching a configured role. Delete a line to use the skill default. `inherit-parent` and `auto` use the parent model natively and still count as one panel lane.

feature, refactoring: grok:grok-4.6@xhigh
bug-fix: codex:gpt-5.6-sol@max
perf-issue: codex:gpt-5.6-sol@max
hillclimb: codex:gpt-5.6-sol@max
judgment and prose: claude:claude-fable-5@max
hardest tasks: claude:claude-fable-5@max
how explorer: grok:grok-4.6@xhigh
how explainer: claude:claude-fable-5@max
how critics: claude:claude-fable-5@max, codex:gpt-5.6-sol@max, grok:grok-4.6@xhigh, claude:claude-opus-5@xhigh
why investigators, synthesizer: inherit-parent
reflect tooling, judgment, divergent, synthesizer: inherit-parent
arena runners: claude:claude-fable-5@max, codex:gpt-5.6-sol@max, grok:grok-4.6@xhigh, claude:claude-opus-5@xhigh
arena cross-judge pool: claude:claude-fable-5@max, codex:gpt-5.6-sol@max, grok:grok-4.6@xhigh, claude:claude-opus-5@xhigh
swarm workers: grok:grok-4.6@xhigh
architect runners: claude:claude-fable-5@max, codex:gpt-5.6-sol@max, grok:grok-4.6@xhigh, claude:claude-opus-5@xhigh
interrogate reviewers: claude:claude-fable-5@max, codex:gpt-5.6-sol@max, grok:grok-4.6@xhigh, claude:claude-opus-5@xhigh
```

### 8. Wire it in

On Claude, add the `@~/.claude/pstack-models.md` include only if absent. On Codex, update the existing pstack routing block in `~/.codex/AGENTS.md` rather than appending duplicates. Do not copy the model sheet between harnesses without rerunning the parent-specific probes; route availability can differ even on the same host.

### 9. Behavioral smoke

Before declaring setup complete, run one small read-only mixed panel from this parent: all four chosen descriptors, distinct output/receipt paths, and an independent cross-judge. Launch Claude-native agents and every external process in the background with retained handles, then drain them. Verify the native transcript entries and every external receipt. A structural config check or unit test is not a substitute.

Report the sheet path, parent route table, requested-effort probe results, smoke results, and external elapsed/token/cost receipts. Re-running this skill re-probes and updates the same sheet. Do not claim the provider exposed hidden applied-effort observability.
