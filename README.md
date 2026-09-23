# Experimentally Grounded Language Representations for Copolymer Exploration

## Overview

This repository contains the experimental data and Python scripts used to construct numeric and language-compatible representations of crosslinked copolymers from chemical composition and NMR measurements and to evaluate representation-dependent materials exploration.

The analysis integrates three sources of information:

1. **Chemical structure and composition**
2. **Time-domain NMR (TD-NMR) molecular dynamics**
3. **Solution-state NMR responses to lipid–copolymer interactions**

Chemical structure and TD-NMR information are used to construct the material representations used for exploration. Solution-NMR responses are kept separate from representation construction and material selection and are used only as independent experimental evaluation endpoints.

The overall workflow is:

```text
Chemical composition / molecular structure
        │
        ├── RDKit numeric representation
        └── Chemical Language
                     │
                     │
TD-NMR measurements │
        │            │
        ├── TD-NMR numeric descriptors
        └── Dynamic-State Language
                     │
                     ▼
           Material representation (X)
                     │
                     ▼
          Sequential material selection
                     │
                     ▼
      Independent Solution-NMR evaluation (Y)
```

---

## Repository structure

```text
Copolymer-lipid-interaction-study_Ver.2/
│
├── README.md
├── requirements.txt
├── LICENSE
│
├── scripts/
│   ├── 01_TDNMR_descriptor_extraction.py
│   ├── 02_Dynamic_State_Language.py
│   ├── 03_SolutionNMR_endpoint_generation.py
│   ├── 04_Chemical_representation.py
│   └── 05_Final_exploration_analysis.py
│
├── data/
│   ├── raw/
│   │   ├── TD_NMR/
│   │   │   └── AddD2O_CPMG_T2_Relative_Intensity_smoothed_44copolymers.csv
│   │   │
│   │   ├── Solution_NMR/
│   │   │   ├── roi_peak_features_with_blank_relative.csv
│   │   │   ├── ZG_to_FunctionalGroup_weights.csv
│   │   │   └── 06_Hierarchical_Language_Representation.csv
│   │   │
│   │   └── composition/
│   │       ├── copolymer_composition.csv
│   │       └── component_smiles.csv
│   │
│   ├── processed/
│   │   ├── 01_TD_NMR_descriptors.csv
│   │   ├── 01_Dynamic_State_Language.csv
│   │   ├── 03_SolutionNMR_independent_Y.csv
│   │   ├── 03_RDKit_Numeric_Final.csv
│   │   ├── 04_Chemical_Language_Sentences.csv
│   │   └── 05_Chemical_Language_Embedding_384D.csv
│   │
│   └── source_data/
│       ├── Fig4_AUC_200repeats.csv
│       ├── Fig4_selection_probability_complete_43x4.csv
│       ├── Fig4_selection_shift_by_copolymer.csv
│       ├── Fig4_top12_selection_shift.csv
│       └── Fig4_monomer_level_summary.csv
│
└── docs/
    └── data_dictionary.csv
```

---

# Analysis workflow

## 01. TD-NMR descriptor extraction

**Script**

```text
01_TDNMR_descriptor_extraction.py
```

The first analysis step converts TD-NMR measurements into quantitative descriptors of polymer molecular dynamics.

The final analysis focuses on the D2O-swollen CPMG condition.

The TD-NMR relaxation distribution is analyzed using a non-negative inverse Laplace transform with ridge regularization.

Seven TD-NMR descriptors are extracted:

```text
ShortFraction
MidFraction
LongFraction
Weighted_logmean_T2_ms
Width_log10T2
N_detected_peaks
Entropy_norm
```

The population fractions are defined as:

```text
Short T2: T2 < 1 ms
Mid T2:   1 ≤ T2 < 30 ms
Long T2:  T2 ≥ 30 ms
```

These descriptors constitute the numeric TD-NMR representation used in the subsequent exploration analysis.

**Main output**

```text
01_TD_NMR_descriptors.csv
```

---

## 02. Dynamic-State Language

**Script**

```text
02_Dynamic_State_Language.py
```

TD-NMR-derived molecular-dynamic information is translated into an experimentally grounded language-compatible representation termed **Dynamic-State Language**.

The language representation describes experimentally observed dynamic characteristics such as:

- relaxation-population balance,
- relaxation-distribution topology,
- coexistence of dynamic populations,
- dynamic heterogeneity.

The purpose of this representation is not simply to replace numerical values with words. Instead, it provides a structured description of experimentally measured molecular states that can be combined with other language-compatible material representations.

A standalone full-dataset language table is generated for inspection and reporting.

**Main output**

```text
01_Dynamic_State_Language.csv
```

For the repeated exploration benchmark, however, data-dependent language thresholds are recalculated using only the initial materials in each repeat, as described below, to avoid information leakage.

