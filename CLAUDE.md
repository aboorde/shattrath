# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

A public knowledge base of tips and tricks across various topics, shared with others. It is **content, not software**: Markdown only, nothing to build, run, or test. There is no package manager, no test suite, and no lint config — do not invent build/test commands or scaffold tooling.

## Structure convention

Flat, one folder per topic at the repo root. A topic folder is the unit of organization — do **not** nest sub-folders inside a topic. Each topic folder is self-contained and follows the pattern established by `cowork/`:

- A `README.md` that is the entry point for the topic. It states the intended audience, a "how it was made / methodology" note, a table mapping each doc to what it covers and why you'd read it, and a short distilled mental model.
- A set of focused `.md` docs, one per sub-area, plus optionally a one-page `cheatsheet.md` distillation.

When adding a new topic, create a new root-level folder following this same README + topic-docs shape rather than altering the existing layout.

## Documentation conventions (load-bearing — match these when editing or adding docs)

The existing `cowork/` docs hold to a deliberate, rigorous style. Preserve it:

- **Every non-obvious claim carries an inline source URL.** Prefer primary/official sources; distinguish them from community/press.
- **Each doc ends with two sections:** a numbered `## Sources` list, and a `## Confidence & gaps` section that separates verified-official facts from inference and explicitly flags what may be stale.
- **Dates are absolute** (e.g., `2026-05-18`), never relative ("last month", "recently"). Topics covering fast-moving subjects carry a researched-on date and a re-verification caveat.
- Tone is technically dense and skimmable: tables, tight bullets, an upfront mental model. Don't pad with generic advice.

For fast-moving / externally-sourced topics, treat freshness as a feature: when updating, re-verify sources rather than silently trusting prior text, and update that doc's `## Confidence & gaps`.

## Working here

- Edits are almost always to Markdown prose. Keep changes scoped to the relevant topic folder.
- Commit when asked; the owner controls pushes to the public remote.
