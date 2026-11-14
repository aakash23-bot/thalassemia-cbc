# Context-Aware Haematological Intelligence III: Differentiating Thalassaemia from Iron-Deficiency Anaemia

Analysis of whether routine complete blood count (CBC) parameters carry enough biological signal to separate thalassaemia phenotypes from each other and from iron-deficiency anaemia (IDA). Classical discriminant indices and baseline machine-learning models are benchmarked across five public datasets totalling more than 14,000 individuals.

Part of the *Context-Aware Haematological Intelligence* series:
- I. Iron biomarker discordance: [`discordance`](https://github.com/aakash23-bot/discordance)
- II. Kidney-driven remodelling of anaemia: [`ckd-anemia-network`](https://github.com/aakash23-bot/ckd-anemia-network)
- **III. Thalassaemia vs IDA (this repository)**

## Central hypothesis

The discriminative performance of any method, whether a classical index or a machine-learning model, is bounded by the biological signal in the phenotype rather than by model sophistication. This predicts a performance gradient: methods should fail where haematological perturbation is minimal (silent α-carriers) and succeed where erythropoietic decompensation is marked (β-thalassaemia major).

## Datasets

All datasets are publicly available. None are redistributed here.

| Dataset | Origin | n | Classes |
|---|---|---|---|
| α-thalassaemia cohort 1 | Colombo, Sri Lanka | 203 | Normal, silent carrier, α-trait |
| α-thalassaemia cohort 2 | Colombo, Sri Lanka | 147 | Normal, silent carrier, α-trait |
| Haemoglobinopathy screening | West Bengal, India | 13,031 | Normal (11,783), BTT (562), HbE carrier (546), HbS carrier (51), HbD carrier (25), rarer variants |
| IDA vs BTT | Public | 250 | IDA (205), BTT (45) |
| Severity classification | Bangladesh | 617 | Major (141), Intermedia (302), Minor (174) |

Place the source files in the notebook's working directory before running it.

## Methods

**Pre-processing:** standardisation of features, age converted to continuous years, harmonised diagnostic labels.

**Classical discriminant indices (10):**

| Index | Formula | Thalassaemia trait if |
|---|---|---|
| Mentzer | MCV / RBC | < 13 |
| England & Fraser | MCV − RBC − 5·Hb − 3.4 | < 0 |
| Shine & Lal | MCV² · MCH / 100 | < 1500 |
| Ehsani | MCV − 10·RBC | < 0 |
| Srivastava | MCH / RBC | < 3.8 |
| Bessman | RDW | < 14.5% |
| RDWI | MCV · RDW / RBC | < 220 |
| Ricerca | RDW / RBC | < 4.4 |
| Green & King | MCV² · RDW / (Hb · 100) | < 65 |
| Sirdah | MCV − RBC − 3·Hb | < 0 |

**Statistics:** Welch's t-tests (p < 0.05), means with 95% CI, and Pearson correlation matrices within each diagnostic subgroup, stratified by sex and age where relevant.

**Machine learning:** logistic regression, random forest and XGBoost with a stratified 70/30 split and **default hyperparameters**. Models are used as a standardised lens to measure intrinsic discriminability, not as tuned clinical tools. Metrics are accuracy, weighted precision, recall, F1 and AUC.

**Explainability:** SHAP on random forest models.

## Key results

| Task | Best ML | Classical indices |
|---|---|---|
| Silent α-carrier vs normal | Near chance | AUC 0.451–0.595 |
| α-trait vs silent carrier | RF AUC 0.841 | AUC ≤ 0.531 |
| IDA vs BTT | RF F1 0.97 | Green & King / RDWI F1 0.82 |
| 4-class screening (West Bengal) | XGBoost F1 0.908, AUC 0.908 | — |
| Severity (major / intermedia / minor) | RF F1 0.984, AUC 1.00 | — |

**Findings**
- **Silent α-thalassaemia** sits at the limit of CBC detectability, with no significant Hb, PCV or RBC differences in females (p = 0.38, 0.16 and 0.74).
- **α-trait** gives sex-dependent decompensation: females show lower Hb (10.60 vs 11.50 g/dL, p = 0.009), lower PCV and higher RBC, while males do not.
- **IDA vs BTT:** RDW is the dominant SHAP feature, followed by Hb and RBC. Which parameters reach significance differs by sex (MCH in females, MCV in males).
- **West Bengal:** correlation structure differs by phenotype. In β-thalassaemia major, HbF vs Hb r = −0.91 (n = 13, interpret with caution).
- **Severity:** near-perfect separation likely reflects dataset homogeneity and should not be read as generalisable signal.

## Repository structure

```
.
├── thalanalysisfinal.ipynb   # full analysis pipeline
├── README.md
└── .gitignore                # excludes data files and figures
```

## Requirements

Python 3.10+

```bash
pip install pandas numpy scipy scikit-learn xgboost shap matplotlib seaborn
```

## How to run

1. Download the five datasets and place them alongside the notebook.
2. Open `thalanalysisfinal.ipynb` in Jupyter or Google Colab.
3. Run all cells top to bottom.

## Limitations

- Small subgroups (BTT n = 45, β-thalassaemia major n = 13, HbE/β n = 19).
- Many diagnoses lack molecular confirmation.
- Single-centre, convenience datasets that are not population-representative.
- Default hyperparameters and a single train/test split, with no external validation.
- Not intended for clinical deployment.

## Authors

Aakash Adhya, Nilotpal Banerjee, Prathit Chatterjee\*

\*Corresponding author: prathit.chatterjee@ihub-data.iiit.ac.in

## License

MIT
