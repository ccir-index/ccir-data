# CCIR Data

**Compute Credit Index Research LLC — Published Index Data**

This repository contains the complete daily history for all CCIR GPU compute reference indices. All data is published under [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/) and is freely citable in credit agreements, collateral valuation, and covenant trigger design.

Live indices and documentation at [ccir.io](https://ccir.io).

---

## Files

| File | Index | Description |
|------|-------|-------------|
| [`cri-b200/CRI-B200-history.csv`](./cri-b200/CRI-B200-history.csv) | CRI-B200 | NVIDIA B200 SXM — frontier training; next-gen displacement signal |
| [`cri-h200/CRI-H200-history.csv`](./cri-h200/CRI-H200-history.csv) | CRI-H200 | NVIDIA H200 SXM — large-model training; memory-bound inference |
| [`cri-h100/CRI-H100-history.csv`](./cri-h100/CRI-H100-history.csv) | CRI-H100 | NVIDIA H100 SXM — **primary collateral reference rate** |
| [`cri-a100/CRI-A100-history.csv`](./cri-a100/CRI-A100-history.csv) | CRI-A100 | NVIDIA A100 SXM — prior-gen; cascade velocity signal |
| [`cri-l40s/CRI-L40S-history.csv`](./cri-l40s/CRI-L40S-history.csv) | CRI-L40S | NVIDIA L40S — inference-optimized; commodity cascade pricing |
| [`cri-r/CRI-R-history.csv`](./cri-r/CRI-R-history.csv) | CRI-R | Hyperscaler + RunPod rate cards — enterprise context (not a benchmark index) |
| `cri-r/CRI-R-{date}.csv` | CRI-R | Daily snapshots by observation date |

All history files are append-only. Each daily run adds one row. No historical values are modified after publication.

---

## CSV Schema

All spot index history files (`CRI-*-history.csv`) share the following schema:

| Field | Type | Description |
|-------|------|-------------|
| `observation_date` | DATE | Publication date (YYYY-MM-DD) |
| `methodology_version` | STRING | Methodology version under which this value was calculated (e.g. `1.0.0`) |
| `index_name` | STRING | Index identifier (e.g. `cri_h100`) |
| `index_value` | FLOAT | Trailing 7-day median $/GPU-hour (4 decimal places) |
| `confidence_flag` | STRING | `high` \| `medium` \| `low` — see below |
| `source_count` | INT | Number of distinct data sources contributing to this value |
| `n_valid_days` | INT | Calendar days in window with 2+ distinct sources contributing |
| `total_observations` | INT | Total qualifying listings in the 7-day window |
| `obs_p25` | FLOAT | 25th percentile of per-source medians |
| `obs_p75` | FLOAT | 75th percentile of per-source medians |
| `obs_min` | FLOAT | Minimum per-source median |
| `obs_max` | FLOAT | Maximum per-source median |
| `window_start` | TIMESTAMP | Start of 7-day calculation window (UTC) |
| `window_end` | TIMESTAMP | End of 7-day calculation window (UTC) |
| `computed_at` | TIMESTAMP | Timestamp of Gold computation run (UTC) |

### CRI-R Schema

CRI-R rate card files contain one row per source/model/tier combination:

| Field | Description |
|-------|-------------|
| `observation_date` | Publication date |
| `source_id` | Provider identifier (`GCP_BILLING`, `AZURE_RETAIL`, `AWS_BULK`, `RUNPOD_SPOT`) |
| `gpu_model_canonical` | Canonical GPU name |
| `instance_tier` | Tier classification (`hpc_premium`, `spot`, `managed_secure`, `managed_community`) |
| `rate_median_usd` | Median $/GPU-hour for this source/model/tier |
| `rate_min_usd` / `rate_max_usd` | Min/max observed rates |
| `sku_count` | Number of SKUs contributing |
| `methodology_version` | Methodology version |

---

## Confidence Flags

| Flag | Criteria | Meaning |
|------|----------|---------|
| `high` | source_count ≥ 3 AND n_valid_days ≥ 5 | Full data depth — standard trigger use |
| `medium` | source_count ≥ 2 AND n_valid_days ≥ 3 | Adequate depth — trigger use with buffer |
| `low` | Otherwise | Thin data — informational only; not appropriate for standalone credit reference without lender discretion |

A **valid day** is a calendar day on which at least 2 distinct source IDs contributed qualifying observations. Single-source days do not count toward `n_valid_days`.

---

## Citing This Data

### In Credit Agreements

```
"The Compute Reference Rate means the CRI-H100 index value as published by
Compute Credit Index Research LLC (ccir.io) for the most recent date on which
a high or medium confidence value is available, calculated in accordance with
CCIR Methodology v1.0.0 or any successor version adopted in accordance with
the CCIR Change Control Policy."
```

### Pinning to a Specific Version

The `methodology_version` column in every CSV row identifies the exact methodology version under which that value was calculated. To pin a credit agreement to a specific methodology version, reference the version string in the citation (e.g. `CCIR Methodology v1.0.0`).

### Linking to a Specific Value

GitHub commit history provides an immutable record of every publication event. Each commit message includes the observation date. A specific historical value can be cited by commit hash:

```
https://github.com/ccir-index/ccir-data/blob/{commit-hash}/cri-h100/CRI-H100-history.csv
```

---

## Data Lineage & Audit Trail

This repository contains **Gold layer aggregates only** — no raw listings, no individual offer identifiers, no provider-level detail beyond what is in CRI-R.

Full audit trail is maintained in the production pipeline:

```
Raw API response bytes
  → snapshot_sha256 (SHA-256 hash, computed at collection time, Bronze)
  → raw_payload (complete JSON, Bronze)
  → Bronze row_id (UUID)
    → Silver bronze_row_id (foreign key)
      → Gold observation_date / computed_at
        → CSV row (this repository)
          → GitHub commit (immutable external record)
```

Delta Lake time-travel on the production ADLS Gen2 storage preserves full Bronze/Silver/Gold state for any historical timestamp. Pre-transaction audit coordination available on request.

---

## Data Sources

| Source ID | Provider | Protocol |
|-----------|----------|----------|
| `VASTAI_SPOT` | Vast.ai | REST API (no auth) |
| `SHADEFORM_MKT` | Shadeform | REST API (free key) |
| `RUNPOD_SPOT` | RunPod | GraphQL (free key) |
| `GCP_BILLING` | Google Cloud | REST API (free key) |
| `AZURE_RETAIL` | Microsoft Azure | REST API (no auth) |
| `AWS_BULK` | Amazon Web Services | REST API (no auth) |

No submissions are accepted from any market participant. All collection is automated via public APIs. The `collection_method` field in the production Bronze layer records `api_rest` or `api_graphql` for every row.

---

## Methodology & Governance

Complete methodology and governance documentation:

- **Methodology:** [github.com/ccir-index/ccir-methodology/METHODOLOGY-v1.0.0.md](https://github.com/ccir-index/ccir-methodology/blob/main/METHODOLOGY-v1.0.0.md)
- **Governance:** [github.com/ccir-index/ccir-methodology/GOVERNANCE-v1.0.0.md](https://github.com/ccir-index/ccir-methodology/blob/main/GOVERNANCE-v1.0.0.md)
- **IOSCO Package:** [github.com/ccir-index/ccir-methodology/blob/main/CCIR-IOSCO-Governance-Package-v1.0.0.pdf](https://github.com/ccir-index/ccir-methodology/blob/main/CCIR-IOSCO-Governance-Package-v1.0.0.pdf)

CCIR indices are governed with reference to the [IOSCO Principles for Financial Benchmarks (July 2013)](https://www.iosco.org/library/pubdocs/pdf/IOSCOPD415.pdf).

---

## Update Schedule

Data is published daily by **17:00 UTC**. The production pipeline runs on Azure Databricks (ccir-workspace). CSV export to this repository is automated via the Databricks export_snapshots notebook, which commits daily via the GitHub API.

---

## Contact

**Compute Credit Index Research LLC**
research@ccir.io · [ccir.io](https://ccir.io)

---

*All data published under [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/). Freely citable with attribution to Compute Credit Index Research LLC.*
