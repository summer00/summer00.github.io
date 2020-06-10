---
title: "IT konekt 2019 Note"
date: 2020-05-23 00:30:00 +0800
categories: [video, English]
---

# Clean Architecture

[Video](https://www.youtube.com/watch?v=2dKZ-dWaCiU&t=1101s)

> Not too much detail too early.

> A good architecture allows major decisions to be DEFERRED!

> A good architecture maximizes the number of decisions NOT made.

> You only test the parts of the application that you want to work.

> Things hard to test:<br>
>
> 1. Anything at outer boundary of the system, like GUI
> 2. It's very difficult to test things if you don't know the answer

# Top 10 SQL Performance Tips & Tricks for Java Developers

[Video](https://www.youtube.com/watch?v=4yXRZa_yq0A)
[Slide](https://www.slideshare.net/gvenzl/top-10-sql-performance-tips-tricks-for-java-developers)

## Round trips

- Avoid unnecessary round trips to the database
- Every round trip includes time that you spend on the network
- That does **not** mean to avoid processing anything at the database

## Auto commit

- The driver commits every DML on your behalf, implicitly
- That is, for every single `insert, update, delete, merge` will be a commit
- Implies a second round trip to the databases for every DML operation
- Forces database to write to disk
- You can never rollback your changes
- It's default to **turn on**, turn off it

## Bulk processing

- Database work best with set based process
- Process as much as possible on the side where it makes most sense
- Generally, a single SQL statement execution is the fastest
- Avoids unnecessary round trips

## Bind variables

- A bind variable allows you to bind data to an explicit data type
  - Without having to convert the data type form string
  - Without having to parse the SQL statement again
- Avoids SQL injections

## Fetch size

- Drive fetches more than on row from the database at once
- Avoids unnecessary round trips

## Save point

## KISS (Keep it simple SQL)

- Easier to maintain
- Faster to optimize
- Less likely for the optimizer to get it wrong

## Parallel queries

```sql
SELECT /*+ PARALLEL */ id, create_date, first_name...
```

## Explain plans

## SQL wait events (Oracle only)