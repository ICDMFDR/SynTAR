# SynTAR — Synthetic Data Augmentation for Treatment Recommendation

**SynTAR** is a two-stage pipeline for personalised treatment recommendation from observational survival data.  
**Stage 0** (TabDiff) generates synthetic counterfactual patients to address treatment imbalance and data scarcity.  
**Stage 1** (TabSurv) uses a two-model TabPFN T-Learner to estimate individual treatment effects (CATE) on the augmented cohort.

> **Paper:** *SynTAR: Synthetic Augmentation for Treatment Recommendation in Survival Analysis*  
> Submitted to IEEE ICDM 2025

---

## Results

### Cross-Dataset Comparison (Mean ΔRMST%, 20 stratified resamples)

| Rank | Method | METABRIC | GBSG2 | TCGA-BRCA | DUKE | Mean Δ% | Win% |
|------|--------|----------|-------|-----------|------|---------|------|
| 1 | **SynTAR** (ours) | +7.82 | +5.71 | **+8.73** | **+6.32** | **+7.15** | **100%** |
| 2 | S-Learner | +10.18 | +7.52 | -0.68 | +4.74 | +5.44 | 75% |
| 3 | CoxPH-Joint | +8.86 | +8.42 | -1.79 | +3.16 | +4.66 | 75% |
| 4 | CUTs | +11.48 | +7.04 | +2.30 | -3.13 | +4.42 | 75% |
| 5 | LR-Joint | +10.17 | +7.04 | +1.44 | -2.68 | +3.99 | 75% |
| 6 | CausalForest | **+12.66** | +3.40 | +0.88 | -3.14 | +3.45 | 75% |
| 7 | XGBoost-Joint | +9.43 | +1.72 | -2.65 | +0.80 | +2.33 | 75% |
| 8 | EP-Learner | +7.42 | +2.14 | -1.58 | -1.80 | +1.54 | 50% |
| 9 | X-Learner | +4.32 | +1.34 | -3.29 | -0.13 | +0.56 | 50% |
| 10 | BITES | +4.27 | +0.34 | -1.87 | -0.77 | +0.49 | 50% |
| 11 | R-Learner | +4.22 | +0.64 | -4.61 | +1.17 | +0.35 | 75% |
| 12 | TARNet | +3.18 | +3.58 | -4.51 | -1.13 | +0.28 | 50% |
| 13 | T-Learner-GB | +7.15 | +5.71 | -10.85 | -0.94 | +0.27 | 50% |
| 14 | DragonNet | +3.14 | +1.34 | -5.53 | -0.77 | -0.46 | 50% |
| 15 | DR-Learner | +5.26 | -0.56 | -7.63 | -0.81 | -0.93 | 25% |

SynTAR is the **only method with 100% win rate** (positive ΔRMST% on every dataset × split pair).  
All values are mean ΔRMST% across 20 stratified resamples; 95% bootstrap CIs available in `output_report/tables/`.

### Stage-0 Ablation (SynTAR vs TabSurv on real data only)

| Dataset | TabSurv | SynTAR | Stage-0 Gain |
|---------|---------|--------|-------------|
| METABRIC | +11.26 | +7.82 | -3.43 |
| GBSG2 | +7.52 | +5.71 | -1.81 |
| TCGA-BRCA | -0.68 | +8.73 | **+9.41** |
| DUKE | +4.74 | +6.32 | +1.58 |
| **Mean** | +5.71 | **+7.15** | **+1.44** |

---

## Repository Structure

