# Linters

Extras for Ruff and Oxlint. Each tool keeps its own defaults. A project adds one path.

## Python — Ruff

In the project's `pyproject.toml` or `ruff.toml`:

```toml
[tool.ruff]
extend = "/Users/themiya/Documents/Dev/Github/linters/python/ruff.toml"
```

`ruff.toml` without the `[tool.ruff]` table:

```toml
extend = "/Users/themiya/Documents/Dev/Github/linters/python/ruff.toml"
```

ty has no `extend`. Copy the one rule, or link `python/ty.toml` as the user config (see `python/README.md`).

## TypeScript — Oxlint

In the project's `.oxlintrc.json`:

```json
{
  "extends": ["/Users/themiya/Documents/Dev/Github/linters/typescript/oxlintrc.json"],
  "options": { "typeAware": true }
}
```

Oxfmt has no `extends`:

```sh
oxfmt -c /Users/themiya/Documents/Dev/Github/linters/typescript/oxfmtrc.json --check .
```

Oxlint type-aware rules need `oxlint-tsgolint` installed next to `oxlint`.

A nearest project config still wins if it does not `extend` these files.

Personal fallback (no project file): `python/README.md`, `typescript/README.md`.
