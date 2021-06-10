---
layout: post
title:  Visualizing housing price trends with QGIS, part 2 - Malmö visualized
date:   2020-06-08 16:47:00 +0200
categories: postgis sql osm openstreetmap qgis saga
published: false
---
The last part involved trend analysis of the housing prices during the last year. In this part, we will step back a bit and create visualizations in QGIS that are static in nature. These will be used later to create the animated trend visualizations showed in the introduction.

Before diving into the visualizations, let's make a reasonable subselection of the sold properties data entries using the following criteria:

- Apartments
- Within a box enclosing the center of the city of Malmö
- Sold in 2019 or 2020

{% highlight sql %}
SELECT *                                                                             
FROM sold_properties
WHERE (property_type = 'Bostadsrättslägenhet' OR
       property_type = 'Bostadsrätt' OR
       property_type = 'Andel i bostadsrätt'
      )
      AND
      ST_Within("way", ST_Expand(ST_Transform('SRID=4326;POINT(12.999218 55.586437)'::geometry, 3857),
                                 8000)
               )
      AND
      sold_date >= '2019-01-01'
;
{% endhighlight %}

In QGIS, the Filter functionality for a layer allows you to enter the SQL query part after the `WHERE` statement. Doing this with the above query and then zooming in to the layer extents, the following will appear. Here, [Wikimedia Tool Labs OSM black/white](https://maplayers-demo.toolforge.org/) has been used as the map tile image provider.

[![QGIS screenshot 1](/images/QGIS_Malmo_1.png)](/images/QGIS_Malmo_1.png)

That's a good start, but the image does not convey very much information. Changing the marker type to `x` and encoding the value of the `price_per_area` column as the color (blue = cheap, red = expensive) and the `main_area` as the marker size, generates the following.

[![QGIS screenshot 2](/images/QGIS_Malmo_2.png)](/images/QGIS_Malmo_2.png)

There is a clear, not very surprising, tendency of higher prices closer to the water. Also, there appears to be quite steep price changes at several locations, with significantly increasing sales prices just by moving a couple of blocks. One way to quantify this is the extract the contour lines showing the isolines of the average price level. Thus, by "counting the lines" reminiscent of reading the equidistance height lines on a map, once can 


- Create layer, show points
- Filter on Malmö
- Change marker
- Filter and create contour
- Show
