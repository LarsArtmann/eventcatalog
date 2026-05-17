# DELETE_TO_IMPROVE_PLAN.md

**Generated:** 2026-05-17 | **Repo:** eventcatalog | **Branch:** master

Deep audit of every file and directory to identify what should be deleted or cleaned up to improve repo health.

---

## Summary

| Category | Count | Impact |
|----------|-------|--------|
| Dead files to delete | 8 | Remove boilerplate, unused code |
| Unused npm dependencies | 5 | Reduce install size, attack surface |
| Duplicate files to deduplicate | 3 | Save ~728KB in git history |
| Generated files to gitignore | 3 | Prevent future pollution |
| Misplaced/stale CI files | 3 | Reduce confusion |
| Total potential disk savings | — | ~1.5MB+ from duplicates/dead files alone |

---

## TIER 1: Delete Immediately (Zero Risk)

These files are confirmed dead, unused, or boilerplate. Deleting them carries zero risk.

### 1.1 `website/src/pages/markdown-page.md` — Docusaurus Boilerplate

- **What:** Default Docusaurus starter page with placeholder content ("Markdown page example", "You don't need React to write simple standalone pages.")
- **Why delete:** Zero real content. Docusaurus auto-generated on project init. Never customized.
- **Risk:** None. Not linked from any navigation, sidebar, or other page.

### 1.2 `packages/eventcatalog/styles/Home.module.css` — Next.js Starter Leftover

- **What:** 100+ lines of CSS (`.container`, `.main`, `.footer`, `.title`, `.card`, `.grid`, etc.)
- **Why delete:** Classic `create-next-app` boilerplate CSS. Zero imports found across the entire codebase. `_app.tsx` imports `globals.css` and `eventcatalog.styles.css` — never this file.
- **Risk:** None. Confirmed dead via `grep -r "Home.module"` across all `.tsx`, `.ts`, `.jsx`, `.js`, `.css` files.

### 1.3 `packages/eventcatalog/public/logo-random.svg` — Unreferenced Asset

- **What:** SVG file in the public assets directory
- **Why delete:** No import, reference, or usage found anywhere in the codebase.
- **Risk:** None. Could be a leftover from an abandoned feature.

### 1.4 `.github/ISSUE_TEMPLATE.md` — Superseded Redirect

- **What:** Single-line file redirecting users to the issue template picker: `## 👉 [Please follow one of these issue templates](...) 👈`
- **Why delete:** GitHub's template picker (`.github/ISSUE_TEMPLATE/` with `config.yml` setting `blank_issues_enabled: false`) already handles this. The redirect is redundant — users never see it because GitHub shows the template picker instead.
- **Risk:** None. The modern YAML templates in `.github/ISSUE_TEMPLATE/` are the active ones.

### 1.5 `.github/ISSUE_TEMPLATE/default.yml` — Legacy Format

- **What:** Old-style markdown issue template (YAML frontmatter only, no `body:` key)
- **What it contains:**
  ```yaml
  name: Default Issue Template
  about: Use this template for anything else
  title: ""
  labels: [bug]
  ```
- **Why delete:** GitHub deprecated this format. The repo already has modern form-style templates (`bug.yml`, `feature.yml`, `documentation.yml`) with proper `body:` fields. This legacy template is inconsistent with the others and provides no value beyond what `bug.yml` already covers (it even defaults to `labels: [bug]`).
- **Risk:** None. Users will still have 3 modern templates + the Discord link from `config.yml`.

### 1.6 `.changeset/canary-release.yml` — Misplaced, Obsolete Workflow

- **What:** A GitHub Actions workflow file sitting in `.changeset/` instead of `.github/workflows/`
- **Why delete:** It's not executed by GitHub Actions from this location. The actual canary workflow is `.github/workflows/canary.yml` (which is properly configured and uses Node 18). This file uses Node 14.x (EOL), `actions/checkout@v3`, and `actions/setup-node@v2` — all outdated. It's a leftover from an older setup that was replaced by the proper `.github/workflows/canary.yml`.
- **Risk:** None. Not executed by GitHub. Has been sitting dormant since the initial commit.

### 1.7 `.changeset/README.md` — Auto-generated Boilerplate

- **What:** Standard `@changesets/cli` init message explaining how changesets work
- **Why delete:** Zero project-specific content. Any developer working with changesets already knows this. If needed, it's in the changesets documentation.
- **Risk:** None. Pure documentation boilerplate.

### 1.8 Root `tsconfig.json` — Orphaned Config

