---
layout: post
title: "Deconstructing SQL Queries"
date: 2017-08-19 11:36:19 +0530
comments: true
categories: sql relational-algebra sets
---

Let's look at an average, everyday SQL query:

```sql
SELECT something
FROM table
WHERE conditions
```

Not hard to draw parallels to other known concepts: <!-- more -->

1. FROM table: A set of elements
2. WHERE conditions:  A filter operation on these elements
3. SELECT something: A presenter of an individual element

## Tables are sets

The bare minimum way of interacting with a set is to take a look at all or some of its elements. That is exactly what this query is doing. We're selecting a subset of a set elements based on a few conditions, and representing each element of this subset in some format.  Recall that a subset of a set is _also a set_ in itself.  Which allows us to do something like:

```sql
SELECT something
FROM (
  SELECT something
  FROM table
  WHERE conditions
)
WHERE conditions
```

We have a nested query, where instead of selecting from a table, we're selecting from the result of selecting from a table. [^1]
Sets can have labels or aliases, so:

```sql
SELECT a_name.something
FROM (
  SELECT something
  FROM table
  WHERE conditions
) as a_name
WHERE conditions
```

An assertion is in order: **a select query operates upon a set, and returns a set**.  Naturally, a set can be a that of a single element as well.

```sql
SELECT COUNT(*) FROM table WHERE conditions
--- better yet:
SELECT COUNT(*) as something FROM table WHERE conditions
```

Let's substitute _this set_ in our original query:

```sql
SELECT the same something
FROM
  (SELECT COUNT(*) as something FROM table WHERE conditions)
WHERE conditions
```

Sets can be unioned or intersected:

```sql
  SELECT something
  FROM (a set)
  WHERE conditions
UNION -- or INTERSECTION
  SELECT the same something
  FROM (a similar set)
  WHERE conditions
```

Notice that "something" and "the same something" are important.  We can only union or intersection similar sets.  Apples and oranges can't be unioned in the relational algebra land.

## Joins are Sets
Let's talk about joins.  Chances are, at some point in life you've written in INNER JOIN instead of an OUTER JOIN and got incorrect results.  Or something along those lines.  Joins can be very opaque, even to a regular practitioner.
A few things are important when considering joins:
1. A join is a product of 2 sets. Always. Multi-table joins are just "first join these two", "take the result" and "join the result with the next".
2. Joins are always performed on sets.  So you can "join" any of the above mentioned sets, and you're still good. Do note that language semantics dictate that you use aliases to disambiguate.
3. NULL is always a part of each set.  Implicitly so, for practicality.

This is best explained through an example:

```
numbers = { 1, 2, 3 }
letters = { a, b, c }
```

Joining `numbers` and `letters`

```sql
numbers | letters
--------|---------
NULL    | NULL
NULL    | a
NULL    | b
NULL    | c
1       | NULL
1       | a
1       | b
1       | c
2       | NULL
2       | a
2       | b
2       | c
3       | NULL
3       | a
3       | b
3       | c
```

Do you get the feeling that you are getting more than what you bargained for?  Me too.  Depending on context, we'll want different subsets of this mega joined set.  That's exactly what different kinds of joins are for.  These joins will determine what working set we'll use.

1. Inner join:  Do not consider the entries which have NULL on either side.
2. Left outer join: Do not consider entries which have NULL on the LEFT side.
3. Right outer join: Do not consider entries which have NULL of the RIGHT side.
4. Full outer join: Consider all entries.

For practical reasons, the entry where both sides are NULL is not considered.
A slightly better example:

```
numbers = { [1, a], [2, b], [3, c] }
letters = { [a, x], [b, y], [c, z] }
```

```sql
JOIN
  numbers AND letters
ON second element of number = first element of letter
```

Our working set:

```sql
numbers | letters
--------|---------
NULL    | NULL
NULL    | [a, x]
NULL    | [b, y]
NULL    | [c, z]
[1, a]  | NULL
[1, a]  | [a, x]
[1, a]  | [b, y]
[1, a]  | [c, z]
[2, b]  | NULL
[2, b]  | [a, x]
[2, b]  | [b, y]
[2, b]  | [c, z]
[3, c]  | NULL
[3, c]  | [a, x]
[3, c]  | [b, y]
[3, c]  | [c, z]
```

