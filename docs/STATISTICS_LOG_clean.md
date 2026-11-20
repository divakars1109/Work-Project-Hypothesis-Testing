# Statistics summary: programming capacity and COVID-19 resilience

**For the group.** This is the short, readable version of the statistical work. The full technical record (every check, fingerprint and review) is in `STATISTICS_LOG.md`. Everything was run in one notebook, `stats_analysis.ipynb`, which reads `master_table_v3_240_units.csv`.

---

## 1. The question

Were EU NUTS 2 regions with **excess programming capacity** (more Stack Overflow programmers than expected for their population and income) more resilient to COVID-19?

| | In plain words |
|---|---|
| **H1a (shock)** | Excess regions lost less employment between 2019 and 2020. |
| **H1b (recovery)** | Excess regions recovered more employment between 2020 and 2022. |
| **H0** | No difference. |

---

## 2. Data and sample

- **240 regions** (EU27, NUTS 2, 2021 map). **234** have a Stack Overflow measure; six with zero users are left out (FI20, FRY1 to FRY5).
- We compare regions **inside the same country**. Five countries have only one region (CY, EE, LU, LV, MT), so they add nothing. The result is carried by **229 regions in 22 countries**.
- **Programming capacity (X):** the excess score *z*, built from Stack Overflow users active in 2019, relative to what population and income predict. z above 0 = excess (111 regions), below 0 = deficit (123). We rebuilt z and it matches Moughit's file.
- **Outcome (Y):** employment rate, ages 15 to 64, change in percentage points (pp).
  - Shock = 2020 minus 2019.
  - Recovery = 2022 minus 2020.
  - 2021 is not used (series break).
- **Controls:** log population and log GDP per inhabitant (both 2019), plus a baseline for each country.

---

## 3. Decisions fixed before any result was seen

| What | Decision |
|---|---|
| Size that matters (margin) | **1 pp** of employment per 1 SD of z |
| Test | Two-sided, 5% level (95% interval) |
| Method | **Wild cluster bootstrap-t**, clusters = countries, 9,999 draws, seed 42 |
| Main models | Shock, and Recovery A (without the shock as a control) |
| Check model | Recovery B (with the shock as a control). It never changes the verdict. |
| Two tests, one question | **Holm correction** across shock and recovery A |
| Verdict rule | Fixed in advance (see section 5) |

A few small details of how the verdict rule's parts combine were written down after the step 10 and 11 numbers existed, but before the rule was applied. They do not affect this verdict, because nothing is significant.

---

## 4. What was done, step by step

**Step 8: descriptives (no tests).**
- Outcome summaries (shock, recovery, net) on 234 regions, the z distribution (111 excess, 123 deficit), and group averages for excess vs deficit, both raw (234 regions) and within-country (229 regions, 22 countries). The group averages are in notebook Section 2 and are not repeated here.
- Regions that lost more jobs in 2020 tended to regain more afterwards (correlation −0.31).
- Inside a country, the shock varies by about 1.12 pp (standard deviation).
- Stack Overflow and GitHub rank regions alike (rank correlation 0.785, 228 regions).
- All three match the figures the Guide expected (about −0.31, 1.1 pp, 0.78).

**Step 9: regressions (point estimates).** For each outcome we asked how much it moves per +1 SD of z, holding population and income constant, inside countries.
- The effect is tiny in all three models (see the table below). Share of within-country variation explained by the models: 2% (shock), 2% (recovery A), 21% (recovery B).
- In Recovery B the shock control has a coefficient of −0.63. Recovery B is a check only; Recovery A is the main model.
- Only point estimates here: no p-values or intervals yet.

**Step 10: uncertainty.** The bootstrap re-draws the data many times to see how much the estimate can wobble by chance. It gives a p-value and a 95% interval for each model, and Holm corrects the two main tests (smaller p must be below 0.025, then the larger below 0.05). The bootstrap draws are well centred (mean about 0).

