# Add Coordinates to Station List

Add lat/lon coordinates to a station list via OpenStreetMap geocoding.
Handles co-located stations (same name, different agencies) by using
differentiated coordinates, and expands known abbreviations for better
geocoding accuracy.

## Usage

``` r
cpcb_add_coords(stations)
```

## Arguments

- stations:

  Data frame from cpcb_get_station_list()

## Value

Same data frame with lat and lon columns added
