# Clear PM2.5 cache

Delete all cached NetCDF files from the cache directory to free disk
space.

## Usage

``` r
pm25_clear_cache(cache_dir = "pm25_data")
```

## Arguments

- cache_dir:

  Character. Directory containing cached files. Default: `"pm25_data"`.

## Value

Invisibly returns NULL.

## Examples

``` r
if (FALSE) { # \dontrun{
pm25_clear_cache()
} # }
```
