---
layout: post
title: Save memory with cursors
tags: PostgreSQL
---

Has your application ever run out of memory when running a query that returns a
a significant amount of data?

For example, in PHP, I have seen a script like the following die because it ate
up too much memory on my application server.

```php
$sql = <<<SQL
SELECT *
FROM really_large_table
SQL;

$res = pg_query($sql);

// Never gets here because pg_query() eats up all of the memory on the
// application server
while ($row = pg_fetch_row($res)) {
  // Process $row...
}
```

So how can you reduce your application's memory usage when querying a large
amount of data from Postgres?

Well, imagine you and me are packing up my book collection which consists of
hundreds of titles. Because you are such a good friend, you hold up a box and
ask "Can I get all the books?" I begin to fill up the box as fast as I can
shoveling all of the books in at the same time. With this approach, the box
would quickly become overflowed and dangerously heavy and you would most likely
drop it.

Knowing this, it would make more sense for you to ask me to fill our box with a
few books at a time until it reached a reasonable weight. At this point, you
would stop asking for more books, retrieve another box, then ask for a few more
books. We would continue this process until we'd happily packed all of the books
in my collection.

![merlin packing books](merlin.gif)

Querying your data in a memory efficient way from Postgres works similarly to
this book packing scenario. Instead of requesting all the data (e.g., books) at
once, we can request the data back from Postgres in smaller **chunks** via a
cursor. Take a look at the following example to see how cursors work.

```sql
-- Cursors can only be declared within a transaction
BEGIN;

-- Initialize the cursor, passing the query you want to run
DECLARE my_cursor CURSOR FOR (
  SELECT *
  FROM really_large_table
);

-- Fetch the results, 100 at a time in this case
FETCH 100 FROM my_cursor;
FETCH 100 FROM my_cursor;

-- Clean up
CLOSE my_cursor;
```

Let me explain what is happening in this example. The first thing we do is
create our cursor and give it a name "my_cursor". At that point, Postgres runs
our query and holds the results on the Postgres server. Once we are ready, we
can request the results in chunks using a FETCH query with the number of rows
we want (100) and the name of our cursor ("my_cursor").

Now let's rewrite our initial example using a cursor to reduce the memory usage
and fix that pesky error.

```php
pg_query("BEGIN");

$sql = <<<SQL
SELECT *
FROM really_large_table
SQL;

pg_query("DECLARE my_cursor CURSOR FOR ({$sql})");

do {
    $res = pg_query("FETCH 100 FROM my_cursor");
    while ($row = pg_fetch_row($res)) {
      // Process $row...
    }
} while (pg_num_rows($res) > 0);

pg_query("CLOSE my_cursor");
```

The first thing you might notice is that this is not as simple as the original
example. In a case like this, it can be worth it to tradeoff some simplicity for
the sake of optimization. Remember,

> "Everything should be made as simple as possible, but not simpler" - Albert Einstein

I hope this gave you a good introduction to cursors and gave you a good solution
to a common problem.

**Links**
- Check out the PostgreSQL
[documentation](https://www.postgresql.org/docs/current/static/sql-declare.html)
for more information on cursors.
- Do you feel like a [Wizard](https://www.youtube.com/watch?v=7bd5YUEOwlE)?
