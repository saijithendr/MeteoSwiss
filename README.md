# MeteoSchweiz

## Description

A comprehensive Python library for accessing, managing, and analyzing Swiss weather data from MeteoSwiss (Federal Office of Meteorology and Climatology). This toolkit provides high-level interfaces for working with weather stations, parameters, historic data, and forecasts.

## Installation

The `meteoschweiz` project can be installed like any standard Python package using `pip install <package>`. However, due to ongoing development, the project is not yet available on PyPI. The recommended way to install the project in your local environment is through the company's GitLab repository:

**Installation Command:**
* ```GIT_SSL_NO_VERIFY=true pip install git+https://oauth2:<PAT_TOKEN>@lab.iabg.de/Gangireddy/meteoschweiz.git ```

**Access Token:**
- The Personal Access Token (PAT) is stored in `Master KeyPass`
- If you don't have access, contact `Gangireddy@iabg.de`


## Usage


### 1. Metadata

#### 1.1 Stations
```python
from meteoschweiz.metadata.stations import SwissWeatherStations

stations = SwissWeatherStations()
stations_df = stations.load() # It returns the all available weather stations in swiss

# One could also get a specific station info
gre = stations.get_by_abbr('GRE')
print(f"{gre.point_name}: {gre.elevation:.0f}m")
```
### 1.2 Metaparameters data
As shown below, using ```MetaParametersLoader``` one could load the parameters from a source url i.e.., local forecast paramter, histroic parameter and make use of defined methods as demonstrated below

```python

from src.meteoschweiz.metadata.parameters import MetaParametersLoader

loader = MetaParametersLoader()

# Add sources
loader.add_source(
    name="Forecast Parameters",
    url="https://data.geo.admin.ch/ch.meteoschweiz.ogd-local-forecasting/ogd-local-forecasting_meta_parameters.csv",
    description="MeteoSwiss forecast parameters",
    encoding="latin-1",
    delimiter=";",
    key_column="parameter_shortname"
)

loader.add_source(
    name="Historic Parameters",
    url="https://data.geo.admin.ch/ch.meteoschweiz.ogd-smn/ogd-smn_meta_parameters.csv",
    description="MeteoSwiss historic/SMN parameters",
    encoding="latin-1",
    delimiter=";",
    key_column="parameter_shortname"
)

print(f" Added {len(loader.list_sources())} sources:")
for name in loader.list_sources():
    print(f"  • {name}")

# Load data
# Load Parameters
loader.load_all()
print(f" Loaded {len(loader.data)} sources")

# Get parameters
## Get All Parameters
df_params = loader.get_all_params(source="Forecast Parameters")
print(f"✓ Forecast Parameters: {len(df_params)} rows × {len(df_params.columns)} columns")
print(f"  Columns: {list(df_params.columns)[:5]}...")

# Search parameters
## Search Parameters by Keyword
temp_params = loader.search("temperature", source="Forecast Parameters")
print(f"✓ Found {len(temp_params)} temperature-related parameters")

# Get specific parameter
## Get Specific Parameter
param = loader.get("tre200h0", source="Historic Parameters")
if param:
    print("  Parameter: tre200h0")
    print(f"  Source: {param.source}")
    print(f"  Data: {list(param.data.keys())}")

# Get statistics
## Loader Statistics
print(loader.summary())


```

### 2. Get Historic Weather Data
```python

from meteoschweiz.historic.historic_handler import HistoricWeatherHandler, MeteoSwissClient


client = MeteoSwissClient()
historic_handler = HistoricWeatherHandler(
    stations_handler=stations, # from 1.1
    params_loader=loader, # from 1.2
    meteoswiss_client=client,
    language="en" # or de
)
print("✓ Historic handler initialized")

## Get Historic Data by Station ID
start_date = "2023-06-01"
end_date = "2023-06-30"

# the method will return a data structure HistoricQueryResult contains
'''
station_id: str
station_name: str
canton: Optional[str]
latitude: float
longitude: float
altitude: float
aggregation: str
parameters: List[str]
start_date: Optional[pd.Timestamp]
end_date: Optional[pd.Timestamp]
data: pd.DataFrame

'''
result = historic_handler.get_historic_by_station_id(
    station_id="GRE",
    start_date=start_date,
    end_date=end_date,
    aggregation="daily",
    parameters=["tre200d0", "tre200dn", "tre200dx", "rre150d0"],
    rename_columns=True,
    include_units=True )



```

### 3. Get Local Forecast data

```python
from meteoschweiz.forecasts.forecast_handler import LocalForecastHandler

forecast_handler = LocalForecastHandler(
    stations_handler=stations,
    params_loader=loader
)

# Get forecast for Grenchen
result = forecast_handler.get_forecast_for_point_id(
    point_id=774,
    parameters=["tre200h0", "rre150h0"]
)

print(result.summary())
```
The following provides a detailed overview of all available methods.