```
SynTAR/
├── config.py                   # Global config: datasets, seeds, hyperparameters
├── run_pipeline.sh             # Master pipeline script (see Usage)
│
├── s1_step1_generate.py        # Stage 1-A: train all 6 generators (single split)
├── s1_step1_recommend.py       # Stage 1-B: evaluate generators via TabSurv
├── s1_step1_compare_gen.py     # Stage 1-C: rank generators → best_generator.json
│
├── s2_prepare_splits.py        # Stage 2-0: write 20 identical splits to disk
├── s2_step1_generate.py        # Stage 2-1: generate TabDiff aug_10x per split
├── s2_step2_baselines.py       # Stage 2-A: run 15 baseline methods
├── s2_step2_syntar.py          # Stage 2-B: run SynTAR + KM curves + significance
├── s2_report_tables.py         # Stage 3: produce Tables 1–8 (CSV + LaTeX)
├── s2_report_figures.py        # Stage 3: produce Figures 1–9 (PNG)
│
├── input/
│   ├── METABRIC.csv
│   ├── GBSG2.csv
│   ├── TCGA_BRCA.csv
│   ├── DUKE.csv
│   └── splits/                 # Written by s2_prepare_splits.py
│
├── output_step1_generators/    # Stage 1 outputs
├── output_step2_baselines/     # Baseline results
├── output_step2_syntar/        # SynTAR results + KM curves
└── output_report/              # Final tables and figures
```

---

## Datasets

| Dataset | N | Treatment | Features | τ (yr) | Treat% | Event% | Cens% |
|---------|---|-----------|----------|--------|--------|--------|-------|
| METABRIC | 1,978 | Chemotherapy | 22 clinical | 10 | 21% | 58% | 42% |
| GBSG2 | 686 | Hormone therapy | 7 | 7 | 36% | 44% | 56% |
| TCGA-BRCA | 1,079 | Lumpectomy | 19 | 10 | 12% | 14% | 86% |
| DUKE | 281 | Chemotherapy | 34 | 4 | 17% | 12% | 88% |

Place raw CSV files in `input/` before running. Each CSV must contain columns:
`time`, `event`, and the treatment column listed above.

