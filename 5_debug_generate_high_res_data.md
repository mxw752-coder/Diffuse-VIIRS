# Crop and Export CARE-SST Final Product

This script is a debug version for the CARE-SST workflow.

It crops a smaller **Area of Interest (AOI)** from the full CARE-SST result, visualizes the cropped product for inspection, and exports the cropped image as a GeoTIFF while preserving its geographic referencing.



# Workflow

```
Full CARE-SST Product
        │
        ▼
Build Longitude/Latitude Grid
        │
        ▼
Specify Geographic Crop
        │
        ▼
Extract Pixel Window
        │
        ▼
Crop All Intermediate Products
        │
        ▼
Visual Inspection
        │
        ▼
Save Cropped GeoTIFF
```

---

# Inputs

This script assumes the complete CARE-SST processing has already been finished.

Required variables include:

- Final CARE-SST product
- IDW result
- Land mask
- Guide image
- Bias map
- Raster transform
- Coordinate reference system (CRS)

The cropping step uses geographic coordinates rather than pixel indices.

Example:

```python
CROP_LON_MIN = 38.95
CROP_LON_MAX = 40.77

CROP_LAT_MIN = -17.67
CROP_LAT_MAX = -15.95
```

The script automatically converts these geographic limits into raster row and column indices.

---

# Processing

For the selected Area of Interest, the script

1. Builds the longitude/latitude grid from the raster transform.
2. Identifies all pixels inside the geographic extent.
3. Computes the crop window automatically.
4. Crops every intermediate CARE-SST product.
5. Displays the cropped results for visual inspection.
6. Builds the correct affine transform for the cropped image.
7. Saves the cropped GeoTIFF.

Because the crop is based on geographic coordinates, no manual pixel calculations are required.

---

# Output

The exported file is written to

```
result/

└── debug_check/
        YYYYMMDD_care_sst_cropped2.tif
```

The output GeoTIFF contains

| Property | Value |
|----------|-------|
| Format | GeoTIFF |
| Data type | Float32 |
| CRS | Same as original CARE-SST product |
| Transform | Updated for cropped region |
| Compression | LZW |
| NoData | -9999 |

---

# Cropped Products

The following layers are cropped simultaneously for visualization:

- Input IDW product
- Land mask
- VIIRS after land masking
- Land-filled guide
- Smoothed guide
- Raw CARE-SST
- Bias map
- Final CARE-SST product

Only the **final CARE-SST product** is exported as a GeoTIFF.

---

# User Parameters

## Geographic Crop

```python
CROP_LON_MIN
CROP_LON_MAX

CROP_LAT_MIN
CROP_LAT_MAX
```

These values define the output region.

---

## Output Directory

```python
OUT_DIR
```

Directory used to save the cropped GeoTIFF.

---

## Output Filename

```python
OUT_TIF
```

Name of the exported GeoTIFF.

---

# Important Quality Control

## **Always verify the exported GeoTIFF before using it in any downstream analysis.**

After the GeoTIFF is saved:

1. Open the file in **ArcGIS Pro**, **QGIS**, or another GIS software.
2. Overlay it with the original satellite imagery, coastline, or land mask.
3. Carefully verify that:
   - the geographic location is correct;
   - the coastline aligns with the land mask;
   - the crop covers the intended Area of Interest;
   - the image is not shifted, rotated, or flipped;
   - the CRS and affine transform are correct.

> **This verification step is strongly recommended whenever the crop region, projection, or raster transform changes.** A small error in the affine transform can cause the exported GeoTIFF to appear in the wrong geographic location, even if the image itself looks correct.

---

# Notes

- The crop is performed using geographic coordinates rather than pixel indices.
- The affine transform is automatically updated so that the exported GeoTIFF retains its correct spatial reference.
- No resampling is performed during cropping; the original raster resolution is preserved.
- Only the selected Area of Interest is written to disk.

---

# Typical Use Cases

- Creating regional study areas
- Publication-quality figures
- Reducing file size for sharing
- Site-specific validation
- Preparing data for GIS analysis

---

# Recommended Workflow

```
Run CARE-SST
        │
        ▼
Generate Final Product
        │
        ▼
Crop Area of Interest
        │
        ▼
Export GeoTIFF
        │
        ▼
✔ Open in ArcGIS Pro / QGIS
✔ Verify geographic alignment
✔ Confirm coastline and projection
        │
        ▼
Use for analysis or publication