**Step 11: smallest detectable effect.** The smallest true effect our data could reliably pick up (80% power) is 2.94 × the standard error (the Guide's 2.8 gives 0.28 and 0.45 pp, almost the same). The 1 pp margin is about 10 standard errors for the shock and 6 for recovery A. Caveat: this treats the estimated standard error as the true one, and with about 22 countries it is itself uncertain.

**Step 12: reading the answer.** The fixed rule was applied once, mechanically. The rule function was first tested on made-up cases (supported, against, mixed, no meaningful difference, inconclusive, Holm edge cases) and the verdict was confirmed by a second, separately written version of the rule.

---

## 5. Results (effects in pp per 1 SD of z)

| Model | Effect | Standard error | Bootstrap p | 95% interval | Holm-adjusted p | Smallest detectable effect |
|---|---|---|---|---|---|---|
| **Shock (H1a)** | 0.030 | 0.100 | 0.77 | −0.16 to 0.28 | 1.0 | 0.29 |
| **Recovery A (H1b)** | 0.048 | 0.162 | 0.77 | −0.27 to 0.45 | 1.0 | 0.48 |
| Recovery B (check) | 0.066 | 0.158 | 0.67 | −0.24 to 0.46 | not applied | not applied |

**The rule.**
1. If anything is significant after Holm, the answer is "supported", "evidence against" or "mixed", depending on the signs.
2. Otherwise, if both 95% intervals lie inside ±1 pp **and** both smallest detectable effects are below 1 pp, the answer is **"no meaningful difference"**.
3. Otherwise the answer is "inconclusive".

---

## 6. Verdict

> **No meaningful difference.**
> Neither test is significant (adjusted p = 1.0). Both 95% intervals lie well inside ±1 pp. Our data could have detected effects of about 0.3 pp (shock) and 0.5 pp (recovery), so a 1 pp effect would have shown up. Recovery B points the same way.

**What it means.** The result is an **association, not a cause**, for Stack Overflow activity in 2019 as a **proxy** for programming capacity. It rules out effects of 1 pp or more per 1 SD of z. It does **not** prove that there is no relationship of any size.

**For the paper:** write "associated with", never "caused". Do not copy the sentence "the intervals are wide" from Guide Section 9. Here the intervals are narrow compared with the 1 pp margin.

---

## 7. How it was checked

- The master table and the fixed-decisions cell were fingerprinted (SHA-256) before z was first set against the outcomes. They were re-checked after every step and never changed.
- Every key number (effects, standard errors, p-values, intervals, smallest detectable effects, verdict) was **recomputed independently with separate code** and matched.
- The notebook was rewritten twice for readability (plain formulas, small cells). Every number was shown to be identical before and after.
- **Deviations from the plan: none.**
- **Known blind spot in the notebook's own checks:** when parts of the bootstrap were deliberately corrupted, the checks caught 4 of 6 problems. The two missed ones were a sign flip of the coin flips (which does not change the result mathematically) and scrambling the coin flips at region level (the rerun check compares the function with itself). The independent recomputation matches exactly, so nothing is wrong now. An independent check of the first draws could be added later.

---

## 8. Main limitations

- **Proxy:** Stack Overflow counts active platform users, not all programmers. Only about 61% of EU-located users could be placed in a region.
- **Small number of countries:** about 22 countries carry the comparison, so the standard error itself is uncertain.
- **Six regions** with zero users are left out (FI20, FRY1 to FRY5).
- **Employment rate** does not capture hours, and some series have breaks.
- **Association only**, not a cause.

The full list (21 points) is in Guide Section 9.

---

## 9. Still open (not statistics)

- Moughit's script that builds the activity counts, and its README, are missing.
- Document how users with deleted accounts get a location.

---

## 10. How to re-run

1. Open `stats_analysis.ipynb` and set `MASTER_PATH` in Section 0 if your folder is different.
2. Run all cells from top to bottom.
3. Do **not** change the settings in Section 0. They are the decisions fixed in advance.
4. Files with `before` in the name are old backups. Use only `stats_analysis.ipynb`.