---

## 03. Independent Solution-NMR evaluation endpoints

**Script**

```text
03_SolutionNMR_endpoint_generation.py
```

Solution-state NMR measurements characterize lipid responses associated with interactions with the copolymer materials.

ROI-derived responses are mapped to functional-group-associated response categories using the predefined ROI-to-functional-group mapping.

The final independent Solution-NMR response table contains the following response dimensions:

```text
Alkenyl_Response
AlkylChain_Response
Glycerol_Response
Other_Response
```

and their corresponding within-regime standardized descriptors.

**Main output**

```text
03_SolutionNMR_independent_Y.csv
```

### Important methodological role

Solution-NMR responses are **not used to construct the material representations used for selection**.

They are retained as independent experimental response variables and are used only to evaluate the materials selected in representation space.

The current workflow uses the previously established within-regime standardized Solution-NMR descriptors contained in:

```text
06_Hierarchical_Language_Representation.csv
```

Accordingly, the present script reconstructs the ROI-to-functional-group processing and retrieves the predefined within-regime standardized Solution-NMR descriptors used as independent evaluation endpoints.

---

## 04. Chemical representation

**Script**

```text
04_Chemical_representation.py
```

Chemical composition and molecular structures of the copolymer building blocks are used to construct two complementary chemical representations:

1. **RDKit-based numeric representation**
2. **Chemical Language representation**

Input information includes:

```text
copolymer_composition.csv
component_smiles.csv
```

RDKit-derived physicochemical descriptors are calculated from the component structures and combined with the role and composition of the building blocks.

The Chemical Language representation describes the polymer in terms of its constituent building blocks and chemically interpretable relationships.

The resulting sentences are converted into sentence embeddings using:

```text
sentence-transformers/all-MiniLM-L6-v2
```

Embedding dimension:

```text
384
```

Embeddings are generated with normalization enabled.

**Main outputs**

```text
03_RDKit_Numeric_Final.csv
04_Chemical_Language_Sentences.csv
05_Chemical_Language_Embedding_384D.csv
```

---

## 05. Final materials-exploration benchmark

**Script**

```text
05_Final_exploration_analysis.py
```

The final analysis evaluates how numeric and language-compatible representations influence sequential materials exploration.

Four representation conditions are compared.

### N0 — Numeric

```text
RDKit numeric
+
TD-NMR numeric
```

### N1 — Chemical Language

```text
RDKit numeric
+
TD-NMR numeric
+
Chemical Language
```

### N2 — Dynamic-State Language

```text
RDKit numeric
+
TD-NMR numeric
+
Dynamic-State Language
```

### N3 — Combined Language

```text
RDKit numeric
+
TD-NMR numeric
+
Chemical Language
+
Dynamic-State Language
```

---

# Exploration benchmark

The final benchmark uses:

```text
Number of materials: 43
Initial materials:   30
Held-out materials:  13
Repeated splits:     200
Random state:        42
```

Materials are sequentially selected from the held-out set using a maximin strategy in representation space.

Selection is therefore determined exclusively from the representation variables (**X**).

Solution-NMR responses are not available to the selection algorithm.

---

## Independent response spaces

After selection, the selected materials are evaluated in independent Solution-NMR response spaces.

The primary response space is:

```text
Glycerol-associated response
×
Alkyl-chain response
```

A secondary response space is:

```text
Glycerol-associated response
×
Alkenyl response
```

These response spaces are used only for post-selection evaluation.

---

# Prevention of information leakage

A central design principle of the analysis is the strict separation of representation construction and experimental evaluation.

```text
Chemical structure/composition ─┐
                                ├── X ──> material selection
TD-NMR molecular dynamics ──────┘

Solution-NMR responses ─────────────> Y ──> independent evaluation
```

Solution-NMR response descriptors are therefore:

- not used to construct the RDKit representation,
- not used to construct Chemical Language,
- not used to construct Dynamic-State Language,
- not used to calculate material-selection distances,
- not used during sequential maximin selection.

They are used only after selection as independent experimental evaluation endpoints.

In addition, data-dependent thresholds used to construct Dynamic-State Language during the repeated exploration benchmark are estimated using only the **30 initial materials within each repeat**.

This prevents information from the held-out materials from contributing to representation construction.

---

# Distance construction and material selection

Each representation block is treated separately before multimodal integration.

Numeric representations are standardized using the initial materials within each repeat.

For each representation block, pairwise distances are normalized relative to the distance distribution of the initial materials.

Multiple representation blocks are subsequently combined with equal block weighting.

Sequential maximin selection then identifies the held-out material that maximizes its minimum distance from the already selected materials.

This procedure is repeated until all held-out materials have been selected.

---

# Evaluation metrics