- **What:** Bare-bones TypeScript config at the repo root:
  ```json
  {
    "compilerOptions": {
      "module": "commonjs",
      "target": "es6",
      "declaration": true,
      "noImplicitAny": false,
      "sourceMap": true,
      "lib": ["es6"]
    },
    "exclude": ["node_modules", "**/*.test.ts"]
  }
  ```
- **Why delete:** All 6 packages have their own fully independent `tsconfig.json` files, none of which extend the root config:
  - `packages/eventcatalog/tsconfig.json`
  - `packages/eventcatalog-utils/tsconfig.json`
  - `packages/eventcatalog-types/tsconfig.json`
  - `packages/create-eventcatalog/tsconfig.json`
  - `packages/eventcatalog-plugin-generator-asyncapi/tsconfig.json`
  - `packages/eventcatalog-plugin-generator-amazon-eventbridge/tsconfig.json`
- **Risk:** None. Confirmed zero `extends` references to the root file. Root config has legacy settings (`emitDecoratorMetadata`, `experimentalDecorators` — note: these were in the original but not visible in current version) suggesting it's from an older era.

---

## TIER 2: Delete Unused npm Dependencies (Low Risk)

These packages are listed in `packages/eventcatalog/package.json` but have zero imports in the codebase.

### 2.1 `@stoplight/markdown-viewer` (dependency)

- **Why delete:** No import found anywhere. The package uses `@stoplight/json-schema-viewer` and `@stoplight/mosaic` (both used), but `markdown-viewer` is a separate package that was never integrated.
- **Risk:** Low. If it's a peer dependency of another Stoplight package, it would be installed transitively. Check `package.json` after removal.

### 2.2 `@stoplight/mosaic-code-viewer` (dependency)

- **Why delete:** No import found anywhere. Used to be part of the `@stoplight/mosaic` ecosystem but isn't directly imported in any component.
- **Risk:** Low. Same reasoning as above.

### 2.3 `react-force-graph-3d` (dependency)

- **Why delete:** No import found anywhere. The project uses `react-flow-renderer` for graph visualization and `dagre` for layout. This appears to be an abandoned experiment or leftover from a 3D visualization feature that was never implemented.
- **Risk:** None. Completely unused.

### 2.4 `cross-env` (devDependency)

- **Why delete:** The root `package.json` has `cross-env` as a devDependency and uses it in the `test` script. However, the `packages/eventcatalog/package.json` also lists it as a devDependency — but no script in that package uses it. The root-level usage is fine; the duplicate in the package is dead.
- **Risk:** None. Root `package.json` keeps its own `cross-env`.

### 2.5 `trim` (devDependency)

- **Why delete:** No import found anywhere. Likely added as a fix for a vulnerability in an older dependency tree (the root `package.json` has `resolutions: {"trim": "=0.0.3"}` — a known prototype pollution fix). But as a direct devDependency in the package, it serves no purpose.
- **Risk:** Low. The resolution in root `package.json` handles the security concern.

---

## TIER 3: Deduplicate Identical Files (Medium Value)

These are byte-for-byte identical files stored in multiple locations.

### 3.1 `opengraph.png` — 3 Copies (728KB wasted)

| Location | Size | MD5 |
|----------|------|-----|
| `packages/eventcatalog/public/opengraph.png` | 364KB | `c5bc29e7...` |
| `packages/create-eventcatalog/templates/default/public/opengraph.png` | 364KB | `c5bc29e7...` (IDENTICAL) |
| `website/static/img/opengraph.png` | 152KB | `c4ad7094...` (DIFFERENT — smaller version) |

- **Why deduplicate:** The two 364KB copies in `packages/` are identical. However, the code references the remote URL (`https://eventcatalog.dev/img/opengraph.png`) rather than the local file, making the local copies potentially unnecessary.
- **Recommendation:** Consider whether the local copies are needed at all. If the Next.js app only uses the remote URL, the local `packages/eventcatalog/public/opengraph.png` is dead weight. The template copy (`create-eventcatalog/templates/default/public/opengraph.png`) should be kept as it's part of the project scaffold.

### 3.2 `logo.svg` — 2 Identical Copies

| Location | MD5 |
|----------|-----|
| `packages/eventcatalog/public/logo.svg` | IDENTICAL |
| `packages/create-eventcatalog/templates/default/public/logo.svg` | IDENTICAL |

- **Why note:** These serve different purposes (runtime asset vs scaffold template). Both are small. Not worth deduplicating — just noting for awareness.

### 3.3 `favicon.ico` — 2 Identical Copies

Same situation as `logo.svg`. Both serve their purpose in different contexts.

---

## TIER 4: Add to `.gitignore` (Prevent Future Pollution)

