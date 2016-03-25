---
title: Filtering duplicates by most recent in PostgreSQL
updated: 2016-03-24
---

## Filtering Duplicates By Most Recent in PostgreSQL

Let's say you have a `user` table that doesn't enforce `unique` emails (common in data warehousing).  There's a possibility you'll have multiple rows containing the same email.  Typically, you'll want to retrieve the most recent.  Here's a little recipe using Postgres [window functions](http://www.postgresql.org/docs/9.1/static/tutorial-window.html):

```
SELECT * FROM
  (SELECT email, created_at, RANK()
                             OVER (PARTITION BY email ORDER BY created_at DESC) AS rank1
  FROM users) AS dups
WHERE rank1 = 1;
```

