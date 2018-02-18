---
layout: post
comments: true
title:  "Correlated to Complete: A SQL Subquery Refactor"
date:   2018-02-17 09:29:00 -0500
categories: sql
---
I recently constructed a SQL query that preformed rather poorly.  The query
involved selecting about 10,000 records with a complicted subquery.  It was
noticeably slow and needed to be improved. It was easy to see that the subquery
was running for each record, like a loop. This is known as a correlated subquery.
The refactor involved removing the loop and executing the subquery just once.
Here's how it went down:

## Correlated Subquery
A correlated subquery is like a loop.  It will executed once for each row in the
outer query.  Thus, correlated subqueries can be costly.  Here is a somewhat
contrived example using Postgres:

{% highlight sql %}
SELECT name,
       id,
       (SELECT AVG((relative_velocity->> 'miles_per_hour')::Numeric)
        FROM close_approaches
        WHERE id = close_approaches.asteroid_id) AS avg_velocity
FROM asteroids
ORDER BY avg_velocity DESC;
{% endhighlight %}

Here, we want the average `relative_velocity` of the close approaches for each
asteroid.  The `avg_velocity` subquery is correlated since it runs for each
selected asteroid in the outer query.

If you're wondering about the `relative_velocity->> 'miles_per_hour'` syntax,
that's just PostgreSQL [JSON operator](https://www.postgresql.org/docs/current/static/functions-json.html)
for selecting the `miles_per_hour` key from the `relative_velocity` JSON field.

To demonstrate the expense of such a query, for let's say 10,000 asteroids
each with several close approaches, we can use the `EXPLAIN` function on the
query.  Using `EXPLAIN` will give us a ton of information about the full
execution plan, but we're just interested in the `Total Cost` for this
demonstration. We'll use the `Total Cost` as a benchmark comparison when
optimizing to use a complete subquery below.

By the way, I am using `EXPLAIN (FORMAT JSON)` because I find the format a bit
easier to read. Here are the results for the above query:

{% highlight json %}
 [
   {
     "Plan": {
       "Node Type": "Sort",
       "Parallel Aware": false,
       "Startup Cost": 236908.35,
       "Total Cost": 236933.34,
       "Plan Rows": 9993,
       "Plan Width": 61,
       "Sort Key": ["((SubPlan 1)) DESC"],
       "Plans": [
       ...
{% endhighlight %}

There's a lot more to this, but I'm only showing the first part with
the `Total Cost`, which is `236933.34`.  Let's see how much lower our cost can
be when we refactor this query to get rid of the correlated subquery in favor
of a complete subquery.

## Complete Subquery

A complete subquery (also just known as a subquery), can manifest itself as a
`JOIN`.  We'll take this `JOIN` approach and refactor our query to join
asteroids to the result of a subquery like this:

{% highlight sql %}
SELECT name,
       id,
       avg_velocity
FROM asteroids
     JOIN (SELECT asteroid_id,
                  (SELECT AVG((relative_velocity->> 'miles_per_hour')::Numeric)) AS avg_velocity
           FROM close_approaches
           GROUP BY asteroid_id) AS avg_velocities
     ON avg_velocities.asteroid_id = asteroids.id
ORDER BY avg_velocity DESC;
{% endhighlight %}

Here, in addition to the `avg_velocity` computation, our subquery also selects
the `asteroid_id` from `close_approaches`, since we use it to join back to
asteroids and group the average velocities per asteroid. The important thing here
is that the subquery runs only once, rather than for each asteroid like the first
example.  All we need to do is join the `asteroids` to the result of the subquery.

Overall, re-writing the query this way makes it a bit more verbose and complex,
but lets see how it performs:

`EXPLAIN (FORMAT JSON)` ...

{% highlight json %}
 [
   {
     "Plan": {
       "Node Type": "Sort",
       "Parallel Aware": false,
       "Startup Cost": 8575.66,
       "Total Cost": 8600.67,
       "Plan Rows": 10007,
       "Plan Width": 61,
       "Sort Key": ["((SubPlan 1)) DESC"],
       "Plans": [
       ...
{% endhighlight %}

In the first example the `Total Cost` was `236933.34`.  After refactoring to a
complete subquery we're down to a `Total Cost` of `8600.67`. That's over 27
times cheaper! 🙂

# Conclusion

Correlated subqueries aren't always bad.  Sometimes they may perform well enough.
But if you're finding it too slow, look to refactoring the query to use a
single subquery on which to join.  Remember to use `EXPLAIN` for comparison
bench-marking.

# Resources
- [Correlated Subquery on Wikipedia](https://en.wikipedia.org/wiki/Correlated_subquery)
- [PostgreSQL EXPLAIN Docs](https://www.postgresql.org/docs/current/static/sql-explain.html)
