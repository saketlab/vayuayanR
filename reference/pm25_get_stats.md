# Get PM2.5 statistics for a region

Compute PM2.5 statistics for a region defined by a GeoJSON file.

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

  Path to GeoJSON file

- year:

  Year of NetCDF data

- month:

  Optional month of NetCDF data

- cache_dir:

  Directory to cache NetCDF files (default: "pm25_data")

- group_by:

  Optional column name(s) to group polygons by

## Value

Data frame or list with PM2.5 statistics
