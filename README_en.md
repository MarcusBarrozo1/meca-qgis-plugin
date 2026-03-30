# MECA: Estimated Water Consumption Model for Agriculture 💧🛰️

A Python/QGIS implementation based on the SAFER algorithm for estimating evapotranspiration and quantifying water consumption in the semi-arid region.

## 📌 Executive Summary: Business Problem
Efficient water resource management in semi-arid regions, especially during the rainy season and drought periods, requires rigorous monitoring. The primary objective of this project is not agronomic crop classification, but rather the **volumetric quantification of water demand (in m³/day and mm/month)** based on the actual planted area (in hectares) belonging to **raw water users**.

The model aims to provide concrete data to support enforcement, water rights granting, and watershed water balance management, restricting processing strictly to the mapped polygons of each user.

---

## 🚧 Current Status: Initial Version (v0.1.0-alpha)
This repository contains the Minimum Viable Product (MVP) code for the algorithm, developed over seven months of research and structured as a plugin for QGIS.

In the current (legacy) architecture:
- Remote sensing mathematical processing is coupled to the graphical interface (`PyQt`).
- Data ingestion requires prior download of satellite scenes and manual selection of optical bands by the user.
- The output is a consumption estimate based on a single temporal cut (daily), generating intermediate matrices (`.tif`) for Albedo, NDVI, and Net Radiation.

---

## 🔬 Scientific and Methodological Foundation
MECA is based on the **SAFER** algorithm (*Simple Algorithm for Evapotranspiration Retrieving*), originally implemented in `R`. The calculation process in this repository involves the surface energy balance, structured in the following main steps:

1. **Radiometric Correction and Calibration:** Obtaining top-of-atmosphere and surface albedo from optical bands.
2. **Shortwave and Longwave Balance:** Calculating Global Solar Radiation, Net Radiation (Rn), and Atmospheric/Surface Emissivity.
3. **Surface Temperature (Ts):** Thermal estimation of the polygon surface.
4. **Evapotranspiration (ET):** Obtaining the evaporation coefficient and daily actual evapotranspiration by crossing satellite data with local climatic variables (Tmax and Tmin).

---

## 🚀 Refactoring and Data Science Roadmap (Next Steps)
To transform this MVP into a scalable **Geographic Data Science** pipeline, the following architectural and scientific milestones are planned for future versions:

### 1. Architectural Decoupling and Performance
- Extract the SAFER mathematical logic to independent service modules (pure Python).
- Replace heavy disk I/O (writing intermediate `.tif` files) with strictly in-memory processing using vectorized matrix operations via `NumPy`.

### 2. Cloud-Native Geospatial (End Manual Downloads)
- Eliminate downloading entire orbital scenes. Implement data access via APIs (STAC/COGs standard) or platforms such as Google Earth Engine.
- The pipeline should dynamically request only the pixel matrix corresponding to the raw water user's polygon.

### 3. Automated Temporal Aggregation (The Monthly Challenge)
- Evolve the daily output into a **real monthly consumption** estimate.
- Create automated routines to process multiple scenes within a month (e.g. 3 to 4 satellite passes) and calculate the weighted temporal average of consumed volume (m³).

### 4. Multimodal Data Fusion (Solution for Cloud Cover)
- **The Physical Challenge:** The rainy season in Ceará generates severe gaps in optical orbital data due to high cloud cover.
- **The Mapped Solution:** Implement data imputation and multimodal fusion methodologies, integrating satellite imagery with data from **UAVs (drones with multispectral sensors)**. This will ensure continuity of the monthly series and serve as a physical validation layer for the equations.

### 5. Scientific Validation and CI/CD
- Build unit tests (`pytest`) to inject control matrices into the Python code and compare the results (pixel by pixel) with the outputs of the original `R` methodology, ensuring scientific rigor and reproducibility.

### 6. Future Features (Optional)
- Develop an auxiliary module for spectral classification of crop type, aiming to refine specific consumption coefficients if requested by water management authorities.
