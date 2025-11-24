# CEO Succession Framing – Paper Repository

This repository contains the reproducible analysis accompanying my master’s thesis:

> **Title:** Succession Framing as Identity Shock: Mapping Founder CEO Transitions onto the CORE Performance System and Stakeholder Responses  
> **Author:** Rong Xia (Columbia University, QMSS)  
> **Program:** MA in Quantitative Methods in the Social Sciences (QMSS), Fall 2025  

The goal of this repo is to provide a clean, self-contained analysis layer for the paper.  
All heavy data collection, model training, and WRDS queries are *archived* and **not** re-run here.

---

## 1. Project Overview

CEO transitions are not only management events; they are also **identity events**.  
When a CEO leaves, firms must explain **who they will be next** – to investors, employees, and the public.

This project treats CEO succession as an **organizational identity shock** and asks:

1. **RQ1 (Descriptive):**  
   How do firms frame CEO succession in SEC 8-K Item 5.02 filings along the CORE dimensions –  
   **Capacity (C)**, **Opportunity (O)**, and **Relevant Exchanges (RE)**?

2. **RQ2 (Explanatory):**  
   Are these framing strategies associated with stakeholder responses such as  
   **short-window stock market reactions** and **longer-term performance / innovation**?

3. **RQ2-Extension (Cross-channel):**  
   Do firms **maintain or shift** their initial framing when they move from  
   formal 8-K announcements to **earnings calls**?

To answer these questions, I combine:

- The **CORE performance model** (Marshall, Aguinis, & Beltran, 2024)  
- **NLP / weak supervision / multi-label BERT** for succession framing  
- A standard **event-study** around CEO succession 8-K filings  
- A cross-corpus comparison between **8-K** and **earnings call transcripts**

---

## 2. Conceptual Framework: CORE Model + Succession Framing

The analysis is built on the **CORE performance model** Marshall, Aguinis, & Beltran, 2024:

> **Performance (P) = Capacity (C) + Opportunity (O) + Relevant Exchanges (RE)**

I adapt CORE to CEO succession framing as follows:

- **Capacity (C)** – competence and readiness of the successor  
  - biography, track record, leadership experience, internal promotion language  
- **Opportunity (O)** – strategic direction and renewal  
  - growth, transformation, realignment, long-term plan, market opportunities  
- **Relevant Exchanges (RE)** – relational legitimacy and governance  
  - board approval, stakeholder support, compensation agreements, process legitimacy  

The key idea is to see each CEO succession announcement as a **framing choice** in the  
(C, O, RE) space – from “procedural continuity” to “strategic renewal” to “comprehensive legitimacy”.

---

## 3. Repository Structure

This repo is deliberately split into:

- A **clean, frozen layer** used by the paper notebook
- An **archival layer** (not re-run) that documents scraping / training

```text
.
├── notebooks/
│   ├── paper_notebook.ipynb
│   │   # Clean analysis for the paper:
│   │   # - reads from data_frozen/ and outputs_frozen/
│   │   # - reproduces all figures/tables for RQ1, RQ2, and EC appendix
│   └── framing_archive_2025-11-23.ipynb
│       # Full pipeline archive: scraping, weak labels, BERT training, WRDS queries
│
├── data_frozen/
│   ├── item502_filings_clean.csv
│   ├── item502_core_bootstrap_scores.csv
│   ├── core_seed_labels.csv
│   ├── 8k_events_with_funda.csv
│   ├── event_window_CAR.csv
│   ├── earnings_calls.csv
│   ├── earnings_call_core.csv
│   ├── 8k_call_aligned_core.csv
│   ├── ec_vs_8k_firm_core.csv
│   └── core_strategy_cards.json
│
├── outputs_frozen/
│   ├── figs/
│   │   # All paper figures, exported as .png
│   └── tables/
│       # All paper tables, exported as .csv/.tex
│
├── src/
│   ├── layer1_fill_clean_sec.py        # SEC 8-K scraping + cleaning
│   ├── layer2_core_bert_bootstrap.py   # weak supervision + multi-label BERT (CORE)
│   ├── layer3_core_strategies.py       # RQ1: framing combos, clusters, simplex, time trends
│   ├── layer4_core_to_market.py        # RQ2: CORE → CAR regressions (CRSP/Compustat)
│   └── layer5_earnings_calls.py        # EC: CORE on earnings calls + cross-channel alignment
│
├── environment.yml / requirements.txt
├── LICENSE
└── README.md
```
**Note:**  
All files in `data_frozen/` are analysis-ready and small enough to share.  
They do **not** contain any raw WRDS data (CRSP/Compustat/CIQ tables).  
See **Data Sources & Access** below.

---

