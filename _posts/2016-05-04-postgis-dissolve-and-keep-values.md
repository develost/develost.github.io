---
layout: post
title:  Postgis dissolve and keep values
date:   2016-05-04 00:00:00
summary : How to dissolve features and keep values
categories: postgis
tags: postgis postgres sql
featured: false
---

How to merge adjacent polygons of the same type keeping attributes?

This is a graphical idea of the problem.

![An idea of the problem]({{ site.url }}/posts-images{{page.url}}.png)

The trick is to save all the grouped values into an array and then extract values from that array.

This is the query I have tested in a very simple case:

{% highlight sql %}
select geom, my_type , 
    case when the_path is not null then values1[the_path] else values1[1] end as value1, 
case when the_path is not null then values2[the_path] else values2[1] end as value2
from (
    select 
        st_asewkt(  (st_dump(st_union(geom))).geom  ) as geom,
        (st_dump(st_union(geom))).path[1] as the_path  ,
        my_type, 
        array_agg(value1) as values1, 
        array_agg(value2) as values2
    from t1
    group by my_type
) tx
{% endhighlight %}

This is teh result you have:

![The result]({{ site.url }}/posts-images{{page.url}}/result_table.png)

This is the set query to run for the setup of this simple test case:

{% highlight sql %}
drop table t1;
create table t1(
    value1 text,
    value2 text,
    my_type text,
    geom geometry(Polygon)
);

insert into t1 (value1,value2,my_type,geom) values ('1-one','2-one','red',ST_GeomFromText('POLYGON((0 0, 0 1, 1 1, 1 0, 0 0))'));
insert into t1 (value1,value2,my_type,geom) values ('1-two','2-two','red',ST_GeomFromText('POLYGON((1 0, 1 1, 2 1, 2 0, 1 0))'));
insert into t1 (value1,value2,my_type,geom) values ('1-three','2-three','blue',ST_GeomFromText('POLYGON((4 0, 4 1, 5 1, 5 0, 4 0))'));
insert into t1 (value1,value2,my_type,geom) values ('1-four','2-four','blue',ST_GeomFromText('POLYGON((7 0, 7 1, 8 1, 8 0, 7 0))'));
{% endhighlight %}

Look at [this](http://stackoverflow.com/questions/36720548/how-to-save-the-information-contains-by-all-the-polygons-after-a-st-union-postg/) discussion on stackoverflow.


