# Limited-Footprint IDW Gap Filling for CARE-SST

This script contains two Python scripts that perform **Inverse Distance Weighting (IDW)** gap filling on CARE-SST outputs before the diffusion stage.

Unlike conventional interpolation, IDW is **strictly limited to the valid tilted VIIRS/L3S observation footprint**, preventing interpolation into areas that were never observed by the satellite.

The final product remains **300 × 300**, making it directly compatible with the downstream CARE-SST workflow.

---

# Repository Structure

```
├── debug cell
│
│   Debug version
│   • Process only the first GeoTIFF
│   • Display intermediate results
│   • Save nothing
│
└── batch process cell
    • Process all GeoTIFFs
    • Apply the same algorithm
    • Save final GeoTIFF products
```

Both scripts use **exactly the same processing algorithm**. The only difference is that the debug version is intended for visualization and quality control, while the batch version processes every image automatically. 

---

# Workflow

```
CARE-SST GeoTIFF
      │
      ▼
Crop center 256×256
(real VIIRS footprint)
      │
      ▼
Read original L3S geometry
      │
      ▼
Reproject land mask
to L3S grid
      │
      ▼
Build valid VIIRS footprint
      │
      ▼
Limited IDW interpolation
(ocean only)
      │
      ▼
Paste back into
300×300 image
      │
      ▼
Save final GeoTIFF
```

---

# Inputs

The scripts require the following datasets.

```
CARE-SST Output
│
├── geotiff/
│      20200101.tif
│      20200102.tif
│      ...
│
├── L3S STAR NetCDF
│      Used to recover the original
│      geographic coordinates
│
└── High-resolution land mask
       landmask.tif
```

### Input Description

| Input | Description |
|--------|-------------|
| CARE-SST GeoTIFF | 300 × 300 SST product |
| L3S STAR | Original Level-3S NetCDF used to recover the true satellite geometry |
| Land Mask | High-resolution binary land mask (1 = land, 0 = water) |

---

# Processing Steps

For every image:

1. Read the 300 × 300 CARE-SST GeoTIFF.
2. Extract the center **256 × 256** L3S observation area.
3. Recover the original L3S geographic transform from the corresponding NetCDF file.
4. Reproject the high-resolution land mask onto the L3S grid.
5. Build the actual tilted VIIRS observation footprint from valid pixels.
6. Remove land pixels.
7. Apply IDW interpolation **only inside the valid observation footprint**.
8. Leave all pixels outside the footprint unchanged.
9. Paste the completed 256 × 256 image back into the original 300 × 300 grid.
10. (Batch version only) Save the final GeoTIFF.

---

# IDW Characteristics

The interpolation is intentionally constrained.

✔ Only valid ocean pixels are used as neighbors.

✔ Land pixels are never interpolated.

✔ Pixels outside the original VIIRS observation footprint remain unchanged.

✔ The final output preserves the original CARE-SST image dimensions.

---

# Debug Script

Purpose

- Process only the first image.
- Display every intermediate processing step.
- Verify land-mask alignment.
- Verify IDW interpolation region.
- Save nothing.

Displayed panels include

1. Original 256 × 256 VIIRS crop
2. Tilted VIIRS footprint
3. Reprojected land mask
4. Land removed
5. IDW result
6. Difference (IDW − Original)

This script should always be used before batch processing to verify that the processing pipeline behaves as expected. :contentReference[oaicite:2]{index=2}

---

# Batch Script

Purpose

- Process all available GeoTIFFs.
- Apply the identical workflow used by the debug script.
- Save the final IDW-filled GeoTIFF for every scene.

Example output

```
idw_filled_geotiff/

├── 2020/
│      20200101.tif
│      20200102.tif
│      ...
│
├── 2021/
│      ...
│
└── 2025/
```

Each output GeoTIFF

- remains **300 × 300**
- uses **EPSG:4326**
- preserves the original L3S geographic location
- is ready for downstream CARE-SST processing. :contentReference[oaicite:3]{index=3}

---

# Important Quality Control

**Always verify the exported GeoTIFF before running downstream processing.**

After running either script:

1. Open the output GeoTIFF in **ArcGIS Pro**, **QGIS**, or another GIS software.
2. Overlay it with the original VIIRS/L3S image or coastline.
3. Confirm that:
   - the GeoTIFF is located in the correct geographic position;
   - the coastline aligns with the land mask;
   - the image is not flipped vertically or horizontally;
   - no unexpected spatial shift or rotation has occurred.

> **This quality check is strongly recommended whenever the AOI, projection, land mask, or L3S source data changes.** Even small georeferencing errors can propagate into subsequent CARE-SST analyses.

---

# User Parameters

| Parameter | Description |
|------------|-------------|
| `IDW_K` | Number of nearest neighbors |
| `IDW_POWER` | Inverse-distance weighting exponent |
| `FLIP_UP_DOWN` | Correct image orientation before processing |
| `FOOTPRINT_DILATE_ITER` | Expand the valid VIIRS footprint |
| `TARGET_SIZE` | Final image size (300 × 300) |
| `CROP_SIZE` | Real L3S observation area (256 × 256) |

---

# Outputs

| Product | Description |
|----------|-------------|
| Debug Script | Visualization only (no files written) |
| Batch Script | Final IDW-filled GeoTIFF for every date |

The exported GeoTIFFs are ready for the next stage of the CARE-SST processing pipeline.
