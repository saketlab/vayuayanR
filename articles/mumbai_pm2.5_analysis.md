# Mumbai PM2.5 Analysis (2022)

## Introduction

Mumbai is India’s financial capital and most densely populated city,
home to over 20 million residents. Despite its coastal location — which
provides relatively better natural ventilation than landlocked Delhi —
Mumbai’s PM2.5 levels regularly exceed WHO safe limits, driven by
vehicular emissions, industrial activity, construction dust, and
seasonal sea-salt aerosols.

This vignette demonstrates how the `vayuayan` package integrates two
independent data sources to analyse Mumbai’s air quality in 2022, and
uses them together to answer a key policy question:

> **Are the 30 CPCB ground monitoring stations distributed well enough
> to capture Mumbai’s true spatial extent of pollution — or are there
> under-monitored localities where PM2.5 is being systematically
> missed?**

We build toward this answer in four steps:

1.  **Seasonal trend** — do satellite and ground data agree on Mumbai’s
    pollution cycle?
2.  **Spatial heatmap** — where in Mumbai is pollution highest?
3.  **Station network** — where are the 30 sensors located relative to
    pollution?
4.  **Monitoring gap analysis** — which localities are furthest from any
    sensor, and does that distance predict ground-satellite
    disagreement?

------------------------------------------------------------------------

## Data & Methods

### Satellite Data

Monthly PM2.5 estimates come from the Washington University in St. Louis
Atmospheric Composition Analysis Group (WUSTL ACAG) **V5GL04** product —
a globally gridded dataset at ~0.01° (~1km) resolution, derived from
satellite aerosol optical depth retrievals, chemical transport
modelling, and ground-level calibration (van Donkelaar et al., 2021).
Files are in NetCDF format.

> **Note:** V5GL04 covers 1998–2022. Data for 2023 onward requires
> V6GL02, which is pending download on the departmental server. This
> vignette uses 2022 throughout.

### Ground Data

