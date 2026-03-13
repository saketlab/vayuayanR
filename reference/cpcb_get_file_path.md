# Get file path for CPCB data download

Get file path for downloading data from CPCB.

## Usage

``` r
cpcb_get_file_path(
  station_id = "",
  station_name = "",
  state = "",
  city = "",
  year = "",
  frequency = "daily",
  data_type = "cityLevel"
)
```

## Arguments

- station_id:

  Station ID

- station_name:

  Station name

- state:

  State name

- city:

  City name

- year:

  Year for data

- frequency:

  Data frequency ('hourly' or 'daily')

- data_type:

  Type of data ('cityLevel' or 'stationLevel')

## Value

List of file path information