After applying conditions, and removing both sides NULL entry:

```sql
numbers | letters
--------|---------
NULL    | [a, x]
NULL    | [b, y]
NULL    | [c, z]
[1, a]  | NULL
[1, a]  | [a, x]
[2, b]  | NULL
[2, b]  | [b, y]
[3, c]  | NULL
[3, c]  | [c, z]
```

Rearranging a little for better understanding:

```sql
numbers | letters | included in
--------|---------|-----------
[1, a]  | [a, x]  | Full, Inner, Left and Right
[2, b]  | [b, y]  | Full, Inner, Left and Right
[3, c]  | [c, z]  | Full, Inner, Left and Right
[1, a]  | NULL    | Full and Left
[2, b]  | NULL    | Full and Left
[3, c]  | NULL    | Full and Left
NULL    | [a, x]  | Full and Right
NULL    | [b, y]  | Full and Right
NULL    | [c, z]  | Full and Right
```

## Functions are Sets
That sounded nice, but it's not true. Functions aren't sets, they _operate_ on sets.  Remember, a single value is also a set, so each function takes a set as an argument, and returns a set.

```sql
SELECT anything.today
FROM
( SELECT now() AS today ) AS anything
```

My apologies for dropping a query on you without any domain context, but consider this slightly more complex function, which returns all the sibling branches of a given restaurant branch.  Restaurant has many Restaurant Branches, and Restaurant Branch belongs to a Restaurant, to aid your understanding.

```sql
CREATE OR REPLACE FUNCTION co_branches(branch_id BIGINT)
  RETURNS TABLE(id BIGINT)
AS $function$
SELECT b2.id
FROM
  restaurant_branches b1
  JOIN restaurant_branches b2 ON b1.restaurant_id = b2.restaurant_id
WHERE b1.id = branch_id;
$function$
LANGUAGE SQL;

--- used simply:
SELECT * FROM COBRANCHES(42); -- Co-branches of Branch#42
--- But more powerful, when used like:
SELECT id, (SELECT COUNT(*) FROM COBRANCHES(id)) as branch_count FROM restaurant_branches;

```

Another assertion is in order: **Sets and set operations tend to compose well.**  Functional programming nerds practically live by this motto.  Fundamentally, SQL is not so different.  An important thing to keep in mind that this composing behaviour is mainly about the data and how the data is interpreted. The query language itself leaves a lot to be desired when it comes to composing.  A lot of things like aliases, joins can easily be taken care by a competent library.  More or on this, and the advantages of using something like ARel in a later post.


## Reading is destructuring, Writing is composing

Let's collect all the set-like behaviour we've seen so far:

- We can interact with a set by looking at all or some of its elements
- A subset of a set is _also a set_ in itself
- A set can have a label or an alias
- A set can be a that of a single element as well
- A set can be unioned or intersected with another set

Reading or writing complex queries becomes much easier, if we think of it as composing queries together, or decomposing a large query into smaller parts.

### How to read complex queries

- Start with the innermost, or smallest "SELECT" clauses
- Replace them with an appropriately and descriptively named function, say `co_branches_of_given_branch` instead of just `co_branches`.
	- If these inner queries, now functions, use a column / value from the outer queries, treat them as function arguments. ( `co_branches` used `branch_id` )
- Keep applying this method until you reach the outermost query.

### How to write complex queries
This really boils down to a top-down vs bottom-up approach. If you're a top-down person:
- Write the top-most query, assume all the lower level functions exist, with appropriate and descriptive names.
- Recursively, apply the same strategy to each lower level function

Conversely, if you're a bottom-up person:
- Figure out the lowest level functions / queries you need, and write them
- Build up your larger query by composing these functions.



<!-- Links and footnotes -->

[^1]: Food for thought: How many nested select queries does your favourite relational database allow?
