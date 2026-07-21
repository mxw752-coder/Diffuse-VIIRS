# Prepare Data Pipeline 

This script is used to process pipeline that aligns, crops, and pads high-resolution sea surface temperature (SST) satellite data from **VIIRS** (Visible Infrared Imaging Radiometer Suite) and **OSTIA** (Operational Sea Surface Temperature and Sea Ice Analysis) for a specific region off the coast of Northern Mozambique.

The script processes daily data for an entire year, extracts a region of interest, generates land masks, handles invalid pixels (NaNs), and pads the outputs to a uniform `300x300` resolution suitable for deep learning or machine learning applications (e.g., gap-filling, downscaling).

## 🚀 Features
*   **Automated Date Generation:** Processes every day for a specified year (e.g., `2021`).
*   **Smart File Matching:** Automatically searches and matches valid VIIRS netCDF files and OSTIA GeoTIFF files based on naming conventions.
*   **Geospatial Alignment:** Crops VIIRS data to a `256x256` bounding box centered on a specific coordinate (`-12.9300, 40.5250`). Resamples OSTIA data using nearest-neighbor interpolation to match the exact VIIRS coordinate grid.
*   **Land Mask Generation:** Automatically identifies land or invalid pixels (values `< 15.0` or `NaN`) based on the OSTIA dataset.
*   **Uniform Padding:** Pads the raw `256x256` data up to a fixed `300x300` grid using constant zero-padding (or `1.0` for landmasks) to standardize input dimensions for modeling.
*   **Debugging Visualizations:** Automatically plots the first `N` successfully processed dates (VIIRS, OSTIA, and Land Mask) for quick visual validation.
*   **Logging:** Outputs summary text files (`successful_dates.txt` and `failed_dates.txt`) for tracking missing or corrupted files.

## 🛠 Prerequisites and Dependencies

Ensure you have the following Python libraries installed. You can install them via pip or conda:

```bash
pip install numpy xarray rioxarray matplotlib tqdm
```
## Directory Structure
../../

├── L3S_STAR/raw/

│   └── {YYYY}-{MM}/PM/

│       └── (VIIRS netCDF files e.g., *STAR-L3S*.nc)

├── {AOI}/

│   ├── L4_OSTIA/geotiff/

│   │   └── {YYYY}/{MM}/

│   │       └── {YYYYMMDD}.tif

│   └── result/

│       ├── ready_whole_{YEAR}/cloud/

│       │   ├── viirs/       # Raw 256x256 outputs

│       │   ├── oisst/       

│       │   ├── land_mask/   

│       │   └── lon_lat/     

│       └── ready_whole_{YEAR}_300/cloud/

│           ├── viirs/       # Padded 300x300 outputs (.npy)

│           ├── oisst/       

│           ├── land_mask/   

│           └── lon_lat/

## Inputs

- YEAR = "2021"                                   **Editable**
- SITE = "Mozambique_N"                           **Editable**

- CENTER_LON = 40.52502367602088                  **Editable**
- CENTER_LAT = -12.930035789266949                **Editable**

- VIIRS_PRODUCT = "_PM_N-" 

- TARGET_SIZE = 300  # Final padded output size
- CROP_SIZE = 256    # Initial crop size from raw data

## What happens during execution?
The script generates a list of all days in the target year.

1. It iterates through the days, finding the matching VIIRS and OSTIA files.

2. If both files exist, it opens them, handles Kelvin-to-Celsius conversions if necessary, crops them to the target coordinates, and generates the masks.

3. The raw 256x256 arrays are saved as .npy files.

4. The arrays are then padded to 300x300 and saved in a separate directory structure.

5. A visualization will pop up for the first few successful dates to allow you to verify the alignment.

6. Upon completion, a summary report is printed to the console, and text logs are saved.

## Outputs

For every successful day, the script generates four ```.npy``` arrays at both 256x256 and 300x300 resolutions:

1. viirs: High-resolution satellite SST.

2. oisst (OSTIA): Interpolated baseline SST grid.

3. land_mask: Binary mask (1 = land/invalid, 0 = ocean).

4. lon_lat: 2D coordinate grid (shape H, W, 2).
