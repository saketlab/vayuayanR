# Get all India monitoring stations

Get all air quality monitoring stations in India from CPCB live
dashboard.

## Usage

``` r
cpcb_get_all_india()
```

## Value

List of station data

## Examples

``` r
if (FALSE) { # \dontrun{
stations <- cpcb_get_all_india()
df <- stations_to_dataframe(stations)
head(df)
} # }
```
