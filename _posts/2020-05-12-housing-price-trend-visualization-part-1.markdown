---
layout: post
title:  Visualizing housing price trends with QGIS, part 1 - Data overview and some queries
date:   2020-05-12 14:36:00 +0200
categories: postgis sql osm openstreetmap qgis saga
---
So let's have a look at the data! All property sales are stored as a row in a PostGIS enabled PostgreSQL database table. For every sale, there is information about e.g. the property type (villa, apartment, plot of land etc), the number of rooms, the sales date and price. And of course, since this will eventually be about geospatial analysis, the location is available as a PostGIS geometry datatype.

An example of a table entry of a house sale is shown below:

| Column | SQL table column name | Value |
|---|---|
| Address | address | Stoppgatan 3 |
| District | district | Tollarp |
| Municipality | kommun | Kristianstads kommun |
| Property type | property_type | Villa |
| Sales date | sold_date | 2020-04-27 |
| Broker | broker | Länsförsäkringar Fastighetsförmedling Kristianstad
| Sales price | final_price | 1520000 |
| Ask price | ask_price | 1395000 |
| Price per area | price_per_area | 11692 |
| Number of rooms | num_rooms | 5 |
| Main living area | main_area | 130 |
| Misc. area | misc_area | 0 |
| Plot area | plot_area | 263 |
| Build year | build_year | 1976 |
| Maintentance cost | maintenance | 32692
| Latitude | lat | 55.9313614 |
| Longitude | lon | 13.99752036 |
| Geometry | way | 0101000020110F0000F777698C00BD374166BF31F1EEC75C41 |

In this part, we'll get started by doing some basic traditional trend analysis using SQL. We want to answer the question:

> What is the monthly price trend for apartments in the largest cities of Sweden during the last 12 months?

I'll start with posting the desired end result, and then go into the details of the SQL queries to generate it.

#### Desired result

|month    | Stockholm | Göteborg | Malmö | Uppsala | Solna
|------------+-----------+----------+-------+---------+-------
|2019-05-01 |     100.0 |    100.0 | 100.0 |   100.0 | 100.0
|2019-06-01 |     102.4 |     97.1 |  98.2 |    95.7 |  97.9
|2019-07-01 |      98.1 |    101.8 |  99.1 |   112.1 | 101.6
|2019-08-01 |     102.7 |    105.1 | 104.1 |   112.4 | 103.3
|2019-09-01 |     106.9 |    102.6 | 104.6 |   104.6 |  99.3
|2019-10-01 |     105.9 |    103.1 | 101.0 |   102.0 | 100.3
|2019-11-01 |     105.9 |    102.3 | 100.0 |    99.5 | 102.0
|2019-12-01 |     108.4 |    100.7 |  93.9 |   108.3 | 103.8
|2020-01-01 |     109.5 |    105.1 | 103.6 |   107.9 | 106.1
|2020-02-01 |     110.5 |    103.5 | 109.5 |   105.7 | 105.8
|2020-03-01 |     108.5 |    102.3 |  97.2 |    99.6 | 100.0
|2020-04-01 |     102.2 |    103.3 | 100.1 |    98.7 | 100.0

