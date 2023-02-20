# map-with-ai-postgis
### Add datasets from [map-with-ai](https://github.com/facebookmicrosites/Open-Mapping-At-Facebook/wiki/Available-Countries) to postgis and filter the major roads. 
<br>

#### **Dependencies:** 
* postgreSQL
* postgis
* ogr2ogr - GDAL

## Data

* download country dataset from [map-with-ai](https://github.com/facebookmicrosites/Open-Mapping-At-Facebook/wiki/Available-Countries) and unzip _country_data_.gpkg.tar
* convert _country_data_.gpkg to _country_data_.shp using ogr2ogr <br>`ogr2ogr -f "ESRI Shapefile" country_data.shp country_data.gpkg` - where country_data is *your data*. <br>
* **If you see this error:** <br>`Warning 1: 2GB file size limit reached for test.dbf. Going on, but might cause compatibility issues with third party software`<br> you will have to split your data into multiple shapefiles because of the 2GB .dbf limitation 

## DB

* connect to localhost <br> `psql -U userName;`
* create new database <br>`CREATE DATABASE map_with_ai WITH ENCODING 'UTF8';`
* connect to the created db <br>`\c map_with_ai;`
* create postgis extension <br>`CREATE EXTENSION POSTGIS;`

## Load data into postgreSQL

* open a terminal in the folder where you have the  _country_data_.shp
* import the  _country_data_.shp to postgreSQL using shp2pgsql <br>`shp2pgsql -D -I -s 4326 country_data.shp | psql dbname=map_with_ai postgres`

## Filter data

* drop duplicated geometry column <br>`ALTER TABLE map_with_ai DROP COLUMN wkt;`
* see number of cases grouped by highway tag column <br>`select highway_ta AS highway_tag,count(highway_ta) AS cases_count from map_with_ai GROUP BY highway_ta ORDER BY count(highway_ta);`
* create new table with major roads <br>`create table output AS (select * from map_with_ai WHERE highway_ta IN ('motorway', 'motorway_link', 'trunk', 'trunk_link', 'primary', 'primary_link', 'secondary', 'secondary_link', 'tertiary', 'tertiary_link'));`
* exit database <br>`exit;`

## Export data
* export the _output_ table to shapefile format <br>`pgsql2shp -f output.shp -h localhost -u postgres -P password map_with_ai output`<br> where -P is your DB password