# TypeScript

Oxlint lints. Oxfmt formats. `oxlint-tsgolint` is required for type-aware rules.

These two files are extras on top of each tool's defaults. They are not auto-discovered (no leading dot), so a project has to add them.

## Add to a project

Point the project's own config at this repo. `extends` only inherits `rules`, `plugins`, and `overrides`. `options.typeAware` has to live in the project's root file.

`.oxlintrc.json`:

```json
{
  "extends": ["/Users/themiya/Documents/Dev/Github/linters/typescript/oxlintrc.json"],
  "options": { "typeAware": true }
}
```

Oxfmt has no `extends`. Pass the file, or copy it to `.oxfmtrc.json`:

```sh
oxfmt -c /Users/themiya/Documents/Dev/Github/linters/typescript/oxfmtrc.json --check .
```

A project that already has `.oxlintrc.json` / `oxlint.config.ts` keeps that file and only picks up these extras if it `extends` them.

## Personal fallback (no project config)

```sh
oxlint -c /Users/themiya/Documents/Dev/Github/linters/typescript/oxlintrc.json
oxfmt  -c /Users/themiya/Documents/Dev/Github/linters/typescript/oxfmtrc.json --check .
```

`-c` uses this file as the root, so `options.typeAware` applies. Install `oxlint-tsgolint` next to `oxlint` or type-aware rules will not run.

`oxlint --type-check` is a separate command, like `ty check`.

Why these extras: `awesome-eslint-oxlint-report.md`.
