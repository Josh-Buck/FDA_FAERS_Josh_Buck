# Data Provenance and Regeneration Plan

Purpose: ensure the analysis can be regenerated on request (for example, if a reviewer asks us to
reproduce results months or a year from now). This documents what to preserve, what can be
re-obtained, and how to rebuild the pipeline end to end.

## 1. Raw data sources

| Source | Where | Re-obtainable? | Action |
|---|---|---|---|
| FAERS Q1-Q4 2025 ASCII | `/Users/joshbuck/Downloads/faers_ascii_2025q1..Q4/ASCII/` | Yes, public FDA portal | Archive the 4 quarterly zips anyway, in case the FDA revises a quarter |
| DrugBank 6.0 XML | `/Users/joshbuck/Downloads/full_database_drugbank.xml` | Only with a license | Keep your licensed copy; do NOT commit or redistribute. Record the release (DrugBank 6.0, 2024) |

FDA FAERS portal: https://fis.fda.gov/extensions/FPD-QDE-FAERS/FPD-QDE-FAERS.html

## 2. Reproducibility anchors (CRITICAL: not identically re-obtainable, must preserve)

- **`random.seed(42)`** in notebook 02 makes the 20-drug down-sampling and pair extraction deterministic. Do not change it.
- **RxNorm cache** `data/cache/rxnorm_mapping_cache.json` (64,552 cached API results). The RxNav API evolves over time, so a fresh run a year from now could return different mappings. This cache freezes the exact mappings used. PRESERVE IT. Same for the **ATC cache** `data/cache/atc_mapping_cache.json`.
- If the caches are lost, the drug-name standardization (and therefore pair counts and every downstream number) may shift slightly.

## 3. Environment

- Python 3.9.6; pandas 2.3.3, numpy 2.0.2, scikit-learn 1.6.1, scipy 1.13.1, statsmodels, networkx, matplotlib, seaborn.
- Freeze a lock file (`pip freeze > requirements-lock.txt`) and store it with the backup so the exact versions are recoverable.

## 4. Pipeline order to regenerate

Notebooks run in sequence; only notebook 02 needs the raw FAERS files. Everything downstream reads CSVs.
02 (data prep) -> 03 (ROR/signals) -> 04 (DrugBank validation) -> 05 (ATC) -> 07 (temporal) -> 08 (classes/validation) -> 09 (stats) -> 10 (figures) -> 11/13 (network/centrality) -> 14 (AUC).

## 5. Verification checksums (latest seed=42 run, verified 2026-06-04)

If a regeneration matches these, it reproduced correctly:
- 1,617,444 raw demo -> 1,469,305 deduplicated cases
- 7,801,018 raw drug -> 6,645,494 (cohort) -> 6,645,476 (cleaned)
- 823,213 cases with outcome data; 418,587 serious (50.8%)
- 64,552 names -> 59,316 mapped (91.9%) -> 30,575 concepts
- 556,666 patients with 2+ drugs; median 1, mean 3.119 drugs/patient
- 14,925,815 raw pair-observations; 2,211,481 unique pairs pre-filter
- 226,153 final pairs; 8,321,499 observations; 5,323,633 serious (64.0%)
- 52,203 signals; 5,433 high-confidence; 300 priority; 81 known / 69 novel / 61 cleaned

## 6. Known consistency caveat

The saved `results/` CSVs (temporal stability, the validation 2x2 table showing 52,319 signals) were
produced before the seed=42 re-run; the current pipeline yields 52,203 signals. Re-run notebooks
07/08/09 before final submission to bring downstream tables fully in sync. Conclusions do not change.

## 7. Backup checklist (store off-machine: external drive + cloud)

- [ ] FAERS Q1-Q4 2025 raw zips
- [ ] DrugBank 6.0 XML (licensed; private)
- [ ] `data/cache/` (RxNorm + ATC caches) <- most important, cannot be regenerated identically
- [ ] `data/intermediate/`, `data/signals/`, `data/validated/`, `results/`, `figures/`, `network/`
- [ ] The notebooks and `requirements-lock.txt`
- [ ] Git repo (already remote on GitHub); the `v1-original-outputs` tag is a snapshot
