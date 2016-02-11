---
layout: post
title:  Bulk change table owner in postgres
date:   2015-09-29 00:00:00
summary : Bulk change table owner in #postgres
tags: postgres dba sql 
featured: false
---

# How to modify the owner of many tables in a PostgreSQL database?

While there is a command (**REASSIGN OWNED**)[http://www.postgresql.org/docs/current/static/sql-reassign-owned.html] to change the ownership of database objects owned by a database role,
sometimes you need to change the owner of some tables only.

This is the script I run from pgAdmin.

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

