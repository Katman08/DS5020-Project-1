# Customers Like You — GiftLane recommender (DS5020 Mini-Project 01)

PCA / SVD on the UCI Online Retail II log, to answer two things Sarah Okafor asked for:
a "customers who bought this also bought" box that runs nightly, and one readable picture
of the main customer groups.

## Success targets (J1)

Agreed on Day 1, from §1.4 of the brief. These are what the project is measured against:

| #   | Target                                                                          | Status                                                               |
| --- | ------------------------------------------------------------------------------- | -------------------------------------------------------------------- |
| 1   | The stored factors are **much smaller** than the full matrix — aim for ≥ 20×    | ◐ matrix is 30× smaller (136.2 MB → 4.5 MB); factor size pending ML2 |
| 2   | The nightly job (fit + score every product) runs in **under 60 seconds**        | ⬜ ML4                                                               |
| 3   | **Hit rate @ 10 beats the best-seller baseline** on held-out purchases (R3)     | ⬜ ML4                                                               |
| 4   | The customer map yields **3–6 groups marketing can name** (R4: no customer IDs) | ⬜ ML6                                                               |

## Team

| Role          | Owner        | Scope                                                              |
| ------------- | ------------ | ------------------------------------------------------------------ |
| Data Engineer | Alex Katzman | DE1–DE6 — raw data, cleaning, split, matrix                        |
| ML Engineer   | Adrien Mery  | ML1–ML7 — method choice, recommendations, evaluation, customer map |

Joint: J1 targets (above), J2 memo, J3 slides. Contribution log (C2): `CONTRIBUTIONS.md`.

## Data (Step 0)

UCI Online Retail II, downloaded Day 1 from
<https://archive.ics.uci.edu/dataset/502/online+retail+ii>.
Two sheets (`Year 2009-2010`, `Year 2010-2011`), **1,067,371 rows**, 1 Dec 2009 – 9 Dec 2011.
Licence CC BY 4.0.

The raw file is **not committed** (`data/` is gitignored). Put it at
`data/raw/online_retail_II.xlsx` before running anything.

## How to run

```bash
pip install -r requirements.txt
jupyter nbconvert --to notebook --execute --inplace build_matrix.ipynb   # ~30 s
```

Or just open `build_matrix.ipynb` and run it top to bottom. It rebuilds every output below
from the raw file in one pass (R6). Seed is fixed at 42 throughout.

| Output                                | What it is                                                  |
| ------------------------------------- | ----------------------------------------------------------- |
| `data/processed/train_matrix.npz`     | sparse customer × product matrix, **training rows only**    |
| `data/processed/customer_index.csv`   | matrix row → AnonID                                         |
| `data/processed/product_index.csv`    | matrix column → ProductCode                                 |
| `data/processed/test_holdout.csv`     | held-out `(customer_row, product_col)` pairs for evaluation |
| `data/processed/product_names.csv`    | ProductCode → Description, for naming segments (ML6)        |
| `data/processed/customer_country.csv` | matrix row → Country, for naming segments (ML6)             |

The ML notebook loads these — it should never read `data/raw/` or re-derive the matrix.

## Results so far

**Cleaning (DE2).** 1,067,371 → **794,163** rows. Dropped: 6,093 non-product stock codes
(postage, bank charges, adjustments, samples, gift vouchers, test rows), 241,000-odd lines with
no customer ID, 61 lines priced ≤ 0, and 26,402 exact duplicates. Cancellation lines are **kept**
and netted against purchases, so a returned order cancels out instead of counting as one —
6,885 customer/product pairs disappear that way.

**Split (DE3).** Time-based, cutoff **2011-10-09**: 675,720 training rows, 118,443 test rows.
Per R2 the hold-out keeps only products a customer had **never bought before** — 93,928 test
pairs become 48,467 after dropping 13,554 cold-start pairs and 31,907 repeat purchases.
43,800 of those are scoreable against the filtered matrix, covering 1,915 customers.

**Matrix (DE4, DE5).** Purchase value is `log1p(net purchase occasions)`; products kept if
bought by ≥ 10 customers (3,762 → 3,273).

|                 |                                  |
| --------------- | -------------------------------- |
| shape           | 5,454 customers × 3,273 products |
| non-zeros       | 394,764                          |
| sparsity        | 97.79%                           |
| dense (float64) | 136.2 MB                         |
| sparse (CSR)    | 4.5 MB — **30× smaller**         |

Every cleaning and modelling decision is written down with its reason in the notebook (R1).

## Privacy (R4)

Real customer IDs are replaced by a seeded random `AnonID` before anything is written to disk.
Nothing in `data/processed/` contains a real customer ID. The memo, slides and every figure use
segment names or anonymised indices only.

## Repository

```
build_matrix.ipynb      DE1–DE5: profile, clean, split, build the matrix
model.ipynb             ML1–ML7: method choice, recommendations, evaluation, customer map
                        (not created yet — owned by the ML Engineer)
requirements.txt        pinned versions
data/raw/               the Excel file (gitignored — download it yourself)
data/processed/         rebuilt by build_matrix.ipynb (gitignored)
```