This table shows that there is a considerable drop in the median price per area unit during March and/or April in Stockholm, Malmö, Uppsala and Solna. Interestingly, the same effect is not showing in Göteborg. However, this is based on the very blunt tool provided by the median. Examining the price distribution more carefully might reveal another story. (This may be the topic for another future post, so let's leave it the way it is for now.)

The SQL query to achieve this table will be built up in three steps below. First, the median price for the big municipalities are calculated. Second, the normalization to the first month is done. And lastly, the resulting table is pivoted to create the desired output.

#### Query 1 - Calculate the median price per month for selected cities

{% highlight sql %}
SELECT date_trunc('month', sold_date)::date AS month,
       kommun,
       percentile_disc(0.5) WITHIN GROUP (ORDER BY price_per_area) AS median
FROM sold_properties
WHERE property_type LIKE 'Bostadsr%' AND
      sold_date BETWEEN '2019-05-01' AND '2020-04-30' AND
      kommun IN (SELECT kommun
                 FROM sold_properties
                 WHERE property_type LIKE 'Bostadsr%' AND
                       sold_date BETWEEN '2019-05-01' AND '2020-04-30'
                 GROUP BY kommun
                 HAVING count(*) > 2000
                 )
GROUP BY 1,2
;
{% endhighlight %}

#### Results

|month    |      kommun       | median
|------------+-------------------+--------
|2019-05-01 | Göteborgs kommun  |  47561
|2019-05-01 | Malmö kommun      |  28947
|2019-05-01 | Solna kommun      |  55705
|2019-05-01 | Stockholms kommun |  68478
|2019-05-01 | Uppsala kommun    |  35294
|2019-06-01 | Göteborgs kommun  |  46190
|2019-06-01 | Malmö kommun      |  28429
|2019-06-01 | Solna kommun      |  54509
|2019-06-01 | Stockholms kommun |  70089
|2019-06-01 | Uppsala kommun    |  33769
|2019-07-01 | Göteborgs kommun  |  48438
|2019-07-01 | Malmö kommun      |  28687
|...|...|...

Now, let's normalize the median prices to the first value of the time period (May 2019) to represent the percentage change. This is done using the above query as a sub-query and scaling by the first value of a sorted list of price values for all municipalities.

#### Query 2 - Normalize to the first month in the period
{% highlight sql %}
SELECT month,
       kommun,
       100*median/first_value(median) OVER (PARTITION BY kommun
                                            ORDER BY month) AS norm_price
FROM (SELECT date_trunc('month', sold_date)::date AS month,
             kommun,
             percentile_disc(0.5) WITHIN GROUP (ORDER BY price_per_area) AS median
      FROM sold_properties
      WHERE property_type LIKE 'Bostadsr%' AND
            sold_date BETWEEN '2019-05-01' AND '2020-04-30' AND
            kommun IN (SELECT kommun
                       FROM sold_properties
                       WHERE property_type LIKE 'Bostadsr%' AND
                             sold_date BETWEEN '2019-05-01' AND '2020-04-30'
                       GROUP BY kommun
                       HAVING count(*) > 2000
                       )
      GROUP BY 1,2
     )
     AS monthly_price
;
{% endhighlight %}

#### Results

|month    |      kommun       |     norm_price     
|------------+-------------------+--------------------
|2019-05-01 | Göteborgs kommun  |                100
|2019-06-01 | Göteborgs kommun  |  97.11738609364815
|2019-07-01 | Göteborgs kommun  | 101.84394777233447
|2019-08-01 | Göteborgs kommun  | 105.12815121633271
|2019-09-01 | Göteborgs kommun  | 102.56302432665419
|2019-10-01 | Göteborgs kommun  |  103.0928702087845
|2019-11-01 | Göteborgs kommun  | 102.27497319232144
|2019-12-01 | Göteborgs kommun  | 100.74851243666029
|2020-01-01 | Göteborgs kommun  | 105.12815121633271
|2020-02-01 | Göteborgs kommun  |  103.5344084438931
|2020-03-01 | Göteborgs kommun  | 102.25184499905384
|2020-04-01 | Göteborgs kommun  | 103.28420344399824
|2019-05-01 | Malmö kommun      |                100
|2019-06-01 | Malmö kommun      |  98.21052267937955
|2019-07-01 | Malmö kommun      |  99.10180675026773
|2019-08-01 | Malmö kommun      | 104.13859812761254
|...|...|...

To get to the final result, we want to distribute the municipalities (kommuns) to separate columns, i.e. pivot on the kommun column. This functionality is available in the [`table_func` extension](https://www.postgresql.org/docs/12/tablefunc.html) of PostgreSQL by means of the `crosstab` function. The full query above (Query 2) needs to be passed as a string to this function, with the addition of the `ORDER` keyword to ensure that the rows are ordered correctly.

#### Query 3 - Make a pivot table

{% highlight sql %}
SELECT *                                                                             
FROM crosstab('SELECT month,
               kommun,
               100*median/first_value(median) OVER (PARTITION BY kommun
                                                    ORDER BY month) AS norm_price
               FROM (SELECT date_trunc(''month'', sold_date)::date AS month,
                            kommun,
                            percentile_disc(0.5) WITHIN GROUP (ORDER BY price_per_area) AS median
                     FROM sold_properties
                     WHERE property_type LIKE ''Bostadsr%'' AND
                           sold_date BETWEEN ''2019-05-01'' AND ''2020-04-30'' AND
                           kommun IN (SELECT kommun
                                      FROM sold_properties
                                      WHERE property_type LIKE ''Bostadsr%'' AND
                                            sold_date BETWEEN ''2019-05-01'' AND ''2020-04-30''
                                      GROUP BY kommun
                                      HAVING count(*) > 2000
                                      )
                     GROUP BY 1,2
                    )
                    AS monthly_price
               ORDER BY 1,2',
               $$VALUES('Stockholms kommun'::text),
                       ('Göteborgs kommun'::text),
                       ('Malmö kommun'::text),
                       ('Uppsala kommun'::text),
                       ('Solna kommun'::text)
               $$                     
              )                       
AS ct("month" date,
      "Stockholm" numeric(5,1),
      "Göteborg" numeric(5,1),
      "Malmö" numeric(5,1),
      "Uppsala" numeric(5,1),
      "Solna" numeric(5,1)
    )
;
{% endhighlight %}

#### Results

|month    | Stockholm | Göteborg | Malmö | Uppsala | Solna
|------------+-----------+----------+-------+---------+-------
|2019-05-01 |     100.0 |    100.0 | 100.0 |   100.0 | 100.0
|2019-06-01 |     102.4 |     97.1 |  98.2 |    95.7 |  97.9
|2019-07-01 |      98.1 |    101.8 |  99.1 |   112.1 | 101.6
|2019-08-01 |     102.7 |    105.1 | 104.1 |   112.4 | 103.3
|2019-09-01 |     106.9 |    102.6 | 104.6 |   104.6 |  99.3
|2019-10-01 |     105.9 |    103.1 | 101.0 |   102.0 | 100.3
|2019-11-01 |     105.9 |    102.3 | 100.0 |    99.5 | 102.0
|2019-12-01 |     108.4 |    100.7 |  93.9 |   108.3 | 103.8
|2020-01-01 |     109.5 |    105.1 | 103.6 |   107.9 | 106.1
|2020-02-01 |     110.5 |    103.5 | 109.5 |   105.7 | 105.8
|2020-03-01 |     108.5 |    102.3 |  97.2 |    99.6 | 100.0
|2020-04-01 |     102.2 |    103.3 | 100.1 |    98.7 | 100.0

And voilà, we have reached the desired result.

In the next post we'll <sub><sup>probably</sup></sub> dive more into the geo-spatial information available and ways to visualize the housing price trends in QGIS.
