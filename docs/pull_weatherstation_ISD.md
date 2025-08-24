# 🌡️ How-to: Download & Summarize NOAA ISD Weather Station Data

## Goal
Pull hourly ISD observations for one or more stations and summarize to daily/monthly CSVs.

---

## Requirements
- R (≥ 4.1 recommended)  
- Packages: `rnoaa`, `dplyr`, `lubridate`, `readr`

---

## Steps
1. **Find stations** near your AOI (USAF/WBAN IDs).  
2. **Download hourly** data (per station × year).  
3. **Summarize** to daily, then monthly.  
4. **Export** tidy CSVs.

---

## Minimal R workflow

```r
# install.packages(c("rnoaa","dplyr","lubridate","readr"))
library(rnoaa); library(dplyr); library(lubridate); library(readr)

# 1) Search stations near a point (Kinshasa example)
stns <- isd_stations_search(lat = -4.325, lon = 15.322, radius = 150)
print(stns[, c("name","usaf","wban")])

# 2) Pull hourly for a selected station (N’djili: 642100/99999) and year
x <- isd(usaf = "642100", wban = "99999", year = 2023)

# 3) Summaries
daily <- x %>%
  mutate(date = as_date(time)) %>%
  summarize(
    mean_t     = mean(temp, na.rm = TRUE),
    prcp_total = sum(prcp, na.rm = TRUE),
    .by = date
  )

monthly <- daily %>%
  summarize(
    month   = floor_date(date, "month"),
    t_mean  = mean(mean_t, na.rm = TRUE),
    prcp_mm = sum(prcp_total, na.rm = TRUE),
    .by = month
  )

# 4) Save
dir.create("data/processed/weather", recursive = TRUE, showWarnings = FALSE)
write_csv(monthly, "data/processed/weather/monthly_weather_ndjili.csv")
```

---

## Notes & QC
- **Units/flags vary** across years/stations → inspect carefully.  
- Expect **gaps** (maintenance outages, clogged gauges, etc.).  
- Extend summaries with medians, maxima, or QC filters as needed.
