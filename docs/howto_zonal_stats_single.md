# 📍 How-to: Zonal Statistics for a Single Raster

## Goal
Extract summary values (mean, min, max, etc.) from **one raster** for a set of polygons.

---

## Requirements
- [QGIS](https://qgis.org/) installed with the **Processing toolbox** enabled  
- Polygon layer (e.g., health areas) and raster layer (e.g., MODIS LST) in compatible CRS  

---

## Minimal QGIS Python Console workflow

```python
import processing

params = {
  'INPUT_RASTER': 'path/to/MODIS_LST_2022_01.tif',
  'RASTER_BAND': 1,
  'INPUT_VECTOR': 'path/to/health_areas.gpkg|layername=areas',
  'COLUMN_PREFIX': 'LST_202201_',
  'STATS': [2, 3, 5]  # mean, min, max
}

processing.run("qgis:zonalstatistics", params)
```

---

## Tips
- Run this inside the **QGIS Python Console**.  
- `COLUMN_PREFIX` ensures the new fields are labeled clearly in your vector attribute table.  
- CRS must match between raster and vector layers — reproject if necessary.  
- Use the GUI (`Vector > Zonal statistics`) if you prefer a no-code option.  

---

## Typical Use Case
- You have one monthly raster (e.g., **MODIS LST Jan 2022**) and want the average value for each health area polygon.  
- Output: your polygon layer’s attribute table gains columns like `LST_202201_mean`, `LST_202201_min`, etc.  
