# map-with-ai-postgis
### Add datasets from [map-with-ai](https://github.com/facebookmicrosites/Open-Mapping-At-Facebook/wiki/Available-Countries) to postgis and filter the major roads. 
<br>

#### **Dependencies:** 
* postgreSQL
* postgis
* ogr2ogr - GDAL

## Data

* download country dataset from [map-with-ai](https://github.com/facebookmicrosites/Open-Mapping-At-Facebook/wiki/Available-Countries) and unzip _country_data_.gpkg.tar
* convert _country_data_.gpkg.tar to _country_data_.shp using ogr2ogr <br>`ogr2ogr -f "ESRI Shapefile" country_data.shp country_data.gpkg` <br>where country_data is *your data*

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
* see number of cases grouped by highway tag column <br>`select highway_ta,count(highway_ta) from map_with_ai GROUP BY highway_ta ORDER BY count(highway_ta);`
* create new table with major roads <br>`create table output AS (select * from map_with_ai WHERE highway_ta IN ('motorway_link', 'trunk_link', 'teriary_link', 'motorway', 'secondary_link', 'primary_link', 'trunk', 'primary', 'secondary', 'tertiary'));`
* exit database <br>`exit;`

## Export data
* export the _output_ table to shapefile format <br>`pgsql2shp -f output.shp -h localhost -u postgres -P password map_with_ai romania`<br> where -P is your DB password