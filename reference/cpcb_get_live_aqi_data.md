# Get live AQI data

Get live AQI data with flexible parameters. Automatically resolves
station from coordinates or IP location if not provided.

## Usage

``` r
cpcb_get_live_aqi_data(
  station_id = NULL,
  coords = NULL,
  date = NULL,
  hour = NULL
)
```

## Arguments

- station_id:

  Optional character. Station ID. If NULL, nearest station is found
  automatically using `coords` or IP geolocation.

- coords:

  Optional numeric vector `c(lat, lon)`. Used to find nearest station if
  `station_id` is not provided.

- date:

  Optional character. Date in `"YYYY-MM-DD"` format. Defaults to today.

- hour:

  Optional integer (0–23). Hour of the day. Defaults to the most recent
  full hour.

## Value

List with processed live AQI data

## Examples

``` r
if (FALSE) { # \dontrun{
# Get live AQI at current location and time
aqi <- cpcb_get_live_aqi_data()

# Get live AQI for a specific station and time
aqi <- cpcb_get_live_aqi_data(
  station_id = "site_103",
  date       = "2024-01-15",
  hour       = 10
)
} # }
```
