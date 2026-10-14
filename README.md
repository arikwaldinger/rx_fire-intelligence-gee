# Wisconsin Prescribed Fire Burn Severity & Recovery Tool

![GEE Prescribed Fire Intelligence](https://img.shields.io/badge/Google_Earth_Engine-JavaScript-4285F4?style=for-the-badge&logo=googleearthengine&logoColor=white)
![Platform](https://img.shields.io/badge/Sentinel--2-10m_Resolution-00A859?style=for-the-badge)
![Machine Learning](https://img.shields.io/badge/Random_Forest-200_Trees-FF6600?style=for-the-badge)

## Overview
Automated Google Earth Engine (GEE) machine learning pipeline designed to delineate prescribed burn perimeters, quantify burn severity (dNBR), evaluate management effectiveness, and track multi-year vegetation recovery across complex Wisconsin ecosystems. 

Developed by **Arik Waldinger** (M.S. Environmental Observation & Informatics, UW–Madison).

---

## Key Technical Highlights

* **100% Field-Verified Ground Truth:** Trained on **128 hand-digitized training polygons** delineated from first-hand, ground-based participation in active prescribed burn operations across Wisconsin.
* **Adaptive Phenology & Dynamic Temporal Windows:** Resolves variable spring green-up rates across woodland, grassland, barrens, and wetland communities by pairing custom pre/post date ranges. Employs a **year-over-year leaf-on baseline (2024 vs. 2025)** for late-season woodland fires where May canopy leaf-out hides post-burn ground scars.
* **33-Band Feature Stack:** Fuses raw Sentinel-2 MSI spectral bands, specialized burn indices (NBR, NBR2, MIRBI, BAIS2, BAI, BSI, CSI, SWIR ratio), vegetation condition metrics (NDVI, EVI, NDRE), and multi-temporal change deltas ($\Delta \text{NBR}, \Delta \text{NDVI}, \Delta \text{BAIS2}, \Delta \text{MIRBI}$, etc.).
* **Adaptive Minimum NBR Reducer:** Extracts post-fire ash/char signatures across multi-week post-burn image collections before canopy leaf-out or vegetation green-up obscures the signal.
* **Fire Effectiveness Index (FEI):** Translates raw severity values into an operational management scorecard:
  $$\text{FEI} = \begin{cases} +1 & \text{Target Burn Severity (Low / Moderate)} \\ 0 & \text{Unburned / Mosaic} \\ -1 & \text{Overburn / Excessive High Severity} \end{cases}$$
* **Long-Term Vegetation Recovery Tracking:** Computes peak summer (July–August) multi-year NDVI trajectories (2022–2025) and percentage vigor change tables per unit.

---

## 33-Band Feature Stack Architecture

The Random Forest model evaluates 33 engineered spatial features per pixel across three seasonal composites:

| Feature Category | Band / Index Names | Ecological Significance |
| :--- | :--- | :--- |
| **Post-Fire Spectral Core** | `NBR`, `NBR2`, `NDVI`, `EVI`, `BAI`, `BSI`, `MNDWI`, `CRI` | Baseline post-burn spectral response and ash reflectance. |
| **Pre-Fire Baselines** | `preNBR`, `preNDVI`, `preBSI`, `preBAI`, `preNDRE`, `preCSI` | Pre-burn vegetation vigor and bare soil context. |
| **Delta ($\Delta$) Indices** | `dNBR`, `dBSI`, `dBAI`, `dNDVI`, `dNDRE`, `dMIRBI`, `dBAIS2` | Absolute spectral change between pre- and post-fire states. |
| **Specialized Ash/Char** | `BAIS2`, `MIRBI`, `NDRE`, `CSI`, `SWIR_ratio`, `NBR_NDVI` | High-sensitivity shortwave-infrared (SWIR) and red-edge indices for char detection. |
| **Mid-Season Phenology** | `midNBR`, `midNDVI`, `midBSI`, `midBAI`, `midBAIS2`, `midNDRE` | Green-up velocity context across April–May transitions. |

---

## Monitored Prescribed Burn Units (16 Units)

The tool processes 16 active management units across diverse Wisconsin ecoregions:

| Unit Name | Target Burn Date | Primary Ecological Cover Type | Baseline Phenology Strategy |
| :--- | :--- | :--- | :--- |
| **Avoca** | 2025-03-10 | Wetland / Woodland | Standard 2025 Early Baseline |
| **Cruson Slough** | 2025-03-13 | Wetland / Woodland | Standard 2025 Early Baseline |
| **Arena** | 2025-03-18 | Woodland / Wetland | Standard 2025 Early Baseline |
| **Westport** | 2025-03-25 | Grassland / Savanna | Standard 2025 Early Baseline |
| **Mazomanie** | 2025-03-27 | Barrens / Savanna | Standard 2025 Early Baseline |
| **Marbleseed** | 2025-04-01 | Grassland | Standard 2025 Early Baseline |
| **Blue River** | 2025-04-04 | Barrens | Standard 2025 Early Baseline |
| **Devil's Lake** | 2025-04-08 | Woodland | Year-Over-Year (May 2024 vs May 2025) |
| **Dells WI** | 2025-04-12 | Woodland | Year-Over-Year (May 2024 vs May 2025) |
| **Hardscrabble** | 2025-04-16 | Woodland / Savanna | Year-Over-Year (Apr/May 2024 vs 2025) |
| **Yellowstone** | 2025-04-25 | Woodland / Grassland / Savanna | Year-Over-Year (May 2024 vs May 2025) |
| **Yorks** | 2025-04-26 | Grassland | Standard 2025 Early Baseline |
| **Bakkens** | 2025-04-27 | Savanna / Grassland | Standard 2025 Early Baseline |
| **Belmont** | 2025-04-30 | Grassland | Standard 2025 Early Baseline |
| **Lower Narrows** | 2025-05-01 | Woodland | Year-Over-Year (May 2024 vs May 2025) |
| **Natural Bridge** | 2025-05-09 | Woodland / Grassland | Year-Over-Year (May 2024 vs May 2025) |

---

## Machine Learning & Evaluation Methodology

1. **Random Forest Classifier:** Trained using `ee.Classifier.smileRandomForest` (200 trees, 4 variables per split, seed 42).
2. **Train/Test Validation Split:** 80% of ground-truth sample points allocated to training, 20% holdout set reserved for unbiased evaluation.
3. **Validation Metrics:**
   * **Overall Accuracy & Kappa ($\kappa$) Statistic**
   * **Jaccard Index (Intersection over Union)** specifically calculated for the Burned class:
     $$\text{Jaccard} = \frac{\text{True Positives}}{\text{True Positives} + \text{False Positives} + \text{False Negatives}}$$
   * **Ranked Variable Importance:** Evaluates feature contribution across all 33 bands to identify dominant spectral discriminators.

---

## How to Run in Google Earth Engine

1. Open the master script file: [`scripts/wi_prescribed_fire_tool.js`](scripts/wi_prescribed_fire_tool.js).
2. Copy the entire JavaScript code block.
3. Paste into your [Google Earth Engine Code Editor](https://code.earthengine.google.com/).
4. Ensure access to the training asset (`projects/code-test-476615/assets/Burn_TrainingData`) or swap in your own `ee.FeatureCollection` with a `class_id` property.
5. Click **Run**.
6. Inspect the Console tab for accuracy benchmarks, the 33-band feature glossary, per-unit acreage summaries, and the multi-year NDVI recovery chart.

---

## Acknowledgments & Disclaimers

* **Field Operations & Research:** Delineated and analyzed by Arik Waldinger through direct operational involvement in Wisconsin prescribed burns.
* **AI Code Optimization:** Assistance provided by Claude (Anthropic AI) for debugging, structural optimization, and array handling. All ecological logic, temporal parameterization, field data collection, and model design decisions were directed by the primary author.
* **Data Sources:** Uses public Copernicus Sentinel-2 Harmonized Surface Reflectance (`COPERNICUS/S2_SR_HARMONIZED`) imagery hosted on Google Earth Engine.
