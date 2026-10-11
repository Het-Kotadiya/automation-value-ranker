# Spec: Automation Value Ranker

This file is the source of truth. If the code and this spec disagree, the spec wins. Any change to a formula or rule must be made here first.

---

## 1. Goal

Help a small team decide **which automation to build next**, and **which running automation to retire**, using a clear and explainable score.

Every number on screen must be traceable: a person should be able to click "Why?" and see exactly how the score was calculated.

## 2. Who uses it

- **The person who builds automations** (for example, an AI enablement lead). They enter ideas, compare them and decide what to build next.
- **Teammates** who suggest ideas. They need plain explanations, not formulas.

## 3. Data: one task

Each task is one object in `data/sample-tasks.json` (or one row the user adds in the browser).

| Field | Type | Required? | Rules |
|---|---|---|---|
| `id` | text | always | Unique. New tasks get an id made by the app. |
| `name` | text | always | 1–80 characters. |
| `description` | text | no | Up to 300 characters. |
| `team` | text | no | Free text, for example "Research". |
| `status` | `"idea"` or `"running"` | always | `idea` = not built yet. `running` = already built and in use. |
| `type` | `"routine"` or `"new"` | always | `routine` = replaces work people do now. `new` = work nobody has had time to do. |
| `minutesPerRun` | number | if `type` is `routine` | ≥ 0 |
| `runsPerMonth` | number | if `type` is `routine` | ≥ 0 |
| `people` | number | if `type` is `routine` | ≥ 0, whole number |
| `shareHandled` | number | if `type` is `routine` | 0 to 1 (the form shows it as 0–100%) |
| `valueHoursPerMonth` | number | if `type` is `new` | ≥ 0. What the new work is worth, in hours per month. |
| `buildHours` | number | if `status` is `idea` | ≥ 0. Ignored for `running` tasks (see 4.5). |
| `upkeepHoursPerMonth` | number | always | ≥ 0 |
| `confidence` | `"high"`, `"medium"` or `"low"` | always | How sure we are about the estimates. |
| `sensitive` | true / false | always | Touches private data or outside accounts. |

## 4. Scoring

Work in hours, not money. Hours are easier to estimate and compare across people, and they don't depend on anyone's salary.

### 4.1 Confidence weights

| Confidence | Weight |
|---|---|
| high | 1.0 |
| medium | 0.7 |
| low | 0.4 |

### 4.2 Formulas

1. **Hours saved per month**
   - `routine`: `minutesPerRun × runsPerMonth × people × shareHandled ÷ 60`
   - `new`: `valueHoursPerMonth`
2. **Expected hours saved per month** = hours saved × confidence weight
3. **Net hours per month** = expected hours saved − `upkeepHoursPerMonth`
4. **First-year value** (ideas only) = net hours per month × 12 − `buildHours`
5. **Payback time in months** (ideas only) = `buildHours` ÷ net hours per month
   - If net hours per month ≤ 0: payback is **never** (store as `Infinity`, show "Never").
   - If `buildHours` is 0 and net > 0: payback is **0** (show "Right away").

### 4.3 Why confidence only discounts the savings

Savings are the uncertain part, so only savings are discounted. Upkeep and build time are counted in full, because they get spent either way.

This means **being less sure never makes a task look better.** (An earlier version applied confidence to the whole net value. For a task that loses time, that made low confidence look *better* than high confidence. See `BUILD_LOG.md`, Day 1.)

### 4.4 Ranking ideas

- Rank ideas by **first-year value**, highest first.
- Ties: fewer `buildHours` first, then name A–Z.
- Tasks with missing or invalid fields are **not ranked** (see 6.4).

### 4.5 Running automations

- Not ranked by first-year value. Build time is already spent, so it is ignored on purpose (it is a sunk cost and should not affect whether to keep something).
- Sorted by **net hours per month, lowest first**, so problems appear at the top.
- Status label: **Keep** if net > 0, otherwise **Consider retiring**.

### 4.6 Rounding

Calculate with full precision. Round only for display: one decimal place, always shown (for example, `33.6` or `136.0`). Tests compare numbers within 0.01.

Computers store some decimals with tiny errors. For example, t10's net hours is 1.85, but JavaScript stores it as `1.8499999999999996`, so plain `toFixed(1)` shows `1.8` instead of `1.9`. Use this helper for all display rounding:

