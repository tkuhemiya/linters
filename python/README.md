# Python

Ruff lints and formats. ty type-checks. Both are already installed with uv.

These two files are the fallback for a directory that has no setup of its own. Ruff keeps its defaults and adds the rules in `ruff.toml`. ty keeps its defaults and enables `missing-type-argument`.

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

Ruff's defaults stay in force: `E4`, `E7`, `E9`, and `F`. Format is Black-compatible at 88 columns. `target-version` is `py310` unless the project's `requires-python` says otherwise. `UP` follows that version, so it will not suggest syntax from a newer Python than the project declares.
