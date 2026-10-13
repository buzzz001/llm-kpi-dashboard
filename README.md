# LLM Evaluation Programme KPI Dashboard

Model quality, rater accuracy, agreement and audit throughput across an 80-rater evaluation programme.

**[Live dashboard](https://app.powerbi.com/view?r=eyJrIjoiYzM0N2Y0YjQtNTkwNC00OWRiLWJkZTEtNmYzYTAyZTIzYTcxIiwidCI6ImI0NGFhODMyLWI2YWMtNGUyNy1iNGVjLTUyYjgyYmFkZTBkMCJ9)** · **[PDF](LLM%20Evaluation%20KPI%20Dashboard.pdf)** · **[.pbix](LLM%20Evaluation%20KPI%20Dashboard.pbix)**

![Quality Overview](screenshots/01-quality-overview.png)

Power BI, DAX, star schema over four fact tables. 80 raters, 24 auditors, 8 languages, 26 weeks. Synthetic data.

## What it shows

**A guideline change in week 14 degraded four independent measures at once.** Cohen's Kappa 0.755 to 0.539, quiz pass rate 78.7% to 34.2%, disagreement rate 4.7% to 8.3%, audit lag 3.5 to 8.9 days. All recovered within four weeks.

**Over-rejection was a team problem, not a language problem.** Arabic raters reject at 11.4% against an auditor-confirmed rate of 5.8%, the lowest of any language. Four of the eight flagged raters are Arabic and all four sit on one team.

**Quiz performance predicts rating quality.** Correlation of +0.64. Raters scoring under 75 average 79.0% F1, those at 85 or above average 90.3%.

**Model releases show a measurable trade-off.** The v2.1 factuality tune cut factual errors 29% and raised tone issues 30%. Pass rate still improved, 74.2% to 76.5% to 80.8% across three releases, but the trade would have been invisible without per-label tracking.

## Pages

| Page | Question |
|---|---|
| Quality Overview | Where is the programme this week, and what moved |
| LLM Quality | How good are the responses, and what is wrong with them |
| Rating Accuracy | Where are raters wrong, by label and by language |
| Rater Watchlist | Which raters need attention, and why |
| Inter-Rater Reliability | Do two raters agree, and does that agreement mean anything |
| Production | How much work got done, and how much produced nothing |
| Quiz | Does quiz performance predict rating quality |
| Auditor Stats | Is the audit function keeping up, and are auditors calibrated |

<details>
<summary><b>Data model</b></summary>

Star schema. Four facts, six dimensions.

- `Fact_Production` — 123,000 rated jobs, one row per job
- `Fact_Audit` — 483,000 rows, one per job per label
- `Fact_IRR` — 40,000 double-rated items
- `Fact_Quiz` — 1,956 weekly attempts

Dimensions: Rater, Auditor, Queue, Label, Model, Date.

**Ragged grain.** `item_rejected` is scored on every audited job. The five content labels are only scored on mutually ratable items, meaning neither the rater nor the auditor rejected them. Scoring content labels on rejected items would have invented true negatives and inflated accuracy.

**Date anchoring.** Every page is anchored on `rating_date` rather than `audit_date`, so quality metrics and their denominators cover the same population. Audit lag averages 4.2 days, so anchoring on audit date would smear each week's quality across two or three rating weeks.

</details>

<details>
<summary><b>Key measures</b></summary>

Cohen's Kappa, chance-corrected agreement:

```dax
Cohens Kappa = DIVIDE([Raw Agreement] - [Pe Expected Agreement], 1 - [Pe Expected Agreement])
```

Raw agreement runs at 0.94 while Kappa sits at 0.80. Both are on the dashboard, because raw agreement is inflated by labels where both raters say "no" most of the time, and Kappa is what says whether the agreement means anything.

Over-rejection detection, combining rate with correctness:

```dax
Over-Rejection Flag =
VAR RaterRate = [Reject Rate]
VAR TeamRate  = CALCULATE([Reject Rate], REMOVEFILTERS(Dim_Rater))
VAR RaterPrec = [Reject Precision]
RETURN
    SWITCH(TRUE(),
        ISBLANK(RaterRate),                            BLANK(),
        RaterRate > TeamRate * 1.5 && RaterPrec < 0.6, "Review - likely over-rejecting",
        RaterRate > TeamRate * 1.5,                    "High reject rate",
        RaterPrec < 0.6,                               "Low reject precision",
        "Normal")
```

Rate alone is not evidence. A rater working a genuinely poor queue will legitimately reject more. It is high rate combined with low precision that identifies someone rejecting items they should have rated.

</details>

## About the data

Synthetic, generated for this piece. No client or production data, and no real rater, auditor or queue is represented.

It was built to behave like a real programme rather than to look tidy: rater skill varies, a handful of raters game the workflow by over-rejecting, audit lag responds to backlog pressure, and a guideline change in week 14 degrades quality before the team recovers. The findings above follow from that design.

The .pbix carries its data cached in the model, so it opens and works without the source files. Refreshing is the only thing it will not do.
