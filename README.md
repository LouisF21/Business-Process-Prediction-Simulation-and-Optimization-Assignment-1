# Business Process Prediction, Simulation, and Optimization – Assignment 1

Process and data science on the **BPIC-17** event log (loan application process of a Dutch financial institute).
Course: Business Process Prediction, Simulation, and Optimization, TUM (Chair of Information Systems and Business Process Management), Winter Semester 2026/2027.
Author: Louis Forster

The goal is a sound, simple base process model for simulation that replays about 80 % of the cases with high precision. On top of it, the notebook performs decision mining at every XOR gateway and an XGBoost/SHAP analysis of the credit decision (granted vs. denied).

## Repository contents

| File | Description |
|---|---|
| `notebook.ipynb` | All code: log analysis, preprocessing, process discovery and evaluation, final model, decision mining, XGBoost + SHAP |
| `final_model_decisions.bpmn` | Final BPMN 2.0 model with a decision block (business rule task) before every XOR gateway, annotated in bpmn.io |
| `requirements.txt` | Python package versions used |

The event log is **not** included in this repository (see below).

## Data

BPI Challenge 2017, application log:
B. F. van Dongen (2017). *BPI Challenge 2017*. 4TU.ResearchData. DOI: [10.4121/uuid:5f3067df-f10b-45da-b98b-86ae4c7a310b](https://doi.org/10.4121/uuid:5f3067df-f10b-45da-b98b-86ae4c7a310b)

1. Download the application event log (`BPI Challenge 2017.xes.gz`) from the DOI above.
2. Rename it to `BPIC17.xes.gz` and place it in a folder `data/` next to the notebook:

```
.
├── notebook.ipynb
├── final_model_decisions.bpmn
├── requirements.txt
└── data/
    └── BPIC17.xes.gz
```

## Setup

Tested with Python 3.11.

```bash
python -m venv .venv
source .venv/bin/activate        # Windows: .venv\Scripts\activate
pip install -r requirements.txt
```

PM4Py renders process models via [Graphviz](https://graphviz.org/download/), which must be installed separately as a system package (e.g. `brew install graphviz` or `sudo apt install graphviz`).

## How to run

Open `notebook.ipynb` in Jupyter and run all cells in order (*Kernel → Restart & Run All*).
The alignment-based evaluations are the slowest part (about 1–2 minutes per model).

### Notebook structure

1. **Basic log analysis** – log statistics, lifecycle transitions, outcomes, variant coverage (Table 1, Fig. 2 of the report)
2. **Preprocessing** – five cleaning steps (Table 2):
   1. keep only `complete` events
   2. keep only cases with an end state
   3. merge end state with its partner offer event (A_Pending/O_Accepted, A_Denied/O_Refused, A_Cancelled/O_Cancelled; < 2 min apart)
   4. collapse directly repeated activities
   5. cut events after the end state
3. **Discovery comparison** – Inductive Miner (noise thresholds 0, 0.1, 0.2, 0.5, 0.9) and Heuristics Miner (dependency thresholds 0.2, 0.5, 0.9) on a sample of 1,500 cases, with PM4Py metrics and own simplicity metrics (size, XOR-splits, control-flow complexity)
4. **Iterative model construction** – Inductive Miner on the full log after each preprocessing step, then on the top-k variants, evaluated on all cases
5. **Final model** – top-50 variants, Inductive Miner with noise threshold 0; soundness check and BPMN export
6. **Decision mining** – decision tree per XOR-split of the final Petri net
7. **Advanced analysis** – XGBoost classifier for the credit decision (Pending vs. Denied), explained with gain importance, SHAP and partial dependence plots

### Generated outputs

Running the notebook writes these files to the working directory:

- `petrinet nt05.pdf`, `petrinet dep09.pdf` – discovered models from the sample
- `final_model_petri.pdf`, `final_model_bpmn.pdf`, `final_model.bpmn` – final model
- `xgb_importance.png`, `shap_bar.png`, `shap_beeswarm.png`, `pdp.png` – XGBoost explanations

`final_model_decisions.bpmn` was created by opening `final_model.bpmn` in [bpmn.io](https://bpmn.io) and adding the decision blocks and branch annotations by hand.

## Reproducibility

- Case sample for the discovery comparison: `random_state=42`
- XGBoost train/test split (70/30, stratified) and model: `random_state=42`
- Decision-mining train/test splits (50/50): `random_state=0`
- All model parameters are set explicitly in the notebook.

## Main results

| Final model (top-50, IM noise 0), evaluated on all 31,411 cases | Value |
|---|---|
| Fitting traces (TBR / alignments) | 0.862 / 0.819 |
| Fitness (TBR / alignments) | 0.990 / 0.983 |
| Precision (ETC / alignments) | 0.642 / 0.650 |
| Generalization | 0.991 |
| Simplicity (arc degree) | 0.682 |
| Size / XOR-splits / CFC | 60 / 12 / 26 |
| Sound | yes |

- Preprocessing reduces the log from 15,930 to 4,779 variants.
- The XGBoost model for the credit decision reaches an AUC of 0.762. The number of incompleteness loops (`n_incomplete`) is the most important feature by mean absolute SHAP value: applications without an incompleteness loop are denied most often.

## Viewing the BPMN model

Open `final_model_decisions.bpmn` in [demo.bpmn.io](https://demo.bpmn.io) (drag and drop the file) or in the Camunda Modeler.

## Tools

[Python](https://www.python.org), [pandas](https://pandas.pydata.org), [NumPy](https://numpy.org), [PM4Py](https://pm4py.fit.fraunhofer.de), [Matplotlib](https://matplotlib.org), [scikit-learn](https://scikit-learn.org), [XGBoost](https://xgboost.readthedocs.io), [SHAP](https://shap.readthedocs.io), [Graphviz](https://graphviz.org), [bpmn.io](https://bpmn.io). No commercial tools were used.

## GenAI usage

Claude (Anthropic) was used to improve the clarity of the report text, format LaTeX tables, check facts, and help translate ideas into code. All results were checked against the notebook outputs.
