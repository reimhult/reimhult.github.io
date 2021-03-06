---
layout: post
title:  "Notes on PostGIS, OpenStreetMap and QGIS"
date:   2020-04-23 12:53:02 +0200
categories: postgis osm openstreetmap qgis
---
**Preface:** This post is mainly a collection of notes on installation and basic usage of a few important tools for geo-spatial analysis. It is a work in progress.

# Installation
## Postgresql and PostGIS
### Links
* [Installng Postgresql](https://www.postgresql.org/download/linux/ubuntu/)
* [Installing PostGIS](https://wiki.postgresql.org/wiki/Apt)

### Enable PostGIS extension
To use the PostGIS functionality, it needs to be enabled by creating the relevant extensions. In `psql`, create the database (`CREATE DATABASE <db name>;`), switch to the database (`\c <db name>`) and run the following lines of code:
{% highlight sql %}
-- Enable PostGIS (as of 3.0 contains just geometry/geography)
CREATE EXTENSION postgis;
-- enable raster support (for 3+)
CREATE EXTENSION postgis_raster;
-- Enable Topology
CREATE EXTENSION postgis_topology;
{% endhighlight %}

## QGIS
Currently, the version available in the Ubuntu APT repository is 3.4, which is quite old. The latest LTS version is 3.10 and the latest release is 3.12, and there have been many bug fixes and improvements done since version 3.4. Information about how to install a newer version can be found [here](https://qgis.org/en/site/forusers/alldownloads.html#debian-ubuntu).

The QGIS plugin framework requires Python 2.x. A convenient way to run QGIS without problems is to create a virtual environment with Python version 2.7 and then activate this before starting QGIS.

* Create virtual environment: `conda create --name qgis python=2.7`
* Activate: `conda activate qgis`

## Saga
Saga is an abbreviation for System for Automated Geoscientific Analyses. It is a comprehensive suite of GIS tools, all open-source, and QGIS has support for running SAGA tools directly from the QGIS Processing toolbox.

Installing SAGA on a recent version of Ubuntu is a bit cumbersome. The long term stable version of SAGA is quite old, it's version 2.3.lts released in 2017. The latest release is 7.6, but QGIS is only compatible with the LTS version. However, this version is not available in any apt repository for recent releases of Ubuntu, so it needs to be compiled from source unless you are running Ubuntu 16.04 or older.

Here's how to do it:
* Clone the 2.3.lts release from the SAGA git repository: `git clone --branch release-2-3-lts git://git.code.sf.net/p/saga-gis/code saga-gis-code`
* Follow the [compilation instructions](https://sourceforge.net/p/saga-gis/wiki/Compiling%20SAGA%20on%20Linux/).

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

There is a nice plugin in QGIS call QuickMapServices, where tile providers are collected in a convenient and searchable way. It can be found in the plugin installer in QGIS.

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

# Some mathematics to test MathJax in Jekyll :)
{% raw %}
$$a^2+b^2=c^2$$
{% endraw %}

{% raw %}
$$\boxed{\displaystyle\int_0^1\dfrac{\ln\left(x^4-2x^2+5\right)-\ln\left(5x^4-2x^2+1\right)}{1-x^2}\ dx=\pi\arctan\left(\sqrt{\dfrac{\sqrt{5}-1}{2}}\right)}$$
{% endraw %}