**Sources:**
- METABRIC: Curtis et al., *Nature*, 2012 — available from [cBioPortal](https://www.cbioportal.org/)
- GBSG2: Schumacher et al., *J. Clinical Oncology*, 1994 — available from  [`R package`](https://r-packages.io/datasets/GBSG2)
- TCGA-BRCA: TCGA Research Network, 2012 — available from [GDC Data Portal](https://portal.gdc.cancer.gov/)
- DUKE: Saha et al., British journal of cancer, 119(4), pp.508-516., 2018 — available from [Duke-Breast-Cancer](https://www.cancerimagingarchive.net/collection/duke-breast-cancer-mri/)

---

## Environment & Dependencies

### Requirements

- Python 3.10
- CUDA 12.1+ (GPU strongly recommended; CPU fallback available but slow)

### Installation

```bash
# 1. Clone the repository
git clone https://github.com/your-org/SynTAR.git
cd SynTAR

# 2. Create virtual environment
python3.10 -m venv t_env
source t_env/bin/activate          # Linux / macOS
# t_env\Scripts\activate           # Windows

# 3. Install dependencies
pip install --upgrade pip
pip install -r requirements.txt
```

### `requirements.txt`

```
torch==2.2.2
torchvision==0.17.2
numpy==1.26.4
pandas==2.2.1
scikit-learn==1.5.2
scikit-survival==0.23.1
lifelines==0.27.8
tabpfn==2.5.0
sdv==1.14.0
synthcity==0.2.9
xgboost==2.0.3
lightgbm==4.3.0
econml==0.15.1
matplotlib==3.8.4
seaborn==0.13.2
pyarrow==15.0.2
```

### HPC / Gadi setup

```bash
module load python3/3.10.4 cuda/12.8.0
source /scratch/sq95/tv9849/t_env/bin/activate
cd /scratch/sq95/tv9849/SynTAR
```

---

## Usage

### Quick reference

```bash
# Full pipeline: Stage 1 (generator screen) + Stage 2 (comparison) + reports
bash run_pipeline.sh

# Stage 1 only — screen all 6 generators (~2–3 hrs on GPU)
bash run_pipeline.sh --stage1

# Stage 2 only — SynTAR vs 15 baselines, 20 splits (~6–8 hrs)
bash run_pipeline.sh --stage2

# Stage 2 with TabDiff aug_10x, 20 splits — recommended for paper reproduction
bash run_pipeline.sh --compare

# Fast test: TabDiff, aug_10x, 5 splits (~2 hrs)
bash run_pipeline.sh --fast

# Debug: TabDiff, GBSG2+DUKE only, 2 splits (~15 min)
bash run_pipeline.sh --debug

# Resume Stage 2 from baselines (splits + generation already done)
STEP=2 bash run_pipeline.sh --compare

# Reports only (all steps already run)
STEP=3 bash run_pipeline.sh
```

### Step-by-step (manual)

```bash
# ── Stage 1: Generator screen (single split per dataset) ──────────────────
python s1_step1_generate.py                        # train all 6 generators
python s1_step1_recommend.py                       # evaluate via TabSurv
python s1_step1_compare_gen.py                     # select best generator

# ── Stage 2: SynTAR vs baselines (20 splits per dataset) ─────────────────
python s2_prepare_splits.py --n_splits 20          # write identical splits
python s2_step1_generate.py --generators TabDiff \
                            --aug_sizes aug_10x \
                            --n_splits 20          # generate per-split parquets
python s2_step2_baselines.py --n_splits 20         # run 15 baselines
python s2_step2_syntar.py    --n_splits 20         # run SynTAR

# ── Stage 3: Reports ──────────────────────────────────────────────────────
python s2_report_tables.py                         # Tables 1–8 (CSV + LaTeX)
python s2_report_figures.py                        # Figures 1–9 (PNG)
```


## Reproducing Paper Results

To reproduce the exact numbers in the paper (Tables 3–5, Figures 4–6):

```bash
# Step 1: Place datasets in input/
cp /path/to/METABRIC.csv    input/
cp /path/to/GBSG2.csv       input/
cp /path/to/TCGA_BRCA.csv   input/
cp /path/to/DUKE.csv        input/

# Step 2: Run full comparison pipeline
bash run_pipeline.sh --compare

# Step 3: Check outputs
ls output_report/tables/    # table3_full_comparison.csv, table4_cross_dataset_summary.csv, ...
ls output_report/figures/   # fig4_method_comparison.png, fig6_ablation.png, ...
ls output_step2_syntar/km_curves/   # km_METABRIC.png, km_GBSG2.png, ...
```

**Expected runtime:** ~4–5 hours on a single V100 GPU (Gadi gpuvolta).  
**Expected key outputs:**

| Output file | Content |
|-------------|---------|
| `output_step1_generators/best_generator.json` | `{"generator": "TabDiff", "aug_size": "aug_10x"}` |
| `output_step2_syntar/summary_step2_syntar.csv` | SynTAR mean ΔRMST% per dataset |
| `output_step2_baselines/summary_step2_baselines.csv` | All 15 baselines |
| `output_step2_syntar/significance_vs_baselines.csv` | Paired bootstrap p-values |
| `output_step2_syntar/km_curves/km_TCGA_BRCA.png` | KM: Followed vs Not-Followed REC |
| `output_report/tables/table3_full_comparison.csv` | Full Table 3 |
| `output_report/figures/fig4_method_comparison.png` | Figure 4 |

### Key hyperparameters (from `config.py`)

| Parameter | Value | Description |
|-----------|-------|-------------|
| `SEED` | 42 | Global random seed |
| `N_SPLITS` | 20 | Stratified resamples |
| `N_BOOT` | 1000 | Bootstrap CI iterations |
| `ALPHA` | 0.05 | Significance level |
| `AUG_SIZES` | `{aug_3x:2, aug_6x:5, aug_10x:9}` | Synthetic multipliers |
| `TABPFN_BATCH` | 512 | TabPFN inference batch size |
| Stage-0 epochs | 500 | TabDiff training epochs |
| Stage-0 batch | 128 | TabDiff batch size |
| Stage-0 lr | 8×10⁻⁴ | AdamW learning rate |
| Stage-1 RSF trees | 200 | RandomSurvivalForest estimators |

---



---

## License

This project is released under the MIT License. See `LICENSE` for details.
