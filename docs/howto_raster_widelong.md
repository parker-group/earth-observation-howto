# 🧰 How-to: Convert Raster Time Series Output from Wide → Long

## Goal
Start with a **wide** table (one row per subject/site; many time-stamped raster columns) and convert it to a **long** “tidy” table with one row per (subject/site × date × variable). This makes it easy to merge with GPS/survey data and run time-series analyses.

---

## When to use this
- You’ve already run **zonal statistics** (e.g., QGIS, Python, R) to extract raster-based values (NDVI, LST, ERA5 temp/precip, etc.) to points or polygons.  
- Your result is a wide table with columns named by **variable + year + day-of-year (DOY)** (e.g., `NDV_23_123`, `LST_23_137`, `TMP_22_015`).  
- You need a long table with explicit `date`, `var_type`, and `value`.

---

## Input format (expected)
- Non-raster columns first (IDs, coordinates, metadata).  
- Then all raster time-series columns, **named in a fixed pattern** that the helper parses by position:  
  - `var_type` = first 3 characters (e.g., `NDV`, `LST`, `TMP`)  
  - Year = characters 5–6 (two-digit year; assumes `20YY`)  
  - DOY  = characters 8–10 (three-digit day-of-year)  

Example: `NDV_23_123`, `LST_23_197`, `TMP_22_015`  

*(If your naming differs — e.g., `NDVI_2023_123` — see caveats below.)*

---

## Minimal R usage

```r
library(stringr)
library(lubridate)

# process_raster_data()
# Original concept/code: Catalina Medina (https://github.com/CatalinaMedina/aedes-serology)
# Adapted for general raster time series use (e.g., MODIS, ERA5, Landsat)

process_raster_data <- function(
  df_og,             # data frame with non-raster cols followed by raster cols
  var_names,         # vector of variable names (for clarity; currently unused internally)
  raster_col_start   # index of the first raster column
){
  df_og <- data.frame(df_og)
  raster_cols <- raster_col_start:ncol(df_og)
  n_raster_cols <- length(raster_cols)
  non_raster_cols <- 1:(raster_col_start - 1)

  new_df <- data.frame(matrix(NA,
    nrow = nrow(df_og) * n_raster_cols,
    ncol = length(non_raster_cols) + 3
  ))
  colnames(new_df) <- c(colnames(df_og)[non_raster_cols], "date", "var_type", "value")

  col_names <- colnames(df_og[, raster_cols])
  var_type <- str_sub(col_names, 1, 3)
  new_df$var_type <- rep(var_type, nrow(df_og))

  year_seq <- paste0("20", str_sub(col_names, 5, 6))
  doy_seq  <- str_sub(col_names, 8, 10)
  date_seq <- as.Date(paste0(year_seq, "-01-01")) + days(as.numeric(doy_seq) - 1)
  new_df$date <- rep(date_seq, nrow(df_og))

  new_row <- 1
  for(i in 1:nrow(df_og)){
    rows_i <- new_row:(new_row + n_raster_cols - 1)
    new_df[rows_i, non_raster_cols] <- as.data.frame(lapply(df_og[i, non_raster_cols], rep, n_raster_cols))
    new_df[rows_i, "value"] <- as.numeric(df_og[i, raster_cols])
    new_row <- new_row + n_raster_cols
  }

  new_df
}

# Example:
# df_wide = your zonal-stats output
long_df <- process_raster_data(df_wide, var_names = c("NDV","LST","TMP"), raster_col_start = 5)
```

---

## Wide → Long: tiny example

**Wide (input)** — one row per site, many raster columns:
```
id   lat   lon   NDV_23_123   NDV_23_137   LST_23_123
A    -4.3  15.3  0.52         0.48         302.1
```

**Long (output)** — multiple rows per site:
```
id   lat   lon   date        var_type   value
A    -4.3  15.3  2023-05-03  NDV        0.52
A    -4.3  15.3  2023-05-17  NDV        0.48
A    -4.3  15.3  2023-05-03  LST        302.1
```

---

## Notes & caveats
- **Column name pattern matters.** Parsing assumes fixed positions; adjust if your column names differ.  
- `var_names` is not yet used internally — but helps document expectations.  
- **Two-digit years** (`20YY`) are assumed; adjust parsing for 4-digit years.  
- Works with any raster-derived time series (MODIS, Landsat, ERA5, etc.), not just MODIS.  

---

## Where this fits in the pipeline
1. **Obtain rasters** (e.g., MODIS, ERA5, Landsat).  
2. **Run zonal statistics (time series)** → wide-format table.  
   👉 See [How-to: Zonal statistics (time series)](howto_zonal_stats_timeseries.md)  
3. **Convert wide → long** with this guide (adds `date`, `var_type`, `value`).  
4. Merge with GPS/survey/health data and proceed with time-series analysis.

---

## Attribution

The original version of this helper function was authored by [Catalina Medina](https://github.com/CatalinaMedina) as part of the [aedes-serology](https://github.com/CatalinaMedina/aedes-serology) project.  
This adaptation generalizes the approach for any raster-derived time series (e.g., MODIS, ERA5, Landsat) and is maintained here with attribution.
