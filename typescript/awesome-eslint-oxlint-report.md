# Awesome ESLint × Oxlint report (2026)

Source list: [dustinspecker/awesome-eslint](https://github.com/dustinspecker/awesome-eslint) (raw readme, 2026). Compatibility: [oxc.rs linter](https://oxc.rs/docs/guide/usage/linter), [built-in plugins](https://oxc.rs/docs/guide/usage/linter/plugins), [JS plugins](https://oxc.rs/docs/guide/usage/linter/js-plugins), [type-aware linting](https://oxc.rs/docs/guide/usage/linter/type-aware), plus the [plugin compatibility discussion](https://github.com/oxc-project/oxc/discussions/14862).

This is the TypeScript counterpart of `python/`. Python is Ruff (lint + format) and ty (types). The 2026 Oxc stack is the same split:

| Python | TypeScript |
| --- | --- |
| `ruff check` | `oxlint` |
| `ruff format` | `oxfmt` |
| `ty check` | `oxlint --type-aware --type-check` (needs `oxlint-tsgolint`) |

Python keeps tool defaults and adds a short extra list. Same bar here. Shareable ESLint presets (Airbnb, Standard, XO, Antfu, Sheriff, Hardcore) are not the model. They pull in style, formatting, and framework plugins that Oxlint either already covers natively, or that belong in oxfmt, or that are project-specific.

Oxlint has no `~/.config` fallback the way Ruff does. A nearest `.oxlintrc.json` replaces the defaults. Passing `-c` disables nested lookup. The personal fallback has to be invoked with `-c /path/to/linters/typescript/.oxlintrc.json` (and the same for oxfmt), or a tiny wrapper. A project file still wins if you run without `-c`.

## What Oxlint already is in 2026

- 871 native rules. 111 on by default, all in the `correctness` category.
- Default plugins: `eslint`, `typescript`, `unicorn`, `oxc`. Enabling `plugins` replaces that list; the array has to name everything you want.
- Optional native plugins: `import`, `jsdoc`, `jsx-a11y`, `react` (includes hooks, refresh, React Compiler), `react-perf`, `nextjs`, `node`, `promise`, `jest`, `vitest`, `vue` (script tags only).
- Type-aware mode is stable: 59 of 61 typescript-eslint type-aware rules via `tsgolint` / TypeScript 7. `no-floating-promises` and the other type-aware correctness rules turn on when `--type-aware` is set. JS plugins still cannot be type-aware.
- JS plugins (alpha, ESLint v9 API): most JS/TS plugins work. Official conformance: cypress, `@e18e/eslint-plugin`, mocha, playwright, react-hooks, regexp, sonarjs, storybook, `@stylistic`, testing-library.
- Not supported for JS plugins: custom parsers / file formats (Vue templates, Svelte, Angular templates, HTML, Markdown, JSON/YAML/TOML as languages), and type-aware plugin rules.
- Formatting rules moved out of ESLint. Oxfmt is the Prettier-compatible formatter, with import sort, Tailwind class sort, and `package.json` sort built in.

## What we should add

Keep Oxlint correctness defaults. Do not turn on whole categories (`suspicious`, `pedantic`, `style`). Pick extras the way `python/ruff.toml` picks `B904`, `ARG`, `UP`, `I` (Ruff 0.16 already turned on `B006`, `B008`, `BLE001`, `C417`). Python mapping: `python/oxlint-compat.md`.

### Always-on extras (native)

These are the `extend-select` list.

| Extra | Why (Python analog) |
| --- | --- |
| `options.typeAware: true` | ty: type-aware checks that `tsc` alone does not give you. Needs `oxlint-tsgolint`. |
| plugin `import` | ruff `I`, but for problems, not sorting: `no-cycle`, `no-self-import`, `no-duplicates`, `no-absolute-path`. |
| plugin `promise` | promise correctness only: `no-callback-in-promise`, `no-new-statics`, `valid-params`. |
| `eslint/preserve-caught-error` | ruff `B904`: rethrow should keep the cause. |
| `eslint/array-callback-return` | ruff `C417` family: map/filter/reduce callbacks that forget to return. |
| `eslint/no-throw-literal` | throw `Error`, not a string. |
| `eslint/no-promise-executor-return` | returning from a `new Promise` executor is almost always a bug. |
| `unicorn/prefer-includes` | ruff `UP`: `indexOf !== -1` → `includes`. |
| `unicorn/prefer-array-flat-map` | `UP`: `.map().flat()` → `flatMap`. |
| `unicorn/prefer-at` | `UP`: `arr[arr.length - 1]` → `arr.at(-1)`. |
| `unicorn/prefer-string-replace-all` | `UP`: `/g` replace → `replaceAll`. |
| `unicorn/prefer-node-protocol` | `UP`: `fs` → `node:fs`. |
| `unicorn/prefer-spread` | `UP`: `Array.from` / `concat` / `.apply` when spread is the modern form. |
| `typescript/no-misused-promises` | extra ty rule: promises in `if` / void slots. Pedantic but high-signal; needs type-aware. |
| `typescript/only-throw-error` | typed version of no-throw-literal. Needs type-aware. |
| `typescript/return-await` | `B904`-adjacent in async: lost `try/catch` if you `return promise` instead of `return await promise`. Needs type-aware. |

Import *sorting* is oxfmt, not oxlint: `{ "sortImports": true }` in `.oxfmtrc.json`. That is the ruff `I` analog.

Do **not** enable `options.typeCheck` in the fallback. Python keeps ty as its own command. Type-aware linting is extra rules; compiler diagnostics stay a separate `oxlint --type-check` (or `tsgo`) when you want them.

### Plugins to skip in the personal fallback

| Plugin | Reason |
| --- | --- |
| `react`, `react-perf`, `jsx-a11y`, `nextjs`, `vue` | framework. Enable in that repo. |
| `jest`, `vitest` | test runner. Enable in that repo. Oxlint has both natively. |
| `jsdoc` | TypeScript types already cover most of this. |
| `node` | native coverage is thin (11 rules, mostly style / old CJS). Useful Node checks (`prefer-node-protocol`, `no-path-concat`) are already unicorn/eslint. |

### JS plugins: none in v1

JS plugins are alpha, slower than native, and the good 2026 ones either overlap native unicorn/typescript or need types (which JS plugins cannot see). Revisit later:

| Plugin | Verdict later |
| --- | --- |
| `@e18e/eslint-plugin` | Official conformance. Modernization + `ban-dependencies`. Overlaps unicorn `prefer-*` and `eslint-plugin-depend`. Type-aware rules will not run. |
| `eslint-plugin-depend` | Same idea as e18e `ban-dependencies`. Community-tested. Add if we want dependency-bloat detection without the rest of e18e. |
| `eslint-plugin-regexp` | Official conformance. Native eslint already has `no-invalid-regexp`, `no-control-regex`, `no-useless-backreference`. Worth it only if we want the full regexp style/correctness set. |
| `eslint-plugin-sonarjs` | Official conformance. Huge overlap with `oxc` + eslint correctness. Noisy as a fallback. |
| `eslint-plugin-de-morgan` | Rewrites `!(a \|\| b)` to `!a && !b`. Nice, stylistic. |
| `eslint-plugin-no-secrets` | Credential scan. Useful, not a TS-syntax linter. |
| `eslint-plugin-playwright` / `cypress` / `testing-library` / `storybook` | Per-project. All conformance-tested. |
| `eslint-plugin-erasable-syntax-only` | 2026 TS 5.8 `erasableSyntaxOnly`. Only if we adopt that flag. |

Do not pull `@stylistic`. Formatting is oxfmt.

## Full catalog

Legend: **native** already in Oxlint; **js-ok** ESLint v9 plugin should load as `jsPlugins`; **js-typed** needs TypeScript types (will not work as a JS plugin); **parser** custom parser/processor (not supported); **oxfmt** formatting; **dead** unmaintained or ESLint 8-only; **n/a** not a lint rule set.

### Configs

Shareable configs are ESLint flat-config packages. Oxlint cannot `extends` them. Steal ideas, do not install them.

| Project | 2026 idea | Oxlint | Add? |
| --- | --- | --- | --- |
| Airbnb / Airbnb-babel / airbnb-extended | Strict React/JS style guide | Style + deprecated formatting. Native react/import cover the useful bits. | no |
| Alloy | React/Vue/TS preset | Framework-specific | per-project |
| ESLint's own config | Dogfood | Already in default eslint plugin | no |
| Facebook / Feedzai / Shopify / Wikimedia | Company style | Style | no |
| Auto / Canonical / Standard / XO | One-command style | XO ≈ unicorn, already default | no |
| Antfu | Current “best” TS preset: unicorn, import, regexp, perfectionist, stylistic, optional React/Vue/Svelte | Official oxlint integration still blocked (no ESLint flat config). Perfectionist/stylistic → oxfmt. regexp → JS plugin later. | steal import + unicorn extras; do not depend on the package |
| Adjunct / Ash-Nazg / Cecilia / Supermind | Kitchen-sink plugin bundles | Noise | no |
| clean-typescript | Ban TS-only keywords | Conflicts with using TypeScript | no |
| Hardcore / Sheriff | Maximal practical / TS-opinionated | Too many plugins and style rules | no |
| Problems | Correctness only, no style | Closest preset to Oxlint defaults | already the default |
| Node.js Standard / Superlint / Lintier | Scaffolding | n/a | no |
| eslint-config-prettier | Turn off formatting rules | Oxlint is already not a formatter | no |

### Code quality plugins

| Project | 2026 idea | Oxlint | Add? |
| --- | --- | --- | --- |
| **Unicorn** | Modern JS | **native**, default, 13/138 rules on | yes: the `prefer-*` extras above, not the whole recommended (it bans `forEach`, `reduce`, `null`) |
| **SonarJS** | Bug/suspicious patterns | **js-ok** (conformance). Overlaps `oxc` | later, not v1 |
| **depend** | Ban bloated/polyfill deps | **js-ok** (community) | later |
| **@e18e/eslint-plugin** | 2026 modernization + module replacements + perf | **js-ok** (conformance). Type-aware rules **js-typed** | later; unicorn extras first |
| De Morgan | Simplify negated boolean | **js-ok** | later, style |
| GitHub | Misc GitHub rules | **js-ok** if ESLint 9 | no |
| @mysticatea / @brettz9 | Misc | mostly superseded by unicorn/eslint | no |
| Deslint | Design-system / a11y for AI UI | **js-ok**, CSS/JSX | per-project UI |
| code-complete | Clean-design heuristics | **js-ok** | no, vague |
| ai-guard | AI-typical bugs (empty catch, SQL concat, secrets) | **js-ok** | later; overlap with no-useless-catch, no-secrets, no-eval |

### Compatibility plugins

| Project | Oxlint | Add? |
| --- | --- | --- |
| Compat | **js-ok** as of oxlint 1.32 (browserslist). Needs project `browserslist` | per-project browser libs |
| ecmascript-compat / es-x / es5 / ie11 | Restrict language level | skip. We are on modern TS |

### CSS-in-JS

Emotion, styled-components, vanilla-extract, css-modules: **js-ok** or broken on old CJS. Not a general TS fallback. Per-project.

### Deprecation

`eslint-plugin-deprecate`: **js-ok**, only if a repo marks its own APIs. `eslint-plugin-disable`: n/a.

### Embedded / parsers / non-JS languages

Oxlint lints script blocks in `.vue` / `.svelte` / `.astro`. It does not take ESLint custom parsers.

| Project | Oxlint | Add? |
| --- | --- | --- |
| HTML (BenoitZugmeyer) | **parser**, failed to load | no |
| Markdown (eslint-plugin-markdown) | **parser** | no; oxfmt formats markdown |
| html-eslint / json / jsonc / json-schema / package-json / toml / yaml / mdx / SQL | **parser** | no; oxfmt covers json/yaml/toml/md |
| babel-eslint-parser | n/a | Oxlint parses JS/TS itself |
| TypeScript parser | n/a | native |
| GraphQL / BrightScript | **parser** | per-project GraphQL |

### Frameworks

All per-project. Native vs JS:

| Project | Oxlint | Notes |
| --- | --- | --- |
| React | **native** `react` | includes hooks, refresh, compiler rules |
| React Hooks | **native** + **js-ok** | use native |
| React Refresh | **native** | |
| JSX a11y | **native** `jsx-a11y` | |
| Next.js | **native** `nextjs` | |
| Vue | **native** `vue` for script; templates limited | |
| Vue scoped CSS | **parser** | no |
| Angular | **js-ok** for most TS rules; templates **parser**; a few rules need parserServices | Angular repos only |
| Astro / Svelte / Solid | Svelte/Astro **parser**; Solid **js-ok** with some API holes | per-project |
| React Native / React-Redux | **js-ok** | per-project |
| AngularJS / Backbone / Ember / Hapi / Meteor | dead or niche | no |

### Languages and environments

| Project | Oxlint | Add? |
| --- | --- | --- |
| **TypeScript** (typescript-eslint) | **native** + type-aware | yes, already default; add the three extras above |
| erasable-syntax-only | **js-ok**, no types needed | only if we adopt `erasableSyntaxOnly` |
| expect-type | **js-typed** / special comments | test-only |
| N (eslint-plugin-n) | **native** `node` is a subset; full plugin **js-ok** (needs alias, name `node` is reserved) | not v1 |
| Flow / Babel plugin | obsolete for this repo | no |
| eslint-plugin-eslint-plugin | linting ESLint plugins | n/a |

### Libraries

| Project | Oxlint | Add? |
| --- | --- | --- |
| JSDoc | **native** `jsdoc` | skip in TS fallback |
| GraphQL-eslint | mixed: many rules **js-ok**, schema-aware ones need parserServices | per-project |
| Tailwind / better-tailwindcss | class lint; oxfmt has `sortTailwindcss` | per-project; sort via oxfmt |
| Lodash / Ramda / jQuery / Mongo / RequireJS / TypeGraphQL | library-specific | only if that library is the stack |

### Misc / tools / formatters / globals

Diff, only-warn, only-error, notice, woke, putout, typelint: process wrappers or unrelated. ESLint formatters (html, SARIF, GitHub) are ESLint CLI. Oxlint has its own reporters. Globals packages are n/a (Oxlint has `env` / `globals`). Developing-for-ESLint and tutorials: n/a.

### Practices and specific ES features

| Project | Oxlint | Add? |
| --- | --- | --- |
| **import** / **import-x** | **native** `import` | **yes, enable plugin** |
| **Promise** | **native** `promise` | **yes, enable plugin** |
| array-func | overlap unicorn prefer-array-* | covered by extras |
| error-cause | overlap `preserve-caught-error` | native extra instead |
| no-use-extend-native | overlap unicorn / `no-extend-native` | `eslint/no-extend-native` is suspicious, not default; skip unless we see need |
| RegExp | **js-ok** | later |
| ReDoS / ReDoSDetector | **js-ok** | later; `no-invalid-regexp` is already default |
| Math | **js-ok** | later |
| unused-imports | **js-ok**; native `no-unused-vars` already default | no |
| boundaries / hexagonal / signature-design / fp / functional / immutable / mutate / pure / this | architecture or FP religion | per-project |
| perfectionist / simple-import-sort / split-and-sort / filenames / padding / paths | **oxfmt** (imports) or style | oxfmt `sortImports` |
| ESLint Stylistic | **js-ok** (conformance, ~100% tests) | **oxfmt, do not add** |
| const-case / editorconfig / switch-case | style | no |
| no-restricted-syntax (query plugin) | `no-restricted-syntax` is not native; **js-ok** via oxlint-plugin-eslint | only for a custom ban list |
| toplevel / no-loops / no-comments / no-argument-spread / proper-arrows | niche | no |
| eslint-comments | **js-ok** | later |
| exception-handling / neverthrow | **js-typed** | incompatible |

### Performance

DOM / optimize-regex / perf-standard: niche. Native `react-perf` exists for React. `oxc/no-accumulating-spread` and `oxc/no-map-spread` are the useful general perf rules; they are off by default (category `perf`). Not v1 unless we want them.

### Security

| Project | Oxlint | Add? |
| --- | --- | --- |
| no-secrets | **js-ok** | later |
| no-unsanitized / xss | **js-ok**, DOM | per-project web |
| Security (nodesecurity) | **js-ok**, Node | per-project Node |
| pii / pg | niche | no |

### Testing tools

Native: `jest`, `vitest`. JS-ok and conformance-tested: Cypress, Playwright, testing-library, mocha, storybook. Also jest-dom (mostly ok). AVA, Jasmine, QUnit, Cucumber, TestCafe, Chai: **js-ok** if ESLint 9. None of these belong in a language fallback; enable next to the test runner.

## How this should look in files

Wired as shareable files (not auto-discovered, so a project has to add them):

```
typescript/
  README.md
  oxlintrc.json      # extras; extend this from .oxlintrc.json
  oxfmtrc.json       # sortImports; oxfmt has no extends, pass -c
```

A project:

```json
{
  "extends": ["/path/to/linters/typescript/oxlintrc.json"],
  "options": { "typeAware": true }
}
```

`extends` only inherits `rules`, `plugins`, and `overrides`. Set `options.typeAware` in the project's root file. `-c oxlintrc.json` uses this file as the root, so `typeAware` applies there.

The extras live in `oxlintrc.json` and `oxfmtrc.json`.

## Explicit non-goals for v1

- ESLint beside Oxlint (`eslint-plugin-oxlint`). Only needed if a JS plugin we want is still incompatible.
- Antfu / Sheriff / Hardcore as a base.
- Stylistic rules, `eqeqeq`, `no-explicit-any`, `consistent-type-imports`, filename case.
- Whole Unicorn recommended.
- Type-aware `no-unsafe-*` (the `any` hammer). Python did not turn ty up to maximum.
- Framework and test plugins in the global fallback.
