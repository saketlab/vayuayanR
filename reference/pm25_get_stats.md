# Get PM2.5 statistics for a region

Compute mean, SD, min, max, and pixel count of satellite PM2.5 for a
region defined by a GeoJSON file. Optionally group by a column in the
GeoJSON (e.g. district name) to return per-group statistics.

## Usage

``` r
pm25_get_stats(
  geojson_file,
  year,
  month = NULL,
  cache_dir = "pm25_data",
  group_by = NULL
)
```

## Arguments

- geojson_file:

  Character. Path to GeoJSON file defining the region.

- year:

  Integer. Year of the NetCDF data (e.g. `2023`).

- month:

  Optional integer (1–12). If NULL, uses annual data.

- cache_dir:

  Character. Directory to cache NetCDF files. Default: `"pm25_data"`.

- group_by:

  Optional character. Column name(s) in the GeoJSON to group polygons by
  (comma-separated for multiple columns).

## Value

List (or data frame if `group_by` is specified) with PM2.5 statistics:
`mean`, `std`, `min`, `max`, `count`.

## Examples

``` r
if (FALSE) { # \dontrun{
# Single region
stats <- pm25_get_stats("delhi.geojson", year = 2023, month = 11)
cat("Mean PM2.5:", round(stats$mean, 1), "ug/m3")

# Grouped by district
stats_by_district <- pm25_get_stats(
  "delhi_districts.geojson",
  year     = 2023,
  month    = 11,
  group_by = "district_name"
)
} # }
```
