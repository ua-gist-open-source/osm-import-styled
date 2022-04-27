# OSM Import Using `osm2psql`
## Prerequisites
- Docker installed
- Docker running with PostGIS with a host volume.
- PGAdmin or other postgres client installed
- QGIS installed

## Deliverables
Create a new branch named `osm2pgsql` and submit a `pull request` to merge with master. Your branch should contain:
- `osm2pgsql-screencap.png`
- `styled-osm-screencap.png`

## Background
In a previous assignment we used the `shp2pgsql` to import canned shapefiles that have been derived from OSM data. However, the full power of OSM is really in the rich annotations of the primitive database design of nodes, ways, and relations. Importing the raw data into postgresql allows a much more flexible and extensible database for querying and displaying the OSM features. 

Read the following:
- [OpenStreetMap to PostGIS - The Basics](https://www.cybertec-postgresql.com/en/open-street-map-to-postgis-the-basics/)
- [Visualizing OSM Data in QGIS](https://www.cybertec-postgresql.com/en/visualizing-osm-data-in-qgis/) 

## Assignment
We will be downloading an extract of OSM From Iceland, importing it into a postgres database in which there are tables for: 
- points
- lines
- roads
- polygons

In this database, all non-road line features will be shared from within a single table and symbolized using rule-based symbology.

## Prepare data
We will use OSM data extracted from the OSM database in the .pbf format. Technically you could use any export but we will use the GeoFabrik export for Iceland. 
- OSM Data for Iceland: [download](https://download.geofabrik.de/europe/iceland-latest.osm.pbf)
- Styles for OSM Data: [github](https://github.com/yannos/Beautiful_OSM_in_QGIS)

1. Download the OSM Data.
2. Clone the `Beautiful_OSM_in_QGIS` repo to your local machine.

## Create the PostGIS database
With your postgis docker container running, issue the following commands from a shell to create the database and enable the postgis and hstore extensions:

```
docker run --network gist604b --entrypoint sh mdillon/postgis -c 'psql -h postgis -U postgres -c "CREATE DATABASE iceland"'
docker run --network gist604b --entrypoint sh mdillon/postgis -c 'psql -h postgis -U postgres -d iceland -c "CREATE EXTENSION postgis"'
docker run --network gist604b --entrypoint sh mdillon/postgis -c 'psql -h postgis -U postgres -d iceland -c "CREATE EXTENSION hstore"'
 ```
 
 ## Populate the PostGIS database with OSM pbf data
 We will use a pre-built docker container with the osm2psql tool in it. This container [openfirmware/osm2pgsql](https://hub.docker.com/r/openfirmware/osm2pgsql/), will run in our `gist604b` network and connect to our postgis database. We will have to mount a volume to make our downloaded pbf file accessible to the docker container. 
 
 In my example, `/Users/aaryn/gist604b_data/downloads` is where I downloaded the pbf file to but for you it will be different.
 ```
 docker run -i -t -e PGPASSWD=postgres -v /Users/aaryn/gist604b_data/downloads:/data/downloads --network gist604b --rm openfirmware/osm2pgsql -c 'osm2pgsql -U postgres -W -d iceland -H postgis -s -G --number-processes 8 -C 20480 /data/downloads/iceland-latest.osm.pbf' 
```

This could take some time so be prepared to let it sit. When it is complete, you will see a message like:
`Osm2pqsql took 7196s overall.`

Take a screenshot of your shell screen with the output of the above command and name it `osm2pgsql-screencap.png`.

## Visualize your OSM data in QGIS
Read the blog, [Visualizing OSM Data in QGIS](https://www.cybertec-postgresql.com/en/visualizing-osm-data-in-qgis/). You should already have cloned the [Beautiful OSM in QGIS](https://github.com/yannos/Beautiful_OSM_in_QGIS) repo and have the styles accessible to you. Load the four layers into QGIS and apply the beautiful OSM styles.

Take a screenshot of your final map in QGIS and save it to this repo as `styled-osm-screencap.png`.

## Deliverables:
- `osm2pgsql-screencap.png`
- `styled-osm-screencap.png`

