# Download PM2.5 NetCDF file

Download NetCDF file from AWS if not cached locally.

## Usage

``` r
pm25_download_netcdf(
  year,
  month = NULL,
  cache_dir = "pm25_data",
  force_download = FALSE
)
```

## Arguments

- year:

  Year for data

- month:

  Optional month (1-12). If NULL, downloads annual data.

- cache_dir:

  Directory to cache NetCDF files (default: "pm25_data")

- force_download:

  Force re-download even if file exists

## Value

Path to the downloaded NetCDF file
