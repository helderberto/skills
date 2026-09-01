---
name: perf-audit
description: Audit frontend bundle size and performance. Use when user asks to "audit performance", "/perf-audit", "analyze bundle", "check bundle size", or wants to find performance bottlenecks. Don't use for backend performance, database query optimization, projects without a frontend build step, accessibility (use /a11y-audit), or translation coverage (use /i18n).
---

# Performance Audit

## Workflow

1. Detect build tool from `package.json` (vite, webpack, next, rollup)
2. Run the production build via the project's task runner (auto-detect from package.json).
3. Analyze the build output — non-interactive only. Never launch a browser or a long-running server in an audit; those hang an automated run. Prefer, in order:
   - **Measure the emitted files directly**: for each file in the output dir (e.g. `dist/assets/*.js`), report raw + gzipped size — `gzip -c <file> | wc -c`. This always works and needs no extra tooling.
   - **Per-module breakdown, as a static artifact** (not a server):
     - Vite/Rollup: `npx source-map-explorer 'dist/assets/*.js'`, or add `rollup-plugin-visualizer` configured to emit JSON/HTML to a file
     - webpack/Next.js: `npx webpack-bundle-analyzer -m static -r report.html <stats-file>` (writes a file; do **not** use the default server mode)
   - `npx vite-bundle-visualizer` opens a browser and blocks — only suggest it to the user for manual exploration; do not run it here.

4. Check `package.json` for known heavy packages
5. Report findings with size impact

Compare against standard industry thresholds; report gzipped sizes.

## Output format

- **Bundle summary**: total size / gzipped size vs budget
- **Large chunks**: name + size + % of total
- **Heavy deps**: package + size + lighter alternative
- **Quick wins**: sorted by estimated savings

## Rules

- Report gzipped sizes (what the browser downloads)
- Never auto-change dependencies -- report and suggest only

## Error Handling

- Build fails -- report the error and stop; fix build issues before auditing
- No build output found -- run production build first
- Build tool unrecognized -- fall back to scanning `package.json` for heavy packages only
