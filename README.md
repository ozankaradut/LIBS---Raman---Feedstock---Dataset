[README.md](https://github.com/user-attachments/files/33778356/README.md)
# LIBS–Raman Fusion for Characterization of Heterogeneous Gasifier Feedstocks

Research data accompanying **Machine Learning Enabled LIBS and Raman Fusion for Online Dynamic Characterization of Heterogeneous Gasifier Feedstocks**.

**Authors:** Ozan Karadut, Carlos E. Romero, and Zheng Yao  
**Institution:** Lehigh University, Bethlehem, Pennsylvania, USA  
**Contact:** Ozan Karadut — sok522@lehigh.edu  
**Article DOI:** [Add when available]  
**Dataset DOI and release version:** [Add after archiving the dataset]

## Overview

This project investigates the combination of laser induced breakdown spectroscopy (LIBS), Raman spectroscopy, and machine learning regression for online and dynamic characterization of heterogeneous waste derived gasifier feedstocks. Timely estimates of feedstock properties can support feedstock assessment and feedforward adjustment of gasifier operating conditions.

LIBS provides elemental emission information, while Raman spectroscopy provides complementary molecular and structural information. The study compares LIBS only and Raman only prediction with two fusion strategies: mid-level fusion and sequential fusion. Laboratory measurements provide the reference values for predicting 17 physicochemical properties.

This repository accompanies the study and documents the experimental data, observation pairing, and modeling framework. The file inventory below describes the supplied Excel workbook.

## Materials and Sample Identifiers

The experimental sample matrix contains **13 distinct materials organized into 23 sample subgroups**, identified as TT1–TT23. Results are reported using five broader material groups.

| Reporting group | Sample identifiers | Materials |
|---|---|---|
| RDF pellets 1 | TT1–TT5 | Five batches of refuse derived fuel pellets from Ames, Iowa |
| MSW fluffy | TT6–TT12 | Seven batches of fluffy municipal solid waste from Ames, Iowa |
| RDF pellets 2 | TT13 | Ames RDF |
| RDF pellets 2 | TT14 | Fulcrum pellets |
| RDF pellets 2 | TT15 | UK RDF pellets |
| Plastics | TT16 | Polymethyl methacrylate (PMMA) |
| Plastics | TT17 | Polyethylene terephthalate (PET) |
| Plastics | TT18 | High density polyethylene (HDPE) |
| Plastics | TT19 | Polypropylene (PP) |
| Plastics | TT20 | Polystyrene (PS) |
| Plastics | TT21 | Polycarbonate (PC) |
| Paper cup/cotton | TT22 | Paper cups |
| Paper cup/cotton | TT23 | Cotton |

TT identifiers denote sample subgroups, not individual measurements. Individual observations within a subgroup must retain their measurement identifiers.

## Measurement Pairing and Reference Values

Each sample subgroup was divided into portions. Each portion was measured separately using LIBS and Raman spectroscopy. Measurements corresponding to the same material portion were linked to form a paired observation for multimodal analysis.

Each paired observation was matched with the laboratory analysis results for that portion. These measurement specific laboratory values served as the reference targets for model training and evaluation.

Pairing is therefore performed at the **portion level**. It does not imply simultaneous acquisition by the two instruments or measurement of the same microscopic location. When combining files, use the documented observation or portion identifiers rather than assuming that row order establishes a match.

## Spectroscopic Measurements

### LIBS

LIBS measurements were acquired dynamically from material moving on a conveyor at a mass flow rate of 50 kg/h. The system used a 1064 nm pulsed laser with approximately 300 mJ pulse energy and a repetition rate of 2 Hz.

Each observation averaged 800 consecutive pulses, corresponding to 400 s of acquisition. Ten independent LIBS observations were collected for each of the 23 subgroups, yielding 230 observations in the acquisition library. Target specific modeling counts may differ after eligibility filtering.

LIBS repeatability was assessed using relative standard deviation (RSD) across observations within each subgroup. Values above 15% were flagged for evaluation; a high RSD did not automatically invalidate an entire observation.

### Raman

Raman measurements used 1064 nm excitation over a spectral range of 225–2500 cm⁻¹. The conveyor advanced between scans, and the probe sampled multiple positions across the material.

Each representative spectrum averaged 40 acquisitions: four acquisitions at each of 10 spatial positions, with a 1000 ms integration time per acquisition. Preprocessing included dark spectrum subtraction, averaging, Savitzky–Golay smoothing, and asymmetric least squares baseline correction. Quality assessment considered spectral shape agreement, signal to noise ratio, and spatial repeatability.

## Target Properties

The study evaluated the following 17 properties:

| Category | Properties |
|---|---|
| Heating value | Higher heating value (HHV), reported on a moisture  and ash free basis |
| Proximate properties | Ash content, fixed carbon, and volatile matter |
| Other fuel quality indicators | Sulfur, chlorine, and alkali content |
| Inorganic constituents | SiO₂, Al₂O₃, Fe₂O₃, CaO, MgO, K₂O, Na₂O, TiO₂, and P₂O₅ |
| Ash thermal behavior | Ash softening temperature |

HHV is reported in BTU/lb and alkali content in lb/MMBtu in the manuscript. Use the database documentation for the exact unit and reporting basis of every column. Do not assume that all percentage values share the same basis, particularly when comparing fuel composition with ash mineral composition.

## Prediction Strategies

| Strategy | Regression inputs |
|---|---|
| LIBS only | LIBS derived predictors |
| Raman only | Target specific Raman features represented by partial least squares (PLS) latent variables |
| Mid-level fusion | LIBS predictors combined with target specific Raman PLS latent variables |
| Sequential fusion | LIBS predictors combined with material class probabilities generated by a Raman based classifier |

For sequential fusion, a logistic regression classifier uses Raman PLS scores to estimate material class probabilities. The complete probability vector supplies additional information to the regression model. Development set probability features are generated using out of fold prediction.

The regression algorithms include support vector regression (SVR), random forests (RF), and multilayer perceptrons (MLP), together with a weighted ensemble.

## Model Evaluation and Interpretation

For each target, observations were divided into an 80% development set and a 20% external test set using a fixed random seed and class stratified sampling. The initial partition was kept consistent across regression algorithms and input configurations for that target. Model development used repeated nested five fold cross validation.

Performance was assessed using root mean square error (RMSE), the coefficient of determination (R²), and normalized root mean square deviation (RMSD, %). For comparisons with LIBS only prediction, improvement was calculated as:

**Improvement (%) = 100 × (RMSD_LIBS − RMSD_method) / RMSD_LIBS**

The comparison uses the same target property and regression algorithm. Positive values indicate reduced prediction error, while negative values indicate increased error.

Important interpretation limits:

- The external test set contains previously unseen measurements from material classes represented during development. It does not establish performance on entirely unseen materials or independently collected batches.
- Target specific Raman feature preparation preceded the outer regression cross validation folds. Those cross validation results are conditional on the selected Raman representation.
- External performance was used to compare algorithms and feature sets. The resulting comparisons are post selection evidence rather than an additional independent confirmatory evaluation.
- The study evaluates feedstock property prediction; it does not demonstrate closed loop control of an operating gasifier.

## Database Files and Column Definitions

The database is provided in **`LIBS_Raman_Fusion_Feedstock_Dateset.xlsx`**. The filename is reproduced exactly as distributed. The workbook contains five worksheets; the first row of each sheet contains column headers. Counts below exclude the header row.

| Worksheet | Data rows | Columns | Contents |
|---|---:|---:|---|
| `SNV_CARS_PLS-DA_v1_3_class3` | 231 | 13 | A `Label` column and 12 Raman latent variable columns, `LV1`–`LV12`. These are latent variable scores, not class probabilities. |
| `Raman` | 231 | 513 | A `Source` column and 512 Raman spectral intensity channels. Numeric column headers represent Raman shifts from 188.773 to 2523.819 cm⁻¹. |
| `LIBS` | 239 | 35 | A `Class` column, 16 LIBS related features, and 18 reference property fields. |
| `RL` | 230 | 47 | A `Class` column, 16 LIBS related features, 12 Raman latent variables (`LV1`–`LV12`), and 18 reference property fields. |
| `RAMAN_LIBS` | 230 | 547 | A `Class` column, 16 LIBS related features, 512 Raman spectral channels, and 18 reference property fields. |

### LIBS Feature Columns

The 16 LIBS related columns comprise nine emission line fields and seven derived features:

- Emission line fields: `C I`, `Fe II.1`, `Mg II`, `Si I.7`, `Al I.1`, `Ti II.3`, `Ca II`, `Na I`, and `O I`.
- Derived features: `Silica Ratio`, `Base`, `Acid`, `R250`, `Dolomite Ratio`, `Al+Ti`, and `B/A`.

Column names are preserved exactly, including suffixes such as `.1`, `.3`, and `.7`. The presence of a feature in the workbook does not establish that it was retained in every published model. The workbook does not provide a complete feature definition or normalization dictionary.

### Reference Property Columns

| Column | Description |
|---|---|
| `%Ash_DB` | Ash content, dry basis |
| `HHV_MAF_BTU` | Higher heating value, moisture  and ash free basis; BTU/lb in the manuscript |
| `%Sulfur_DB` | Sulfur content, dry basis |
| `Fixed_Carbon_DB` | Fixed carbon, dry basis |
| `Volatile_%` | Volatile matter percentage |
| `SiO2`, `Al2O3`, `TiO2`, `Fe2O3`, `CaO`, `MgO`, `K2O`, `Na2O`, `P2O5` | Inorganic oxide reference values |
| `Alkali_Content` | Alkali content; lb/MMBtu in the manuscript |
| `Chlorine_ppm` | Chlorine concentration in parts per million |
| `Softening_Temp` | Ash softening temperature; the unit is not specified in the column header |

The workbook includes **17 reference-property fields**. Reporting bases and units that are not explicit in the headers must be checked against the laboratory documentation before conversion or reuse.

### Spectra, Derived Features, and Release Scope

The workbook contains Raman spectral intensity arrays, extracted LIBS features, Raman latent variable scores, and reference property values. The Raman arrays are supplied as measurement level spectra; the workbook does not independently identify their precise preprocessing stage or intensity units. The study's Raman preprocessing procedure is described above, but raw versus processed status cannot be established from the worksheet names alone.

The workbook does not contain full LIBS spectra, individual laser shot records, explicitly labeled development/test assignments, model predictions, class probability vectors, or analysis scripts. The `RL` sheet combines LIBS features with latent variables and should not be interpreted as a table of sequential fusion class probabilities.

### Observation Records and Material Labels

Each data row represents a unique measurement observation. Repeated values in the `Class`, `Source`, or `Label` columns identify observations belonging to the same material category; they do not indicate duplicate observations.

The worksheets contain different representations and selections of the study data, as summarized by the row counts above. The supplied combined sheets organize LIBS features, Raman information, and reference properties within individual observation rows.

## Using the Data

1. Read the file descriptions and column definitions before importing the data.
2. Use the combined worksheets for the representations already supplied. For new cross sheet joins, obtain the original observation/portion mapping; class labels and row positions alone do not establish correspondence.
3. Select the target property and retain observations with valid inputs and reference values.
4. Obtain the original split assignments or the exact splitting procedure when reproducing the reported comparisons; split assignments are not included in this workbook.
5. Fit any new supervised preprocessing and feature selection within the relevant training folds, keeping the external test set separate.
6. Report the dataset version, retained observation count, preprocessing, validation design, and evaluation metrics when presenting new results.

Reproducing the published numerical results requires the corresponding processing settings, feature definitions, split assignments, and model configuration. Dataset availability alone should not be interpreted as a claim that executable reproduction code is included.

## Citation

Please cite the associated article and the archived dataset version when using these data.

**Article:** Karadut, O., Romero, C. E., and Yao, Z. *Machine Learning Enabled LIBS and Raman Fusion for Online Dynamic Characterization of Heterogeneous Gasifier Feedstocks*. [Add final publication details and DOI.]

**Dataset:** [Add the author-approved dataset citation, release version, year, archive name, and DOI.]
**Dataset:** The experimental data were collected and compiled by Ozan Karadut at the Energy Research Center, Lehigh University, Bethlehem, Pennsylvania, USA, as part of this research.
## License

[Add the approved data license and a link to its full text. If code is included under a separate license, identify its scope explicitly.]

## Funding

This work was supported by the U.S. Department of Energy, Office of Energy Efficiency and Renewable Energy (EERE), Bioenergy Technologies Office (BETO), under Award Number **DE-EE0009668**.

## Contact

For questions about the dataset, measurement pairing, or study methodology, contact **Ozan Karadut** at **sok522@lehigh.edu** or **ozan.karadut@gmail.com* .
