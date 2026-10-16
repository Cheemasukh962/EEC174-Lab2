# EEC 174 Lab 2: Detecting Ventilator Breathing Problems

This lab uses machine learning to spot problems in how ICU patients breathe with a ventilator. Each breath is labeled as one of:

- **0: Normal**
- **1: Breath stacking (BSA):** the patient doesn't fully breathe out, so air gets trapped.
- **2: Double trigger (DTA):** the ventilator fires twice in a row.

All the work is in **`Lab2.ipynb`**.

## What's in this folder

| File / folder | What it is |
|---|---|
| `Lab2.ipynb` | The main notebook: all code and results |
| `ece174_pva_train_x.csv` / `ece174_pva_train_y.csv` | Training data (breath features / labels) |
| `ece174_pva_validation_x.csv` / `ece174_pva_validation_y.csv` | Validation data (breath features / labels) |
| `data/` | Raw flow and pressure waveforms, one folder per patient |
| `REPORT_QUESTIONS.md` | Written report questions and answers |

## How to run it

### 1. Install Python
Use **Python 3.10 or newer**.

### 2. Install the packages
In a terminal, run:

```bash
python -m pip install pandas numpy scipy scikit-learn matplotlib jupyter
```

### 3. Open the notebook
Open `Lab2.ipynb` in **VS Code** (with the Jupyter extension) or run:

```bash
jupyter notebook Lab2.ipynb
```

> **Important:** the notebook must be run from inside this folder, because it loads the CSV files and the `data/` folder from here.

### 4. Run everything
Click **Run All** (or **Kernel → Restart & Run All**).

- It takes about **1–3 minutes**. Building the features reads every waveform file.
- Cells depend on the ones above them, so always run from the top. Running a single cell on its own gives errors like `NameError: name 'train_x' is not defined`.

## What each part does

| Part | What happens |
|---|---|
| **Task 1: Featurization** | Reads each breath's waveform and calculates 8 new features (breathing-out time, breath rate, air volume in, pressure stats, and so on), giving 18 columns in total |
| **Task 2: Random Forest** | Scales the data and trains a Random Forest 10 times, then reports the average precision, recall and F1 per class |
| **Task 3: Model comparison** | Trains L1 logistic regression and compares it with the Random Forest |
| **Task 4a: Expert rules** | Predicts breath problems using simple rules from doctors (e.g. "air out ÷ air in < 0.9 → breath stacking") with no ML |
| **Task 5: Cross-validation** | Tests a Random Forest with 5-fold stratified cross-validation and prints scores per fold and the average |

## Expected results (approximate)

Random Forest scores change slightly every run, so your numbers may differ a little.

| Method | Accuracy | Macro F1 |
|---|---|---|
| Random Forest (validation set) | ~0.89 | ~0.64 |
| L1 logistic regression (validation set) | ~0.77 | ~0.53 |
| Expert rules (training set) | ~0.97 | ~0.94 |
| 5-fold cross-validation (Random Forest) | n/a | ~0.92 |

## Common problems

| Error | Fix |
|---|---|
| `ModuleNotFoundError: No module named 'scipy'` (or `sklearn`) | Run the install command in step 2, using the **same Python** your notebook kernel uses. Then restart the kernel. |
| `NameError: name '...' is not defined` | Cells were run out of order. Use **Run All** from the top. |
| `IndexError: list index out of range` on `data_files[0]` | The notebook can't find the `data/` folder. Make sure you're running it from inside this folder. |
| `ImportError: cannot import name 'simps'` | Newer SciPy renamed `simps` to `simpson`. The notebook already handles this; make sure you have the latest version of `Lab2.ipynb`. |
