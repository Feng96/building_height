# building-height

## USA Structures

- https://disasters.geoplatform.gov/publicdata/Partners/ORNL/USA_Structures/
- Download data by state manually and save them to `Team-Drives/Buildings/USA_Structures/`
- 51 files in total: 50 states + DC
- Total file size: 21.1 GB
- Rename the files

```bash
rename 'v2_OCC' '' *
rename '_OCC' '' *
```

- Read GeoFiledabase using Geopandas

```python
import geopandas as gpd
url = 'USA_Structures/Deliverable20211222DC/DC_Structures.gdb/'
gdf = gpd.read_file(url, layer='DC_Structures')
gdf.head()[['HEIGHT', 'geometry']].explore()
```

## Microsoft US Footprints data

- https://github.com/microsoft/USBuildingFootprints
- Vector tile: https://www.arcgis.com/home/item.html?id=f40326b0dea54330ae39584012807126
- Download using Google Colab
- 50 states + DC. 4.5 GB in total

```python
import requests
from bs4 import BeautifulSoup

url = "https://github.com/microsoft/USBuildingFootprints/blob/master/README.md"
response = requests.get(url)
soup = BeautifulSoup(response.content, "html.parser")

links = []
for link in soup.find_all("a"):
    href = link.get("href")
    if href.endswith(".zip"):
        links.append(href)

out_dir = '/media/hdd/Data/Buildings/MS_USBuildingFootprints/'
leafmap.download_files(links, out_dir=out_dir, unzip=False)
```

## Microsoft Global Building Footprints

- https://github.com/microsoft/GlobalMLBuildingFootprints
- https://docs.google.com/spreadsheets/d/1uq56aNQ9M3pLR5Tlm5ILqvFlVJVG-gV6zRIfqrvteYo-
- https://github.com/microsoft/GlobalMLBuildingFootprints/issues/34

```python
import pandas as pd
import geopandas as gpd
from shapely.geometry import shape
dataset_links = pd.read_csv("https://minedbuildings.blob.core.windows.net/global-buildings/dataset-links.csv")
greece_links = dataset_links[dataset_links.Location == 'Greece']
for _, row in greece_links.iterrows():
    df = pd.read_json(row.Url, lines=True)
    df['geometry'] = df['geometry'].apply(shape)
    gdf = gpd.GeoDataFrame(df, crs=4326)
    gdf.to_file(f"{row.QuadKey}.geojson", driver="GeoJSON")
```

## OSM Microsoft Building Footprint Data

- https://wiki.openstreetmap.org/wiki/Microsoft_Building_Footprint_Data

- Download files manually
- Merge shapefiles

```python
import os
import leafmap
cwd = os.getcwd()

out_dir = os.path.join(cwd, 'Shapefile')
files = leafmap.find_files(cwd, ext='zip', recursive=False)
files.sort()

for file in files:
    gdf = gpd.read_file(file)
    basename = os.path.basename(file).replace(" ", "").replace('zip', 'shp')
    filename = os.path.join(out_dir, basename)
    gdf.to_file(filename)
```
