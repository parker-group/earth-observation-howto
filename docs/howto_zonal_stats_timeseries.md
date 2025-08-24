# 📊 How-to: Zonal Statistics for a Raster Time Series

## Goal
Batch zonal statistics for a folder of rasters (e.g., monthly .tif files) and append results into one polygon attribute table + CSV.

---

## Requirements
- [QGIS](https://qgis.org/) with Python/Processing enabled  
- Polygon shapefile/GeoPackage for your AOI  
- Rasters named with a consistent pattern, e.g. `<PREFIX>_YYYY_MM.tif`  

---

## Using the Parker Group batch script

Reference: [`scripts/python/RemoteSensZonalStats.py`](https://github.com/parker-group/Kinshasa_EO/blob/main/scripts/python/RemoteSensZonalStats.py)

---

## Example configuration (edit at top of the script)

```python
# Paths
input_vector = "data/shapefiles/health_areas.gpkg"
raster_root  = "data/processed/rasters"   # contains subfolders per variable
output_csv   = "data/processed/zonal/KinshasaZonalStats_All.csv"

# Stats to compute
stats = ["mean"]  # add "min","max","sum" as needed
```

---

## How it works
1. Loops through raster subfolders (e.g., `MODIS_EVI/`, `ERA5_Precip/`).  
2. Extracts the `YYYYMM` string from filenames.  
3. Runs zonal stats for each raster × polygon.  
4. Adds new columns to the polygon attribute table (e.g., `EVI_202201`, `LLST_202307`).  
5. Saves both updated polygons and a tidy `.csv`.  

---

## Output
- Shapefile/GeoPackage with time-series columns.  
- CSV table ready for plotting/analysis in R or Python.  

---

## Tips
- Keep rasters organized into subfolders per variable.  
- Ensure AOI vector and rasters share the same CRS.  
- Test first with a small subset before running the full time series.  
