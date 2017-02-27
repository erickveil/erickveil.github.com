---
layout: post
title: How to Perform Some More Complex SQL Queries
date: 17-02-27 08:20:00 -0800
categories: SQL, SQLite
---

Building all complex queries is a matter of stacking simple queries together in a legal way.
(Note that for the purposes of this article, I will be using SQLite flavored queries.)

Say we have 2 tables:

- alerts
- district

Alert table has the fields:
id, type, district_id

District table has the fields:
id, name

Some simple, common queries to start with would be:

**We want all the alerts in district id 2:**


    Select * from alerts 
    where district_id=2;


**List of all district id's with medical alerts:**


    Select distinct district_id from alerts 
    where type="medical";



## List District Names That Have Medical Alerts:

This type of query will require information from two tables. This is a clue that we might need a `join` clause. We will use a left join, here, as SQLite does not support right joins. Either way, I've found that anything expressed as a right join can also be expressed as a left join in reverse.

    Select distinct name from alerts A 
    left join district B 
    on B.id=A.district_id;


My first thought was to select from the districts table, since "name" is a column in that table. Unfortunately I was wrong. It gets us all district names because we are selecting from districts. So even if, say, district id 4 is never used, we will still get district 4 from the name field. This, by the way, is the same as performing a right join on the previous query.

    /* Wrong */
    Select distinct name from districts A 
    left join alerts B 
    on A.id = B.district_id;

This is why it is important that the first table for the select statement with a join define which table is the limiting data set. In this case, `district` has ALL the district names, while `alerts` LIMITS to the districts that are alerted.


    Select distinct <data source field> from <limiting table> <limiting table variable> 
    left join <data source table> <data source variable> 
    on <data source variable>.<reference field to data relative to limiting field>=<limiting table variable>.<limiting field>;


## List All of the Alert Types Found in District "City Center"

This requirement sounds complex at first, but by using a subquery, we find that it is actually rather simple.

First, since we were smart enough to set up our normalized relational tables with matching IDs, let's figure out the ID for the "City Center":

    select id from districts 
    where name="City Center"

Then we want to write part of a query for what we think we want:

    Select distinct type from alerts 
    where district_id ???;

We can now see that the first query will be a subquery for the second, using the `in` clause:
    

    Select distinct type from alerts 
    where district_id in (
        select id from districts 
        where name="City Center"
    );


## List the Number of "fire" Type Alerts in District ID 2:

This simple query obviously uses the `and` keyword, and sets us up for the next query, which will be more complex:

    select count(*) from alerts 
    where type="fire" and district_id=2;


## List the Number of "medical" Type Alerts in "City Center":

It's actually rare that the provided data will include IDs. If the user of your CRUD application is entering strings for searches, it's going to force you to do the lookup yourself. In this case, you will almost always be replacing the ID of a simple query with a subquery that gets the ID for you.

    select count(*) from alerts 
    where type="medical" and district_id=(
        select id from district 
        where name="City Center"
    );


## Get the District Name With the Most Alerts.

Now this one is rather complex.

First, we want to figure out which district_id has the most alerts, as that will narrow our query down by one table and simplify the problem. However, the issue is still complex.

Start by getting all of the district_ids:


    select district_id from alerts;


We want a count of each of the id's, which suggests the use of the count function: `select district_id from alerts order by count(*);` However, this will give us an error. We cannot use an aggregate function to tease apart values this way. If we try to use `group by count(*)` we will also get an error, because aggregates are not allowed in the group by clause.

We want to count each individual district_id, so we will need a list of these ids individually.

We get this list by grouping by district_id. This will return one record per unique district_id. This list will be in the order they were inserted, and the returned record for each value will be the last one inserted for that value.

To get a better idea of what records are being returned, replace `select district_id` with `select *`.

    select district_id from alerts 
    group by district_id;


Now, we want to order by the count of each id. This will return the same list of records, but sorted from the least occurring number of ids to the most. This time, we've first grouped the id list, and so the `order by count(*)` is applied to that instead of the whole list of repeated IDs, making it legal when used together.

Again, you can get a clearer idea of the records by replacing the select argument with the wildcard.

    select district_id from alerts
    group by district_id
    order by count(*);


Next, to isolate the highest counted id, we limit the responses to just 1 record. However, the limit function returns just the first entry, while we want the last. So we reverse the order of the list with `desc`.


    select district_id from alerts
    group by district_id
    order by count(*)
    desc
    limit 1;


Now we have the id of the district that has had the most alerts. But the user doesn't know IDs. The user only know human readable names. So this is what we want to return. We know that it is easy to get the value by id in a table with the simple query using `select name from district where id=<ID>;` To do this all in one query, we simply replace `<ID>` with our sub-query:


    select name from district where id=(
        select district_id from alerts
        group by district_id
        order by count(*)
        desc
        limit 1
    );


And that will return the name of the district with the most alerts. Going step by step through the query in this way is the easiest way to reason it out. Unfortunately, glancing at the query during maintenance, five years from now, it will not be clear what it is supposed to accomplish without using a little brain power to tease the query apart.

Make sure you `/* comment */` all of your queries. Lazy developers of the future (a.k.a future you) will thank you.

## Get the District Name With the Most Fire Alerts

Having done all that work, we can extrapolate further to build off of it.

Remember that while we were selecting the district_id from the alerts table using a variety of refinements, we were always able to change the response of what we returned. We were returning district_id, but when we wanted to be clear about exactly what line that ID came from, we changed `select district_id` to `select *`.

Likewise, we can refine our primary query by the type of alert record using our basic `where` clause. `select district_id from alerts where type="fire";` limits the response to just the fire type alerts. By meshing this idea together with the previous one, we can get the value we are looking for here.

But where do we put this `where` clause in our query to create this refinement?

Consider:

1. The filter belongs to the query we are doing on the alerts table, because it is a filter on the "type" column. So we know it is going to go in the subclause inside the parentheses.
2. We know the rule about the `group by` clause (because we read the documentation for every clause before we used it, of course) that it must come after the `where` clause in a query and before an `order by` clause.

That pretty much narrows it down. We can now query for the most fire alerts by simply inserting the new clause in the appropriate place:

    select name from district where id=(
        select district_id from alerts
        where type="fire"
        group by district_id
        order by count(*)
        desc
        limit 1
    );

