# Get live AQI data for a station

Get live AQI data for a specific station at a specific time.

## Usage

``` r
cpcb_get_live_aqi_data_for_station(station_id, date_time)
```

## Arguments

- station_id:

  Character. Station ID (e.g. `"site_103"`).

- date_time:

  Character. Date and time in `"YYYY-MM-DDTHH:00:00Z"` format.

## Value

List with live AQI data including pollutant readings

## Examples

``` r
if (FALSE) { # \dontrun{
aqi <- cpcb_get_live_aqi_data_for_station(
  station_id = "site_103",
  date_time  = "2024-01-15T10:00:00Z"
)
} # }
```
