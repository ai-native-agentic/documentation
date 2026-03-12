# Harness Integration Rollout

This document tracks rollout of harness-engineering 6-gate QA across Python projects in this ecosystem.

## Enforcement policy

- One entry command per repository: `./.harness/run-gates.sh`
- Gate scripts live in `<repo>/.harness/`
- CI enforcement via `<repo>/.github/workflows/harness-gates.yml`
- Local pre-commit enforcement via `<repo>/.git/hooks/pre-commit`

## Timeline

1. Optional (now)
   - Gates integrated and runnable locally
   - Gate D/E/F placeholders kept disabled until baselines exist
2. Recommended (next)
   - Gate A + Gate C required for all PRs
   - Gate B required for repositories with stable layer boundaries
3. Mandatory (target)
   - Gate A/B/C required for all configured repos
   - Gate D/E/F enabled per-project after deterministic baseline setup

## Project status

| Project | Gate A | Gate B | Gate C | Gate D/E/F | Notes |
|---|---|---|---|---|---|
| nanobot | enabled | enabled | enabled | disabled | Gate A/C currently failing due baseline debt |
| ClawWork | enabled | disabled | enabled | disabled | Gate A/C failing; Gate B deferred |
| lunark-ondevice-ai-lab | enabled | disabled | enabled | disabled | parser/env issues surfaced by gate run |
| ai-native-self-learning-agents | enabled | enabled | enabled | disabled | Gate A/C parse + style debt; Gate B passed |
| lunark-chatbot-lab | enabled | disabled | enabled | disabled | Gate A/C failing; import cycle detected |
| trpg-chatbot-lab | enabled | enabled | enabled | disabled | local Python version mismatch for parsing |
| openmanus | enabled | enabled | enabled | disabled | parse issue blocks B/C |
| ai-native-agentic-economic-sdk | enabled | disabled | disabled | disabled | Gate A(✅ black+ruff+mypy), Gate E(✅ pytest 12 tests), B/C/D/F disabled |

## Failure examples and fixes

1. Formatting/lint/type failure (Gate A)
   - Symptom: black reports files would be reformatted
   - Fix: run formatter/linter in check-fix loop; then resolve type errors in narrow modules
2. Architecture boundary failure (Gate B)
   - Symptom: forbidden source -> target layer import is reported
   - Fix: extract shared contracts/utilities to neutral layer; remove direct cross-layer import
3. Structural ratchet failure (Gate C)
   - Symptom: function length/complexity threshold exceeded
   - Fix: split responsibilities and reduce branching
4. Parser/runtime mismatch
   - Symptom: AST parsing fails on newer syntax under older interpreter
   - Fix: run gates under repository-supported Python version in local and CI

## Non-Python Repos

| Repo | Language | Harness Status |
|------|----------|----------------|
| openclaw | TypeScript | `.harness/run-gates.sh` wraps pnpm format/lint/build/test |
| symphony/elixir | Elixir | `.harness/run-gates.sh` wraps mix format/credo/dialyzer/test |
| seedbot | Bash | No harness (single-file script) |

## Deliverables

- `.harness/` created in all 7 target Python repositories with Gate A-F scripts and `config.yaml`
- `HARNESS.md` and `HARNESS_REPORT.md` created in all 7 target repositories
- CI gate workflow added to each target repository
- Root integration policy created in this file
