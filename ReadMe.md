If you do not have python or jupyter notebook installed yet, 
Go to python.org/downloads
Download the latest Python 3 installer
Run it and check the box “Add Python to PATH”
Click Install Then  open Command Prompt (Windows) or Terminal (Mac) and type:
python --version

You should see something like this:
Python 3.10.8

Then install Jupyter notebook by typing:
pip install notebook

Then install  libraries
pip install requests geopandas


Then navigate to your project folder... the terminal should look something like this
C:\\ > 

navigate to your directory:
C:\\ > cd "Your Path"

Mine looks something like this:
C:\Users\TPM>cd "C:\Users\Thomas.McKeon\folderToSaveWebScrapingData"

Once you're in your directory, type:
jupyter notebook

A new window will popup with Jupyter notebook.

Click on New > Python 3, now you're ready to build your script.


Getting the data:
Get the map into full view of new jersey, then inspect > Network. Search "query" and make sure filters is set to ALL.
https://experience.arcgis.com/experience/1e53539c4ad7469284037abc85acf292/page/Data-Maps 

Click on one of the urls that show up for query. Each one is a tile for the map, but the first part is what you need:
https://services7.arcgis.com/Z0rixLlManVefxqY/arcgis/rest/services/MDC_Clinical_Outcome_Variables/FeatureServer/1/query

Run the following code in your jupyter notebook:
```python
import requests
import geopandas as gpd

service_url = "https://services7.arcgis.com/Z0rixLlManVefxqY/arcgis/rest/services/MDC_Clinical_Outcome_Variables/FeatureServer/1/query"

all_features = []
offset = 0
page_size = 2000  # You can't download more than 2000 rows at a time so you need to pull batches. There were about 15000

while True:
    print(f"Fetching offset {offset}...")

    params = {
        "where": "1=1",
        "outFields": "*",
        "returnGeometry": "true",
        "f": "geojson",
        "resultOffset": offset,
        "resultRecordCount": page_size
    }

    r = requests.get(service_url, params=params)
    data = r.json()

    if "features" not in data or len(data["features"]) == 0:
        print("No more features. Done.")
        break

    all_features.extend(data["features"])
    offset += page_size

# Build final GeoDataFrame WITH CRS
gdf = gpd.GeoDataFrame.from_features(all_features, crs="EPSG:4326")

# Save to GeoJSON, you can remove this if you only want shapefile.
gdf.to_file("MDC_full.geojson", driver="GeoJSON")

# Save as Shapefile (creates a folder called MDC_full_shapefile) in the same directory you save your python script
gdf.to_file("MDC_full_shapefile", driver="ESRI Shapefile")


print("Complete dataset downloaded.")
print("Total features:", len(gdf))
```
