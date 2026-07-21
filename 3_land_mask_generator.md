# Generate Land Mask from ESA WorldCover

This script generates a high-resolution binary land mask using the **ESA WorldCover v200 (10 m)** dataset in **Google Earth Engine (GEE)** and exports the result as a GeoTIFF to Google Drive.


---

# Features

- Uses **ESA WorldCover v200** (10 m resolution)
- Automatically buffers the Area of Interest (AOI)
- Creates binary land and water masks
- Removes small coastal artifacts using morphological filtering
- Exports the final land mask to Google Drive
- Reports Earth Engine task status after submission

---

# Workflow

```
Area of Interest
        │
        ▼
Apply Buffer
        │
        ▼
Load ESA WorldCover v200
        │
        ▼
Extract Water Class (80)
        │
        ▼
Invert to Land Mask
        │
        ▼
Morphological Cleaning
        │
        ▼
Export GeoTIFF to Google Drive
```

---

# Input

The only required input is the Area of Interest (AOI).

```python
min_lon = 124.137
max_lon = 127.764

min_lat = -9.739
max_lat = -7.619
```

An optional geographic buffer is added around the AOI.

```python
BUFFER_DEG = 0.2
```

This expands the region before generating the land mask, helping preserve coastline continuity near image boundaries.

---

# Data Source

**ESA WorldCover v200**

- Resolution: **10 m**
- CRS: WGS84 (EPSG:4326)
- Source: Google Earth Engine

Water is identified using:

```
Class 80 = Permanent Water Bodies
```

The binary land mask is generated as

```
Land = WorldCover ≠ 80
```

Resulting values are

| Value | Meaning |
|--------|---------|
| 0 | Water |
| 1 | Land |

---

# Morphological Cleaning

Small coastal artifacts are removed using two morphological operations.

```python
land_mask_clean = (
    land_mask
        .focal_max(radius=20, units="meters")
        .focal_min(radius=20, units="meters")
)
```

These operations

- fill tiny holes
- smooth jagged coastlines
- remove isolated pixels
- improve coastline continuity

The final product remains a binary mask.

---

# Output

The script exports a GeoTIFF to Google Drive.

Example:

```
Google Drive

└── GEE_exports/
        land_mask_timor.tif
```

The exported raster has

| Property | Value |
|----------|-------|
| Format | GeoTIFF |
| CRS | EPSG:4326 |
| Resolution | 10 m |
| Data type | uint8 |
| Values | 0 = Water, 1 = Land |

---

# User Parameters

## AOI

```python
min_lon
max_lon
min_lat
max_lat
```

---

## Buffer

```python
BUFFER_DEG = 0.2
```

---

## Export Settings

```python
EXPORT_FOLDER = "GEE_exports"

EXPORT_FILENAME = "land_mask_timor"

EXPORT_SCALE = 10

EXPORT_CRS = "EPSG:4326"
```

---

# Google Earth Engine Requirements

Before running the script, install the required packages.

```bash
pip install earthengine-api geemap
```

Authenticate Earth Engine (first time only):

```bash
earthengine authenticate
```

or allow the script to authenticate interactively.

---

# Outputs

The exported GeoTIFF can be used directly in downstream workflows including

- SST reconstruction (CARE-SST)
- Chlorophyll-a reconstruction
- Cloud masking
- Ocean-only analysis
- Coastline masking
- Spatial interpolation
- Remote sensing preprocessing

---

# Notes

- The script uses **ESA WorldCover v200**, which provides a global 10 m land cover map.
- Water is defined exclusively as **WorldCover Class 80 (Permanent Water Bodies)**.
- Inland lakes and rivers are also classified as water.
- The exported raster is already in **EPSG:4326**, making it directly compatible with most satellite products (VIIRS, OLCI, Sentinel-3, Landsat, etc.).
- The generated land mask is binary and suitable for masking land pixels in geospatial analysis.

---

# Example Output

```
Input AOI
        │
        ▼
Buffered AOI
        │
        ▼
ESA WorldCover (10 m)
        │
        ▼
Binary Land Mask
        │
        ▼
Morphological Cleaning
        │
        ▼
land_mask_timor.tif
```
