# KanarekLabSkylineTyrosineTracingAnalysisPipeline
R Scripts and supplemental files to process mass spectrometry data from tyrosine stable isotope tracing experiments

This repository contains the full analysis pipeline for processing ¹³C/¹⁵N tyrosine stable isotope tracing data, from peak picking in Skyline through QC/normalization and natural abundance correction.

---

## Pipeline Overview

```
Raw LC-MS data
      │
      ▼
Stage 0: Peak Picking (Skyline)
      │
      ▼
Stage 1: QC & Normalization (R)
      │
      ▼
Stage 2: Isotope Natural Abundance Correction (IsoCorrectoR)
      │
      ▼
Corrected isotopologue distributions (tyrosine, fumarate, malate)
```

---

## Repository Contents

| File | Description |
|------|-------------|
| `Skyline_Tyrosine_Labeling_Database.xlsx` | Skyline small molecule database for peak picking (166 metabolites, QReSS-13C-15N-Tyrosine list) |
| `STAGE_1_QC_and_normalization_skyline.R` | R script for QC and normalization of Skyline output |
| `STAGE_1_QC_and_normalization_skyline.docx` | Step-by-step user guide for the Stage 1 R script |
| `STAGE_2_Natural_abundance_correction_using_El_Maven_and_IsoCorrectoR.docx` | User guide for natural abundance correction using IsoCorrectoR |
| `IsoCorrectoR_MoleculeInfo_Tyrosine_Tracing.xlsx` | IsoCorrectoR molecule file for tyrosine, fumarate, and malate |
| `IsoCorrectoR_ElementInfo_Tyrosine_Tracing_99purity.csv` | IsoCorrectoR element file (C, N, O, S natural abundances; 99% tracer purity) |

---

## Stage 0: Peak Picking in Skyline

**Input:** Raw LC-MS data files  
**Reference file:** `Skyline_Tyrosine_Labeling_Database.xlsx`

The Skyline small molecule database (`QReSS-13C-15N-Tyrosine` sheet) contains 166 metabolites with precursor formulas, charges, adducts (M+H or M-H), and explicit retention times with windows.

### Steps

1. Open Skyline and import the small molecule database (`Skyline_Tyrosine_Labeling_Database.xlsx`).
2. Import your raw LC-MS data files.
3. Review peak picking and manually curate integration boundaries as needed.
4. Export peak area data as a consolidated `.csv` file (one file with all samples as columns and metabolites as rows).

---

## Stage 1: QC and Normalization

**Input:** Consolidated Skyline `.csv` export  
**Script:** `STAGE_1_QC_and_normalization_skyline.R`  
**Guide:** `STAGE_1_QC_and_normalization_skyline.docx`

### Prerequisites

- R and RStudio installed
- Required packages:

```r
install.packages(c("dplyr", "purrr"))
```

### Running the Script

Run the full script in RStudio (`Cmd + Option + R` on Mac; `Ctrl + Alt + R` on Windows). The script is interactive and will prompt you for input at several steps:

1. **File selection** — A file picker dialog will open; select your consolidated Skyline `.csv`.
2. **Sample removal (optional)** — Review the data table and optionally remove blanks, washes, or failed injections by entering unique name identifiers.
3. **QC sample definition** — Enter keywords identifying your 1× pool injections (minimum 3 replicates for CV) and dilution series factors for R² calculation.
4. **QC thresholds** — Set minimum R² (e.g., `0.95`) and maximum CV (e.g., `0.30`) cutoffs for metabolite filtering.

### Outputs

All outputs are saved to an `RscriptOutput/` subfolder in the same directory as your input file:

| Output File | Description |
|-------------|-------------|
| `MSdataframe.csv` | Transposed raw data |
| `filteredstandards.csv` | Peak areas of internal standards |
| `normalization_vector_fromstandards.csv` | Scaling factors from internal standards |
| `sampledata_normalized_withRsq_and_CV.csv` | Standard-normalized data with R² and CV per metabolite |
| `RsqCVfiltered_normalized_sampledata.csv` | Data after removing metabolites failing QC thresholds |
| `finaldata.csv` | **Final result** — normalized to total polar metabolites with QC metrics |

---

## Stage 2: Natural Abundance Correction (IsoCorrectoR)

**Input:** Isotopologue peak areas exported from Skyline (formatted as an IsoCorrectoR measurement file)  
**Reference files:** `IsoCorrectoR_MoleculeInfo_Tyrosine_Tracing.xlsx`, `IsoCorrectoR_ElementInfo_Tyrosine_Tracing_99purity.csv`  
**Guide:** `STAGE_2_Natural_abundance_correction_using_El_Maven_and_IsoCorrectoR.docx`

Natural abundance correction is performed for **tyrosine** (¹³C and ¹⁵N dual-tracer), **fumarate** (¹³C), and **malate** (¹³C).

### Molecule File

The molecule file (`IsoCorrectoR_MoleculeInfo_Tyrosine_Tracing.xlsx`) defines the molecular formulas and tracer labeling schemes:

| Molecule | MS Ion Formula |
|----------|----------------|
| tyrosine | C9H11N1O3LabC9LabN1 |
| fumarate | C4H4O4LabC4 |
| malate | C4H6O5LabC4 |

### Element File

The element file (`IsoCorrectoR_ElementInfo_Tyrosine_Tracing_99purity.csv`) specifies natural isotope abundances for C, N, O, and S, with 99% purity specified for ¹³C and ¹⁵N tracers.

### Formatting the Measurement File

Before running IsoCorrectoR, format your isotopologue data as a measurement file:

- Row 1, Column A must read `Measurements/samples`
- Row headers = sample names (must be unique)
- Column A entries = `metabolitename_MID`, e.g.: `tyrosine_C0.N0`, `tyrosine_C1.N0`, `tyrosine_C0.N1` (dual tracer: `C` and `N` MIDs separated by a period)
- Replace all `0` values with blanks (empty cells)
- Save as `.csv`

### Running IsoCorrectoR

Open **R (not RStudio)** and run:

```r
library(IsoCorrectoRGUI)
IsoCorrectionGUI()
```

In the GUI, provide paths to:
- Your measurement file (`.csv`)
- `IsoCorrectoR_MoleculeInfo_Tyrosine_Tracing.xlsx`
- `IsoCorrectoR_ElementInfo_Tyrosine_Tracing_99purity.csv`
- Your desired output directory

**Important settings:**
- ✅ Check **High Resolution Mode** (required for dual ¹³C/¹⁵N tyrosine tracing)
- ✅ Check **Correct Tracer Impurity** (tracer purity 0.99 is provided in the element file)
- ✅ Check **Calculate Mean Enrichment**

### Outputs

IsoCorrectoR outputs corrected isotopologue distributions, fractional enrichments, and mean enrichment values for tyrosine, fumarate, and malate.

---

## Dependencies Summary

| Tool | Version | Purpose |
|------|---------|---------|
| [Skyline](https://skyline.ms) | Latest | Peak picking |
| R / RStudio | ≥ 4.0 | QC & normalization |
| dplyr | CRAN | Data wrangling |
| purrr | CRAN | Functional programming |
| [IsoCorrectoR](http://bioconductor.org/packages/devel/bioc/vignettes/IsoCorrectoR/inst/doc/IsoCorrectoR.html) | Bioconductor | Natural abundance correction |
| IsoCorrectoRGUI | Bioconductor | GUI wrapper for IsoCorrectoR |

---

## References

- IsoCorrectoR documentation: http://bioconductor.org/packages/devel/bioc/vignettes/IsoCorrectoR/inst/doc/IsoCorrectoR.html
