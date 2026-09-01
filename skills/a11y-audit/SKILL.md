---
name: a11y-audit
description: Audit accessibility compliance in frontend code. Use when user asks to "check accessibility", "/a11y-audit", "audit a11y", "check WCAG", or wants to find accessibility issues. Don't use for backend code, non-UI files, projects without HTML/JSX output, bundle size (use /perf-audit), or translation coverage (use /i18n).
---

# Accessibility Audit

## Workflow

1. Detect framework from `package.json` (React, Vue, Svelte)
2. Run static analysis -- grep JSX/TSX/HTML files for violations
3. If dev server is running, optionally run axe-core CLI:
   ```bash
   npx @axe-core/cli@4 http://localhost:3000
   ```
4. Report findings grouped by severity

## Output format

Group by WCAG criterion:
- **Critical** (A) -- blocks assistive technology users
- **Serious** (AA) -- significantly impacts usability
- **Suggestions** -- best practice improvements

Use `file:line` references. Include the fix for each finding. Prioritize keyboard navigation and screen reader issues; never auto-fix -- show what to change and why.

## Error Handling

- Framework undetected -- scan all `.html`, `.jsx`, `.tsx`, `.vue` files
