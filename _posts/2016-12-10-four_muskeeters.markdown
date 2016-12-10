---
layout: post
title:  "Four Muskeeters:"
date:   2016-12-09 20:36:49 -0500
---

## includes, joins, preload & eager_load

Rails uses four methods to load associated object data from ActiveRecord.  Using the Models-Class-Methods-Lab for my examples, hopefully we'll be able to figure out what each one does (me included!)

1. `preload`:  Fires two SQL (SELECT) rockets to retrieve associated object data  Runs a query for the first data object table and then runs a second query to find associated data from a second data object table (with attributes like :id of the original table).  This makes it unable to use `where` or `order` on attributes from the second table.

2. `includes`: Also fires two SQL rockets to retrieve associated object data but wont stop at `where` or `order` methods.   It also loads the object into memory! so you dont have to make seperate SQL queries to load associated data of the initial object.  but that's not within the domain of this post.

3. `eager_load`: Will issue only one query rocket with a 'left outer join'.  No smart functionality like with `includes` but gets the job done when you need those attributes from a second data table.
 
4. `joins`:  Also fires one query rocket to retrieve associated data with an `inner join`. It selects only the first table and uses an `inner join` to query the second table for associated data. In this case its able to grab attributes from the second table using a single line of query.

**Using the following method:**

```
  def self.sailboats
    sub_query_method(:classifications).where("classification.name = ?", "Sailboat")
  end
```

`preload` in place of sub_query_method returns the following error:

`ActiveRecord::StatementInvalid: SQLite3::SQLException: no such column: classification.name: 

SELECT "boats"."name" FROM "boats" WHERE (classification.name = 'Sailboat')`

`preload` doesn't even load the second table "classifications" in this case because `where` is asking for an attribute from  the table not selected.  had the method block just been `preload(:classifications)` it woulds have SELECT "boats" and SELECT "classifications" in the SQL statements, however this still leaves you the problem of grabbing those attributes from the second selected table

`includes`  returns a similar error as `prelude`:

```
ActiveRecord::StatementInvalid: SQLite3::SQLException: no such column: classification.name: 

SELECT "boats". FROM "boats" WHERE (classification.name = 'Sailboat')
```

I'm pretty sure `include` gave up on it's way to the marina.  It seems smart enough to realize that two SELECT tables or `inner join` just wont work to get that classsification.name data and just gave up.

`eager_load ` used in the above query returns the following error:

` ActiveRecord::StatementInvalid: SQLite3::SQLException: no such column: classification.name: 

SELECT "boats"."id" AS t0_r0, "boats"."name" AS t0_r1, "boats"."length" AS t0_r2, "boats"."captain_id" AS t0_r3, "boats"."created_at" AS t0_r4, "boats"."updated_at" AS t0_r5, "classifications"."id" AS t1_r0, "classifications"."name" AS t1_r1, "classifications"."created_at" AS t1_r2, "classifications"."updated_at" AS t1_r3 FROM "boats" LEFT OUTER JOIN "boat_classifications" ON "boat_classifications"."boat_id" = "boats"."id" LEFT OUTER JOIN "classifications" ON "classifications"."id" = "boat_classifications"."classification_id" WHERE (classification.name = 'Sailboat')`

Whole lotta `left outer join`(s) for nothing.  Again what attribute are we after and from which data table?

`joins` is the golidlocks of query methods in this case.  it's `inner join` connects the appropriate data table and with its one SELECT susses out that valuable data.

```
SELECT "boats". FROM "boats" INNER JOIN "boatclassifications" ON "boat_classifications"."boat_id" = "
boats"."id" INNER JOIN "classifications" ON "classifications"."id" = "boat_classifications"."classification_id" WHERE (class
ifications.name = 'Sailboat')
```


Lets take a look at methods where `eager_load` and `includes` works:

```
  def self.sailors
    eager_loads(boats: :classifications).where("classifications.name = ?", "Sailboat").uniq
  end
```

With `eager_loads` with get the following sucessful query:

```
SELECT DISTINCT "captains"."id" AS t0_r0, "captains"."name" AS t0_r1, "captains"."admiral" AS t0_r2, "captain
s"."created_at" AS t0_r3, "captains"."updated_at" AS t0_r4, "boats"."id" AS t1_r0, "boats"."name" AS t1_r1, "boats"."length"
 AS t1_r2, "boats"."captain_id" AS t1_r3, "boats"."created_at" AS t1_r4, "boats"."updated_at" AS t1_r5, "classifications"."i
d" AS t2_r0, "classifications"."name" AS t2_r1, "classifications"."created_at" AS t2_r2, "classifications"."updated_at" AS t
2_r3 FROM "captains" LEFT OUTER JOIN "boats" ON "boats"."captain_id" = "captains"."id" LEFT OUTER JOIN "boat_classifications" ON "boat_classifications"."boat_id" = "boats"."id" LEFT OUTER JOIN "classifications" ON "classifications"."id" = "boat_classifications"."classification_id" WHERE (classifications.name = 'Sailboat')
```

```
  def self.longest
    includes(:boats).order("boats.length desc").limit(2).uniq
  end
```

```
SELECT  DISTINCT "classifications"."id" FROM "classifications" LEFT OUTER JOIN "boat_classifications" ON "boat_classifications"."classification_id" = "classifications"."id" LEFT OUTER JOIN "boats" ON "boats"."id" = "boat_classificati
ons"."boat_id"  ORDER BY boats.length desc LIMIT 2

SELECT DISTINCT "classifications"."id" AS t0_r0, "classifications"."name" AS t0_r1, "classifications"."create
d_at" AS t0_r2, "classifications"."updated_at" AS t0_r3, "boats"."id" AS t1_r0, "boats"."name" AS t1_r1, "boats"."length" AS
 t1_r2, "boats"."captain_id" AS t1_r3, "boats"."created_at" AS t1_r4, "boats"."updated_at" AS t1_r5 FROM "classifications" L
EFT OUTER JOIN "boat_classifications" ON "boat_classifications"."classification_id" = "classifications"."id" LEFT OUTER JOIN"boats" ON "boats"."id" = "boat_classifications"."boat_id" WHERE "classifications"."id" IN (5, 6)  ORDER BY boats.length desc
```
	
`joins` and `eager_loads` works here as well...but that requires more investigation on my part.


I purposly left out examples of `preload` bacause the other methods seem more flexible for querying data.  maybe it's because I'm a noob but it seems really limited.  It's going to have to sixt back there next to the weird cousin  I don't want to meet until I absolutely have to.

To infinity and beyond...




