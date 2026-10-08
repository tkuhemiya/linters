# Oxlint ideas that apply to Python (2026)

Oxlint does not lint Python. This is an idea map: [oxc.rs/docs/guide/usage/linter](https://oxc.rs/docs/guide/usage/linter) → Ruff 0.16 and ty, against `python/ruff.toml` and `python/ty.toml`.

Ruff 0.16 (July 2026) changed the ground. Defaults went from 59 rules (`E4`, `E7`, `E9`, `F`) to 413. Most of the extras this repo added when the Python fallback was written are now on with no config.

## How many Oxlint rules are even about Python-shaped problems?

871 native Oxlint rules. Grouped by plugin:

| Oxlint plugin | Rules | Python idea? | Ruff / ty analog |
| --- | ---: | --- | --- |
| `eslint` | 187 | yes | Ruff `F`, `E`, `RUF`, `SIM`, `PIE` |
| `unicorn` | 138 | yes | Ruff `UP`, `FURB`, `C4` |
| `typescript` | 111 | yes (types) | ty (137 rules) |
| `import` | 33 | partial | Ruff `I`, `F401`, `PLW0406`. No cycle checker |
| `oxc` | 27 | yes | Ruff `B`, `RUF` |
| `jsdoc` | 23 | docs, not types | Ruff `D` (only `D419` is default) |
| `promise` | 16 | async | Ruff `ASYNC`, ty `unused-awaitable` |
| `node` | 11 | no | — |
| `react` + `react-perf` + `jsx-a11y` + `nextjs` + `vue` | 192 | no | — |
| `jest` + `vitest` | 133 | tests | Ruff `PT` (pytest), per-project |
| **total** | **871** | **~535 language / types / imports / async (61%)** | |
| | | **~192 JS frameworks (22%)** | skip |
| | | **~133 JS test runners (15%)** | pytest only |
| | | **~11 Node (1%)** | skip |

Features on that page, not rules:

| Oxlint feature | Python 2026 |
| --- | --- |
| Correctness-first defaults (111 rules) | Ruff 0.16 correctness/suspicious/complexity/performance defaults (413 rules) |
| Type-aware linting (`tsgolint`) | ty. Separate command, like this repo already does |
| `--type-check` compiler diagnostics | `ty check` is the type checker. Do not fold it into Ruff |
| Multi-file analysis / `import/no-cycle` | **Gap.** Ruff has no import-cycle rule. ty `missing-direct-dependency` is package deps, off by default, not a module-graph cycle check |
| Auto-fix | `ruff check --fix` |
| Formatter split | `ruff format` (Black-compatible, 2026 style guide) vs oxfmt |
| JS plugins | n/a. Ruff is all native |
| Ignore comments | `# noqa`, `# ruff: ignore`, `# ruff: disable` (0.15+), `# ty: ignore` |

## The TypeScript extra list, one by one

These are the extras proposed for Oxlint in `typescript/awesome-eslint-oxlint-report.md`.

| Oxlint extra | Valid in Python? | Ruff 0.16 / ty | In our fallback? |
| --- | --- | --- | --- |
| `options.typeAware` | yes | ty is the type-aware engine | already a separate command |
| plugin `import` (`no-cycle`) | idea yes | **no Ruff rule** | gap, leave it |
| `import/no-self-import` | yes | `PLW0406` **default** | covered |
| `import/no-duplicates` | yes | `I001` / `F401` **default** | covered |
| `import/no-absolute-path` | yes | `PTH124` **default** | covered |
| plugin `promise` | yes (asyncio) | `ASYNC*` **mostly default**; ty `unused-awaitable` **warn** | covered |
| `eslint/preserve-caught-error` | yes | `B904` **not default** | **keep** `B904` |
| `eslint/array-callback-return` | yes | `C417` **default** | drop from extras |
| `eslint/no-throw-literal` | yes | `B016` **default** (raise a literal) | covered |
| `eslint/no-promise-executor-return` | no | JS `new Promise` executor | — |
| `unicorn/prefer-includes` | yes | `SIM118` **default** (use `key in dict`) | covered |
| `unicorn/prefer-array-flat-map` | yes | `UP` / comprehensions `C417` **default** | keep `UP` for the rest of pyupgrade |
| `unicorn/prefer-at` | weak | negative index is already normal Python | — |
| `unicorn/prefer-string-replace-all` | yes | `str.replace` already replaces all | — |
| `unicorn/prefer-node-protocol` | no | `node:` specifiers | — |
| `unicorn/prefer-spread` | yes | `UP` unpacking, `C4` comprehensions **default** | keep `UP` |
| `typescript/no-misused-promises` | partial | ty `unused-awaitable` **warn**; `truthiness-test-of-callable` / `iterable` **warn**. No “promise used as bool” that matches TS exactly | already on as warn; do not promote in v1 |
| `typescript/only-throw-error` | yes | `B016` + `TRY002` **default** | covered |
| `typescript/return-await` | partial | `B904` (cause) and `TRY300` (return in try). `TRY300` **not default**, noisy | keep `B904`; skip `TRY300` |
| oxfmt `sortImports` | yes | `I001` **default** in Ruff 0.16; `ruff format` does not sort imports | keep group `I` so `I002` stays available |

Oxlint extras with a real Python counterpart: **13 of 19**. Already on in Ruff 0.16 or ty defaults: **9**. Still a gap we choose to fill: **`B904`**. Still a gap with no good rule: **import cycles**.

## What our Python extras should be now

`extend-select` adds to whatever Ruff’s defaults currently are. On 0.16 that is already 413 rules, including `B006`, `B008`, `BLE001`, `C417`, most of `UP`, and `I001`.

Keep:

| Extra | Why it is still extra |
| --- | --- |
| `B904` | raise-inside-except should name `from`. Direct match for `preserve-caught-error`. Still off by default |
| `ARG` | unused arguments. Direct match for unused-params. Still off by default |
| `UP` | the rest of pyupgrade beyond the default subset (`UP042`, `UP038`, …) |
| `I` | the rest of isort beyond `I001` |

Drop as redundant on 0.16: `B006`, `B008`, `BLE001`, `C417`.

ty stays as-is: defaults plus `missing-type-argument`. Do not turn on `possibly-unresolved-reference` (too many false positives; ty itself defaults it to ignore). Do not promote `unused-awaitable` from warn to error in this pass.

## ty off-by-default rules (16)

Only `missing-type-argument` is on in this repo. The others stay off:

`blanket-ignore-comment`, `disjoint-cast`, `division-by-zero`, `dynamic-function-decorator-return`, `missing-direct-dependency`, `missing-override-decorator`, `missing-type-argument` *(we enable)*, `possibly-missing-attribute`, `possibly-missing-import`, `possibly-unresolved-reference`, `redundant-condition-strict`, `truthiness-test-of-none-union`, `unsound-assignment`, `unsound-return-statement`, `unsound-yield`, `unsupported-dynamic-base`.

None of these is a clean match for a remaining Oxlint extra.

## Bottom line

About **three fifths** of Oxlint is language-level and has a Ruff or ty counterpart. The other **two fifths** is React/Vue/Next/a11y/Jest/Vitest/Node and does not belong in a Python fallback.

After Ruff 0.16, the Python fallback does not need to grow. It needs to shrink: keep `B904`, `ARG`, `UP`, `I`. The TypeScript Oxlint list does not justify new Ruff groups (`S` bandit, `D` pydocstyle, `ANN` annotations) or new ty rules.
