# Data Availability Audit — Greater Accra ET Inputs

Last updated: <!-- fill in date after audit -->

## Source Comparison Table

| Source | Variable | Temporal Resolution | Spatial Resolution | Native CRS | Cloud/Gap Frequency (Accra) | Notes |
|--------|----------|--------------------|--------------------|------------|----------------------------|-------|
| MODIS MOD11A1 / MYD11A1 | Land Surface Temperature (LST) | Daily (Terra + Aqua) | 1 km | Sinusoidal (EPSG:6842) | High in wet season — can be >50% missing | Two overpasses/day; combine for better coverage |
| MODIS MOD13A2 | NDVI / EVI | 16-day composite | 1 km | Sinusoidal | Composited — gaps mostly filled within 16d | 250m (MOD13Q1) also available |
| MODIS MOD13Q1 | NDVI / EVI | 16-day composite | 250 m | Sinusoidal | Same as MOD13A2 | Aggregate up to 1km for common grid |
| SMAP L3 | Soil Moisture (surface) | Daily (~2-3 day revisit) | ~9 km (EASE-Grid 2.0) | EASE-Grid 2.0 | Low — microwave, cloud-transparent | L-band signal degrades under dense tropical canopy |
| ESA CCI Soil Moisture | Soil Moisture (merged active+passive) | Daily | ~25 km (0.25°) | Geographic (EPSG:4326) | Low — microwave | Longer record (1978–present); coarser than SMAP |
| GPM IMERG Final | Precipitation | 30-min → daily | ~11 km (0.1°) | Geographic (EPSG:4326) | None — microwave/gauge merged | Use daily accumulation for ET drivers |
| ERA5-Land | T2m, wind, humidity, radiation | Hourly → daily | ~9 km (0.1°) | Geographic (EPSG:4326) | None — reanalysis | Needed for Penman-Monteith met forcing |

## Common Grid Decision

**Proposed common grid: 1 km**

Rationale:
- MODIS LST (primary ET driver) is native 1 km — no upscaling of the key variable
- SMAP, ESA CCI, GPM, ERA5-Land are all coarser; they will be bilinearly interpolated *up* to 1 km (still coarse information, just registered to the grid)
- DEM (30 m, from FloodWatch) and NDVI (250 m) will be aggregated *down* to 1 km (mean elevation, mean slope, mean NDVI per 1 km cell)
- Avoids false spatial precision from pushing 9–11 km data to 30 m

**Reference CRS:** EPSG:32630 (WGS 84 / UTM Zone 30N) — consistent with FloodWatch layers, metric units for area calculations

**Bounding box (Greater Accra):** approximately `[-0.35, 5.45, 0.10, 5.80]` (lon_min, lat_min, lon_max, lat_max) — confirm against FloodWatch AOI polygon

## Gap Frequency Notes

Fill in after pulling sample data in `01_sample_pull.ipynb`. Key questions:
- What is the actual % of valid LST pixels per month over Accra for 2022?
- Does the 16-day NDVI composite reliably fill within-composite cloud gaps, or are some composites still partly masked?
- Maximum contiguous gap length (days) for LST in wet season (Apr–Jul, Sep–Nov)?

## Decision: Maximum Acceptable Gap

*To be filled after step 5 (temporal alignment EDA).*

Candidate rules:
- Exclude any scene where >30% of AOI pixels are masked
- For gap-fill: use nearest valid scene within N days (N = TBD), else fall back to monthly climatology
- Flag any time step where gap-fill is used so downstream models can weight accordingly
