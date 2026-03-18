# Get station list for a city

Get station list available in given city from CPCB.

## Usage

``` r
cpcb_get_station_list(city)
```

## Arguments

- city:

  Character. City name (e.g. `"Delhi"`, `"Mumbai"`).

## Value

Data frame with columns `value` (station ID) and `label` (station name)

## Examples

``` r
if (FALSE) { # \dontrun{
stations <- cpcb_get_station_list("Delhi")
head(stations)
} # }
```
