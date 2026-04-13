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
#    IMPORTANT: slides.pdf on main is NOT auto-rebuilt - if slides.md
#    has changed since the last PDF commit, the committed PDF is stale
#    and students will get out-of-date handouts.
npx slidev export slides.md        # traditional headless export
# ...or use the new in-browser exporter (Slidev 52.x):
# npx slidev                        # then visit http://localhost:<port>/export
git add slides.pdf
git commit -m "docs: regenerate slides.pdf"
git push origin main
```

If any step surfaces a surprise, check the release notes at
<https://geminicli.com/docs/changelogs/latest/> before class.

> **Heads-up for April 2026 delivery:** `slides.md` was refreshed for
> Gemini CLI 0.30–0.37 (commit `b60409b`), but `slides.pdf` has NOT
> been regenerated from a sandboxed environment. Rebuild it from your
> laptop and commit the result before class.

## Dependency vulnerabilities

Dependabot routinely reports findings on `main`. **None of them affect
students or the taught material.** Triage below.

### Current state (after April 2026 refresh)

As of commits `0d4c9e0` and `3b4d7a1`:

| Lockfile | Before | After |
|---|---|---|
| Root `package-lock.json` (Slidev toolchain) | 20 (1 low / 14 mod / 5 high) | **6** (all moderate) |
| `exercises/javascript/my-task-manager` | 4 (1 mod / 3 high) | **0** |

The remaining 6 are all in the `@slidev/cli` → `monaco-editor` →
`dompurify` chain (moderate mutation-XSS). They only clear with
`npm audit fix --force`, which downgrades `@slidev/cli` from 52.12.0
to 52.6.0 (a major-version downgrade). **Don't bother** — the attack
surface for a slide deck the instructor builds themselves is nil.
Wait for Slidev upstream to publish a clean update.

### Re-running the audit next time

```bash
# Root (Slidev toolchain - only affects the laptop building the deck)
npm audit fix --ignore-scripts   # --ignore-scripts avoids a Playwright
                                  # post-install that tries to fetch
                                  # Chromium (can fail behind proxies)

# Exercise lockfile
cd exercises/javascript/my-task-manager
npm audit fix --ignore-scripts
```

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
