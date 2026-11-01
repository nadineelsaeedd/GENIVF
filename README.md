# GenIVF-RAG

**A Data Audit and Leakage-Free Evaluation of Retrieval-Augmented Language Models for Personalised Ovarian Stimulation Treatment**

*Nadine Y. Elsaeed · M. Z. Rashad · Samir Elmougy · Rasha Saqr*  
Mansoura University · Egypt University of Informatics

📄 **Paper:** *Journal of Big Data*, Springer Nature — *under review*  
📦 **Dataset:** [Mendeley Data — Embryo dataset v1](https://doi.org/10.17632/x5s3ky9cpv.1) (Wang M., 2026, CC BY 4.0)  
📓 **Notebook:** [`GenIVF_RAG_Experiments_v8.ipynb`](./GenIVF_RAG_Experiments_v8.ipynb)

---

## Overview

This repository accompanies the paper and provides a **single, fully executable notebook** (`GenIVF_RAG_Experiments_v8.ipynb`) that reproduces every number, table, and figure in the paper — from raw data loading through the data audit, all machine-learning experiments, hybrid retrieval evaluation, and local LLM generation with and without retrieved cases.

**All random seeds are fixed.** The LLM step (Step 9) halts with an error if any API call fails; no fallback generator exists. This guarantees that every reported LLM result comes from a complete, uninterrupted run.

---

## Key findings

### 1 — Label leakage in a public IVF dataset

The [Mendeley Embryo dataset](https://doi.org/10.17632/x5s3ky9cpv.1) contains **25 outcome-specific imputed constants across 13 clinical columns** that appear almost exclusively in one pregnancy-outcome class (all p < 10⁻⁶, exact binomial test on the training split). These values affect **836 of 1,415 patients (59.1%)**.

| Model | AUROC — raw file | AUROC — after masking |
|---|---|---|
| Logistic Regression | 0.588 | 0.582 |
| **Histogram Gradient Boosting** | **0.858** | **0.621** |

The gradient-boosting AUROC drops by **0.237 points** once the constants are masked. Any model trained on the unmasked file should be expected to overestimate its performance by a comparable margin.

Examples of flagged values (training split):

| Column | Value | Non-pregnant patients | Pregnant patients | p |
|---|---|---|---|---|
| Basal progesterone | 1.39 | 116 | 0 | 9.7 × 10⁻²⁶ |
| Basal progesterone | 1.48 | 0 | 92 | 4.0 × 10⁻³⁸ |
| Female fasting glucose | 5.05 | 152 | 4 | 1.4 × 10⁻²⁷ |
| Prepared-semen density | 37.55 | 130 | 0 | 9.9 × 10⁻²⁹ |

### 2 — Leakage-free benchmark (1,415 patients, patient-level 80/20 split)

On corrected data, all models discriminate modestly. The best test AUROC is **0.621 [95% CI 0.558–0.684]** (histogram gradient boosting, baseline variables). Top permutation importances: age (Δ AUROC 0.039), AMH (0.028), uric acid (0.022).

### 3 — Retrieval quality

Hybrid BM25 + cosine retrieval with RRF (k = 60) consistently finds clinically closer patients than random selection:

| Retrieval | Mean \|Δ age\| (years) | Mean \|Δ log AMH\| | Mean \|Δ FSH\| (IU/L) | Outcome AUROC (k = 50) |
|---|---|---|---|---|
| Random | 4.88 | 0.65 | 2.69 | — |
| Dense | 3.42 | 0.45 | 2.01 | 0.599 |
| Sparse | 3.13 | 0.30 | 2.01 | 0.563 |
| **Hybrid** | **3.12** | **0.33** | **1.95** | **0.595** |

### 4 — LLM with and without retrieved cases

| System | Exact match with clinic | Valid JSON output |
|---|---|---|
| Population mode | 24.7% | — |
| RF regressor | 27.2% | — |
| Retrieval — all neighbours | 25.4% | — |
| Retrieval — successful neighbours | 20.1% | — |
| **LLM without retrieval** | **24.7%** | **100.0%** |
| **LLM + RAG** | **19.1%** | **100.0%** |

McNemar exact test (paired comparison): p = 0.026. Without retrieval, the LLM recommended **10 days for every patient**. With retrieval, it produced 3 distinct recommendations (10, 11, 12 days).

**Retrieval reduced AMH factual errors** in rationales from **35.1% → 8.7%** — the clearest benefit of RAG in this study.

---

## Repository structure

```
GENIVF/
│
├── GenIVF_RAG_Experiments_v8.ipynb   ← main notebook; reproduces all results
│
├── figures/
│   ├── fig1_workflow.png              ← study workflow diagram
│   ├── fig2_leakage.png              ← leakage detection and AUROC comparison
│   └── fig3_results.png              ← ROC curves, retrieval quality, Gn-duration agreement
│
├── data/
│   └── README.md                     ← dataset download instructions (file not included)
│
├── requirements.txt                  ← Python dependencies
├── LICENSE                           ← MIT license (code only)
└── README.md                         ← this file
```

---

## Reproducing the results

### 1. Install Python dependencies

```bash
pip install -r requirements.txt
```

### 2. Get the dataset

Download `Embryo dataset_2542.xlsx` from Mendeley Data:  
👉 https://doi.org/10.17632/x5s3ky9cpv.1

Place it so the notebook can find it:

```
data/Embryo dataset_2542.xlsx
```

The notebook searches several common paths (including `/content/` for Colab). Edit the `DATA_CANDIDATES` list in Step 1 if your path differs.

### 3. Set up Ollama for the LLM step

Install [Ollama](https://ollama.com) and pull the model:

```bash
ollama pull llama3.1:8b
ollama serve          # keep this running in a separate terminal
```

Then set the environment variables before launching the notebook:

```bash
export GENIVF_LLM_PROVIDER=openai_compatible
export GENIVF_LLM_MODEL=llama3.1:8b
export GENIVF_LLM_BASE_URL=http://localhost:11434/v1
```

> **To skip the LLM step** (runs Steps 1–8 only, no LLM results reported):  
> leave `GENIVF_LLM_PROVIDER` unset or set it to `none`.

### 4. Run the notebook

```bash
jupyter notebook GenIVF_RAG_Experiments_v8.ipynb
```

Run all cells in order. The notebook will:

| Step | What it does |
|---|---|
| 0 | Install / import dependencies |
| 1 | Configuration (paths, seeds, LLM settings) |
| 2 | Load `Embryo dataset_2542.xlsx`, translate Chinese column headers, collapse embryo rows → 1,415 patient records |
| 3 | Plausibility cleaning (360 out-of-range values → NaN) |
| 3b | **Data audit**: exact binomial test detects 25 class-specific constants in 13 columns; constants masked in train and test |
| 4 | Feature sets, patient-level 80/20 stratified split |
| 5 | ML outcome prediction on both raw (leaky) and leakage-free data; permutation importance |
| 6 | Hybrid BM25 + cosine retrieval knowledge base (training patients only) |
| 7 | Retrieval evaluation: clinical similarity + outcome AUROC; k selected by train leave-one-out CV |
| 8 | Protocol recommendation (Gn days): mode, RF regressor, retrieval variants; exploratory concordance analysis |
| 9 | LLM generation (with / without RAG) via Ollama; McNemar test; grounding checks |
| 9b | Rationale factual-consistency check (AMH statements vs. measured AMH) |
| 10 | Figures and full results export to `genivf_outputs_v8/` |

### Expected runtimes

| Steps | Approximate time |
|---|---|
| 1–8 (no LLM) | ~5–8 minutes |
| 9 (LLM, 283 × 2 patients, GPU) | ~10–15 minutes |
| 9 (LLM, 283 × 2 patients, CPU) | ~45–90 minutes |

LLM calls are **cached** in `genivf_outputs_v8/llm_generation_log.csv`. A re-run resumes from where it left off.

---

## Leakage detection — standalone function

The core detection logic (Step 3b) is self-contained and can be applied to any tabular clinical dataset:

```python
from scipy.stats import binomtest
import numpy as np, pandas as pd

def find_class_constants(df, target_col, min_count=10, alpha=1e-6):
    """
    For every non-integer value occurring >= min_count times in df,
    tests whether its distribution across outcome classes is consistent
    with the base rate using a two-sided exact binomial test.
    
    A value that passes the threshold was almost certainly imputed with a
    class-specific statistic (e.g. the within-class mean), leaking the label.
    
    Parameters
    ----------
    df         : pd.DataFrame  (training split only)
    target_col : str           binary outcome column (0/1)
    min_count  : int           minimum occurrences to test a value
    alpha      : float         significance threshold (default 1e-6)
    
    Returns
    -------
    pd.DataFrame with columns: column, value, mostly_in, n_non_preg, n_preg, p
    """
    base = df[target_col].mean()
    found = []
    for col in df.columns:
        if col == target_col:
            continue
        vc = df.groupby([col, target_col]).size().unstack(fill_value=0)
        for v, row in vc.iterrows():
            n0, n1 = int(row.get(0, 0)), int(row.get(1, 0))
            if float(v).is_integer() or n0 + n1 < min_count:
                continue  # skip genuine integer measurements
            p = binomtest(n1, n0 + n1, base).pvalue
            if p < alpha:
                found.append({'column': col, 'value': v,
                              'mostly_in': 'pregnant' if n1/(n0+n1) > base else 'non-pregnant',
                              'n_non_preg': n0, 'n_preg': n1, 'p': p})
    return pd.DataFrame(found).sort_values('p')
```

---

## Outputs

After a full run, `genivf_outputs_v8/` contains:

| File | Contents |
|---|---|
| `table_class_specific_constants.csv` | All 25 flagged values with p-values (Table 1 in paper) |
| `table_leakage_effect.csv` | AUROC before/after masking (Table 2) |
| `table_outcome_ml.csv` | ML prediction results (Table 3) |
| `table_retrieval.csv` | Retrieval similarity and outcome AUROC (Table 4) |
| `table_protocol_gn_days.csv` | Stimulation-duration agreement (Table 5) |
| `table_llm_gn_days.csv` | LLM agreement and grounding checks |
| `table_rationale_consistency.csv` | AMH factual-consistency check (Table 6) |
| `summary_metrics_v8.json` | All key numbers in one machine-readable file |
| `genivf_v8_dashboard.png` | Figure 3 (ROC curves, retrieval, duration agreement) |
| `genivf_v8_leakage.png` | Figure 2 (leakage detection and AUROC comparison) |
| `llm_generation_log.csv` | Full per-patient LLM log (cached; enables reruns) |

---

## Citation

If you use this code or the leakage-detection method, please cite:

```bibtex
@article{elsaeed2026genivf,
  title   = {A Data Audit and Leakage-Free Evaluation of Retrieval-Augmented
             Language Models for Personalised Ovarian Stimulation Treatment},
  author  = {Elsaeed, Nadine Y. and Rashad, M. Z. and Elmougy, Samir and Saqr, Rasha},
  journal = {Journal of Big Data},
  year    = {2026},
  note    = {Under review}
}
```

*Update with volume, issue, pages, and DOI once published.*

---

## License

**Code** (this repository): [MIT License](./LICENSE)  
**Dataset**: [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/) — see the [Mendeley Data page](https://doi.org/10.17632/x5s3ky9cpv.1) for attribution requirements.

---

## Contact

**Nadine Y. Elsaeed** — nadine.elsaeed@eui.edu.eg  
Egypt University of Informatics (EUI), Cairo, Egypt  
Mansoura University, Mansoura, Egypt
