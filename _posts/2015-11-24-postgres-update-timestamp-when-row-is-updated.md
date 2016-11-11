---
layout: post
title: PostgreSQL - update timestamp when row(s) is updated
tags:
- PostgreSQL
- Hacker
- Data
---

In PostgreSQL, if you want to set current timestamp as default value, you can simply keep a column's default expression as `now()`. However, by default there is no function defined to update the timestamp when a particular row (or multiple rows) need to be updated.

In such scenario, you may create your custom method and trigger it using <b>PostgreSQL's Triggers</b>. Following snippet will make it more clear:

Here, we are creating a new method, `method_get_updated_at()`

{% highlight sql %}
CREATE OR REPLACE FUNCTION method_get_updated_at() RETURNS TRIGGER
LANGUAGE plpgsql
AS $$
    BEGIN
      NEW.<column_name> = now();
      RETURN NEW;
    END;
$$;
{% endhighlight %}

Once it is created, use the following snippet to trigger it:

{% highlight sql %}
CREATE TRIGGER trigger_<column_name>
BEFORE UPDATE ON <table_name>
FOR EACH ROW
EXECUTE PROCEDURE method_get_updated_at();
{% endhighlight %}

If you want to delete a Trigger, you can use this simple query:

{% highlight sql %}
DROP TRIGGER IF EXISTS trigger_<column_name> ON <table_name>
{% endhighlight %}

<b>Note:</b> Please update the <table_name> and <column_name> accordingly and execute the code for your particular database. Also, note that, some web frameworks (like Rails) manage such columns(created_at, updated_at) automatically.

ALso, if you want to view all existing methods, run this query:
{% highlight sql %}
SELECT  p.proname
FROM    pg_catalog.pg_namespace n
JOIN    pg_catalog.pg_proc p
ON      p.pronamespace = n.oid
WHERE   n.nspname = 'public'
{% endhighlight %}

And, run this query to view all Triggers:
{% highlight sql %}
SELECT * FROM pg_trigger;
{% endhighlight %}

Thanks!
