---
layout: post
title: Save memory with cursors
tags: postgresql
---

Has your application ever run out of memory when running a query that returns a
large amount of data?

For example, in PHP, I have seen a script like the following die because it ate
up too much memory on my application server.

```php
$sql = <<<SQL
SELECT *
FROM really_large_table
SQL;

$res = pg_query($sql);
while ($row = pg_fetch_row($res)) {
  // Process $row...
}
```

So how can you reduce your application's memory usage when querying a large
amount of data from Postgres?

Well, your application is running out of memory because you are trying to query
all of the data at once, and your application server cannot (or will not) hold
that many results in memory&mdash;this is to be expected.

Imagine you and I are packing up all of the books at my house (and I have a lot
of books). If you were holding a box and said, "Hey, lets pack up the books, put
them all in here.", and I continued by putting 150 books in your box, you would
eventually drop the box because it would become too heavy.

Instead, you would probably say, "Hey, lets pack up the books. Can you give me 5
at a time?", and once you had about 15 in there you could get another box, then
ask for 5 more, until we had packed all of the boxes.

So instead of asking for all of the books at once, you would ask for the books a
few at a time, or in **chunks**.

Querying your data in a memory efficient way from Postgres works in a similar
way. Instead of asking for all of the data at once, we can request the data back
from Postgres in **chunks** via a `CURSOR`. Take a look at the following example
to see how you can use cursors.

```sql
-- Cursors can only be declared within a transaction
BEGIN;

-- Initialize the cursor, passing the query you want to run
DECLARE my_cursor NO SCROLL CURSOR FOR (
  SELECT *
  FROM reall_large_table
);

-- Fetch the results, 100 at a time in this case
FETCH 100 FROM my_cursor;
FETCH 100 FROM my_cursor;

-- Clean up
CLOSE my_cursor;
```

Now that we see how cursors work, let's rewrite our initial example to use them.

```php
pg_query("BEGIN");

$sql = <<<SQL
SELECT *
FROM really_large_table
SQL;

pg_query("DECLARE my_cursor NO SCROLL CURSOR FOR ({$sql})");

do {
    $res = pg_query("FETCH 100 FROM my_cursor");
    while ($row = pg_fetch_row($res)) {
      // Process $row...
    }
} while (pg_num_rows($res) > 0);

pg_query("CLOSE my_cursor");

// Make sure you check for errors
```

This example will run now, and will only use as much memory as it takes to hold
100 rows of data.

You can read more about cursors in the PostgreSQL [documentation](https://www.postgresql.org/docs/current/static/sql-declare.html).
