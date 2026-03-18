# Download city-level AQI data

Download daily PM2.5 AQI data for a city from the CPCB data repository
for a given year. Data is saved as a CSV file with one row per day and
one column per month.

## Usage

``` r
cpcb_download_city_data(city, year, save_location)
```

## Arguments

- city:

  Character. City name (e.g. `"Delhi"`, `"Mumbai"`).

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
cpcb_download_city_data(
  city          = "Delhi",
  year          = 2022,
  save_location = tempfile(fileext = ".csv")
)
} # }
```
