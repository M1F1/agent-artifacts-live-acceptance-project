# AART live acceptance project

A consumer repository, not a registry. Everything under `.claude/`, `.mcp.json`, `CLAUDE.md`, and
`.agent-artifacts/` was installed by [AART](https://github.com/M1F1/agent-artifacts) `2.0.0` from
two independently published registries, and is committed exactly as AART wrote it.

The point of the repository is that the installed state is verifiable: CI reinstalls nothing, it
reconciles what is committed here against what the registries currently publish.

## Sources

| Alias | Registry | Repository |
| --- | --- | --- |
| `registry-a` | `la-registry-a` | [M1F1/agent-artifacts-registry](https://github.com/M1F1/agent-artifacts-registry) |
| `registry-b` | `la-registry-b` | [M1F1/agent-artifacts-registry-2](https://github.com/M1F1/agent-artifacts-registry-2) |

Source configuration is user-global, not part of a project, so it is not committed. The workflow in
`.github/workflows/consumer-acceptance.yml` re-adds both sources from scratch on every run, which is
also the exact sequence a new contributor runs locally.

## What is installed

Eleven installations across both registries and all five artifact kinds, all at project scope for
the `claude` profile in copy mode:

| Coordinate | Kind | Lands in |
| --- | --- | --- |
| `registry-a/skill/systematic-debugging` | skill | `.claude/skills/systematic-debugging/` |
| `registry-a/skill/test-driven-development` | skill | `.claude/skills/test-driven-development/` |
| `registry-a/skill/verification-before-completion` | skill | `.claude/skills/verification-before-completion/` |
| `registry-a/guideline/la-house-style` | guideline | `.claude/guidelines/la-house-style.md` |
| `registry-a/hook/la-guard` | hook | `.claude/hooks/la-guard/` + `.claude/settings.json` |
| `registry-a/memory/la-team-memory` | memory | `CLAUDE.md` (sentinel-delimited block) |
| `registry-a/mcp/context7` | mcp | `.mcp.json` (`mcpServers.context7`) |
| `registry-b/skill/wayfinder` | skill | `.claude/skills/wayfinder/` |
| `registry-b/skill/residual-03-stressors` | skill | `.claude/skills/residual-03-stressors/` |
| `registry-b/skill/using-residues` | skill | `.claude/skills/using-residues/` |
| `registry-b/guideline/residuality-theory` | guideline | `.claude/guidelines/residuality-theory.md` |

Ten of those were requested. `registry-b/skill/using-residues` was not: every `residual-*` stage
skill declares it as a required artifact, and AART resolves that closure before planning any effect.
Installing a stage without its kernel is no longer possible.

Nothing here carries a credential. `.mcp.json` references `${CONTEXT7_API_KEY}` by name only, and
the MCP artifacts that need a secret keep it in a separate, reviewed setup step that this repository
does not run.

## Reproducing the installation

```sh
python3 -m pip install --no-deps \
  https://github.com/M1F1/agent-artifacts/releases/download/v2.0.0/agent_artifacts-2.0.0-py3-none-any.whl

aart source add --alias registry-a --kind registry-git \
  --location https://github.com/M1F1/agent-artifacts-registry.git --ref main --default
aart source add --alias registry-b --kind registry-git \
  --location https://github.com/M1F1/agent-artifacts-registry-2.git --ref main --no-default

aart marketplace install \
  registry-a/skill/systematic-debugging registry-a/skill/test-driven-development \
  registry-a/skill/verification-before-completion registry-a/guideline/la-house-style \
  registry-a/hook/la-guard registry-a/memory/la-team-memory registry-a/mcp/context7 \
  registry-b/skill/residual-03-stressors registry-b/skill/wayfinder \
  registry-b/guideline/residuality-theory \
  --profile claude --scope project --mode copy
```

The command above only reviews. Re-run it with `--yes` to apply the exact reviewed plan.

## Verifying what is committed

```sh
aart marketplace status --profile claude --scope project
```

`current` on every line means the committed tree still matches the artifacts the registries publish.
`drifted` means a file here was edited by hand; `outdated` means a registry moved ahead, and
`aart marketplace update` (review first, then `--yes`) reconciles it.

## Version window

Both registries require AART `>=2.0.0,<3.0.0`. Registry A publishes setup-v2 recipes with a
package-root `SETUP.md`, and Registry B declares artifact dependencies — no released `1.x` executable
can read either, and both registries say so rather than letting an old client fail late.
