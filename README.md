# Automation Value Ranker

A simple web tool that helps a team decide **which tasks are worth automating first**.

Most teams have more automation ideas than time to build them. This tool ranks each idea by its expected value: the hours it saves, minus what it costs to build and keep running, adjusted for how confident we are in the estimates.

> **Status:** In progress (October 2026). Being built with Claude Code over one week. See the [build plan](#build-plan).
>
> **Live demo:** coming on Day 6.
>
> **About the data:** The tool comes with sample tasks from a made-up small research team. All tasks and numbers are invented for demonstration. You can replace them with your own.

---

## The problem

A team can almost always automate more than one person can build. Without a clear method, the next project often gets picked because it is interesting, or because someone asked loudly, not because it is the most valuable.

This tool makes the choice visible and easy to explain: every rank comes with the math behind it.

---

## How it works

For each recurring task, you enter a few estimates:

| Input | Example |
|---|---|
| Minutes per run | 45 |
| Runs per month | 4 |
| People who do it | 3 |
| Share an automation could handle | 70% |
| Build effort (hours, one time) | 10 |
| Upkeep (hours per month) | 1 |
| Confidence in these estimates | Medium |
| Type | Replaces routine work / Adds new capacity |
| Touches private data or outside accounts? | Yes / No |

The tool then calculates:

- **Hours saved per month** = minutes per run × runs per month × people × share handled ÷ 60
- **Net hours per month** = hours saved − upkeep
- **First-year value** = net hours per month × 12 × confidence − build effort
  *(confidence: high = 1.0, medium = 0.7, low = 0.4)*
- **Payback time** = build effort ÷ net hours per month

Tasks are ranked by **first-year value**.

**Worked example** (the row above):
- Hours saved: 45 × 4 × 3 × 0.7 ÷ 60 = **6.3 hours/month**
- Net: 6.3 − 1 = **5.3 hours/month**
- First-year value: 5.3 × 12 × 0.7 − 10 ≈ **34.5 hours**
- Payback: 10 ÷ 5.3 ≈ **1.9 months**

**"Adds new capacity" tasks** are work the team has never had time to do, so there is no current time to save. For these, you enter what the work would be worth in hours per month, and the rest of the math stays the same.

### Flags

- **Needs review:** touches private data or outside accounts. Check access and security before building.
- **Low confidence:** the estimate is a guess. Check with the people who do the task before building.
- **Consider retiring:** for automations already running, upkeep now takes as much time as the automation saves.
- **Not worth it yet:** first-year value is below zero.

---

## Features (planned)

- Ranked list of tasks, with sample data loaded
- Add, edit and delete tasks in the browser
- **"Why this rank?"**: a plain-language explanation of each score
- Flags (see above)
- Value vs. effort chart to spot quick wins
- Export to CSV
- Edits saved in your browser

## Left out on purpose

- **Accounts and logins:** not needed for a team-sized tool, and they add setup and risk.
- **A server or database:** a single web page is easier to host, hand over and maintain.
- **AI-made estimates:** every number is entered by a person, so the ranking stays easy to check and discuss. This could be added later as a suggestion, never as the final answer.

---

## How it's being built

Built with **Claude Code**, using a spec-first approach:

1. Write a clear spec (`SPEC.md`) before any code.
2. Give Claude Code project rules (`CLAUDE.md`) and break the work into small tasks.
3. Test the scoring math before building the interface.
4. Record what I asked for, what went wrong and what I changed (`BUILD_LOG.md`).
5. Write a handover guide (`HANDOVER.md`) so someone else can run and fix the tool.

## Build plan

| Day | Goal |
|---|---|
| 1 | Write `SPEC.md`, `CLAUDE.md` and the sample data |
| 2 | Scoring logic, with tests for the math |
| 3 | Ranked table and add/edit form |
| 4 | Flags, "Why this rank?", and odd cases (blank fields, zero values, negative value) |
| 5 | Value vs. effort chart and CSV export |
| 6 | Put it online with GitHub Pages; write `HANDOVER.md` |
| 7 | Polish, short demo video, final README update |

## Tech

One static web page (HTML, CSS, JavaScript). Sample data in a JSON file. Hosted free on GitHub Pages. Nothing to install.
