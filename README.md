# Machine Learning Summative

### Comparing traditional machine learning and deep learning

**Author:** Kethia Kayigire Ngabire, ALU

| Resource | Link |
|---|---|
| Google Drive (raw dataset zip + saved models) | *https://drive.google.com/drive/folders/15kfUuwXItGAFNCbbMBx-kquxYI65G_v6?usp=drive_link* |
| Video presentation | *https://youtu.be/TIlIosnyKZA* |

## Problem

Network intrusion detection systems (NIDS) flag malicious traffic so that security teams can respond before damage is done. Signature-based tools miss new attack variants, so this project asks a concrete question: **how well do traditional machine-learning models and deep neural networks separate benign network flows from malicious ones, and which attacks remain hard to detect?**

The motivation is the protection of people and institutions in Rwanda and across Africa. Digital public services and mobile money are expanding quickly there, while the number of trained security analysts remains small. An accurate, explainable detector can help reduce the workload of a small team.

## Dataset

**CIC-IDS2017**, Canadian Institute for Cybersecurity, University of New Brunswick [Sharafaldin et al., ICISSP 2018].

- Official page: https://www.unb.ca/cic/datasets/ids-2017.html (download needs a short registration form)
- Mirror used by the notebook (the official `MachineLearningCVE` CSVs): https://www.kaggle.com/datasets/chethuhn/network-intrusion-dataset
- About 2.83 million labelled flows, 78 CICFlowMeter features, and 15 labels (BENIGN plus 14 attack types: DoS/DDoS, PortScan, brute force, web attacks, botnet, infiltration and Heartbleed).
- In this project the target is binary (`BENIGN` = 0, any attack = 1). The original labels are kept for per-attack error analysis.

The raw data is **not** committed to this repository. The notebook downloads it automatically.

## Method summary

1. **Cleaning:** drop identifier columns (`Destination Port` etc.) to prevent shortcut learning; drop `inf`/`NaN` rows, exact duplicates, conflicting-label duplicates, constant columns and duplicated columns.
2. **Sampling and splitting:** a stratified sample of 200 k flows in which rare attacks keep at least 1 000 rows (or all of their rows if they have fewer). The sample is split 70/15/15 into train, validation and test. Every fitted statistic uses the training split only.
3. **Feature engineering:** six behavioural ratio features, a signed log transform for heavy-tailed features, and a correlation filter (|r| > 0.98). Each model is compared on a `raw` view and an `fe` view of the features.
4. **Traditional ML (scikit-learn), 8 experiments:** Logistic Regression (baseline → feature engineering → class weights), Random Forest (default → RandomizedSearchCV), HistGradientBoosting (default → RandomizedSearchCV), and top-k feature selection.
5. **Deep learning (TensorFlow/Keras + `tf.data`), 8 experiments:** Sequential MLPs that vary capacity, regularisation (BatchNorm/Dropout/L2), learning rate, optimiser (Adam vs SGD-Nesterov), class weights, early stopping and LR scheduling, plus a Functional-API residual MLP and a run without feature engineering.
6. **Evaluation:** precision, recall, F1, ROC-AUC, PR-AUC, FPR, confusion matrices, ROC/PR curves, Keras learning curves, scikit-learn learning curves, a bias–variance plot, decision-threshold tuning on validation, per-attack-family recall and an inspection of missed attacks.
7. **Tracking:** every run is logged to TensorBoard and to Weights & Biases in offline mode. `results/experiment_results.csv` is the experiment table.

## Repository contents

| File | Purpose |
|---|---|
| `CIC_IDS2017_Intrusion_Detection.ipynb` | Full pipeline. Runs top to bottom in Google Colab. |
| `requirements.txt` | Python dependencies |
| `COLAB_STEPS.md` | Step-by-step instructions for running the notebook in Colab |
| `REPORT_GUIDE.md` | Report outline, where each figure/table goes, and reference list |

## How to run

1. Open the notebook in Google Colab (*File → Upload notebook*, or open it from GitHub).
2. *Runtime → Change runtime type → T4 GPU* (optional, but it speeds up the neural networks).
3. *Runtime → Run all*. Allow Google Drive access when asked.
4. Outputs are written to `MyDrive/ML_Summative/`: `figures/`, `results/`, `models/`, `logs/` and the raw dataset zip.

Run it locally with:

```bash
pip install -r requirements.txt
```

```bash
jupyter notebook Kethia_Kayigire_Intrusion_Detection_Notebook.ipynb
```

All seeds are fixed (`CONFIG["seed"] = 42`). Expected run time on free Colab is about 45–75 minutes.

## Acknowledgements

The dataset is by the Canadian Institute for Cybersecurity (UNB). No pretrained models were used. Libraries: scikit-learn, TensorFlow/Keras, pandas, matplotlib/seaborn, Weights & Biases.
