# EDA Findings — Proxy ET Ghana

This document records decisions made during the exploratory data analysis phase. It is the authoritative reference for the ingestion and pipeline scripts that follow. Each section maps to one EDA notebook.

---

## 1. Data Availability Audit (`00_data_availability_audit.md`)

**Common grid:** 1 km, EPSG:32630 (UTM Zone 30N)

Rationale: MODIS LST (primary ET driver) is native 1 km. Coarser sources (SMAP 9 km, ERA5-Land 9 km, GPM 11 km) carry no sub-1km spatial information — reprojecting them below native resolution would add false precision. Fine-resolution layers (DEM 30 m, NDVI 250 m) are aggregated up to 1 km.

**Input sources confirmed:**

| Source | Variable | Res | Temporal | Access |
|--------|----------|-----|----------|--------|
| MODIS MOD11A1 | LST | 1 km | Daily | NASA Earthdata (earthaccess) |
| MODIS MOD13Q1 | NDVI/EVI | 250 m | 16-day | NASA Earthdata (earthaccess) |
| SMAP SPL3SMP_E | Soil Moisture | 9 km | ~Daily | NASA Earthdata (earthaccess) |
| GPM GPM_3IMERGDF | Precipitation | 11 km | Daily | NASA Earthdata (earthaccess) |
| ERA5-Land | Met forcing | 9 km | Hourly | Copernicus CDS (cdsapi) |

*Update this table with any sources added or dropped after the sample pull.*

---

## 2. Visual Inspection (`02_visual_inspection.ipynb`)

*Fill in after running the notebook.*

- LST urban heat island visible: <!-- Y/N -->
- NDVI vegetation pattern expected: <!-- Y/N -->
- Georeferencing offsets observed: <!-- Y/N — describe if yes -->
- Artefacts noted: <!-- list any -->
- FloodWatch DEM cross-reference result: <!-- consistent / inconsistent -->

---

## 3. CRS and Resolution Reconciliation (`03_crs_resolution.ipynb`)

**Reference grid:** `<!-- fill in pixel-aligned UTM bounds after running notebook 03 -->`

**Resampling decisions:**

| Source | Direction | Method | Notes |
|--------|-----------|--------|-------|
| MODIS LST 1km | Reproject only | Bilinear | |
| MODIS NDVI 250m | Aggregate → 1km | Block mean (4×4) | |
| SMAP 9km | Disaggregate → 1km | Bilinear | No new info added |
| GPM 11km | Disaggregate → 1km | Bilinear | |
| ERA5-Land 9km | Disaggregate → 1km | Bilinear | Check lat orientation on open |
| DEM 30m | Aggregate → 1km stats | Mean elev + std slope | Static auxiliary layer |

*Update this table after confirming alignment in section 6 of notebook 03.*

---

## 4. Temporal Alignment and Gap-Fill Strategy (`04_temporal_alignment.ipynb`)

*This section is the primary methodological contribution of the EDA phase.*

**Scene exclusion threshold:** <!-- e.g. "Scenes where <70% of AOI pixels are valid are treated as gaps" -->

**Gap-fill rules by source:**

| Source | Rule | Max fill window | Fallback |
|--------|------|----------------|---------|
| MODIS LST | <!-- e.g. nearest valid within 5 days --> | <!-- N days --> | <!-- exclude / climatology --> |
| MODIS NDVI | 16-day composite provides implicit fill | N/A | Nearest composite |
| SMAP | <!-- microwave — few gaps expected --> | | |
| GPM | Gap-free (reanalysis-merged) | N/A | N/A |
| ERA5-Land | Gap-free (reanalysis) | N/A | N/A |

**Worst-case gap found (LST, wet season):** <!-- N days, date range -->

**QA flag convention:**
- `0` — direct observation
- `1` — gap-filled (within acceptable window)
- `2` — excluded (gap exceeds threshold or fill quality insufficient)

**Note:** Any time step where QA flag = 1 for LST should be treated as lower-confidence in downstream ET estimation. Consider weighting by flag in model training.

---

## 5. Distribution and Correlation Checks (`05_distributions_correlations.ipynb`)

*Fill in after running the notebook.*

**Physical sanity check results:**

| Check | Expected | Observed | Pass? |
|-------|----------|----------|-------|
| corr(LST, NDVI) | < −0.2 | | |
| corr(SM_t, precip_t−1) | > +0.1 | | |
| corr(LST, T2m ERA5) | > +0.5 | | |
| NDVI in [−0.1, 0.9] | 100% | | |
| LST in [20, 55] °C | 100% | | |

**Recommended feature set for ET model:**
- *To be filled after correlation evidence reviewed*

**Data quality issues to carry forward:**
- *List any layers that raised concerns*

---

## Open Questions for Pipeline Phase

*Record anything that the EDA raised but did not resolve — these become issues/decisions in the pipeline design phase.*

1. <!-- e.g. "ESA CCI at 25km may be too coarse to add signal over SMAP at 9km — confirm via partial correlation" -->
2.
3.
