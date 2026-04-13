# Maintenance Notes (instructor-only)

Private notes for keeping the course deliverable. Not intended for students.

## Pre-class smoke test

Run through this the day before teaching to catch anything that drifted
since the last delivery:

```bash
# 1. Confirm you're on a current Gemini CLI
npm install -g @google/gemini-cli
gemini --version                   # expect 0.37.x or later

# 2. Launch and verify Plan Mode is the default (since 0.34)
gemini
#   -> should draft a plan before executing anything
#   -> Shift+Tab cycles out of plan mode; Ctrl+G opens external editor

# 3. Verify the current model list includes Gemini 3.1 Pro Preview
/model

# 4. Smoke-test one custom command to confirm the TOML format still loads
#    (e.g. /review, /test-gen, /docs, /refactor in commands/)

# 5. Confirm the Policy Engine flag is present and inspect current rule schema
gemini --policy --help

# 6. Regenerate the slides PDF so the handout matches the source
npm run build                      # or: npx slidev export
```

If any step surfaces a surprise, check the release notes at
<https://geminicli.com/docs/changelogs/latest/> before class.

## Dependency vulnerabilities

Dependabot routinely reports ~25+ findings on `main`. **None of them affect
students or the taught material.** Triage below.

### Root `package-lock.json` — Slidev toolchain (~20 findings)

These are all transitive deps of `@slidev/cli`. They only affect YOUR
laptop when rebuilding the deck — nothing ships to students.

**Fix:**

```bash
npm audit fix
# Optional (only if you want to bump Slidev to a new major):
# npm audit fix --force
```

Known high-severity packages flagged: `defu`, `lodash-es`, `picomatch`,
`rollup`, `vite`. Moderate: `dompurify`, `yaml`, `unhead`, and the
`@mermaid-js/parser` / `chevrotain` chain.

### `exercises/javascript/my-task-manager` — 4 findings

Jest + Express dev dependencies. Students don't touch these directly, and
the lockfile gets regenerated when they run `npm install` anyway.

**Fix:**

```bash
cd exercises/javascript/my-task-manager
npm audit fix
```

Known: `minimatch`, `path-to-regexp`, `picomatch` (high);
`brace-expansion` (moderate).

### Python / Java exercises

Not covered by `npm audit`. If the Dependabot count is bothering you:

```bash
# Python
cd exercises/python/weather-app
pip-audit -r requirements.txt

# Java
cd exercises/java/bookstore-api
mvn org.owasp:dependency-check-maven:check
```

These are hands-on exercise codebases — students will be modifying them
in class regardless, so a stale dep or two is not a teaching hazard.

## Known "do NOT do in a hurry" items

- **Do not** run `npm audit fix --force` at the root without testing the
  deck rebuild afterwards. It will bump Slidev across a major version.
- **Do not** upgrade Gemini CLI mid-class. Pin a version on students'
  machines during the prereq step.
- **Do not** delete `slides.pdf` without regenerating it — students get
  the PDF as a handout.
