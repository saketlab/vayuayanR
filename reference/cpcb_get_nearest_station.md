# Find Nearest Monitoring Station

Find nearest monitoring station to given coordinates or IP location.

## Usage

``` r
cpcb_get_nearest_station(coords = NULL)
```

## Arguments

- coords:

  Optional numeric vector c(lat, lon). If NULL, uses IP location.

## Value

List with station_id, station_name, and distance_km
