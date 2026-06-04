# Causal ML for Oncocardiology
### Personalized Treatment Recommendation to Minimize Cardiotoxicity Risk in Breast Cancer Patients

---

## 🎯 Motivation

Cancer treatments are effective — but their side effects can be life-threatening.  
**Cardiotoxicity** (heart damage caused by cancer therapy) is one of the leading causes of mortality among cancer survivors, yet cardiovascular disease and oncology have historically been studied in silos.

This project sits at the intersection of both fields (**onco-cardiology**) and asks:

> *Given a patient's clinical profile, which treatment minimizes the risk of cardiotoxicity while remaining therapeutically appropriate?*

This project is a **Proof of Concept (PoC)** — the primary goal is not to deliver a production-ready clinical tool, but to demonstrate that causal ML is a technically feasible and clinically motivated approach to this problem. It builds an end-to-end pipeline: from simulated clinical data generation, through individualized treatment effect estimation, to an interpretable recommendation interface.

The goal is not to predict a patient's outcome under their actual treatment — it's to estimate what **would have happened** under every possible treatment, and recommend the safest one. This is a **counterfactual reasoning** problem, which is why standard classifiers are insufficient and causal inference methods are needed.

---

## 🧠 Why Causal Inference, Not Just Classification?

A conventional classifier trained on observational data learns:
> *"Patients who received Treatment X tended to have outcome Y."*

But this conflates **treatment effect** with **selection bias** — sicker patients tend to receive more aggressive treatments, making those treatments look worse than they are.

Causal inference methods are designed to answer:
> *"For **this specific patient**, what would the outcome be under each treatment?"*

This project uses **TARNet (Treatment-Agnostic Representation Network)**, a neural network architecture designed for individualized treatment effect estimation.

---

## 🏗️ Model Architecture: TARNet Multi-Head Neural Network

```
Patient Features (HER2, ER, BRCA1, Stage, Age, Troponin)
        ↓
 Shared Representation Network
 (learns a patient embedding that captures clinical patterns)
        ↓
 ┌──────┬──────┬──────┐
 │Head 1│Head 2│Head 3│
 │Chemo │Target│Combo │
 └──────┴──────┴──────┘
        ↓
 Toxicity probability for each treatment
        ↓
 Recommend treatment with lowest predicted toxicity
```

**Why TARNet?**
- The shared network learns a representation that captures cross-treatment clinical patterns
- Separate heads allow each treatment's outcome to be modeled independently
- Enables counterfactual prediction: estimating outcomes under treatments a patient never received

---

## 📊 Dataset

This version uses **simulated data** (n = 10,000) generated from a clinically informed risk model incorporating:

| Feature | Description |
|---|---|
| `HER2_positive` | HER2 protein overexpression (key for targeted therapy eligibility) |
| `ER_positive` | Estrogen receptor status (relevant for hormonal therapy) |
| `BRCA1_mutation` | DNA repair gene mutation (affects treatment response) |
| `Stage` | Cancer stage 0–4 |
| `Age` | Patient age (30–85) |
| `Troponin_high` | Elevated cardiac biomarker (indicator of existing cardiac stress) |

**Treatment Options:**

| Label | Treatment | Notes |
|---|---|---|
| T1 | Chemotherapy | Broad-spectrum; higher cardiac risk |
| T2 | Targeted Therapy | HER2-directed; generally lower toxicity |
| T3 | Chemo + Targeted | Combination; highest efficacy and highest risk |

**Known Limitation:** The simulated risk function used for data generation is the same as the ground truth used for PEHE evaluation. This means PEHE measures model approximation error of the DGP, not true out-of-sample counterfactual accuracy. Real-world validation requires observational or RCT data with known counterfactuals (see: Roadmap).

---

## 📈 Results

| Metric | Value |
|---|---|
| AUC (observed treatment) | 0.7928 |
| PEHE (Root Mean Squared Error) | 0.0787 |
| Policy Accuracy (correct recommendation rate) | 75.62% |

**Per-treatment PEHE:**

| Treatment | RMSE |
|---|---|
| T1 Chemotherapy | 0.0909 |
| T2 Targeted Therapy | 0.0838 |
| T3 Combination | 0.1032 |

---

## 🖥️ Interactive Demo (Gradio)

Two interfaces are included:

**Single Patient Prediction**
- Input a patient's clinical features via radio buttons and sliders
- Outputs predicted toxicity risk for each treatment + confidence score

**Batch Prediction**
- Upload a CSV file with multiple patients
- Returns a downloadable results table with recommendations and confidence explanations

---

## ⚠️ Limitations & Known Issues

1. **Simulated data**: All results are based on synthetic data. Clinical validity has not been established.
2. **PEHE evaluation circularity**: The evaluation ground truth is derived from the same DGP as the training data (see Dataset section). This is a known limitation of simulation studies in causal inference.
3. **Coarse treatment categories**: T1/T2/T3 represent broad treatment classes, not specific drug protocols or dosages. Real clinical decisions require far more granularity.
4. **No confounding analysis**: The causal structure of the data-generating process is known by design. In real observational data, unmeasured confounders would require additional methods (e.g., instrumental variables, propensity score adjustment).
5. **No external validation**: The model has not been tested on any held-out clinical dataset.

---

## 🗺️ Roadmap

- [ ] **Replace simulated data with TCGA** (The Cancer Genome Atlas) — real breast cancer genomic and clinical data with HER2/ER/BRCA1 annotations
- [ ] **Integrate PhysioNet cardiac data** — link post-treatment ECG signals to cardiotoxicity outcomes
- [ ] **Add SHAP explainability** — visualize per-feature contributions for each recommendation in the Gradio UI
- [ ] **Write SaMD risk management documentation** — ISO 14971 + IEC 62304 framework applied to this model as a clinical decision support tool
- [ ] **Stage 2 model** — drug-level treatment selection within each treatment category

---

## 🛠️ Setup

```bash
pip install -r requirements.txt
jupyter notebook CausalML_for_Oncocardiology.ipynb
```

**Requirements:**
```
torch
numpy
pandas
scikit-learn
seaborn
matplotlib
gradio
```

---

## 📚 References

- Cardiotoxicity and cancer survivors: https://doi.org/10.1038/s41467-020-15639-5
- Onco-cardiology database: https://doi.org/10.1093/database/baad029
- TARNet: Shalit et al. (2017), *Estimating individual treatment effect: generalization bounds and algorithms*

---

## 📌 Disclaimer

This project is for **research and educational purposes only**.  
It is not a validated clinical decision support tool and must not be used for actual medical decisions.
