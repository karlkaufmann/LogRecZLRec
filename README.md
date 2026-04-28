# Hotel extra-services ranking (LogRec / ZLRec)

Learning-to-rank experiments for **per-email product recommendations** in a hotel extra-services context: each `booking_id` is one outbound message with multiple candidate products; labels indicate whether the guest purchased the offered item (`purchased ∈ {0,1}`). The goal is a **within-booking ranking** that surfaces products the guest is most likely to buy.

This public repository contains the **analysis notebook and safety tooling** only. Raw interaction tables, exports, and serialized model weights are **not** versioned here—bring your own dataset and run everything locally.

---

## Problem framing

| Concept | Definition |
|--------|------------|
| **Unit of ranking** | One email / campaign row group keyed by `booking_id` |
| **Positives** | Rows with `purchased = 1` |
| **Negatives** | Rows with `purchased = 0` (shown, not bought)—**do not invent negatives**; the notebook assumes real negatives already exist per group |
| **Target** | Order products so true purchases rank high (ranking metrics, not only pointwise AUC) |

Features combine **guest**, **stay**, and **product** signals (including categorical high-/low-cardinality handling and train-only aggregates where applicable). See `hotel_recommendation_system.ipynb` for the exact feature list and transformations.

---

## Methodology (high level)

1. **Data QA** — Duplicates, missingness, and sanity checks on label structure per group.
2. **Leak-safe splitting** — Grouped splits on `booking_id` so the same reservation never appears in both train and validation/test.
3. **Feature engineering** — Pipelines aligned with training (no test statistics leaking into train features).
4. **Models** — **Logistic regression** baseline vs **LightGBM LambdaRank** (listwise objective appropriate for graded/implicit relevance from binary purchase).
5. **Evaluation** — Ranking-focused reports: **NDCG@K**, **HitRate@K**, **Recall@K** (K chosen in-notebook), plus qualitative checks on score distributions.
6. **Production path (optional)** — ONNX export, metadata, and HTTP batch scoring are described in the internal handoff notes; the runnable **Docker / FastAPI** layout lives under `deploy/` (see [`deploy/README.md`](deploy/README.md)). That directory is **not** part of the minimal public snapshot policy for this repo; clone the full workspace or vendor `deploy/` separately if you need the serving bundle.

---

## Repository layout

| Path | Role |
|------|------|
| `hotel_recommendation_system.ipynb` | End-to-end EDA → features → train → evaluate → export notes |
| `.gitignore` | Blocks data archives, DBs, env secrets, and common model/binary formats |
| `.cursor/rules/` | Agent rules: public-repo safety + commit hygiene |
| `.agent/skills/public-repo-packaging/` | Checklist before publish/commit |

---

## Reproducing the analysis

1. **Environment** — Python 3.11+ recommended; install `pandas`, `numpy`, `scikit-learn`, `lightgbm` (and optional `onnxmltools` / `onnxruntime` if you extend export cells). Pin versions to match your compliance stack.
2. **Data** — Place your interaction CSV where the notebook expects it (the shipped notebook references a **local filename**; update the path cell to your file). Expected columns align with the feature engineering section—inspect the first data-loading cells before running the full pipeline.
3. **Execution** — Run cells in order. Heavy cells (grid search, CV) are intentionally sequential; set smaller grids for a quick smoke run.
4. **Seeds** — The notebook fixes Python / NumPy seeds for repeatable splits and model init where the library allows.

---

## What is intentionally out of scope for this remote

To avoid leaking **PII-heavy logs**, **commercial datasets**, or **byte-identical model artifacts**, the following stay **ignored** and should never be force-added: `*.csv`, `*.parquet`, `*.pkl`, `*.onnx`, archives, local DBs, `.env`, and typical model extensions (see `.gitignore`). If you add `evaluate_model.py` or evaluation markdown reports later, keep them free of absolute local paths and secrets.

---

## Contributing / extending

- Prefer **small, reviewable commits** with factual messages (see `.cursor/rules/commit-style.mdc`).
- Before opening a public PR or tagging a release, walk through `.agent/skills/public-repo-packaging/SKILL.md`.

---

## License / attribution

Add a `LICENSE` file if you open-source beyond a personal portfolio mirror. Until then, treat code and notebook prose as **all rights reserved** unless you state otherwise.

---

## Further reading

- **Serving & PHP integration pattern**: [`deploy/README.md`](deploy/README.md) (batch `POST /score`, model version in response, Docker runbook).
- **Detailed handoff / export checklist** (internal): `HANDOFF_PLAN.md` in a full checkout—not required to understand the ranking problem covered in the notebook.