## 4. Data Sources & Access

### 4.1 Public data (shipped in `data_frozen/`)

These files are derived from public sources and can be safely shared:

- **item502_filings_clean.csv**  
  - 306 SEC 8-K Item 5.02 filings that mention CEO succession  
  - cleaned text of the Item 5.02 block + metadata (company, date, URL)

- **item502_core_bootstrap_scores.csv**  
  - CORE probabilities and binary tags for each 8-K filing  
  - columns: `Capacity_score`, `Opportunity_score`, `Relevant_Exchanges_score`, plus `_pred`

- **core_seed_labels.csv**  
  - seed dictionaries + weak labels used to train the CORE classifier

- **core_strategy_cards.json**  
  - sentence-level “strategy cards” with the top C/O/RE snippets per filing  
  - used for qualitative inspection and examples in the paper

### 4.2 Restricted data (NOT included in raw form)

The following rely on licensed WRDS / CIQ data and are therefore only stored as  
lightweight, derived outputs:

- **event_window_CAR.csv**  
  - event-window abnormal returns (`CAR_1d`, `CAR_3d`) around each 8-K filing  
  - computed from CRSP but only the CARs themselves are shared

- **8k_events_with_funda.csv**  
  - per-event firm characteristics (e.g., log assets, log market cap, ROA, firm age, SIC2)  
  - derived from Compustat; **no raw Compustat rows** are exposed

- **earnings_calls.csv / earnings_call_core.csv / ec_vs_8k_firm_core.csv**  
  - aggregated earnings call transcripts and their CORE scores  
  - sourced from S&P Capital IQ Transcripts (WRDS), but **only processed text and scores** are shared

If you have WRDS access, you can rebuild these from the archive notebook / scripts.  
Otherwise, you can still run the paper notebook using the frozen CSVs in `data_frozen/`.

---

## 5. Method in One Slide: 5-Layer Pipeline

The full pipeline is organized into **five layers** (implemented in `src/` and the archive notebook):

1. **Layer 1 – fill+clean (SEC 8-K corpus)**  
   - Query SEC’s open EDGAR endpoints (`efts.sec.gov`, `data.sec.gov`)  
   - Filter for 8-K Item 5.02 filings mentioning CEOs  
   - Parse HTML, isolate Item 5.02 text, and clean boilerplate  
   - **Output:** `item502_filings_clean.csv`

2. **Layer 2 – Bootstrap multi-label BERT (CORE classifier)**  
   - Build weak lexicons for Capacity/Opportunity/RE (SEC-style roots + phrases + proximity rules)  
   - Generate noisy labels for each filing (`core_seed_labels.csv`)  
   - Fine-tune a multi-label **BERT (bert-base-uncased)** on these weak labels  
   - Use **bootstrap** over random seeds to stabilize predictions (macro-F1 ≈ 0.81 ± 0.04)  
   - **Output:** `item502_core_bootstrap_scores.csv`

3. **Layer 3 – CORE strategies (RQ1)**  
   - Use CORE scores to explore framing patterns in 8-K filings:  
     - combo frequencies (C / C+RE / C+O / C+O+RE)  
     - K-means clustering + PCA visualization  
     - CORE simplex (ternary) plots  
     - yearly trends with bootstrap confidence intervals  
   - **Output:** all paper figures + `core_rq1_summary.json`

4. **Layer 4 – CORE → Market Reaction (RQ2)**  
   - Map 8-K events to CRSP permno (via CIK → gvkey → permno link tables)  
   - Compute event-window CARs (e.g., `CAR[-1,+1]`, `CAR[-3,+3]`) using a market-model estimation window  
   - Run OLS with HC3 and variants (industry FE, PCA-based framing indices, balance metrics)  
   - **Main result:** no robust evidence that CORE framing predicts short-window CARs

5. **Layer 5 – Earnings Calls with CORE (Cross-channel)**  
   - Pull earnings call transcripts from Capital IQ Transcripts (WRDS)  
   - Filter management speech; apply the same CORE classifier  
   - Compare 8-K vs call framing within firms/events:  
     - **ΔCORE (call – 8-K)**  
     - semantic similarity via sentence-BERT  
     - firm-level CORE profiles across channels  
   - Used to interpret whether 8-K framing is event-specific identity work vs general linguistic style

The **paper notebook** (`notebooks/paper_notebook.ipynb`) uses **only Layers 2–5 outputs**  
and does **not** re-run scraping or model training.

---

## 6. Reproducing the Paper Results

### 6.1 Environment

You can use either **conda** or **pip**. A minimal setup:

```bash
# conda example
conda create -n succession-core python=3.10
conda activate succession-core
pip install -r requirements.txt
