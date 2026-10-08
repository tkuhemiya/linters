# Python

Ruff lints and formats. ty type-checks. Both are already installed with uv.

These two files are extras on top of each tool's defaults. Ruff 0.16 already turns on 413 rules; `ruff.toml` adds a short list. ty keeps its defaults and enables `missing-type-argument`.

## Add to a project

Ruff does not merge a nearest `ruff.toml` with this file. The project has to `extend` it:

```toml
[tool.ruff]
extend = "/Users/themiya/Documents/Dev/Github/linters/python/ruff.toml"
```

ty has no `extend`. Put the rule in the project file, or use the user-config link below (project settings still win):

```toml
[tool.ty.rules]
missing-type-argument = "error"
```

## Personal fallback (no project config)

Link them once:

```sh
ln -s /Users/themiya/Documents/Dev/Github/linters/python/ruff.toml ~/.config/ruff/ruff.toml
ln -s /Users/themiya/Documents/Dev/Github/linters/python/ty.toml ~/.config/ty/ty.toml
```

Then, in any directory:

```sh
ruff check .
ruff format .
ty check
```

`ruff format` writes. `ruff format --check` only reports.

If the directory (or a parent) has its own config, that config is what runs.

Ruff looks for `ruff.toml`, `.ruff.toml`, or a `[tool.ruff]` table. The nearest one replaces this file entirely. A project that only sets a line length does not keep rules added here later. To keep them, that project can set `extend = "~/.config/ruff/ruff.toml"`.

ty looks for `ty.toml`, or a `[tool.ty]` table. A `pyproject.toml` with no `[tool.ty]` table is skipped. Project settings merge over this file, and the project value wins.

Ruff 0.16 defaults stay in force (bugbear, pyupgrade, isort `I001`, and many others; not the old `E4`/`E7`/`E9`/`F` set). Format is Black-compatible at 88 columns, 2026 style guide. `target-version` is `py310` unless the project's `requires-python` says otherwise. `UP` follows that version, so it will not suggest syntax from a newer Python than the project declares.

How Oxlint ideas map onto this fallback: `oxlint-compat.md`.
