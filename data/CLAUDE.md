# CLAUDE.md

Rules for Claude Code when working on this project.

## What this is

Automation Value Ranker: a one-page web tool that ranks automation ideas by expected value and flags running automations worth retiring. All data in this repo is made up for demonstration.

**Read `SPEC.md` before any task.** It is the source of truth for every formula, rule, flag and screen. If a request conflicts with the spec, stop and say so instead of guessing.

## Tech rules

- Plain HTML, CSS and JavaScript (ES modules). No frameworks, no build step.
- No libraries or npm dependencies. The chart is drawn by hand in SVG. Ask before adding anything.
- `package.json` exists only for `"type": "module"` and the test script.
- Tests use Node's built-in test runner (Node 20 or newer). Nothing to install.
- All formulas, ranking and flags live in `src/scoring.js` as pure functions (no page code in that file). `src/app.js` only handles the page.

## Commands

- Run tests: `npm test` (runs `node --test`)
- Run the app locally: `python3 -m http.server 8000`, then open http://localhost:8000
  (Opening `index.html` directly from disk won't work, because browsers block modules and JSON loading from `file://`.)

## How to work

1. Before coding, give a short plan: which files change and why. Wait for my OK on anything bigger than one small step.
2. Work in small steps. One feature per step.
3. After any change to `src/scoring.js`, run the tests and show the result. Don't move on with failing tests.
4. Never change a formula, weight or rule in code without changing `SPEC.md` first, and tell me when you do.
5. Use the expected results in `SPEC.md` section 10 as the test cases.
6. Round only for display, with the `round1` helper from `SPEC.md` section 4.6.
7. Suggest a short commit message after each working step.
8. At the end of a session, suggest a `BUILD_LOG.md` entry (what we did, what went wrong, what changed). I will edit it before saving.

## Safety rules

- Never put user-typed text into the page with `innerHTML`. Use `textContent` or create elements.
- Guard CSV export against formula injection (`SPEC.md` section 6.4).
- Wrap all `localStorage` reads and writes in `try/catch`. The app must work without it.
- No outside network requests. No analytics. No real personal or company data.

## Writing style for the page

- Plain, short sentences. Explain numbers in words a non-technical teammate understands.
- Hours, not money.
- Always show the note that the sample data is made up.
