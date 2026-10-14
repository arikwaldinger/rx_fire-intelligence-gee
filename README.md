# Full-Cycle Fire Intelligence & Soil Risk Model in Google Earth Engine

![Fire Intelligence Banner](assets/fire_intelligence_header.png)

## Overview
This repository contains the complete Google Earth Engine (GEE) JavaScript pipeline developed for end-to-end prescribed burn and wildfire analysis across 2,700 acres. The model integrates multi-sensor satellite imagery (Sentinel-2 MSI, Landsat 8/9, Sentinel-1 SAR) and topographic terrain models to evaluate pre-fire fuel moisture, map post-fire burn severity (dNBR), model soil erosion hazards, and monitor long-term post-fire ecological recovery.

Developed by **Arik Waldinger** (M.S. Environmental Observation & Informatics, UW–Madison).

---

## Technical Architecture & Feature Stack

The pipeline constructs a 33-band spectral and topographic feature stack per pixel across four core operational phases:

1. **Pre-Fire Risk & Fuel Assessment:** Analyzes canopy fuel moisture and live fuel density using Normalized Difference Water Index (NDWI), Enhanced Vegetation Index (EVI), and Sentinel-1 C-Band SAR backscatter for structural roughness.
2. **Post-Fire Char & Severity Mapping:** Applies a custom **Minimum NBR Reducer** across multi-week post-fire image collections to capture peak combustion char signatures prior to vegetation green-up. Calculates delta NBR (dNBR) and categorizes USGS burn severity tiers.
3. **Soil Burn Severity & Erosion Risk Model:** Combines dNBR char metrics with USGS 10m DEM slope, flow accumulation, and Revised Universal Soil Loss Equation (RUSLE) soil erodibility ($K$-factor) data to isolate high-risk hydrological runoff zones.
4. **Multi-Year Ecological Recovery Tracking:** Computes normalized recovery trajectories ($\% \text{NDVI}$ recovery) to evaluate vegetation re-establishment and invasive species encroachment over 1- to 3-year post-fire windows.

---

## Pipeline & Script Architecture

| Script | Phase | Description / Core Algorithms | Key Output |
| :--- | :--- | :--- | :--- |
| [`01_spectral_stack_builder.js`](scripts/01_spectral_stack_builder.js) | Data Prep | Synthesizes Sentinel-2, Sentinel-1 SAR, Landsat LST, and USGS 10m DEM into a aligned 33-band raster stack. | 33-Band Stack |
| [`02_prefire_fuel_moisture_indices.js`](scripts/02_prefire_fuel_moisture_indices.js) | Pre-Fire | Calculates NDWI, Moisture Stress Index (MSI), and SAR canopy density prior to ignition. | Pre-Fire Fuel Risk Map |
| [`03_minimum_nbr_char_reducer.js`](scripts/03_minimum_nbr_char_reducer.js) | Fire Event | Custom temporal reducer isolating absolute minimum NBR (darkest char/ash signature) post-burn. | Peak Char Signature |
| [`04_burn_severity_dnbr_classified.js`](scripts/04_burn_severity_dnbr_classified.js) | Severity | Computes dNBR ($\text{NBR}_{\text{pre}} - \text{NBR}_{\text{post}}$) and classifies into USGS unburned, low, moderate, and high severity zones. | Classified Burn Severity Map |
| [`05_soil_burn_severity_erosion_risk.js`](scripts/05_soil_burn_severity_erosion_risk.js) | Soil Hazard | Integrates dNBR with DEM slope, TPI, and CTI to highlight post-fire erosion and debris flow corridors. | Hydrological Soil Risk Map |
| [`06_postfire_recovery_monitoring.js`](scripts/06_postfire_recovery_monitoring.js) | Recovery | Tracks multi-year NDVI trajectory against pre-fire baselines to verify native plant recruitment. | Recovery Rate Surface |

---

## Key Performance Indicators & Results

* **Coverage Extent:** 2,700 acres across heterogeneous Driftless Area topography.
* **Spatial Resolution:** 10-meter native spatial grid.
* **Spectral Sensitivity:** Peak char retention captured via SWIR (B12) and NIR (B8) spectral difference.
* **Erosion Risk Calibration:** Successfully identified steep coulee drainages (>25% slope) prone to post-burn soil loss.

---

## Institutional & Data Disclaimer

*This project was developed for academic and ecological modeling research purposes. Spatial datasets and analytical outputs do not constitute official emergency response protocols or regulatory management decisions. Data processing relies on public Copernicus Sentinel-2/Sentinel-1 and USGS satellite streams hosted within Google Earth Engine.*

---

## How to Run in Google Earth Engine

1. Open any script from the [`scripts/`](scripts/) folder in this repository.
2. Copy and paste the JavaScript code into your [Google Earth Engine Code Editor](https://code.earthengine.google.com/).
3. Define or upload your study area boundary (`ee.Geometry.Polygon` or `ee.FeatureCollection`).
4. Set the pre-burn and post-burn date parameters at the top of the script.
5. Click **Run**.
