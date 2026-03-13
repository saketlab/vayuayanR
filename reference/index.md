# Package index

## CPCB historical data

Fetch historical air quality data from CPCB data repository

- [`cpcb_get_complete_list()`](https://saketkc.github.io/vayuayanR/reference/cpcb_get_complete_list.md)
  : Fetch complete station and city list from CPCB
- [`cpcb_get_state_list()`](https://saketkc.github.io/vayuayanR/reference/cpcb_get_state_list.md)
  : Get list of states with AQI data
- [`cpcb_get_city_list()`](https://saketkc.github.io/vayuayanR/reference/cpcb_get_city_list.md)
  : Get list of cities in a state
- [`cpcb_get_station_list()`](https://saketkc.github.io/vayuayanR/reference/cpcb_get_station_list.md)
  : Get station list for a city
- [`cpcb_add_coords()`](https://saketkc.github.io/vayuayanR/reference/cpcb_add_coords.md)
  : Add coordinates to station list
- [`cpcb_get_file_path()`](https://saketkc.github.io/vayuayanR/reference/cpcb_get_file_path.md)
  : Get file path for CPCB data download
- [`cpcb_download_city_data()`](https://saketkc.github.io/vayuayanR/reference/cpcb_download_city_data.md)
  : Download city-level AQI data
- [`cpcb_download_station_data()`](https://saketkc.github.io/vayuayanR/reference/cpcb_download_station_data.md)
  : Download station-level AQI data

## CPCB live data

Real-time AQI monitoring from CPCB live dashboard

- [`cpcb_get_system_location()`](https://saketkc.github.io/vayuayanR/reference/cpcb_get_system_location.md)
  : Get system location via IP geolocation
- [`cpcb_get_all_india()`](https://saketkc.github.io/vayuayanR/reference/cpcb_get_all_india.md)
  : Get all India monitoring stations
- [`cpcb_get_nearest_station()`](https://saketkc.github.io/vayuayanR/reference/cpcb_get_nearest_station.md)
  : Find nearest monitoring station
- [`cpcb_get_live_aqi_data()`](https://saketkc.github.io/vayuayanR/reference/cpcb_get_live_aqi_data.md)
  : Get live AQI data
- [`cpcb_get_live_aqi_data_for_station()`](https://saketkc.github.io/vayuayanR/reference/cpcb_get_live_aqi_data_for_station.md)
  : Get live AQI data for a station

## Satellite PM2.5 data

Download and analyze WUSTL ACAG satellite PM2.5 estimates

- [`pm25_download_netcdf()`](https://saketkc.github.io/vayuayanR/reference/pm25_download_netcdf.md)
  : Download PM2.5 NetCDF file
- [`pm25_get_stats()`](https://saketkc.github.io/vayuayanR/reference/pm25_get_stats.md)
  : Get PM2.5 statistics for a region
- [`pm25_list_cached_files()`](https://saketkc.github.io/vayuayanR/reference/pm25_list_cached_files.md)
  : List cached PM2.5 NetCDF files
- [`pm25_clear_cache()`](https://saketkc.github.io/vayuayanR/reference/pm25_clear_cache.md)
  : Clear PM2.5 cache

## Utility functions

Helper functions for data processing and analysis

- [`stations_to_dataframe()`](https://saketkc.github.io/vayuayanR/reference/stations_to_dataframe.md)
  : Convert station data to data frame
- [`analyze_station_data()`](https://saketkc.github.io/vayuayanR/reference/analyze_station_data.md)
  : Analyze station data
- [`clean_station_name()`](https://saketkc.github.io/vayuayanR/reference/clean_station_name.md)
  : Clean station name
- [`get_aqi_category()`](https://saketkc.github.io/vayuayanR/reference/get_aqi_category.md)
  : Get AQI category
- [`get_aqi_categories()`](https://saketkc.github.io/vayuayanR/reference/get_aqi_categories.md)
  : Vectorized AQI category
- [`haversine_distance()`](https://saketkc.github.io/vayuayanR/reference/haversine_distance.md)
  : Calculate haversine distance
- [`vayuayan_welcome()`](https://saketkc.github.io/vayuayanR/reference/vayuayan_welcome.md)
  : Print vayuayan welcome message