### SwissWeatherStations (`stations.py`)
**Purpose:** Station lookup and geographic queries

| Method | Purpose |
|--------|---------|
| `load()` | Load all stations from MeteoSwiss |
| `get_by_abbr(abbr)` | Get station by abbreviation (e.g., 'GRE') |
| `get_by_name(name)` | Search stations by name |
| `get_by_id(id)` | Get station by point ID |
| `find_nearby(lat, lon, radius_km)` | Find stations within radius |
| `find_nearest(lat, lon, n)` | Find n nearest stations |
| `filter_by_bbox(lat_min, lat_max, lon_min, lon_max)` | Regional filter |
| `filter_by_elevation(min, max)` | Elevation-based filter |
| `get_highest_stations(n)` | Get n stations with highest elevation |
| `get_statistics()` | Get comprehensive statistics |

### MetaParametersLoader (`parameters.py`)
**Purpose:** Parameter metadata management

| Method | Purpose |
|--------|---------|
| `add_source(name, url, ...)` | Register CSV source |
| `load_source(name)` | Load data from source |
| `load_all()` | Load all sources |
| `get_all_params(source)` | Get all parameters from source |
| `search(keyword)` | Search by keyword |
| `filter(conditions)` | Filter by conditions |
| `get(key)` | Get specific parameter |
| `export_to_csv(source, filepath)` | Export to CSV |
| `summary()` | Get loader summary |

### HistoricWeatherHandler (`historic_handler.py`)
**Purpose:** Historic weather data retrieval

| Method | Purpose |
|--------|---------|
| `get_historic_by_station_id(station_id, ...)` | Query by SMN station ID |
| `get_historic_by_name(station_name, ...)` | Query by Swiss station name |
| `get_historic_by_coords(lat, lon, ...)` | Query nearest station |
| `get_temperature_history(station, start, end)` | Temperature convenience method |
| `get_precipitation_history(station, start, end)` | Precipitation convenience method |
| `list_available_parameters()` | List all available parameters |
| `export_to_csv(result, filepath)` | Export data to CSV |
| `export_to_json(result, filepath)` | Export data to JSON |

### LocalForecastHandler (`forecast_handler.py`)
**Purpose:** Local forecast data retrieval

| Method | Purpose |
|--------|---------|
| `get_forecast_for_point_id(point_id, ...)` | Get forecast by location ID |
| `get_forecast_for_station_name(name, ...)` | Get forecast by station name |
| `get_forecast_for_coordinates(lat, lon, ...)` | Get forecast for nearest station |
| `get_all_parameters()` | List available forecast parameters |
| `export_to_csv(result, filepath)` | Export forecast to CSV |
| `export_to_json(result, filepath)` | Export forecast to JSON |


## Bugs / Issues

If you find any issue / bug please report here https://lab.iabg.de/Gangireddy/meteoschweiz/-/issues using the following format so that it will be clear and easy to traceback.

select type as `issue` and assignee as `@Gangireddy`

![alt text](image.png)

```markdown

name: Bug Report
about: Create a bug report to help us improve
title: '[BUG] '
labels: bug
assignees:


## Bug Description
<!-- Clear and concise description of the bug -->

## Steps to Reproduce
<!-- How to reproduce the issue -->
1.
2.
3.

## Expected Behavior
<!-- What should happen -->

## Actual Behavior
<!-- What actually happens -->

## Environment
- Python version:
- Operating System:
- Library versions (pandas, requests, etc.):

## Code Example
<!-- Minimal code to reproduce -->
```

## Developer Guide for contributions

```bash

# step 1. For the project (main branch) by clicking the fork which is on top right corner in gitlab ui

# step 2. Create a git branch
git checkout -b feature/your-feature-name

# step 3. setup your local environment
pip install -r requirements-dev.txt

# step 4. Add / Update features as you need

# step 5. Run the pytests

pytest tests/test_meteo_swiss.py -v # to run all tests

# step 6. upon passing all the tests, commit the changes and push them to your remote repository

# step 7. Upon successfull git push to your remote repo, create a PR (pull request) to merge your branch into main branch project repository.
```
### Pull Request format

```markdown

name:
about: short info of what is this about.
title: '[Feature] ' # or  Update
labels: feature     # or update
assignees:


## Description
<!-- Clear and concise description of fetaure / updated component -->


## Expected Behavior
<!-- What should happen ? with your change -->

## Example results
<!-- Implementation results with minimal code -->
```

## Roadmap

1. Better visuliaztions (station geo cords, districts, states)
2. Data preprocessing pipelines
3. Individual weather feature handlers

## Authors and acknowledgment
Show your appreciation to those who have contributed to the project.

## License
For open source projects, say how it is licensed.