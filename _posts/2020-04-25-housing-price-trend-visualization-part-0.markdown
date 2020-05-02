---
layout: post
title:  Visualizing housing price trends with QGIS, part 0
date:   2020-04-25 15:47:02 +0200
categories: postgis osm openstreetmap qgis saga
---
This is an introduction to a series of posts to document the results of some self-study sessions carried out during the Covid-19 pandemic period, with its times of isolation and short-term layoffs. Geospatial analysis is very useful for many industries and this was a great opportunity to deepen my knowledge in the field. I have focused on QGIS, which is a massive open-sourced piece of GIS software (Geographic Information System), providing a stack of tools from backend data access to visualization and map generation.

I've always been a fan of learning by doing and, having access to a PostGIS database with housing sales prices in Sweden (enabling this access might become the topic for another blog post in the future), I was considering how to utilize this for the purpose of learning new tools and methodology.

Housing price trends are freely available on the Internet, e.g. at [Mäklarstatistik](https://www.maklarstatistik.se/), and they are presented in the obvious way: aggregated time series split by property type, city and district.

![Mäklarstatistik example](/images/maklarstatistik.png "Mäklarstatistik example")

This is of course very useful and accessible, but I was thinking about a way to present the price trends on a map, covering many cities and/or district at the same time. So, the end goal of this little project was quite clear to me from the start:

**How do we go about to visualize housing price trends in a way reminiscent of weather forecasts, where contour plots of the air pressure and heat map coloring are animated to show the movement of weather systems and changes of temperature?**

<p><a href="https://commons.wikimedia.org/wiki/File:2015-01-20_Surface_Weather_Map_NOAA.png#/media/File:2015-01-20_Surface_Weather_Map_NOAA.png"><img src="https://upload.wikimedia.org/wikipedia/commons/3/3f/2015-01-20_Surface_Weather_Map_NOAA.png" alt="2015-01-20 Surface Weather Map NOAA.png"></a><br><figcaption>By <a href="https://en.wikipedia.org/wiki/National_Centers_for_Environmental_Prediction" class="extiw" title="w:National Centers for Environmental Prediction">National Centers for Environmental Prediction</a> - <a rel="nofollow" class="external text" href="https://www.wpc.ncep.noaa.gov/dailywxmap/index.html">National Centers for Environmental Prediction</a>, Public Domain, <a href="https://commons.wikimedia.org/w/index.php?curid=38139297">Link</a></figcaption></p>

So, perhaps jumping the gun a bit, here is an example of the result (still a work in progress). The contours and colors represent the price per area of apartments in Stockholm. Each image in the video visualizes three months of sales prices (as shown in the upper right corner) and the images are updated once every month. The "equidistance" of the contour lines is 5,000 SEK/m².

<iframe src="https://player.vimeo.com/video/410120629" width="640" height="386" frameborder="0" allow="autoplay; fullscreen" allowfullscreen></iframe>
<p><figcaption><a href="https://vimeo.com/410120629">Apartment price trend, Stockholm 2013-2020</a> from <a href="https://vimeo.com/user113431800">Daniel Reimhult</a> on <a href="https://vimeo.com">Vimeo</a>.</figcaption></p>

There are a few steps involved to go from the apartment sales data points to the above animation. This will be the topic for future posts in this series.
