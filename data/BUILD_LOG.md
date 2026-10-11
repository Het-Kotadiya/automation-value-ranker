# Build Log

What I asked for, what went wrong, and what I changed, day by day. This project is built with Claude Code.

---

## Day 1: Spec and plan (October 11, 2026)

**What I did**
- Wrote `SPEC.md`: data fields, formulas, ranking, flags, screens, edge cases, and expected results for the sample data.
- Wrote `CLAUDE.md`: project rules for Claude Code.
- Made 13 sample tasks for a made-up small research team (10 ideas, 3 running automations).
- Ran the formulas on the sample data before writing any app code.

**What went wrong**
1. **The first formula rewarded uncertainty.** It applied confidence to the whole net value (savings minus upkeep). For a task that loses time, low confidence made the loss look *smaller*: −14.6 hours with low confidence vs −29 hours with high confidence. Being less sure should never make a task look better.
2. **Rounding showed wrong numbers.** A net value of 1.85 is stored by JavaScript as `1.8499999999999996`, so it displayed as `1.8` instead of `1.9`. Two sample tasks were affected.

**What I changed**
1. Confidence now discounts only the hours saved, since that is the uncertain part. Upkeep and build time count in full. The same losing task now scores −36.2 with low confidence, worse than high confidence, as it should.
2. Added a `round1` helper that removes tiny float errors before rounding, and added both cases to the expected test results.

**What I learned**
- Test the formula on odd cases (negative values) before building anything on top of it.
- Check displayed numbers, not just stored ones.

**Next:** Day 2, scoring logic and tests with Claude Code.

---

<!-- Copy this block for each new day.

## Day N: Title (date)

**What I asked Claude Code to do**

**What went wrong**

**What I changed**

**What I learned**

**Next:**

-->
