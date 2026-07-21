# CARE-SST SST Reconstruction with Local VIIRS/OISST Balancing

This script contains two main cells for reconstructing cloud-contaminated
VIIRS Level-3 Sea Surface Temperature (SST) using the CARE-SST diffusion model.

Before reconstruction, valid VIIRS pixels are locally adjusted toward the
corresponding OISST background to reduce systematic differences while preserving
the original spatial variability.

---

# Workflow

VIIRS L3S
      │
      ▼
Cloud Mask
      │
      ▼
Balance valid VIIRS pixels toward OISST
      │
      ▼
CARE-SST Diffusion
      │
      ▼
Hybrid Output

Outside cloud holes:
    Balanced VIIRS

Inside cloud holes:
    CARE-SST reconstruction

Land:
    NaN

---

# CELL 1

## 1. Debug Script

Purpose

- Run CARE-SST on a single scene
- Compare original and balanced inputs
- Visualize intermediate results
- Save nothing

Typical outputs image for QA/QC

- Original VIIRS
- OISST background
- Original CARE-SST result
- Balanced VIIRS
- Balanced CARE-SST result
- Difference before balancing
- Difference after balancing

**This cell is intended for parameter tuning and quality control.**

---
# CELL 2
## Batch Script

Purpose

- Process an entire year of SST scenes
- Run local balancing
- Perform CARE-SST reconstruction
- Save final hybrid products

Outputs

```
caresst_whole_l3s_YYYY/
│
├── geotiff/
│      YYYYMMDD.tif
│
└── npy/        (optional)
```

Only the final hybrid product is exported.

---

# Local Balancing

For valid ocean pixels outside cloud holes,

```
Balanced = OISST + α × (VIIRS − OISST)
```

where

| α | Behavior |
|---|----------|
|1.0|Original VIIRS|
|0.8|Slight adjustment toward OISST|
|0.5|Halfway between VIIRS and OISST|
|0.0|Replace VIIRS with OISST|

Only pixels satisfying

- ocean
- non-cloud
- finite VIIRS
- finite OISST

are adjusted.

Cloud pixels remain unchanged and are reconstructed by CARE-SST.

---

# Hybrid Output

Final SST is constructed as

Outside cloud holes

```
Balanced VIIRS
```

Inside cloud holes

```
CARE-SST prediction
```

Land and invalid pixels

```
NaN
```

Thus the final product preserves observed SST wherever possible while only using
CARE-SST to reconstruct missing regions.

---

# Workflow

```
                +----------------------+
                |  VIIRS L3S SST (.npy)|
                +----------+-----------+
                           |
                           |
                +----------v-----------+
                |  OISST SST (.npy)    |
                +----------+-----------+
                           |
                           |
                +----------v-----------+
                | Binary Land Mask     |
                +----------+-----------+
                           |
                           |
                +----------v-----------+
                | Cloud Mask           |
                +----------+-----------+
                           |
                           |
                +----------v-----------+
                | Local VIIRS/OISST    |
                | Balancing            |
                +----------+-----------+
                           |
                           |
                +----------v-----------+
                | CARE-SST Diffusion   |
                +----------+-----------+
                           |
                           |
                +----------v-----------+
                | Hybrid SST Product   |
                +----------------------+
```

---

# Inputs

The scripts expect the following directory structure:

```
ready_whole_2020_300/

└── cloud/
    ├── viirs/
    │      20200101.npy
    │      20200102.npy
    │      ...
    │
    ├── oisst/
    │      20200101.npy
    │      20200102.npy
    │      ...
    │
    ├── land_mask/
    │      20200101.npy
    │      20200102.npy
    │      ...
    │
    └── lon_lat/
        └── viirs/
               20200101.npy
               20200102.npy
               ...
```

### Input Description

| Input | Description |
|-------|-------------|
| **VIIRS** | 300 × 300 Level-3 SST with cloud gaps |
| **OISST** | Daily background SST used as structural guidance |
| **Land Mask** | Binary land/ocean mask (`1 = land`, `0 = ocean`) |
| **Lon/Lat** | Geographic coordinates for GeoTIFF generation |

---

# Processing

1. Load VIIRS, OISST, land mask, and cloud mask.
2. Balance valid VIIRS pixels toward OISST using:

```
Balanced = OISST + α × (VIIRS − OISST)
```

3. Use the balanced VIIRS as the initial condition for CARE-SST.
4. Reconstruct only cloud-covered pixels.
5. Preserve balanced VIIRS outside cloud holes.
6. Apply land mask.
7. Export the final hybrid SST.

---

# Outputs

The batch script creates:

```
caresst_whole_l3s_2020/

├── geotiff/
│      20200101.tif
│      20200102.tif
│      ...
│
└── npy/        (optional)
       20200101.npy
       ...
```

### Output Description

| Output | Description |
|---------|-------------|
| **GeoTIFF** | Final cloud-free hybrid SST in EPSG:4326 |
| **NumPy (optional)** | Same hybrid SST stored as `.npy` |
| **Land Pixels** | Stored as `NaN` |
| **Ocean Pixels** | Balanced VIIRS outside clouds + CARE-SST reconstruction inside clouds |

---