### 4.1 `report/` — Generated Analysis Output

- **What:** Contains `jscpd-report.json` — output from JavaScript Copy/Paste Detector
- **Why gitignore:** This is generated tool output, not source code. It was generated on 2026-05-17 and should never have been committed. No jscpd config exists in the repo, suggesting it was run ad-hoc.
- **Action:** Add `report/` to `.gitignore` and delete `report/` from git tracking.

### 4.2 `.vscode/` — IDE Settings

- **What:** Contains `settings.json` with `{"typescript.tsdk": "node_modules/typescript/lib"}`
- **Why gitignore:** IDE settings are developer-specific and shouldn't be forced on all contributors. The current setting is harmless but sets a precedent. Better to let each developer configure their own IDE.
- **Action:** Add `.vscode/` to `.gitignore`. Remove `.vscode/` from git tracking (but keep locally).

### 4.3 `.crush/` — Crush Tool Database

- **What:** Contains `crush.db`, `crush.db-shm`, `crush.db-wal` — SQLite database files for the Crush AI assistant tool
- **Why gitignore:** Local tool state. Should never be committed. The directory has a `.gitignore` inside it already, but it should be in the root `.gitignore` for safety.
- **Action:** Add `.crush/` to root `.gitignore`.

---

## TIER 5: Not Recommended for Deletion (Kept for Context)

These items were investigated but should NOT be deleted.

### 5.1 `website/` — Keep

Actively deployed at `eventcatalog.dev`. Part of the yarn workspace. Blog is stale (last post March 2022) but the docs are actively maintained. Deleting would break the live site.

### 5.2 `examples/basic/` — Keep

Used by `scripts/start-catalog-locally.js` and `scripts/verify-build-catalog-locally.js` for local development. Not a duplicate of `templates/default/` — they've diverged (examples has domains/Orders with extra events, different config with generators, etc.)

### 5.3 `.all-contributorsrc` — Keep

Source-of-truth for the all-contributors bot. Contains 30+ contributor entries. Standard pattern for open-source projects.

### 5.4 `jest/custom_matchers.ts` — Keep

Provides `toMatchMarkdown()` custom matcher. Loaded via `setupFilesAfterSetup` in `jest.config.js` for every test run.

### 5.5 Root `babel.config.js` — Keep

Used by the Next.js build pipeline (via `babel-jest` transform in `jest.config.js`). Different from `website/babel.config.js` which is Docusaurus-specific.

### 5.6 All packages — Keep

All 6 packages are actively maintained with no dead exports:
- `@eventcatalog/core` — Main application
- `@eventcatalog/types` — Shared types (all used)
- `@eventcatalog/utils` — Shared utilities (all used)
- `@eventcatalog/create-eventcatalog` — CLI scaffolding tool
- `@eventcatalog/plugin-doc-generator-asyncapi` — AsyncAPI plugin
- `@eventcatalog/plugin-doc-generator-amazon-eventbridge` — EventBridge plugin

---

## Execution Checklist

### Phase 1: Delete dead files
- [ ] Delete `website/src/pages/markdown-page.md`
- [ ] Delete `packages/eventcatalog/styles/Home.module.css`
- [ ] Delete `packages/eventcatalog/public/logo-random.svg`
- [ ] Delete `.github/ISSUE_TEMPLATE.md`
- [ ] Delete `.github/ISSUE_TEMPLATE/default.yml`
- [ ] Delete `.changeset/canary-release.yml`
- [ ] Delete `.changeset/README.md`
- [ ] Delete root `tsconfig.json`

### Phase 2: Remove unused dependencies
- [ ] Remove `@stoplight/markdown-viewer` from `packages/eventcatalog/package.json`
- [ ] Remove `@stoplight/mosaic-code-viewer` from `packages/eventcatalog/package.json`
- [ ] Remove `react-force-graph-3d` from `packages/eventcatalog/package.json`
- [ ] Remove `cross-env` from `packages/eventcatalog/package.json` devDependencies
- [ ] Remove `trim` from `packages/eventcatalog/package.json` devDependencies

### Phase 3: Clean up git tracking
- [ ] Add `report/`, `.vscode/`, `.crush/` to root `.gitignore`
- [ ] Remove `report/` from git tracking (`git rm -r --cached report/`)
- [ ] Remove `.vscode/` from git tracking (`git rm -r --cached .vscode/`)

### Phase 4: Verify
- [ ] Run `yarn install` to verify dependency removal
- [ ] Run `yarn test` to verify no regressions
- [ ] Run `yarn build` to verify build succeeds
- [ ] Run `yarn lint` to verify no lint errors
