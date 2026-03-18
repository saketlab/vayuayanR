# Get list of cities in a state

Get list of cities in a given state from CPCB.

## Usage

``` r
cpcb_get_city_list(state)
```

## Arguments

- state:

  Character. State name (e.g. `"Maharashtra"`, `"Delhi"`).

## Value

Character vector of city names

## Examples

``` r
if (FALSE) { # \dontrun{
cities <- cpcb_get_city_list("Maharashtra")
head(cities)
} # }
```