```js
function round1(x) {
  const clean = Number(x.toFixed(10)); // remove tiny float errors first
  return Math.round(clean * 10) / 10;  // then round to 1 decimal
}
// show with round1(x).toFixed(1)
```

## 5. Flags

A task can have several flags. Each flag has a short label and a one-line reason shown on hover or in "Why?".

| Flag | Applies to | Rule | Reason shown |
|---|---|---|---|
| **Needs review** | all | `sensitive` is true | Touches private data or outside accounts. Check access and security first. |
| **Low confidence** | all | `confidence` is low | These numbers are a guess. Check with the people who do the task. |
| **Not worth it yet** | ideas | first-year value < 0 | Costs more time than it saves in its first year. |
| **Quick win** | ideas | first-year value > 0 and payback ≤ 1 month | Pays back its build time within a month. |
| **Consider retiring** | running | net hours per month ≤ 0 | Upkeep now takes as much time as it saves. |

## 6. Screens and features

### 6.1 Layout (one page)

1. **Header:** title, one-line explanation, and a visible note: *"Sample data: all tasks and numbers are made up for demonstration."*
2. **Ideas to build:** ranked table with: rank, name, team, first-year value (hours), payback (months), flags, and a **Why?** button.
3. **Value vs. effort chart:** ideas only. X axis = build hours, Y axis = net hours per month. Top-left area is labelled "Quick wins". Hovering a dot shows the task name.
4. **Running automations:** table with: name, team, net hours per month, Keep / Consider retiring, flags, and a **Why?** button.
5. **Needs more info:** tasks that can't be scored, each with the missing or invalid fields named in plain words.
6. **Actions:** Add task, Export CSV, Reset to sample data.

### 6.2 "Why?" explanation

Plain sentences, using the task's own numbers. Example for an idea:

> This task takes about **20 hours a month** in total (15 min × 20 times × 5 people × 80% that can be automated).
> We're **medium** sure about these numbers, so we count 70% of that: **14 hours**.
> Minus **2 hours** of upkeep leaves **12 hours a month**.
> Over a year that's 144 hours, minus **8 hours** to build = **136 hours** of first-year value.
> It pays back its build time in about **0.7 months**.

### 6.3 Add / edit form

- Fields change with `status` and `type` (for example, `buildHours` only shows for ideas; `valueHoursPerMonth` only shows for `new`).
- Errors in plain words next to the field, for example: *"Minutes per run can't be negative."*
- Edit and delete buttons on every row. Delete asks "Are you sure?" first.

### 6.4 Edge cases

| Case | What happens |
|---|---|
| A required field is blank | Task goes to **Needs more info**, naming the field. |
| A number is negative, or `shareHandled` is outside 0–1 | Form shows an error and won't save. Bad data loaded from a file goes to **Needs more info**. |
| Hours saved is 0 | Scored normally. Net is negative, so it gets **Not worth it yet** (idea) or **Consider retiring** (running). |
| Net hours per month ≤ 0 | Payback shows "Never". |
| `buildHours` is 0 | Payback shows "Right away" if net > 0. |
| Two ideas have the same first-year value | Tie rule from 4.4. |
| Very long name | Cut off with "…" in tables; full name in "Why?" and on hover. |
| Task text contains HTML or script | Shown as plain text, never run (see 7). |
| A cell in the CSV starts with `=`, `+`, `-` or `@` | Add a `'` in front so spreadsheet apps don't run it as a formula (see 7). |
| Browser storage is blocked or empty | App still works with the sample data; edits just won't be saved. |

### 6.5 Saving

- Edits are saved in the browser (`localStorage`), wrapped in `try/catch`.
- **Reset to sample data** clears saved edits after asking "Are you sure?".

### 6.6 Export CSV

One row per task, with all input fields plus: hours saved, expected hours saved, net hours per month, first-year value, payback, flags. File name: `automation-ranking-YYYY-MM-DD.csv`.

## 7. Safety rules

- Never put user-typed text into the page with `innerHTML`. Use `textContent` or create elements.
- CSV export guards against formula injection (6.4).
- No data leaves the browser. No outside requests except loading the app's own files.

## 8. Out of scope (on purpose)

- **Logins and accounts:** not needed for a team-sized tool, and they add setup and risk.
- **A server or database:** one static page is easier to host, hand over and maintain.
- **AI-made estimates:** people enter every number, so the ranking stays easy to check. Could be added later as a suggestion only.
- **Money values:** hours are enough to compare tasks and avoid salary questions.
- **Sharing between users and edit history:** export CSV covers sharing for now.

