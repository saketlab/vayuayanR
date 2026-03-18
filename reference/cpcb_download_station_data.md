# Download station-level AQI data

Download daily PM2.5 AQI data for a single CPCB monitoring station for a
given year. Data is saved as a CSV file with one row per day and one
column per month.

## Usage

``` r
cpcb_download_station_data(station_id, year, save_location)
```

## Arguments

- station_id:

  Character. Station ID from
  [`cpcb_get_station_list()`](https://saketkc.github.io/vayuayanR/reference/cpcb_get_station_list.md)
  (e.g. `"site_103"`).

- year:

  Integer. Year for data (e.g. `2022`).

- save_location:

  Character. Full file path to save the downloaded CSV.

## Value

Data frame preview (first 6 rows) of the downloaded data, invisibly. The
full data is saved to `save_location`.

## Examples

``` r
if (FALSE) { # \dontrun{
cpcb_download_station_data(
  station_id    = "site_103",
  year          = 2022,
  save_location = tempfile(fileext = ".csv")
)
} # }
```
