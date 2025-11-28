---
title: "Aligning product and engineering OKRs"
description: "Set shared goals through impact and capacity balance instead of roadmap dates."
categories: [Leadership, Product]
tags: [okr, roadmap, leadership]
---

Product brings impact and revenue goals; engineering brings capacity and risk. I bridge the two languages with a measurable outcome plus a technical constraint in the same sentence.

## 3 rules
- **Make capacity explicit:** Cap team throughput using the last three iterations’ average; when adding new work, remove the same amount.
- **Expose risks:** Add architectural debt or scaling needs as their own KRs; measure progress with acceptance criteria.
- **Count experiments:** Make validation steps (A/B, MVP, launch plan) as visible as shipped features.

## Sample OKR
- **Objective:** Increase customer activation while keeping platform stability.
  - KR1: Grow activation rate from 15% → 20% (measure: weekly cohort).
  - KR2: Cut signup→first value from 24h to 12h.
  - KR3: Keep P99 API latency < 400ms, error rate < 0.2%.
  - KR4: Define error budget and SLO for the riskiest module; publish the dashboard.

## Cadence
- 30-minute “input check” before sprint planning to map OKRs/KRs to the new work list.
- For large items, write RFC → decision → implementation; define who approves.
- End of sprint: two slides only—metrics graphs + technical debt decisions.
