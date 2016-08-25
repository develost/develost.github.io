---
layout: post
title:  A report of all the data contained into a postgres schema
date:   2016-05-24 00:00:00
summary : Make use of the information schema catalogue to create a report of a postgres schema
categories: postgres
tags: postgres sql dba report
featured: false
---
Need a simple and effective way read the informative content of a postgres schema?

You can use the following procedure that creates for every table, for every attribute a row.

The row contains:

* table_schema: the name of the schema,  
* table_name: the name of the table,
* table_attribute: the name of the attribute,
* table_attribute_type: the type of the attribute in both declared and internal representation
* record_number: number of the record in the table,
* distinct_values: number of distinct values for the attributes,
* min_value text: minimum value of the attribute (null not counted) ,
* max_value text:maximum value of the attribute (null not counted),
* null_count: number of null occurences for the attributes,
* not_null_count: number of not null occurences for the attributes

{% highlight sql %}
DROP FUNCTION IF EXISTS schema_report(text);
CREATE OR REPLACE FUNCTION schema_report(the_schema text)
  RETURNS TABLE(table_schema text, table_name text, table_attribute text, table_attribute_type text,record_number integer, distinct_values integer, min_value text, max_value text, null_count integer, not_null_count integer) AS
$BODY$
DECLARE
    get_columns_query text;
    get_columns_record record;
    report_query text;

BEGIN
    get_columns_query := '';
    get_columns_query :=  get_columns_query || 'SELECT c.table_schema, c.table_name, c.column_name, c.data_type, c.udt_name ';
    get_columns_query :=  get_columns_query || 'FROM information_schema.tables t LEFT JOIN information_schema.columns c ON t.table_schema = c.table_schema AND t.table_name=c.table_name ';
    get_columns_query :=  get_columns_query || 'WHERE t.table_schema = ''' || the_schema || ''' AND t.table_type = ''BASE TABLE'' ';
    get_columns_query :=  get_columns_query || 'ORDER BY c.table_schema, c.table_name, c.ordinal_position';

    report_query := '';
    RAISE NOTICE 'collecting informations' ;
    FOR get_columns_record IN EXECUTE get_columns_query
    LOOP
        report_query := report_query || 'SELECT ''' || get_columns_record.table_schema || '''::text as table_schema, ';
        report_query := report_query || '''' || get_columns_record.table_name || '''::text as table_name, ' ;
        report_query := report_query || '''' || get_columns_record.column_name || '''::text as table_attribute, ';
        report_query := report_query || '''' || get_columns_record.data_type || '/' || get_columns_record.udt_name ||  '''::text as table_attribute_type, ';
        report_query := report_query || 'count(*)::integer as record_number, ';
        report_query := report_query || 'count(distinct ' || get_columns_record.column_name || ')::integer as distinct_values, ' ;
	if get_columns_record.data_type = 'boolean' then
	    report_query := report_query || 'min(' || get_columns_record.column_name || '::text)::text as min_value, ';
	    report_query := report_query || 'max(' || get_columns_record.column_name || '::text)::text as max_value, ';
	elsif get_columns_record.udt_name = 'hstore' then
	    report_query := report_query || '''hstore not supported''::text as min_value, ';
	    report_query := report_query || '''hstore not supported''::text as max_value, ';
    else
	    report_query := report_query || 'min(' || get_columns_record.column_name || ')::text as min_value, ';
	    report_query := report_query || 'max(' || get_columns_record.column_name || ')::text as max_value, ';
	end if;

        report_query := report_query || 'sum(case when ' || get_columns_record.column_name || ' is null then 1 else 0 end )::integer as null_count, ';
        report_query := report_query || 'sum(case when ' || get_columns_record.column_name || ' is null then 0 else 1 end )::integer as not_null_count ';
        report_query := report_query || 'FROM ' || get_columns_record.table_schema || '.' || get_columns_record.table_name || ' ';
        report_query := report_query || 'UNION ALL ';
    END LOOP;
    report_query := left(report_query,-10);
    RAISE NOTICE 'executing report query' ;
    RETURN QUERY EXECUTE report_query ;

END;
$BODY$
  LANGUAGE plpgsql VOLATILE
  COST 1000
  ROWS 1000;
{% endhighlight %}

{% highlight sql %}
select *
from  schema_report('public');
{% endhighlight %}

The execution of the procedure is a quite long task, it could be reasonable to materialize a view...


Have you found a better approach? I'm open to your suggestions.
