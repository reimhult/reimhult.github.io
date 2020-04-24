---
layout: post
title:  "Notes on PostGIS, OpenStreetMap and QGIS"
date:   2020-04-23 12:53:02 +0200
categories: postgis osm openstreetmap qgis
---

# Installation
## Enable PostGIS extension
    -- Enable PostGIS (as of 3.0 contains just geometry/geography)
    CREATE EXTENSION postgis;
    -- enable raster support (for 3+)
    CREATE EXTENSION postgis_raster;
    -- Enable Topology
    CREATE EXTENSION postgis_topology;
## QGIS
## Saga
https://sourceforge.net/p/saga-gis/wiki/Compiling%20SAGA%20on%20Linux/

Version 2.3.lts is required


# Downloading and ingesting map data
## Download
[Geofabrik](https://download.geofabrik.de/)
## Merge
To speed up the conversion and ingesting of OSM data into PostGIS, it is recommended to first merge the pbf files. This is because the `osm2pgsql` tool is very slow when appending to an existing PostGIS database.

There are several tools available to do the merging (`osmium`, `osmosis`, `osmconvert`). One tool that seems to work well is `osmconvert` (installed with `sudo apt install osmctools`). Note that `osmium` gives problems with duplicate entries in the merged output, rendering failure when running `osm2pgsql`.

    osmconvert sweden-latest.osm.pbf --out-o5m | osmconvert - norway-latest.osm.pbf --out-o5m | osmconvert - finland-latest.osm.pbf --out-o5m | osmconvert - germany-latest.osm.pbf --out-o5m | osmconvert - denmark-latest.osm.pbf -o=merged.osm.pbf

## Ingest
    sudo -u postgres osm2pgsql --multi-geometry -d osm -c --slim -C 12000 --flat-nodes /home/daniel/osm/flat-nodes merged.osm.pbf

# Visualizaion
## QGIS
### XYZ tiles
[Link to an overview of tile layer providers](https://www.spatialbias.com/2018/02/qgis-3.0-xyz-tile-layers/ "Spatial bias")

There is a convenient plugin in QGIS call QuickMapServices, where tile providers are collected in a conventient and searchable way. It can be found in the plugin installer in QGIS.

# SQL
## Example queries
### Create table with all railway border crossings from Sweden
{% highlight sql %}
CREATE TABLE border_railways AS
    SELECT l1.* FROM planet_osm_line AS l1
    CROSS JOIN planet_osm_line AS l2
    WHERE ST_Crosses(l1.way, l2.way) AND
        l1.railway IS NOT NULL AND
        l2.boundary = 'administrative' AND
        l2.name='Sverige'
;
{% endhighlight %}
### Create table with all road border crossings from Sweden
{% highlight sql %}
CREATE TABLE border_roads AS
    SELECT l1.* FROM planet_osm_line AS l1
    CROSS JOIN planet_osm_line AS l2
    WHERE ST_Crosses(l1.way, l2.way) AND
        l1.highway IS NOT NULL AND
        l1.highway NOT LIKE 'path' AND
        l1.highway NOT LIKE 'track' AND
        l2.boundary LIKE 'administrative' AND
        l2.name='Sverige'
;
{% endhighlight %}

# Math test
{% raw %}
  $$a^2+b^2=c^2$$
{% endraw %}

{% raw %}
  $$\frac{e^{\pi(n-\frac{1}{24})^{1/2}}}{4 \cdot 3^{1/4}(n-\frac{1}{24})^{3/4}}$$
{% endraw %}
