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
