# DemoBlaze Performance Testing

Performance testing suite for the [DemoBlaze E-Commerce Web Application](https://www.demoblaze.com) using Apache JMeter 5.6.3, executed via GitHub Actions CI/CD with results published to GitHub Pages.

📊 **[View Live HTML Dashboard Reports](https://hasbyqa.github.io/Performance-test)**

---

## Overview

| Attribute | Value |
|---|---|
| Application | DemoBlaze Product Store |
| Testing Tool | Apache JMeter 5.6.3 |
| Execution | Headless mode via GitHub Actions |
| Prepared By | AmaliTech QA Team |
| Report Date | March 17, 2026 |

---

## How It Works

Each CI/CD step runs the same `.jmx` file but activates only one thread group at a time by passing a unique `-J` property. All other thread groups default to `0` threads and produce no load.

```
CI Step 1: -Jbaseline_threads=50   → only Baseline fires  → reports/baseline/html
CI Step 2: -Jload_threads=150      → only Load fires       → reports/load_150/html
CI Step 3: -Jpeak_threads=400      → only Peak fires       → reports/load_300/html
CI Step 4: -Jstress_threads=800    → only Stress fires     → reports/stress_500/html
CI Step 5: -Jsoak_threads=300      → only Soak fires       → reports/endurance/html
```

All 5 reports are deployed to GitHub Pages after each run.

---

## Test Scenarios

| Scenario | Users | Ramp-up | Loops | Duration Assertion | Think Time |
|---|---|---|---|---|---|
| Baseline Test | 50 | 10s | 1 | None | 1000ms |
| Load Test | 150 | 15s | 1 | None | 1000ms |
| Peak Load Test | 400 | 20s | 2 | 500ms on Login, Add to Cart, Place Order | 800ms |
| Stress Test | 800 | 25s | 2 | 500ms on Login, Add to Cart, Place Order | 500ms |
| Soak Test | 300 | 15s | Infinite (300s) | 500ms on Login, Add to Cart, Place Order | 500ms |

Each virtual user executes 5 transactions in sequence: **Browse Homepage → Login → View Product → Add to Cart → Place Order**.

---

## Results Summary

**17,044 total requests · 7 failures · 0.04% error rate**

| Transaction | Samples | Error% | Avg (ms) | 95th Pct (ms) | Throughput |
|---|---|---|---|---|---|
| 01_Homepage | 3,436 | 0.00% | 462 | 1,009 | 5.75/s |
| 02_Login | 3,436 | 0.12% | 705 | 1,878 | 5.76/s |
| 03_View_Product | 3,393 | 0.00% | 415 | 512 | 5.73/s |
| 04_Add_To_Cart | 3,393 | 0.03% | 521 | 1,103 | 5.73/s |
| 05_Place_Order | 3,386 | 0.06% | 1,627 | 2,497 | 5.76/s |
| **TOTAL** | **17,044** | **0.04%** | **745** | **2,204** | **28.44/s** |

All 7 failures were **502 Bad Gateway** responses on Login (4), Add to Cart (1), and Place Order (2) under combined stress load.

---

## Performance Goals

| Goal | Target | Result |
|---|---|---|
| Average Response Time | < 2,000ms | ✅ PASS — 745ms overall |
| 95th Percentile | < 2,000ms | ⚠️ MARGINAL FAIL — 2,204ms (driven by Place Order at 2,393ms) |
| Error Rate | < 1% | ✅ PASS — 0.04% |
| Login Stability | < 1% errors | ✅ PASS — 0.12% |
| Place Order Performance | < 2,000ms avg | ⚠️ FAIL — 95th pct reached 2,393ms |
| Scalability | Gradual degradation | ✅ PASS — no sudden collapse observed |

---

## Key Findings

- **Login** — 4/7 total failures (502 Bad Gateway). Auth service reached capacity under combined stress load. 95th pct: 1,878ms.
- **Place Order** — Weakest transaction. Average 1,627ms, 95th pct 2,393ms, APDEX score 0.212 (Poor). Write-heavy backend operations under load cause database contention.
- **Homepage & View Product** — Zero failures across all scenarios. Static content delivery layer is not a bottleneck.
- **Autoscaling** — DemoBlaze runs on Google Cloud and autoscales, which kept the overall error rate at 0.04% despite aggressive load. 502 errors mark the moments scaling could not keep pace with sudden spikes.

---

## Repository Structure

```
.
├── .github/
│   └── workflows/
│       └── performance-test.yml   # GitHub Actions CI/CD pipeline
└── demoblaze_perf_test.jmx        # JMeter test plan (single parameterized thread group)
```

---

## Documents

- 📄 [Performance Test Plan](https://docs.google.com/document/d/1YsdDtzpf44kiWU_i9brYFPWFoUvub0h1Y8KCxVWImVg/edit)
- 📄 [Final Performance Report](https://docs.google.com/document/d/1iCKyGEv333Uo2GbYo50a8q2-2HywGUvenb-Cu31E-7s/edit)
