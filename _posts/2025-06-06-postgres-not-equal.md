---
title: 'Postgres Not Equal Only Returns Non-Null Values'
date: 2025-06-06
permalink: /posts/postgres-not-equal-only-returns-non-null-values
tags:
  - databases
---
As part of an analysis I recently ran a PostgreSQL query to return results filtered to remove rows in which the status column contained values not equal to "started". However after running the below query using the <> operator I noticed it returned far less rows than expected.

```sql
SELECT publication_id, start_date, department, pub_status
FROM pub_schema.publications
WHERE pub_status <> 'Started';
```

When I investigated this was due to the pub_status column containing mostly Nulls, which were not being returned in the query results. With only rows containing a non-Null status value other than 'Started' being returned.

As mentioned in the Postgres [functions comparison documentation](https://www.postgresql.org/docs/current/functions-comparison.html) this is due to the not equal to operator (<> or !=) treating Nulls as non-comparable and thereby ignoring and excluding them from the query. This is a common pitfall across most databases as following the SQL Standard if any value on either side of an operator is a Null the equation should evaluate to false, including Null = Null.

This is actually quite logical with the value of Null being by definition unknown, this makes an equivalence with a null value unknowable, as explained in this quote from the [Wikipedia page](https://en.wikipedia.org/wiki/Null_(SQL)) on Nulls in SQL:

**A null should not be confused with a value of 0. A null indicates a lack of a value, which is not the same as a zero value. For example, consider the question "How many books does Adam own?" The answer may be "zero" (we know that he owns none) or "null" (we do not know how many he owns).**

To include rows in the result where status can also be Null, I ended up using the IS DISTINCT FROM predicate (below). This predicate treats Nulls as comparable values rather than unknowns. Meaning it will return Nulls alongside with any other rows that do not equal 'Started' (or any other specified criteria).

```sql
SELECT publication_id, start_date, department, pub_status
FROM pub_schema.publications
WHERE pub_status IS DISTINCT FROM 'Started';
```
