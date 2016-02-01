---
layout: post
title:  Bulk change table owner in postgres
date:   2015-09-29 00:00:00
summary : Bulk change table owner in #postgres
tags: postgres dba sql 
featured: false
---

#Bulk change owner tables in multiple schema postgres

todo description

{% highlight sql %}
---------------------------------------------------------------------------------
--- Bulk alter table owner for all tables in multiple schema
--- to be run into pgAdmin "Execute pgScript"
---------------------------------------------------------------------------------
--- hint: replace MY_USER with your actual user

SET @elements = select schemaname, tablename from pg_tables where schemaname in ('public','test','whatever');
SET @i = 0;
WHILE @i < LINES(@elements)
BEGIN
    SET @T = @elements[@i][0] + '.' + @elements[@i][1];
    ALTER TABLE @T OWNER TO MY_USER;
    SET @i = @i +1;
END
{% endhighlight %}