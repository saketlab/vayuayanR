# Get Live AQI Data

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

  Optional station ID

- coords:

  Optional numeric vector c(lat, lon)

- date:

  Optional date in 'YYYY-MM-DD' format

- hour:

  Optional hour (0-23)

## Value

List with processed live AQI data
