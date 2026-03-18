# List cached PM2.5 NetCDF files

List all cached NetCDF files in the cache directory with file sizes.

## Usage

``` r
pm25_list_cached_files(cache_dir = "pm25_data")
```

## Arguments

- cache_dir:

  Character. Directory containing cached files. Default: `"pm25_data"`.

## Value

Character vector of cached file paths.

## Examples

``` r
if (FALSE) { # \dontrun{
files <- pm25_list_cached_files()
} # }
```
