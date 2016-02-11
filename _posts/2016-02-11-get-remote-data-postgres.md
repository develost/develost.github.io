---
layout: post
title:  Utility function to query data from remote postgres
date:   2016-02-11 00:00:00
summary : Utility function to query data from remote postgres, wrapping dblink
tags: postgres dba sql dblink
featured: false
---

If you find quite cumbersome the syntax to be used for dblink in postgres
and ors you cannot use the [postgres_fdw](http://www.postgresql.org/docs/9.3/static/postgres-fdw.html), introduced in version 9.3, make use of
this function.

This is the execution flow implemented

1. Connect to a remote postgres, query the information schema to get attribute types
1. Compose the remote query
1. Run the query on the remote database
1. Return the query to be run (as text)

Call this function in this way:

{% highlight sql %}
manager.get_remote_select(
    'dbname=planetosm port=5432 host=localhost user=postgres password=secret',
    'public',
    'planet_osm_polygon',
    '{name,osm_id,admin_level,tags,way}',
    'where boundary = ''''administrative'''''
);
{% endhighlight %}

The where_clause parameter allows the remote query to be called with where clause, incrementig speed.

This is the function definition:

{% highlight sql %}
DROP FUNCTION IF EXISTS manager.get_remote_select(text, text, text, text[], text);
CREATE OR REPLACE FUNCTION manager.get_remote_select(dblink_name text,remote_schema_name text,remote_table_name text, remote_attribute_names text[], where_clause text)
    RETURNS text AS
$BODY$
    
DECLARE
    information_schema_query text;
    information_schema_record record;
    remote_attribute_name text;
    remote_attribute_types text[] := '{}';
    i int;
    remote_query text;
    
BEGIN
    FOR i IN array_lower(remote_attribute_names,1) .. array_upper(remote_attribute_names,1)
    LOOP
        remote_attribute_types := array_append(remote_attribute_types, 'ERROR_COLUMN_NOT_FOUND');
    END LOOP;    

    --RAISE NOTICE 'remote_attribute_types: %' , remote_attribute_types ;
    
    information_schema_query := '';
    information_schema_query := information_schema_query || 'SELECT * FROM dblink(''';
    information_schema_query := information_schema_query || dblink_name;
    information_schema_query := information_schema_query || ''',''';
    information_schema_query := information_schema_query || 'select table_schema,table_name,column_name,ordinal_position,data_type,udt_name,character_maximum_length,numeric_precision,numeric_scale ';
    information_schema_query := information_schema_query || 'from information_schema.columns where table_schema = ''''';
    information_schema_query := information_schema_query || remote_schema_name;
    information_schema_query := information_schema_query || ''''' and table_name = ''''';
    information_schema_query := information_schema_query || remote_table_name;
    information_schema_query := information_schema_query || ''''' order by ordinal_position';    
    information_schema_query := information_schema_query || ''') AS t1 (table_schema text, table_name text, column_name text, ordinal_position int,data_type text,udt_name text,character_maximum_length int,numeric_precision int,numeric_scale int)';
    --RAISE NOTICE 'information_schema_query: %' , information_schema_query ;
    
    FOR information_schema_record IN EXECUTE information_schema_query 
    LOOP
        FOR i IN array_lower(remote_attribute_names,1) .. array_upper(remote_attribute_names,1)
        LOOP
            remote_attribute_name := remote_attribute_names[i];
            IF remote_attribute_name = information_schema_record.column_name THEN
                CASE information_schema_record.data_type
                    WHEN 'character varying','varchar' THEN
                        remote_attribute_types[i] := information_schema_record.data_type;
                        remote_attribute_types[i] := remote_attribute_types[i] || '(' || information_schema_record.character_maximum_length || ')';
                    WHEN 'timestamp without time zone','bigint','double precision' THEN
                        remote_attribute_types[i] :=  information_schema_record.data_type;
                    WHEN 'numeric' THEN
                        remote_attribute_types[i] := information_schema_record.data_type;
                        remote_attribute_types[i] := remote_attribute_types[i] || '(' || information_schema_record.numeric_precision || ',' || information_schema_record.numeric_scale || ')';
                    WHEN 'USER-DEFINED' THEN
                        IF information_schema_record.udt_name = 'geometry' OR information_schema_record.udt_name = 'hstore' THEN
                            remote_attribute_types[i] := information_schema_record.udt_name;
                        ELSE
                            remote_attribute_types[i] := 'text';
                        END IF;
                    ELSE
                        remote_attribute_types[i] := information_schema_record.data_type;
                    END CASE;
            END IF;
        END LOOP;
    END LOOP;

    -- RAISE NOTICE 'remote_attribute_types: %' , remote_attribute_types ;
    
    remote_query := '';
    remote_query := remote_query || 'SELECT * FROM dblink(''';
    remote_query := remote_query || dblink_name;
    remote_query := remote_query || ''',''';
    remote_query := remote_query || 'SELECT ';
    FOR i IN array_lower(remote_attribute_names,1) .. array_upper(remote_attribute_names,1)
    LOOP
        remote_query := remote_query || '"' || remote_attribute_names[i] || '"' || ',';
    END LOOP;
    remote_query := trim(trailing ',' from remote_query);
    remote_query := remote_query || ' FROM ' || remote_schema_name || '.' || remote_table_name;
    IF where_clause <> '' THEN
        remote_query := remote_query || ' ' || where_clause;
    END IF;
    remote_query := remote_query || ''') AS t1 (';
    FOR i IN array_lower(remote_attribute_names,1) .. array_upper(remote_attribute_names,1)
    LOOP
        remote_query := remote_query || '"' || remote_attribute_names[i] || '"' || ' ' || remote_attribute_types[i] || ',';
    END LOOP;
    remote_query := trim(trailing ',' from remote_query);
    remote_query := remote_query || ')';
    
    --RAISE NOTICE 'remote_query: %' , remote_query ;
    RETURN remote_query;
END;
$BODY$
LANGUAGE plpgsql VOLATILE COST 100;
{% endhighlight %}

This function is part of the functions I use for the project OpenStreetMap for the dyslexic. Check it on [github](https://github.com/osm-for-the-dyslexic/data_management)

