---
layout: post
title:  An uniform way to get the first point from geometries of different type
date:   2016-05-15 00:00:00
summary : An effective way to extract the firs point foma a postgis geometry regardless of the geometry type
categories: postgis
tags: postgis postgres sql
featured: false
---

Is it difficult to extract the first point from a postgis geometry?

Have you ever try to get if from geometry of different types?
The standard approach will be to use the most suitable postgis function for every geometry type:

| Geomtry Type | How to get first point |
| POINT        | the point itself |
| MULTIPOINT   | the first part of the geometry |
| LINESTRING   | use ST_StartingPoint   |
| MULTILINESTRING | get the first part and then use ST_StartingPoint |
| POLYGON        | use ST_DumpPoints and filter the first |
| MULTIPOLYGON   | use ST_DumpPoints and filter the first |
| GEOMETRYCOLLECTION | use ST_DumpPoints and filter the first |
{: .table-bordered}

But really? In my opinion putting everything together in a single query will be extremely cumbersome.

So after some time spent to find an elegant approach to the problem I gave up, and I build an ugly but effective query:

{% highlight sql %}
DROP FUNCTION IF EXISTS get_first_point(geometry);
CREATE OR REPLACE FUNCTION get_first_point(geom geometry)
RETURNS geometry AS
$BODY$
DECLARE
    retval geometry;
BEGIN

select ST_PointFromText('POINT(' || replace(
    reverse(
        split_part(
            reverse(
                split_part(
                    split_part(
                        left(
                            st_asewkt(geom),100
                        ),',',1
                    ),';',2
                )
            ), '(',1 )
        ) ,')',''
    )||')',ST_SRID(geom)
) into retval;
return retval;
END;
$BODY$
LANGUAGE plpgsql IMMUTABLE COST 1;
{% endhighlight %}

This works quite well in my reference environment.

Have you found a better approach? I'm open to your suggestions.
