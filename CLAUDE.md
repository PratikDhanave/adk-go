# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Start here

**[AGENTS.md](./AGENTS.md) is the primary source of truth** for project context,
commands (build/test/lint/tidy), the "definition of done", the repository layout,
and the framework's core idioms. Read it first. Human-oriented process (CLA, PR
requirements) lives in [CONTRIBUTING.md](./CONTRIBUTING.md).

This repo deliberately keeps one shared instruction file for all AI agents
(`CLAUDE.md`, `GEMINI.md` → `AGENTS.md`). Prefer editing `AGENTS.md` over growing
this file, so guidance doesn't drift across agents. The notes below are only the
things worth surfacing beyond what `AGENTS.md` already covers.

## This repo is now multi-module

The root module `google.golang.org/adk/v2` is no longer the only module — e.g.
`plugin/agentanalytics/` is its own module (`google.golang.org/adk/plugin/agentanalytics`),
and more may be added (policy: integrations with heavy/optional deps must be
separate modules — see CONTRIBUTING.md#multi-module-development).

Consequences that bite silently:
- `go build ./...` / `go test ./...` run from the root **only cover the root
  module** and skip nested modules. To build/test everything, either set up the
  workspace once (`go work init && go work use -r .`, then `go build work && go test work`)
  or `cd` into the submodule and run commands there. CI covers submodules
  automatically; local `./...` does not.
- `go.work` is local-only and gitignored — never commit it, and don't assume it
  exists when reasoning about how consumers fetch modules.
- Submodules are versioned/tagged independently with a path prefix
  (e.g. `plugin/agentanalytics/v0.1.0`), not the root's `vX.Y.Z`.

## Go version

`go.mod` (root and submodules) pins `go 1.26.5`. Use that toolchain; the "Go 1.25+"
phrasing in AGENTS.md is the lower bound, not the pinned version.

## Where complexity lives

`workflow/` is by far the largest and most intricate package (node/graph engine:
dynamic scheduler, join/branch nodes, concurrency limits, HITL, conformance-parity
tests against adk-python). Treat it as high-fan-in — prefer additive, backward-
compatible changes and lean on its extensive existing tests. `agent/`, `runner/`,
`model/`, `session/`, and `tool/` are the other high-fan-in packages AGENTS.md
flags for "ask first" changes.

## Two easy-to-miss testing facts

- Tests are **offline by default**: LLM traffic is replayed from `testdata/*.httprr`
  via `internal/httprr`. Never introduce live network/model calls in tests.
- CI runs tests with `-race -count=1 -shuffle=on`. Reproduce a flake locally with
  the same flags; a single `go test ./pkg -run TestName` won't surface ordering bugs.
