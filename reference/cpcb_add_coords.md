# Add coordinates to station list

Add lat/lon coordinates to a station data frame by querying the CPCB
live coordinates API (`iit_rss_feed_with_coordinates`). This is the same
source used by the Python vayuayan package and returns authoritative
coordinates directly from CPCB - no geocoding, no rate limiting, no NAs
for oddly-named stations.

Falls back to Nominatim geocoding for any stations not found in the CPCB
response (e.g. offline stations absent from the live feed).

## Usage

``` r
cpcb_add_coords(stations)
```

## Arguments

- stations:

  Data frame from
  [`cpcb_get_station_list()`](https://saketkc.github.io/vayuayanR/reference/cpcb_get_station_list.md)

## Value

Same data frame with `lat` and `lon` columns added
