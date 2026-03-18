# Download PM2.5 NetCDF file

Download a WUSTL ACAG V6GL02 NetCDF file from AWS S3 and cache it
locally. If the file is already cached and complete (\>1 MB), the cached
version is returned without re-downloading.

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

  Integer. Year for data (e.g. `2023`).

- month:

  Optional integer (1–12). If NULL, downloads the annual file.

- cache_dir:

  Character. Directory to cache NetCDF files. Default: `"pm25_data"`.

- force_download:

  Logical. Force re-download even if file exists. Default: `FALSE`.

## Value

Character. Path to the downloaded (or cached) NetCDF file.

## Examples

``` r
if (FALSE) { # \dontrun{
# Download monthly data
nc_path <- pm25_download_netcdf(year = 2023, month = 11)

# Download annual data
nc_path <- pm25_download_netcdf(year = 2023)
} # }
```
