# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

This is a **Claude Code plugin marketplace**, not a Python application. Despite the name, there is no Python code to build or run here. The deliverable is a *skill* — a Markdown instruction file that teaches Claude how to scaffold new Python projects. Editing this repo means editing prose (instructions Claude follows), not application logic.

## Layout

Three nested manifests, each a layer of the plugin system:

- `.claude-plugin/marketplace.json` — the marketplace catalog. Lists plugins and points `source` at each plugin directory.
- `plugins/bootstrap-python-repo/.claude-plugin/plugin.json` — the plugin manifest (name, description, author, and — when published — `version`).
- `plugins/bootstrap-python-repo/skills/bootstrap-python-repo/SKILL.md` — **the actual product.** Everything else is packaging around this one file.

`evals/evals.json` holds eval prompts that exercise the skill end-to-end (e.g. hyphenated-name → underscore-package mapping, no-name placeholder flow). Treat these as the regression suite: when you change `SKILL.md` behavior, check the `expected_output` of each eval still holds, and add an eval when you add a behavior.

## The name/description appears in three places — keep them in sync

The plugin name and one-line description are duplicated across `marketplace.json`, `plugin.json`, and the `SKILL.md` frontmatter. Change one, change all three.

The `description:` in `SKILL.md`'s YAML frontmatter is **not cosmetic** — it is the trigger phrase Claude matches against to decide whether to activate the skill. Editing it changes *when the skill fires*, so treat wording there as behavior, not documentation.

## Versioning (required for a distributed marketplace)

Right now no `version` field is set anywhere, so Claude Code falls back to the **git commit SHA** as the version — every commit is treated as a new release, and installed users update on each push. That is fine for active development.

For a stable, distributed plugin, set an explicit semantic `version` in `plugin.json` (it takes precedence over any version in the marketplace entry). **Once you adopt explicit versioning, you must bump `version` on every change to `SKILL.md` (or any plugin file) — otherwise `/plugin update` reports "already at the latest version" and users never receive your change.** This is the single most important habit for maintaining a published skill: a content change without a version bump is invisible to installed users.

Validate manifests before publishing:

```bash
claude plugin validate ./plugins/bootstrap-python-repo --strict
```

Note: `claude plugin validate` reports unrecognized/misspelled fields as warnings, not errors — read its output, don't just check the exit code.

## Editing the skill

`SKILL.md` is the file you'll almost always be changing. Two things make it correct:

- The bootstrap steps it describes must actually produce a working project. The authoritative check is that a scaffolded project passes `uv run ruff check .` and `uv run pytest` — this is asserted in every eval. If you change the generated `pyproject.toml`, `.gitignore`, or git hooks, mentally (or actually) run a scaffold through to confirm it still passes.
- The skill embeds a *template* `CLAUDE.md` (for the projects it generates). Don't confuse it with this file: this CLAUDE.md governs *this* repo; the one inside `SKILL.md` is emitted into the user's new project. Conventions baked into that template (conventional commits, no AI attribution, type hints required, no `print()` in production) are deliberate — preserve them unless changing them on purpose.

## Commits

This repo follows the same conventions it teaches: **conventional commits** (`feat`, `fix`, `docs`, `chore`, `refactor`, etc.) and **no AI attribution** — no `Co-Authored-By` trailers, commit under the repository's own git identity.
