# Clean Station Label for Geocoding

Clean a CPCB station label for Nominatim geocoding by stripping agency
suffixes and expanding known abbreviations.

## Usage

``` r
.cpcb_clean_geocoding_query(label)
```

## Arguments

- label:

  Station label string

## Value

Cleaned query string suitable for geocoding
