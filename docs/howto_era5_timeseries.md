# 🛰️ How-to: Extract ERA5-Land Climate Time Series

## Goal
Obtain ERA5-Land temperature and total precipitation for an area of interest (AOI), aggregated to monthly time series. 

---

## Requirements
- [Google Earth Engine](https://earthengine.google.com/) account (recommended approach)  
- Alternative: [Copernicus CDS API](https://cds.climate.copernicus.eu/api-how-to) for direct scripted pulls  

---

## Path 1 — Google Earth Engine (recommended)

**Why:** simple global access, efficient aggregation, direct export to Google Drive.  

```js
// Define your AOI (Area of Interest - geometry or FeatureCollection)
var aoi = /* your AOI polygon */;

// ERA5-Land hourly
var era = ee.ImageCollection("ECMWF/ERA5_LAND/HOURLY")
  .select(['temperature_2m','total_precipitation']);

// Loop over months in 2022–2023
var months = ee.List.sequence(2022, 2023).map(function(y){
  return ee.List.sequence(1, 12).map(function(m){
    var start = ee.Date.fromYMD(y, m, 1);
    var end   = start.advance(1, 'month');

    var mon = era.filterDate(start, end).mean()
      .set({'YYYY_MM': start.format('YYYY_MM')})
      .clip(aoi);

    return mon;
  });
}).flatten();

// Export loop omitted here — see notebooks/GEE_ERA5_Export_Kinshasa.ipynb
```

### Notes
- **Temperature**: Kelvin → convert to °C after export.  
- **Precipitation**: meters → multiply by 1000 for mm.  
- Use a consistent naming convention (e.g., `ERA5_Temp_YYYY_MM.tif`).  

---

## Path 2 — Copernicus CDS API (direct pull)

You run these scripts locally in **Python** (via `cdsapi`) or **R** (via `ecmwfr`) — for example in a terminal, Jupyter notebook, or RStudio session.  

Before use, you must [register for a free Copernicus CDS account](https://cds.climate.copernicus.eu/) and configure your API key (`.cdsapirc` for Python or `.ecmwfr.yml` for R) to allow authentication.  

Once set up, the scripts will download ERA5 files (usually NetCDF) directly to your working directory or server.


**Why:** works outside Earth Engine; reproducible server-side scripts.  
**Cons:** setup requires a CDS account + API key; queue times for large requests.

- **Python:** [`cdsapi`](https://pypi.org/project/cdsapi/)  
- **R:** [`ecmwfr`](https://github.com/bluegreen-labs/ecmwfr)  

Example (Python):

```python
import cdsapi

c = cdsapi.Client()

c.retrieve(
    'reanalysis-era5-land-monthly-means',
    {
        'variable': ['2m_temperature','total_precipitation'],
        'year': ['2022','2023'],
        'month': [f"{m:02d}" for m in range(1,13)],
        'time': '00:00',
        'format': 'netcdf'
    },
    'era5_land_monthly.nc')
```

---

## QC & Tips
- ERA5-Land resolution: ~9 km.  
- Always document whether values are aggregated over an AOI (polygon mean) or extracted at a point. We normally extract to a polygon, and if we start with point data we draw a buffer around that point and extract to the buffer polygon. How big should that buffer be? See my repo/blog on [activity spaces](https://github.com/DMParker1/activity-spaces) for some thoughts on this.
- Use consistent temporal coverage across variables before merging.  
