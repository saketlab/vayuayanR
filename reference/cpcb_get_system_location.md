# Get system location via IP geolocation

Get approximate system location using IP-based geolocation. Returns
city-level coordinates based on the machine's public IP address.

## Usage

``` r
cpcb_get_system_location()
```

## Value

Numeric vector c(lat, lon)

## Examples

``` r
if (FALSE) { # \dontrun{
loc <- cpcb_get_system_location()
cat("Latitude:", loc["lat"], "Longitude:", loc["lon"])
} # }
```