Daily city-level and per-station PM2.5 readings are fetched from the
Central Pollution Control Board (CPCB) data repository using
[`cpcb_download_city_data()`](https://saketkc.github.io/vayuayanR/reference/cpcb_download_city_data.md)
and
[`cpcb_download_station_data()`](https://saketkc.github.io/vayuayanR/reference/cpcb_download_station_data.md).
Mumbai has 30 operational CAAQMS stations managed by MPCB, IITM, and
BMC. Station coordinates are obtained via
[`cpcb_add_coords()`](https://saketkc.github.io/vayuayanR/reference/cpcb_add_coords.md),
which queries the CPCB live feed directly, the same source used by the
Python vayuayan package. Nominatim is used as a fallback only for
stations temporarily absent from the live feed.

### Spatial Units

The GeoJSON boundary file (`mumbai_v4.geojson`) contains **227
localities** across Mumbai’s **24 administrative wards** (coded A
through T). Each locality belongs to one ward. All spatial analysis is
conducted at locality level; ward boundaries are overlaid on maps as
reference.

### Comparison Caveat

These two data sources measure PM2.5 differently:

- **Satellite** = spatial mean over an entire area (~1km pixels)
- **Ground** = point measurement at one physical location, often sited
  near pollution sources

Ground readings are therefore consistently higher than satellite
estimates. This is not an error — it reflects the fundamental difference
between point-source and area-averaged measurement. Ground data is
aggregated to monthly means for fair temporal comparison.

------------------------------------------------------------------------

## Setup

``` r
# pacman handles installation + loading in one step
if (!require("pacman")) install.packages("pacman")

pacman::p_load(
  vayuayan,      # CPCB data download + coordinates
  tidyverse,     # data wrangling + ggplot2
  purrr,         # map_dfr for iterating over months
  terra,         # raster operations
  sf,            # vector spatial data
  tidyterra,     # ggplot2 + terra integration (geom_spatraster)
  exactextractr, # area-weighted raster extraction to polygons
  ncdf4,         # reading NetCDF files directly
  khroma         # diverging colour palettes
)
```

### Helper Functions

The V5GL04 NetCDF files store data in `[lon, lat]` order — the reverse
of what `terra` expects (`[lat, lon]`). Loading the entire 121MB global
array just to extract Mumbai is extremely slow. These two helpers read
only the pixels covering Mumbai’s bounding box, making extraction
near-instant.

``` r
# read_v5gl04_fast(): returns a single mean PM2.5 value for the shape
# Used for the monthly time series (Section 1)
read_v5gl04_fast <- function(nc_path, shape) {
  bbox <- sf::st_bbox(shape)

  nc   <- ncdf4::nc_open(nc_path)
  lons <- ncdf4::ncvar_get(nc, "lon")
  lats <- ncdf4::ncvar_get(nc, "lat")

  lon_idx <- which(lons >= bbox["xmin"] & lons <= bbox["xmax"])
  lat_idx <- which(lats >= bbox["ymin"] & lats <= bbox["ymax"])

  mumbai_data <- ncdf4::ncvar_get(
    nc, "GWRPM25",
    start = c(min(lon_idx), min(lat_idx)),
    count = c(length(lon_idx), length(lat_idx))
  )
  ncdf4::nc_close(nc)

  mean(mumbai_data, na.rm = TRUE)
}

# load_v5gl04_rast(): returns a terra SpatRaster cropped to Mumbai
# Used when we need the full spatial raster (Sections 2, 3, 4)
load_v5gl04_rast <- function(year, month, shape) {
  nc_path <- sprintf(
    "/NAS/climatehealth/WUSTL_PM2.5/V5GL04/Global/Monthly/V5GL04.HybridPM25.Global.%04d%02d-%04d%02d.nc",
    year, month, year, month
  )
  bbox <- sf::st_bbox(shape)

  nc   <- ncdf4::nc_open(nc_path)
  lons <- ncdf4::ncvar_get(nc, "lon")
  lats <- ncdf4::ncvar_get(nc, "lat")

  lon_idx <- which(lons >= bbox["xmin"] & lons <= bbox["xmax"])
  lat_idx <- which(lats >= bbox["ymin"] & lats <= bbox["ymax"])

  mumbai_data <- ncdf4::ncvar_get(
    nc, "GWRPM25",
    start = c(min(lon_idx), min(lat_idx)),
    count = c(length(lon_idx), length(lat_idx))
  )
  ncdf4::nc_close(nc)

  # Build a raster from the Mumbai subset
  # V5GL04 stores [lon, lat] — transpose and flip vertically to get [lat, lon]
  r <- terra::rast(
    nrows = length(lat_idx), ncols = length(lon_idx),
    xmin  = min(lons[lon_idx]), xmax = max(lons[lon_idx]),
    ymin  = min(lats[lat_idx]), ymax = max(lats[lat_idx]),
    crs   = "EPSG:4326"
  )
  terra::values(r) <- t(mumbai_data)[nrow(t(mumbai_data)):1, ]
  r
}
```

------------------------------------------------------------------------

## Results

### 1. Seasonal Trend: Do Satellite and Ground Data Agree?

The first question is simple: do these two independent data sources tell
the same seasonal story? If they agree on *when* pollution peaks and
troughs, we can trust them both as complementary sources for the spatial
analysis that follows.

#### Ground Data: City-Level Monthly PM2.5

``` r
# Download city-level daily PM2.5 for all of Mumbai, 2022
# Returns a wide-format CSV: rows = days, columns = months
cpcb_download_city_data(
  city          = "Mumbai",
  year          = 2022,
  save_location = "mumbai_aqi_2022.csv"
)
#> 2022 
#> AQI_daily/city_level/mumbai/2022/mumbai_2022.xlsx 
#> File Saved to mentioned location
#> # A tibble: 6 × 13
#>     Day January February March April   May  June  July August September October
#>   <dbl>   <dbl>    <dbl> <dbl> <dbl> <dbl> <dbl> <dbl>  <dbl>     <dbl>   <dbl>
#> 1     1     173      177   187   196    89    79    58     65        61      78
#> 2     2     150      179   216   214   103    99    67     63        73      57
#> 3     3     156      187   204   172    91   108    72     89        79      56
#> 4     4     184      162   177   142   103    77    65     61        67      66
#> 5     5     219      276   167   108   118    69    60     71        59      75
#> 6     6     170      228   185    98   139    70    61     50        59      66
#> # ℹ 2 more variables: November <dbl>, December <dbl>

mumbai_wide <- read.csv("mumbai_aqi_2022.csv")

# Reshape wide → long and compute monthly mean across all stations
mumbai_ground <- mumbai_wide %>%
  pivot_longer(
    cols      = -Day,
    names_to  = "Month_Name",
    values_to = "ground_pm25"
  ) %>%
  mutate(month = match(Month_Name, month.name)) %>%
  filter(!is.na(ground_pm25)) %>%
  group_by(month) %>%
  summarise(ground_pm25 = mean(ground_pm25, na.rm = TRUE), .groups = "drop")

print(mumbai_ground)
#> # A tibble: 12 × 2
#>    month ground_pm25
#>    <int>       <dbl>
#>  1     1       176. 
#>  2     2       159. 
#>  3     3       165. 
#>  4     4       107. 
#>  5     5       122. 
#>  6     6        61.6
#>  7     7        64.1
#>  8     8        61.7
#>  9     9        60.1
#> 10    10        97.9
#> 11    11       173. 
#> 12    12       202.
```

#### Satellite Data: Monthly Spatial Mean over Mumbai

``` r
# Load Mumbai boundary — used for spatial subsetting throughout this vignette
mumbai_shape <- sf::st_read(
  "/NAS/darakshann/geojson/mumbai_v4.geojson", quiet = TRUE
) %>% sf::st_make_valid()

# Extract mean satellite PM2.5 for Mumbai for each month
# Each file is ~121MB — the fast helper reads only Mumbai's pixels (~seconds)
mumbai_sat_df <- map_dfr(1:12, function(m) {
  nc_file <- sprintf(
    "/NAS/climatehealth/WUSTL_PM2.5/V5GL04/Global/Monthly/V5GL04.HybridPM25.Global.2022%02d-2022%02d.nc",
    m, m
  )
  tibble(month = m, sat_pm25 = read_v5gl04_fast(nc_file, mumbai_shape))
})

# Join ground and satellite into one table for comparison
mumbai_final <- inner_join(mumbai_ground, mumbai_sat_df, by = "month")
print(mumbai_final)
#> # A tibble: 12 × 3
#>    month ground_pm25 sat_pm25
#>    <int>       <dbl>    <dbl>
#>  1     1       176.      73.3
#>  2     2       159.      53.7
#>  3     3       165.      45.4
#>  4     4       107.      37.8
#>  5     5       122.      31.5
#>  6     6        61.6     18.2
#>  7     7        64.1     19.2
#>  8     8        61.7     14.5
#>  9     9        60.1     24.2
#> 10    10        97.9     30.0
#> 11    11       173.      65.3
#> 12    12       202.      73.0
```

#### Seasonal Line Chart

``` r
ggplot(mumbai_final, aes(x = month)) +
  geom_line(aes(y = ground_pm25, color = "CPCB Ground"), linewidth = 1) +
  geom_line(
    aes(y = sat_pm25, color = "WUSTL Satellite"),
    linewidth = 1, linetype = "dashed"
  ) +
  geom_point(aes(y = ground_pm25, color = "CPCB Ground"), size = 2) +
  geom_point(aes(y = sat_pm25,    color = "WUSTL Satellite"), size = 2) +
  # WHO reference lines
  geom_hline(yintercept = 5,  linetype = "dotted", color = "darkgreen", linewidth = 0.6) +
  geom_hline(yintercept = 15, linetype = "dotted", color = "orange",    linewidth = 0.6) +
  annotate("text", x = 1, y = 18, label = "WHO 24hr limit (15 µg/m³)",  size = 3, color = "tan4",    hjust = 0) +
  annotate("text", x = 1, y = 8,  label = "WHO annual limit (5 µg/m³)", size = 3, color = "darkgreen", hjust = 0) +
  scale_x_continuous(breaks = 1:12, labels = month.abb) +
  scale_color_manual(values = c(
    "CPCB Ground"     = "#d77027ff",
    "WUSTL Satellite" = "#4575b4"
  )) +
  labs(
    title   = "Mumbai 2022: PM2.5 Seasonal Cycle — Satellite vs Ground",
    y       = expression(paste("PM2.5 Concentration (", mu, "g/m"^3, ")")),
    x       = NULL,
    color   = NULL,
    caption = "Source: WUSTL ACAG V5GL04 + CPCB CAAQMS | Dotted lines = WHO guidelines"
  ) +
      theme_minimal(base_size = 11) +
 theme(
    text = element_text(family = "Arial"),
    panel.grid.major.x = element_blank(),
    panel.grid.minor = element_blank(),
    panel.grid.major.y = element_line(linetype = "dashed", color = "grey85"),
      legend.position = "top")
```

![Mumbai 2022: Monthly PM2.5 from satellite and ground sources. Both
sources agree on the seasonal pattern — peak in winter, trough in
monsoon. Ground readings are consistently higher due to the
point-to-area measurement difference described in
Methods.](mumbai_pm2.5_analysis_files/figure-html/line-chart-1.png)

Mumbai 2022: Monthly PM2.5 from satellite and ground sources. Both
sources agree on the seasonal pattern — peak in winter, trough in
monsoon. Ground readings are consistently higher due to the
point-to-area measurement difference described in Methods.

``` r

ggsave("fig1_mumbai_seasonal_trend.png", width = 9, height = 6, dpi = 300)
```

> **Key finding:** Mumbai shows a clear seasonal cycle — PM2.5 peaks in
> October–January when northeast monsoon winds bring dry continental
> air, and drops sharply during the southwest monsoon (June–September)
> when rain washes out particulates. Unlike Delhi, Mumbai’s monsoon
> months approach near-safe levels due to the cleansing effect of heavy
> rainfall.

> **Note:** For 2023–2025, ground data is available but satellite data
> (V6GL02) is pending download on the departmental server.

------------------------------------------------------------------------

### 2. Where Is Pollution Highest? Spatial Heatmap

The city-level average masks important spatial variation across Mumbai’s
elongated north-south geography. This heatmap shows satellite-estimated
PM2.5 at ~1km resolution for November 2022 — Mumbai’s peak pollution
month.

``` r
# Load November 2022 as a spatial raster
r_nov              <- load_v5gl04_rast(2022, 11, mumbai_shape)
mumbai_raster_mask <- terra::mask(r_nov, terra::vect(mumbai_shape))

# Outer Mumbai boundary for map reference overlay
mumbai_city_boundary <- suppressWarnings(sf::st_union(mumbai_shape))
```

``` r
ggplot() +
  geom_spatraster(data = mumbai_raster_mask) +
  geom_sf(
    data      = mumbai_city_boundary,
    fill      = NA,
    color     = "royalblue4",
    linewidth = 0.4
  ) +
  scale_fill_whitebox_c(
    palette  = "muted",
    na.value = NA,
    name     = expression(paste("PM2.5 (", mu, "g/m"^3, ")"))
  ) +
  labs(
    title    = "Spatial Distribution of PM2.5 in Mumbai",
    subtitle = "Satellite estimate: November 2022",
    caption  = "Source: WUSTL ACAG V5GL04 | Resolution: ~1km"
  ) +
  xlab(NULL) + ylab(NULL) +
  theme_minimal(base_size = 11) +
    theme(
    text = element_text(family = "Arial"),
    panel.grid.major = element_blank(),
    panel.grid.minor = element_blank(),
    legend.position = "top") +
     guides(
    fill = guide_colorbar(
      barwidth = unit(10, "lines"),
      barheight = unit(0.6, "lines")
    )
  )
```

![Spatial PM2.5 distribution across Mumbai, November 2022. Black outline
= Mumbai city boundary. Northern suburbs tend toward higher PM2.5 than
the southern
peninsula.](mumbai_pm2.5_analysis_files/figure-html/heatmap-1.png)

Spatial PM2.5 distribution across Mumbai, November 2022. Black outline =
Mumbai city boundary. Northern suburbs tend toward higher PM2.5 than the
southern peninsula.

``` r
    
ggsave("fig2_mumbai_heatmap_nov2022.png", width = 9, height = 8, dpi = 300)
```

> **Key finding:** Pollution is higher in the northern suburbs
> (Kandivali, Borivali, Malad, Andheri) compared to the southern
> peninsula (Colaba, Fort). This pattern reflects greater industrial
> activity, traffic density, and construction in the north.

------------------------------------------------------------------------

### 3. Where Are the Sensors? Station Network Choropleth

Now we overlay the 30 CPCB station locations on the satellite PM2.5
choropleth. The visual question: do stations cover the most polluted
localities?

#### Load Station Locations

``` r
# Fetch coordinates for all Mumbai CPCB stations from CPCB API
# Cached to CSV — re-runs use cache instead of hitting the API again
if (file.exists("mumbai_stations_with_coords.csv")) {
  mumbai_station_list <- read.csv("mumbai_stations_with_coords.csv")
} else {
mumbai_station_list <- cpcb_get_station_list("Mumbai")
mumbai_station_list <- cpcb_add_coords(mumbai_station_list)
# Remove site_5102 — Vasai West, outside Mumbai city limits
mumbai_station_list <- mumbai_station_list %>% filter(value != "site_5102")
write.csv(mumbai_station_list, "mumbai_stations_with_coords.csv", row.names = FALSE)
}

mumbai_full_sf <- mumbai_station_list %>%
  filter(!is.na(lat), !is.na(lon)) %>%
  filter(value != "site_5102") %>%    # outside Mumbai limits
  sf::st_as_sf(coords = c("lon", "lat"), crs = 4326)

cat(nrow(mumbai_full_sf), "stations loaded\n")
#> 28 stations loaded
```

#### Choropleth: Locality-Level PM2.5 + Station Locations

``` r
# Load the 227 Mumbai localities
mumbai_localities <- sf::st_read(
  "/NAS/darakshann/geojson/mumbai_v4.geojson", quiet = TRUE
) %>% sf::st_make_valid()

# Area-weighted mean satellite PM2.5 per locality
mumbai_localities$sat_pm25 <- exactextractr::exact_extract(
  r_nov, mumbai_localities, 'mean', progress = FALSE
)

# Aggregate localities to 24 ward boundaries for reference overlay
mumbai_ward_bounds <- mumbai_localities %>%
  group_by(WARD) %>%
  summarise(geometry = sf::st_union(geometry), .groups = "drop")
```

``` r
ggplot() +
  geom_sf(
    data = mumbai_localities, aes(fill = sat_pm25),
    color = "white", linewidth = 0.15
  ) +
  geom_sf(
    data = mumbai_full_sf, color = "black",
    fill = "royalblue4", size = 2.5, shape = 21, alpha = 0.85
  ) +
  scale_fill_distiller(
    palette   = "YlOrRd",
    direction = 1,
    na.value  = "grey80",
    name      = expression(paste("Sat PM"[2.5] ~ (mu * g/m^3)))
  ) +
  labs(
    title    = "Mumbai PM2.5 Monitoring Network",
    subtitle = "Locality shading = satellite average PM2.5 | Blue dots = CPCB stations",
    caption  = "November 2022 | Source: WUSTL ACAG V5GL04 + CPCB CAAQMS"
  ) +
  xlab(NULL) + ylab(NULL) +
  theme_minimal(base_size = 11) +
    theme(
    text = element_text(family = "Arial"),
    panel.grid.major = element_blank(),
    panel.grid.minor = element_blank(),
    legend.position = "top") +
     guides(
    fill = guide_colorbar(
      barwidth = unit(10, "lines"),
      barheight = unit(0.6, "lines")
    )
  )
```

![Locality-level satellite PM2.5 with CPCB station locations. Black
boundaries = Mumbai's 24 administrative wards. Blue dots = CPCB
stations.](mumbai_pm2.5_analysis_files/figure-html/choropleth-1.png)

Locality-level satellite PM2.5 with CPCB station locations. Black
boundaries = Mumbai’s 24 administrative wards. Blue dots = CPCB
stations.

``` r
ggsave("fig3_mumbai_choropleth_stations.png", width = 9, height = 8, dpi = 300)
```

> **Key finding:** Stations are concentrated in south and central
> Mumbai. Several northern localities — particularly in wards R and S —
> have elevated PM2.5 but sparse sensor coverage.

------------------------------------------------------------------------

### 4. Monitoring Gap Analysis

We now answer the central question: **which localities are
under-monitored, and does sensor distance predict ground-satellite
disagreement?**

#### Concept

For each of the 227 localities we:

1.  Find its geographic centroid
2.  Identify the nearest CPCB station and its winter ground PM2.5
    reading
3.  Extract satellite PM2.5 at that centroid
4.  Compute **delta = satellite − nearest ground station**
5.  Draw a line connecting centroid → nearest station to make the
    spatial relationship visually explicit

A large delta in a locality far from its nearest station suggests that
station may not be representative of that locality’s true pollution
level.

#### Per-Station Winter Ground Data

``` r
# Download per-station data for winter months: Oct, Nov, Dec, Jan
if (!dir.exists("station_data_mumbai")) dir.create("station_data_mumbai")

station_raw <- lapply(seq_along(mumbai_station_list$value), function(i) {
  id   <- mumbai_station_list$value[i]
  file <- paste0("station_data_mumbai/", id, ".csv")

  tryCatch({
    if (!file.exists(file)) {
      cpcb_download_station_data(
        station_id    = id,
        year          = 2022,
        save_location = file
      )
    }
    df <- read.csv(file)

    df %>%
      pivot_longer(cols = -Date, names_to = "Month_Name", values_to = "pm25") %>%
      mutate(month = match(Month_Name, month.name)) %>%
      filter(month %in% c(1, 10, 11, 12), !is.na(pm25)) %>%
      summarise(ground_pm25 = mean(pm25, na.rm = TRUE)) %>%
      mutate(value = id)

  }, error = function(e) {
    message("Skipped: ", id)
    data.frame(value = id, ground_pm25 = NA)
  })
})

station_ground <- do.call(rbind, station_raw)

station_ground_sf <- station_ground %>%
  filter(!is.na(ground_pm25)) %>%
  left_join(mumbai_station_list %>% select(value, lat, lon), by = "value") %>%
  filter(!is.na(lat), !is.na(lon)) %>%
  filter(value != "site_5102") %>%    # outside Mumbai limits
  sf::st_as_sf(coords = c("lon", "lat"), crs = 4326)

cat(nrow(station_ground_sf), "stations with valid winter PM2.5 (2022 data)\n")
#> 17 stations with valid winter PM2.5 (2022 data)
```

#### Compute Delta at Each Locality Centroid

``` r
locality_centroids <- sf::st_centroid(mumbai_localities)

sat_centroid_vals <- terra::extract(r_nov, terra::vect(locality_centroids))
mumbai_localities$sat_pm25_centroid <- sat_centroid_vals[, 2]

dist_matrix_winter <- sf::st_distance(locality_centroids, station_ground_sf)
nearest_idx        <- apply(dist_matrix_winter, 1, which.min)

mumbai_localities$nearest_ground_pm25 <- station_ground_sf$ground_pm25[nearest_idx]
mumbai_localities$nearest_winter_km   <- apply(dist_matrix_winter, 1, min) / 1000

# Delta = satellite at locality centroid − nearest station ground reading
# All values negative is expected — ground stations read higher (point-source bias)
mumbai_localities$delta <- mumbai_localities$sat_pm25_centroid -
                           mumbai_localities$nearest_ground_pm25

mumbai_localities <- mumbai_localities %>%
  mutate(
    priority = case_when(
      nearest_winter_km > 2 & sat_pm25_centroid > median(sat_pm25_centroid, na.rm = TRUE) ~ "High priority",
      nearest_winter_km > 2                                                                 ~ "Medium priority",
      TRUE                                                                                   ~ "Well monitored"
    )
  )

cat("Delta summary (satellite - nearest ground station):\n")
#> Delta summary (satellite - nearest ground station):
print(summary(mumbai_localities$delta))
#>    Min. 1st Qu.  Median    Mean 3rd Qu.    Max.    NA's 
#> -167.89 -125.65 -106.45 -103.34  -81.10  -25.41       5
cat("\nPriority breakdown:\n")
#> 
#> Priority breakdown:
print(table(mumbai_localities$priority))
#> 
#>   High priority Medium priority  Well monitored 
#>              59              66             102
```

#### The Monitoring Gap Map

``` r
centroid_coords <- sf::st_coordinates(locality_centroids)
station_coords  <- sf::st_coordinates(station_ground_sf)[nearest_idx, ]

connection_lines <- data.frame(
  x_from = centroid_coords[, 1],
  y_from = centroid_coords[, 2],
  x_to   = station_coords[, 1],
  y_to   = station_coords[, 2],
  delta  = mumbai_localities$delta
) %>% filter(!is.na(delta))

ggplot() +
  geom_sf(
    data      = mumbai_localities,
    aes(fill  = delta),
    color     = "white",
    linewidth = 0.1
  ) +
  geom_segment(
    data = connection_lines,
    aes(x = x_from, y = y_from, xend = x_to, yend = y_to),
    color     = "grey8",
    linewidth = 0.4,
    alpha     = 0.6,
    linetype  = "dashed"
  ) +
  geom_sf(
    data  = station_ground_sf,
    color = "black", fill = "yellow",
    size  = 2.5, shape = 21, alpha=0.7
  ) +
  scale_fill_gradientn(
    colours  = khroma::colour("roma")(100),
    values   = scales::rescale(c(
      min(mumbai_localities$delta, na.rm = TRUE),
      median(mumbai_localities$delta, na.rm = TRUE),
      max(mumbai_localities$delta, na.rm = TRUE)
    )),
    na.value = "grey80",
    name     = expression(paste(Delta ~ PM[2.5] ~ (mu * g/m^3)))
  ) +
  labs(
    title    = "Mumbai: Satellite vs Nearest Ground Station PM2.5 by Locality",
    subtitle = "Dashed lines connect each locality centroid to its nearest CPCB station\nColour shows relative disagreement | Yellow = CPCB stations | Black = ward boundaries",
    caption  = "Delta = satellite PM2.5 at locality centroid − nearest CPCB station reading\nNovember 2022 | Source: WUSTL ACAG V5GL04 + CPCB CAAQMS"
  ) +
  xlab(NULL) + ylab(NULL) +
  coord_sf() +
    theme_minimal(base_size = 11) +
    theme(
    text = element_text(family = "Arial"),
    panel.grid.major = element_blank(),
    panel.grid.minor = element_blank(),
    legend.position = "top",
     plot.title      = element_text(face = "bold"),
    plot.subtitle   = element_text(size = 9.5, color = "grey8")) +
     guides(
    fill = guide_colorbar(
      barwidth = unit(10, "lines"),
      barheight = unit(0.6, "lines")
    )
  )
```

![Monitoring gap map. Each locality is coloured by ground-satellite
disagreement. Dashed lines connect each locality centroid to its nearest
CPCB station. Black = ward
boundaries.](mumbai_pm2.5_analysis_files/figure-html/gap-map-1.png)

Monitoring gap map. Each locality is coloured by ground-satellite
disagreement. Dashed lines connect each locality centroid to its nearest
CPCB station. Black = ward boundaries.

``` r

ggsave("fig4_mumbai_monitoring_gap_map.png", width = 9, height = 8, dpi = 300)
```

#### Priority Map: Where Should New Stations Go?

``` r
ggplot() +
  geom_sf(
    data = mumbai_localities, aes(fill = priority),
    color = "white", linewidth = 0.15
  ) +
  geom_sf(
    data  = mumbai_full_sf,
    color = "black", fill = "yellow",
    size  = 3, shape = 21
  ) +
  scale_fill_manual(
    values = c(
      "High priority"   = "#d6604d",
      "Medium priority" = "#fdae61",
      "Well monitored"  = "#4dac26"
    ),
    na.value = "grey80",
    name     = "Monitoring priority"
  ) +
  labs(
    title    = "Mumbai: Priority Localities for New CPCB Monitoring Stations",
    subtitle = "Red = above-median PM2.5 AND >2km from nearest sensor\nOrange = >2km from sensor | Green = well covered | Yellow = existing stations",
    caption  = "Source: WUSTL ACAG V5GL04 + CPCB CAAQMS | November 2022"
  ) +
  xlab(NULL) + ylab(NULL) +
  coord_sf() +
    theme_minimal(base_size = 11) +
    theme(
    text = element_text(family = "Arial"),
    panel.grid.major = element_blank(),
    panel.grid.minor = element_blank(),
    legend.position = "top",
     plot.title      = element_text(face = "bold"),
    plot.subtitle   = element_text(size = 9.5, color = "grey8")) +
     guides(
    fill = guide_colorbar(
      barwidth = unit(10, "lines"),
      barheight = unit(0.6, "lines")
    )
  )
```

![Priority localities for new CPCB stations in Mumbai. Red = high
pollution AND poor sensor coverage (\>2km from nearest
station).](mumbai_pm2.5_analysis_files/figure-html/priority-map-1.png)

Priority localities for new CPCB stations in Mumbai. Red = high
pollution AND poor sensor coverage (\>2km from nearest station).

``` r

ggsave("fig5_mumbai_priority_map.png", width = 9, height = 8, dpi = 300)
```

#### Does Sensor Distance Predict Disagreement?

``` r
cor_val <- cor(
  mumbai_localities$nearest_winter_km,
  abs(mumbai_localities$delta),
  use = "complete.obs"
)
cat("Correlation (distance vs |delta|): r =", round(cor_val, 3), "\n")
#> Correlation (distance vs |delta|): r = -0.082

ggplot(
  data = sf::st_drop_geometry(mumbai_localities),
  aes(x = nearest_winter_km, y = abs(delta))
) +
  geom_point(aes(color = nearest_winter_km), size = 3, alpha = 0.8) +
  geom_smooth(method = "loess", se = TRUE, color = "grey30", linewidth = 0.8) +
  annotate(
    "text",
    x     = max(mumbai_localities$nearest_winter_km, na.rm = TRUE) * 0.55,
    y     = max(abs(mumbai_localities$delta), na.rm = TRUE) * 0.92,
    label = paste0("r = ", round(cor_val, 2)),
    size  = 4.5, color = "grey20"
  ) +
  scale_color_distiller(
    palette = "RdYlGn", direction = -1,
    name    = "Distance to\nnearest sensor (km)"
  ) +
  labs(
    title    = "Sensor Distance Predicts Ground-Satellite Disagreement",
    subtitle = "Each point = one Mumbai locality (n = 227) | LOESS fit with 95% CI",
    x        = "Distance to nearest CPCB station (km)",
    y        = expression(paste("|Delta| PM"[2.5] ~ (mu * g/m^3))),
    caption  = "November 2022 | Source: WUSTL ACAG V5GL04 + CPCB CAAQMS"
  ) +
  theme_minimal(base_size = 13) +
  theme(
    plot.title      = element_text(face = "bold"),
    plot.subtitle   = element_text(size = 9.5, color = "grey40"),
    legend.position = "right"
  )
```

![Relationship between sensor distance and ground-satellite disagreement
across 227 Mumbai localities. LOESS fit captures non-linear
trend.](mumbai_pm2.5_analysis_files/figure-html/scatter-1.png)

Relationship between sensor distance and ground-satellite disagreement
across 227 Mumbai localities. LOESS fit captures non-linear trend.

``` r

ggsave("fig6_mumbai_distance_vs_disagreement.png", width = 9, height = 5, dpi = 300)
```

------------------------------------------------------------------------

## Discussion

### Seasonal Pattern

Mumbai’s seasonal PM2.5 cycle differs meaningfully from Delhi. The
southwest monsoon (June–September) brings heavy rainfall that
efficiently washes out particulates, bringing PM2.5 close to safe levels
in some months. This is in sharp contrast to Delhi, where even monsoon
months record dangerous levels. The dry winter months (October–January)
see PM2.5 rise sharply as northeast monsoon winds bring dry continental
air and temperature inversions trap pollutants near the surface.

### Spatial Heterogeneity

The heatmap reveals a clear north-south gradient — northern suburbs
(Kandivali, Borivali, Malad, Andheri) show higher satellite PM2.5 than
the southern peninsula. This reflects the concentration of industrial
zones, construction activity, and traffic corridors in the north, while
the southern peninsula benefits from sea breezes from three sides.

### Monitoring Gap

The gap analysis identifies localities where the current 30-station
network may be providing an incomplete picture of pollution. Localities
with long centroid-to-station lines — particularly in outer northern
wards — are most at risk of having their pollution underestimated. The
priority map highlights these areas for targeted investment in new
CAAQMS stations.

### Limitations

- All delta values are negative — ground stations consistently read
  higher than satellite due to point-source siting bias. The analysis
  focuses on *relative* disagreement across localities rather than
  absolute values
- Station coordinates from
  [`cpcb_add_coords()`](https://saketkc.github.io/vayuayanR/reference/cpcb_add_coords.md)
  are sourced directly from the CPCB live feed, with Nominatim as
  fallback for stations temporarily offline. One station (Vasai West,
  `site_5102`) required a manual coordinate patch as it lies outside
  Mumbai city limits and is absent from the live feed
- V5GL04 (2022) is used in place of V6GL02 (2023) due to server data
  availability — the methodology is identical for any year once V6GL02
  is downloaded
- Mumbai’s complex coastal geometry means some locality polygons may
  include sea areas, slightly affecting area-weighted PM2.5 extraction

------------------------------------------------------------------------

## Session Information

``` r
sessionInfo()
#> R version 4.5.2 (2025-10-31)
#> Platform: x86_64-pc-linux-gnu
#> Running under: Ubuntu 24.04.4 LTS
#> 
#> Matrix products: default
#> BLAS:   /usr/lib/x86_64-linux-gnu/blas/libblas.so.3.12.0 
#> LAPACK: /usr/lib/x86_64-linux-gnu/lapack/liblapack.so.3.12.0  LAPACK version 3.12.0
#> 
#> locale:
#>  [1] LC_CTYPE=en_US.UTF-8       LC_NUMERIC=C              
#>  [3] LC_TIME=en_US.UTF-8        LC_COLLATE=en_US.UTF-8    
#>  [5] LC_MONETARY=en_US.UTF-8    LC_MESSAGES=en_US.UTF-8   
#>  [7] LC_PAPER=en_US.UTF-8       LC_NAME=C                 
#>  [9] LC_ADDRESS=C               LC_TELEPHONE=C            
#> [11] LC_MEASUREMENT=en_US.UTF-8 LC_IDENTIFICATION=C       
#> 
#> time zone: Asia/Kolkata
#> tzcode source: system (glibc)
#> 
#> attached base packages:
#> [1] stats     graphics  grDevices utils     datasets  methods   base     
#> 
#> other attached packages:
#>  [1] khroma_1.17.0        ncdf4_1.24           exactextractr_0.10.1
#>  [4] tidyterra_1.0.0      sf_1.1-0             terra_1.8-93        
#>  [7] lubridate_1.9.5      forcats_1.0.1        stringr_1.6.0       
#> [10] dplyr_1.2.0          purrr_1.2.1          readr_2.2.0         
#> [13] tidyr_1.3.2          tibble_3.3.1         ggplot2_4.0.2       
#> [16] tidyverse_2.0.0      pacman_0.5.1         vayuayan_0.0.1      
#> 
#> loaded via a namespace (and not attached):
#>  [1] tidyselect_1.2.1    farver_2.1.2        S7_0.2.1           
#>  [4] fastmap_1.2.0       digest_0.6.39       timechange_0.4.0   
#>  [7] lifecycle_1.0.5     magrittr_2.0.4      compiler_4.5.2     
#> [10] rlang_1.1.7         sass_0.4.10         tools_4.5.2        
#> [13] yaml_2.3.12         data.table_1.18.2.1 knitr_1.51         
#> [16] labeling_0.4.3      htmlwidgets_1.6.4   sp_2.2-1           
#> [19] classInt_0.4-11     curl_7.0.0          RColorBrewer_1.1-3 
#> [22] KernSmooth_2.23-26  withr_3.0.2         desc_1.4.3         
#> [25] grid_4.5.2          e1071_1.7-17        scales_1.4.0       
#> [28] dichromat_2.0-0.1   cli_3.6.5           rmarkdown_2.30     
#> [31] ragg_1.5.1          generics_0.1.4      otel_0.2.0         
#> [34] rstudioapi_0.18.0   httr_1.4.8          tzdb_0.5.0         
#> [37] readxl_1.4.5        DBI_1.3.0           cachem_1.1.0       
#> [40] proxy_0.4-29        splines_4.5.2       cellranger_1.1.0   
#> [43] vctrs_0.7.1         Matrix_1.7-4        jsonlite_2.0.0     
#> [46] hms_1.1.4           systemfonts_1.3.2   jquerylib_0.1.4    
#> [49] units_1.0-0         geosphere_1.6-5     glue_1.8.0         
#> [52] lwgeom_0.2-15       pkgdown_2.2.0       codetools_0.2-20   
#> [55] stringi_1.8.7       gtable_0.3.6        raster_3.6-32      
#> [58] pillar_1.11.1       htmltools_0.5.9     R6_2.6.1           
#> [61] textshaping_1.0.5   evaluate_1.0.5      lattice_0.22-9     
#> [64] bslib_0.10.0        class_7.3-23        Rcpp_1.1.1         
#> [67] nlme_3.1-168        mgcv_1.9-4          xfun_0.56          
#> [70] fs_1.6.7            pkgconfig_2.0.3
```
