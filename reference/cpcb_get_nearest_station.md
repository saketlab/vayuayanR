# Find nearest monitoring station

Find nearest CPCB monitoring station to given coordinates. If no
coordinates provided, uses IP-based geolocation automatically. Returns
station ID, name, and distance in kilometres.

## Usage

``` r
cpcb_get_nearest_station(coords = NULL)
```

## Arguments

- coords:

  Optional numeric vector c(lat, lon). If NULL, uses IP location.

## Value

List with station_id, station_name, and distance_km