## Response-space coverage

Exploration efficiency is evaluated by the extent to which sequentially selected materials cover the independent Solution-NMR response space.

Coverage is calculated across the complete selection trajectory.

The resulting coverage trajectory is summarized using the **coverage area under the curve (Coverage AUC)**.

---

## Early-selection probability

For each copolymer, the probability of appearing among the first five selected held-out materials is calculated across the repeated exploration runs.

The representation-dependent selection shift is evaluated as:

```text
Δ Top-5 selection probability
=
P(Top-5 | N3)
-
P(Top-5 | N0)
```

Positive values indicate that a material is selected more frequently after inclusion of both language representations, whereas negative values indicate reduced early-selection frequency.

---

## Monomer-level interpretation

Selection shifts are subsequently summarized according to the chemical roles of the copolymer building blocks:

```text
Hydrophilic monomer
Hydrophobic monomer
Crosslinker
```

Individual points represent copolymers containing the corresponding building block.

Summary statistics are reported as:

```text
mean ± SD
```

across the corresponding copolymers.

This analysis connects representation-dependent changes in material selection back to chemically interpretable building-block identities.

---

# Figure source data

Source data used to generate the final exploration figure are provided in:

```text
data/source_data/
```

The principal files are:

```text
Fig4_AUC_200repeats.csv
Fig4_selection_probability_complete_43x4.csv
Fig4_selection_shift_by_copolymer.csv
Fig4_top12_selection_shift.csv
Fig4_monomer_level_summary.csv
```

### `Fig4_AUC_200repeats.csv`

Contains Coverage AUC values for the four representation conditions across the repeated exploration runs.

### `Fig4_selection_probability_complete_43x4.csv`

Contains Top-5 selection probabilities for all 43 copolymers and all four representation conditions.

Materials that were never selected within the Top-5 under a given condition are explicitly represented with:

```text
Top5_Count = 0
Top5_Probability = 0
```

### `Fig4_selection_shift_by_copolymer.csv`

Contains the material-level difference in Top-5 selection probability between N3 and N0.

### `Fig4_top12_selection_shift.csv`

Contains the 12 copolymers showing the largest absolute changes in Top-5 selection probability between N3 and N0.

### `Fig4_monomer_level_summary.csv`

Contains the monomer-level summary of representation-dependent selection shifts.

---

# Reproducing the analysis

The recommended execution order is:

```bash
python scripts/01_TDNMR_descriptor_extraction.py
python scripts/02_Dynamic_State_Language.py
python scripts/03_SolutionNMR_endpoint_generation.py
python scripts/04_Chemical_representation.py
python scripts/05_Final_exploration_analysis.py
```

The output of each stage is used as input for the subsequent stages where applicable.

The complete workflow can therefore be summarized as:

```text
01
TD-NMR
↓
Numeric dynamic descriptors

02
TD-NMR descriptors
↓
Dynamic-State Language

03
Solution-NMR
↓
Independent experimental Y

04
Composition + molecular structure
↓
RDKit numeric + Chemical Language

05
Chemical + TD-NMR representations
↓
Sequential exploration
↓
Independent Solution-NMR evaluation
↓
Figure source data
```

---

# Python environment

The analysis was developed using Python 3.

Major packages include:

```text
numpy
pandas
scipy
scikit-learn
matplotlib
rdkit
sentence-transformers
sentencepiece
openpyxl
```

The sentence-embedding model is:

```text
sentence-transformers/all-MiniLM-L6-v2
```

A complete list of package versions used for the final analysis is provided in:

```text
requirements.txt
```

For maximum reproducibility, users are encouraged to install the versions specified in that file.

---

# Data dictionary

Descriptions of the principal data files, variables, units, and roles in the analysis are provided in:

```text
docs/data_dictionary.csv
```

The distinction between representation variables and evaluation variables is particularly important:

```text
X:
Chemical structure/composition
RDKit descriptors
Chemical Language
TD-NMR descriptors
Dynamic-State Language

Y:
Independent Solution-NMR responses
```

---

# Reproducibility notes

The repository is organized to distinguish between:

```text
raw/        Experimental or primary analysis inputs
processed/  Intermediate representations used by subsequent scripts
source_data/ Numerical data underlying the final manuscript figure
```

Intermediate and diagnostic files generated by the scripts may not all be included in the repository because they can be regenerated by running the corresponding analysis scripts.

Historical exploratory analyses are not required to reproduce the final manuscript analysis.

---

# Citation

If you use this dataset or analysis workflow, please cite the associated publication.

```text
[Publication information will be added after publication.]
```

---

# License

Please refer to the `LICENSE` file for the terms governing reuse of the code and data.

---

# Contact

For questions regarding the experimental data or analysis workflow, please contact the corresponding author of the associated publication.
