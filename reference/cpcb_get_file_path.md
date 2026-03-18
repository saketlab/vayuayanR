# Get file path for CPCB data download

Get file path for downloading data from CPCB. Used internally by
[`cpcb_download_city_data()`](https://saketkc.github.io/vayuayanR/reference/cpcb_download_city_data.md)
and
[`cpcb_download_station_data()`](https://saketkc.github.io/vayuayanR/reference/cpcb_download_station_data.md).

## Usage

``` r
cpcb_get_file_path(
  station_id = "",
  station_name = "",
  state = "",
  city = "",
  year = "",
  frequency = "daily",
  data_type = "cityLevel"
)
```

## Arguments

- station_id:

  Character. Station ID (e.g. `"site_103"`).

- station_name:

  Character. Station name.

- state:

  Character. State name.

- city:

  Character. City name.

- year:

  Integer or character. Year for data (e.g. `2022`).

- frequency:

  Character. Data frequency: `"daily"` or `"hourly"`.

- data_type:

  Character. Type of data: `"cityLevel"` or `"stationLevel"`.

## Value

List of file path information with `filePath` and `year` elements

## Examples

``` r
if (FALSE) { # \dontrun{
paths <- cpcb_get_file_path(city = "Delhi", frequency = "daily",
                             data_type = "cityLevel")
str(paths[[1]])
} # }
```