## 9. Files

```
index.html               page structure
styles.css               styles
src/scoring.js           all formulas, ranking and flags (no page code)
src/app.js               page code: tables, form, chart, export, saving
data/sample-tasks.json   13 made-up sample tasks
tests/scoring.test.js    tests for src/scoring.js
package.json             only to set "type": "module" and the test command; no dependencies
README.md  SPEC.md  CLAUDE.md  BUILD_LOG.md  HANDOVER.md (Day 6)
```

### `src/scoring.js` functions

| Function | Returns |
|---|---|
| `CONFIDENCE_WEIGHTS` | `{ high: 1, medium: 0.7, low: 0.4 }` |
| `validateTask(task)` | `{ ok, problems }` where `problems` is a list of plain-language messages |
| `scoreTask(task)` | `{ hoursSaved, expectedHoursSaved, netHoursPerMonth, firstYearValue, paybackMonths, flags }` (`firstYearValue` and `paybackMonths` are `null` for running tasks) |
| `rankIdeas(tasks)` | valid ideas, scored and sorted, each with a `rank` starting at 1 |
| `sortRunning(tasks)` | valid running tasks, scored and sorted by net hours, lowest first |
| `explainTask(task)` | the plain-language "Why?" text as a list of sentences |

## 10. Expected results for the sample data

Tests must check these. Values rounded to one decimal.

### Ideas (ranked)

| Rank | id | Task | Hours saved | Expected | Net / month | First-year value | Payback (months) | Flags |
|---|---|---|---|---|---|---|---|---|
| 1 | t03 | Meeting notes to action items | 20.0 | 14.0 | 12.0 | 136.0 | 0.7 | Needs review, Quick win |
| 2 | t02 | Add new AI models to the database | 20.0 | 14.0 | 11.0 | 102.0 | 2.7 | |
| 3 | t09 | Weekly AI company news check | 10.0 | 7.0 | 5.0 | 44.0 | 3.2 | |
| 4 | t01 | Weekly newsletter first draft | 4.8 | 4.8 | 3.8 | 33.6 | 3.2 | |
| 5 | t08 | Monthly benchmark results summary | 8.0 | 5.6 | 3.6 | 23.2 | 5.6 | |
| 6 | t10 | Shared prompt for paper summaries | 3.0 | 2.1 | 1.9 | 21.2 | 0.5 | Quick win |
| 7 | t04 | Sort expense receipts | 1.8 | 1.8 | 1.3 | 5.0 | 8.0 | Needs review |
| 8 | t05 | Social posts for new reports | 1.5 | 1.1 | 0.6 | 0.6 | 10.9 | |
| 9 | t07 | Sort press and partner emails | 3.0 | 1.2 | 0.2 | −5.6 | 40.0 | Needs review, Low confidence, Not worth it yet |
| 10 | t06 | Answer repeated AI tool questions | 2.1 | 0.8 | −0.2 | −6.0 | Never | Low confidence, Not worth it yet |

Exact values for tests (before rounding): t10 net = 1.85, t04 hours saved = 1.75 and net = 1.25, t05 expected = 1.05 and net = 0.55, t06 hours saved = 2.0833… and net = −0.1667…. Tests should also check that t10 net **displays** as `1.9` and t05 net as `0.6` (the rounding trap in 4.6).

### Running automations (lowest net first)

| id | Task | Hours saved | Net / month | Status | Flags |
|---|---|---|---|---|---|
| t12 | Website broken-link checker | 0.5 | −1.5 | Consider retiring | Consider retiring |
| t13 | Monthly AI tool spend report | 0.9 | 0.4 | Keep | Needs review |
| t11 | New papers posted to team chat | 18.0 | 17.0 | Keep | |

### What the sample data is meant to show

- **The top idea needs a check first.** t03 has the highest value but touches private data, so it is flagged for review before building.
- **Rank isn't everything.** t10 is only 6th by first-year value but pays back in about two weeks, so it's a quick win worth doing alongside the top pick.
- **Confidence changes decisions.** t06 and t07 are "not worth it yet" only because the numbers are a guess. If the team confirmed their estimates (high confidence), t06 would score +9.0 and t07 +16.0. The right next step is to ask, not to build.
- **Retire what stopped paying.** t12 breaks often and costs more upkeep than it saves.
