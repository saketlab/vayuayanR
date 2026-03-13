# Convert station data to data frame

Convert nested station data to a flat data frame with geographic
information, AQI values, and categories. This is the primary way to get
station coordinates (latitude/longitude) from live CPCB data.

## Usage

``` r
stations_to_dataframe(data)
```

## Arguments

- data:

  List of cities with nested stations (from cpcb_get_all_india())

## Value

Data frame with columns: station_id, station_name, city_name, city_id,
state_id, longitude, latitude, live, avg_aqi, status, aqi_category
