# Predicting Harvest and Post-Harvest Crop Loss in Nigeria

## Overview

This project uses Nigeria’s National Agricultural Sample Survey (NASS) 2023 to predict whether a household reported harvest or post-harvest loss for a crop during the main agricultural season.

**Research question:** How well can crop, location, household, and farm characteristics predict reported crop loss?

The project also investigates whether adding household and farm information improves predictions beyond using crop and location alone.

## Dataset

### Data origin and selection

The data was downloaded from the National Bureau of Statistics (NBS), Nigeria, through its official microdata catalogue:

[NASS 2023 study description and data access](https://microdata.nigerianstat.gov.ng/index.php/catalog/173/study-description)

- **Survey:** National Agricultural Sample Survey 2023
- **Reference ID:** NGA-NBS-NASS-2023-v01
- **File format:** Stata (`.dta`)
- **Download date:** October 3, 2026
- **Local storage:** `data/raw/`

This dataset was selected because it contains a direct question about reported crop loss alongside crop, location, household, and farm characteristics. Shared identifiers allow information from the household, plot, crop, and harvest tables to be connected.

These features make it suitable for comparing predictions based on crop and location alone with predictions using additional household and farm information. The investigation focuses on the main agricultural season to keep the scope consistent and manageable.

The survey contains 21 Stata files. This investigation uses four:

| File | Information used |
| --- | --- |
| `1_households.dta` | Farm record-keeping and listed plot count |
| `3_Plot_mainSeason.dta` | Land tenure and irrigation infrastructure |
| `4_Crop_mainSeason.dta` | Crop, plot identifiers, and planted area |
| `5_Harvest_mainSeason.dta` | Reported loss, crop, location, and household identifiers |

The modelling table contains **334,133 household-crop records** from **116,406 households**. A household can appear in several records.

The target, `s01b_q11`, records whether losses occurred during or after harvest, post-harvest activities, transport, or storage. It is encoded as:

- **0:** No reported loss
- **1:** Reported loss

Loss was reported for **11.51%** of records.

## Project structure

```text
nass-postharvest-project/
├── README.md
├── data/
│   ├── raw/                 # Original Stata files
│   └── processed/
│       └── modelling_data.csv
├── notebooks/
│   ├── 01_data_exploration.ipynb
│   ├── 02_data_preparation.ipynb
│   └── 03_modelling_evaluation.ipynb
├── outputs/
└── references/
    ├── documentation.pdf
    └── DS mini-project assignment.pdf
```

## Notebook workflow

Run the notebooks in this order:

| Notebook | Purpose |
| --- | --- |
| [01: Data exploration](notebooks/01_data_exploration.ipynb) | Define the goal, inspect the target, understand the source tables, and identify candidate predictors |
| [02: Data preparation](notebooks/02_data_preparation.ipynb) | Validate source data, aggregate plot information, join tables, and save the modelling dataset |
| [03: Modelling and evaluation](notebooks/03_modelling_evaluation.ipynb) | Build models, perform grouped validation, evaluate the selected model, and discuss findings and limitations |

Together, these notebooks cover the five required lifecycle stages: goal definition, data management, modelling, validation, and analysis.

## Methods

Plot characteristics are joined to crop-plot records and then summarised to household-crop level. These summaries and household characteristics are attached to the harvest table.

The models compared are:

- A majority-class dummy benchmark
- Logistic regression
- Class-balanced logistic regression
- Class-balanced random forest
- Random forest with tuned tree depth and minimum leaf size

Approximately 20% of households are reserved for final testing. Five-fold grouped cross-validation is used within the training set, keeping each household’s records together.

Categorical predictors are one-hot encoded. Numeric predictors are scaled for logistic regression. Learned preprocessing is fitted inside each modelling pipeline.

Models are selected using mean validation F1, with precision, recall, and accuracy also reported. F1 gives precision and recall equal importance while focusing on the loss class.

Household IDs and survey weights are excluded from predictors. The reported performance scores are unweighted.

## Main results

The tuned balanced random forest achieved the highest mean validation F1 among the tested approaches. Its selected settings were:

- 200 trees
- Maximum depth: 25
- Minimum records per leaf: 10
- Balanced class weights

| Metric | Validation mean | Final test |
| --- | ---: | ---: |
| Accuracy | 0.707 | 0.712 |
| Precision | 0.233 | 0.239 |
| Recall | 0.674 | 0.688 |
| F1-score | 0.346 | 0.355 |

The model detected **68.8% of actual loss records**, but **76.1% of its loss alerts were false alarms**.

Adding household and farm predictors increased validation F1 from **0.307 to 0.346** under the selected forest settings. These settings were selected using the full predictor set, not tuned separately for each predictor set.

Permutation importance showed that the model relied most strongly on **state**, followed by **crop planted area**.

## Limitations

- The target measures reported loss and may contain reporting errors.
- The data covers one survey year; performance in future years or completely new states has not been tested.
- Before-harvest availability of all predictors has not been confirmed, so this is not yet a validated before-harvest forecasting system.
- Performance differs across crops and zones.
- Scores describe unweighted household-crop records and not national population estimates.
- Predictive relationships and importance scores do not establish causes of crop loss.

The model provides some useful predictive information, but frequent false alarms limit its suitability for costly interventions.

## Running the project

### 1. Open the project folder

Open `nass-postharvest-project` as the workspace folder in VS Code.

The included `.vscode/settings.json` configures notebooks to use the project root as their working directory. All data paths are relative to that root.

### 2. Prepare the Python environment

The notebooks record Python **3.12.7**.

Install the following packages in the Python environment selected as your notebook kernel:

```bash
python -m pip install \
  numpy==1.26.4 \
  pandas==2.2.2 \
  matplotlib==3.9.2 \
  scikit-learn==1.5.1 \
  ipykernel==6.28.0
```

### 3. Add the source data

Place the original survey `.dta` files in `data/raw/`.

Keep the original files unchanged. Notebook 02 creates the processed modelling table.

### 4. Run the notebooks

Select the appropriate Python kernel and run each notebook from top to bottom in this order:

1. `01_data_exploration.ipynb`
2. `02_data_preparation.ipynb`
3. `03_modelling_evaluation.ipynb`

Each notebook can run independently once its input files are available. Notebook 03 requires the CSV generated by notebook 02.

Rerunning notebook 02 replaces `data/processed/modelling_data.csv`. Rerunning modelling cells trains fresh models and may take time.

The random seed is set to **42**.

## Future work

Possible extensions include adding predictors available before harvest, comparing additional models such as gradient boosting, investigating decision thresholds using training-data validation, and evaluating performance on another survey year.

## Data source and citation

National Bureau of Statistics, Nigeria, National Agricultural Sample Survey (NASS 2023)-v1.0.

See the [survey documentation](references/documentation.pdf) for variable descriptions and dataset information.