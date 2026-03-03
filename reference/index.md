# Package index

## CPCB historical data

Fetch historical air quality data from CPCB data repository

- [`cpcb_get_complete_list()`](https://saketkc.github.io/vayuayanR/reference/cpcb_get_complete_list.md)
  : Fetch Complete Station and City List from CPCB
- [`cpcb_get_state_list()`](https://saketkc.github.io/vayuayanR/reference/cpcb_get_state_list.md)
  : Get List of States with AQI Data
- [`cpcb_get_city_list()`](https://saketkc.github.io/vayuayanR/reference/cpcb_get_city_list.md)
  : Get List of Cities in a State
- [`cpcb_get_station_list()`](https://saketkc.github.io/vayuayanR/reference/cpcb_get_station_list.md)
  : Get Station List for a City
- [`cpcb_add_coords()`](https://saketkc.github.io/vayuayanR/reference/cpcb_add_coords.md)
  : Add Coordinates to Station List
- [`cpcb_get_file_path()`](https://saketkc.github.io/vayuayanR/reference/cpcb_get_file_path.md)
  : Get File Path for CPCB Data Download
- [`cpcb_download_city_data()`](https://saketkc.github.io/vayuayanR/reference/cpcb_download_city_data.md)
  : Download City-Level AQI Data
- [`cpcb_download_station_data()`](https://saketkc.github.io/vayuayanR/reference/cpcb_download_station_data.md)
  : Download Station-Level AQI Data

## CPCB live data

Real-time AQI monitoring from CPCB live dashboard

- [`cpcb_get_system_location()`](https://saketkc.github.io/vayuayanR/reference/cpcb_get_system_location.md)
  : Get System Location via IP Geolocation
- [`cpcb_get_all_india()`](https://saketkc.github.io/vayuayanR/reference/cpcb_get_all_india.md)
  : Get All India Monitoring Stations
- [`cpcb_get_nearest_station()`](https://saketkc.github.io/vayuayanR/reference/cpcb_get_nearest_station.md)
  : Find Nearest Monitoring Station
- [`cpcb_get_live_aqi_data()`](https://saketkc.github.io/vayuayanR/reference/cpcb_get_live_aqi_data.md)
  : Get Live AQI Data
- [`cpcb_get_live_aqi_data_for_station()`](https://saketkc.github.io/vayuayanR/reference/cpcb_get_live_aqi_data_for_station.md)
  : Get Live AQI Data for a Station

## Satellite PM2.5 data

Download and analyze WUSTL ACAG satellite PM2.5 estimates

- [`pm25_download_netcdf()`](https://saketkc.github.io/vayuayanR/reference/pm25_download_netcdf.md)
  : Download PM2.5 NetCDF File
- [`pm25_get_stats()`](https://saketkc.github.io/vayuayanR/reference/pm25_get_stats.md)
  : Get PM2.5 Statistics for a Region
- [`pm25_list_cached_files()`](https://saketkc.github.io/vayuayanR/reference/pm25_list_cached_files.md)
  : List Cached PM2.5 NetCDF Files
- [`pm25_clear_cache()`](https://saketkc.github.io/vayuayanR/reference/pm25_clear_cache.md)
  : Clear PM2.5 Cache

## Utility functions

Helper functions for data processing and analysis

- [`stations_to_dataframe()`](https://saketkc.github.io/vayuayanR/reference/stations_to_dataframe.md)
  : Convert Station Data to DataFrame
- [`analyze_station_data()`](https://saketkc.github.io/vayuayanR/reference/analyze_station_data.md)
  : Analyze Station Data
- [`clean_station_name()`](https://saketkc.github.io/vayuayanR/reference/clean_station_name.md)
  : Clean Station Name
- [`get_aqi_category()`](https://saketkc.github.io/vayuayanR/reference/get_aqi_category.md)
  : Get AQI Category
- [`get_aqi_categories()`](https://saketkc.github.io/vayuayanR/reference/get_aqi_categories.md)
  : Vectorized AQI Category
- [`haversine_distance()`](https://saketkc.github.io/vayuayanR/reference/haversine_distance.md)
  : Calculate Haversine Distance
- [`vayuayan_welcome()`](https://saketkc.github.io/vayuayanR/reference/vayuayan_welcome.md)
  : Print Vayuayan Welcome Message
