---
layout: post
title: "Postgres Text Search: Simple, Adequate"
date: 2017-06-18
comments: true
categories: sql postgres full-text-search how-tos
---

Searching for text within your data is a frequently requested feature, and often leads to excellent UX.  Gmail's web interface is entirely built on top of search. No wonder databases have supported basic text search operators like ~, LIKE, ILIKE etc for a long time.  But they often fall short or give inaccurate results, as we try to evolve the feature.  Say, searching in multiple languages, or searching for different variants of the same word: consider realistically, realistic, and realist, or [searching one word, but not the other][goog-example].

This is where full-text search comes in.  <!-- more --> Postgres ships with excellent full-text search capabilities which allow us to implement text search in our application without incurring additional dependencies and operational overhead.  More over, having a built-in search allows us to compose search with other existing queries and procedures.  When I say excellent capabilities, I mean fully-featured.  And when I say fully-featured, I mean that it supports stemming, search relevance, search highlights, fuzzy matching, and multiple languages.

In this post, we'll look at basic text search.  In particular, we'll look at the fundamental components of a text search, and how to use them.

**Document**

This is the source data within which we intend to search. Typically spread across multiple columns. Now, I know any search worth its salt must be able to search across multiple rows in multiple tables, but we'll come to that bit a little later in the post.  Postgres offers a function `to_tsvector` to obtain a search document from text.  It accepts text as an argument, and returns tsvector of the text.

```sql
    -- Example 1a: tsvector
    =# SELECT
       to_tsvector('With great power, comes great responsibility!')
       AS document;

                    document
    --------------------------------------------
     'come':4 'great':2,5 'power':3 'respons':6
    (1 row)

    -- Example 1b: have strings, will concat
    =# SELECT
       to_tsvector('With great power' || ' ' || 'comes great responsibility!')
       AS document;

                      document
    --------------------------------------------
     'come':4 'great':2,5 'power':3 'respons':6
    (1 row)

    -- Example 1c: have tsvectors, will concat
    =# SELECT
       to_tsvector('With great power')
       || to_tsvector('comes great responsibility!')
       AS document;

                      document
    --------------------------------------------
     'come':4 'great':2,5 'power':3 'respons':6
    (1 row)
```

A few things are noticeable:

1. Punctuation is gone, so is 'with'.  All commonly occurring words that don't have search relevance, such as 'with', are removed from the documents.  These differ from language to language, and are called "stop words".
2. Words are reduced to a base form. These are called as "[lexemes][docs]"; they are nothing but normalised forms of words, a term taken from [linguistics][lexeme-wiki].
3. The lexemes are sorted alphabetically, There are numbers associated with the lexemes.
4. The tsvectors can be concatenated, _or_ can operate on concatenated strings

If you want to dig deeper into how the search actually works, these are some of the things you can read about.  For now, we'll focus on just one thing from this:  the to_tsvector accepted our text input, and returned a searchable value.

As a side note, I really like this examples driven approach to learning.  Code and examples make for easier understanding of the subject, and give the reader a starting point to dig deeper.  In that spirit, this blog post is a repl-driven-blog-post.

**Query**

The term we wish to search for.  Typically a word or a phrase , but can be any text. Even a full document if you will. For obtaining a query object from the given search string, Postgres has two functions: `to_tsquery` and `plainto_tsquery`. `to_tsquery` allows us to use control characters like wildcards, but is a lot stricter with the input. `plainto_tsquery` on the other hand doesn't use control characters, but escapes all the input, and is safe from SQL injection.  In this post we'll only look at to_tsquery since it falls in line with our "fully featured" requirement.


```sql
    -- Example 2: tsquery
    =# SELECT to_tsquery('responsibility');
     to_tsquery
    ------------
     'respons'
    (1 row)

    -- Example 3: tsquery multiple words, with escaping
    =# SELECT to_tsquery('great\ responsibility');
         to_tsquery
    ---------------------
     'great' & 'respons'
    (1 row)

    -- Example 4: Wildcard search
    =# SELECT to_tsquery('Eliz:*');
     to_tsquery
    ------------
     'eliz':*
    (1 row)

    -- Example 5: Intersection (AND-ing)
    =# SELECT to_tsquery('Barry') && to_tsquery('Allen');
         ?column?
    -------------------
     'barri' & 'allen'
    (1 row)

    -- Example 5: Union (OR-ing)
    =# SELECT to_tsquery('Barry') || to_tsquery('Wally');
         ?column?
    -------------------
     'barri' | 'walli'
    (1 row)

```


Examples demonstrate the various usages of to_tsquery.  Wildcards, AND and OR do exactly what you'd expect them to do.  The key takeaway is that, these are just regular functions, like `LOWER`, `LENGTH` and so we can just use them in _any_ query.


```sql

-- Example 6
=# SELECT
   name,
   to_tsvector(name) AS document
   FROM states
   ORDER BY name ASC
   LIMIT 10;

         name         |         document
----------------------+---------------------------
 Alabama              | 'alabama':1
 Alaska               | 'alaska':1
 Arizona              | 'arizona':1
 Arkansas             | 'arkansa':1
 California           | 'california':1
 Colorado             | 'colorado':1
 Connecticut          | 'connecticut':1
 Delaware             | 'delawar':1
 District of Columbia | 'columbia':3 'district':1
 Florida              | 'florida':1
(10 rows)

```

**Putting document and query together**

Now the question arises: How to actually use tsvector and tsquery?  Enter the `@@` operator.  This is the operator that actually performs the search. Examples work the best, so let's search for a US State with the word "North" in it:


```sql

=# SELECT
     name,
     to_tsvector(name) AS document
   FROM states
   WHERE to_tsvector(name) @@ to_tsquery('north')
   ORDER BY name ASC;

      name      |        document
----------------+------------------------
 North Carolina | 'carolina':2 'north':1
 North Dakota   | 'dakota':2 'north':1
(2 rows)

```


<br/>
Or, let's search for a state that has a word that starts with "CA":


```sql

=# SELECT
     name,
     to_tsvector(name) AS document
   FROM states
   WHERE to_tsvector(name) @@ to_tsquery('ca:*')
   ORDER BY name ASC;

      name      |        document
----------------+------------------------
 California     | 'california':1
 North Carolina | 'carolina':2 'north':1
 South Carolina | 'carolina':2 'south':1
(3 rows)

```


And that is your standard text search spanning multiple rows of a table.  A friendly, neighbourhood text-search is just a WHERE clause away! Another clear takeaway here is that any `@@` operation is no different from your average `=` operation.  Naturally, a multi-table search is just a join away. Multi-column searches are just a concatenation away.

However, what I like about this idea is that I can now compose this search with my existing SQL queries.  Let's say I have a query for "all contacts of a user that have a facebook profile", and now I can "name search" in just this subset.  To me, this is the best use-case of having the search built-in.  Composability is a very powerful design pattern.

A note on `NULL`s:  As with everything in Postgres, text-search doesn't quite work well with NULLs.  If you have null columns, [COALESCE][coalesce] is your friend, use it liberally!


**Search relevance and rankings**

Searches are often centered around finding "all matching documents", rather than finding a specific document.  In its most basic form, search relevance boils down to two questions:

1. How do I rank the results returned by the search?
2. How do I control the ranking based on my context and requirements?

Answer to the first question is the `ts_rank` function.  It accepts a tsvector and a tsquery as an argument, and returns the "rank"; which is a bit of a misnomer, because unlike a regular rank, where 1 is better than 2 is better than 3, this rank has the property "higher the better".  It's best used in the ORDER clause.  Here's an example, searching all people whose name starts with "Eliz". If that sounds odd, think autocomplete :

```sql

=# SELECT
     first_name,
     ts_rank(to_tsvector(first_name), to_tsquery('eliz:*')) as rank
   FROM person_names
   WHERE to_tsvector(first_name) @@ to_tsquery('eliz:*')
   ORDER BY rank DESC
   LIMIT 10;

   first_name    |   rank
-----------------+-----------
 Elizabeth Eliza | 0.1215850
 Elizabeth       | 0.0607927
 Elizabeth       | 0.0607927
 Elizabeth       | 0.0607927
 Elizabeth       | 0.0607927
 Elizabeth       | 0.0607927
 Elizabeth       | 0.0607927
 Elizabeth       | 0.0607927
 ELIZABETH       | 0.0607927
 Elizabeth       | 0.0607927


```

I think you will find that absolute rank isn't as useful as being able to order results by it.  That brings us to the next question, how can the ts_rank function be configured?  Say, you are searching blog posts, and want a match in title to carry more weight than a match in the body.  For this, Postgres offers "weights".  The weights are called as A, B, C and D, in the order of precedence.  The default value for these weights are 1.0, 0.4, 0.2, 0.1.  Which means a match with A carries 10 times more weight than a match with D.

Think of these weights as "tags", i.e. you tag a tsvector as A or B or C or D, and specify which tags carry how much weight. With that, Postgres will yield appropriate rank.  The tags analogy will make more sense once we look at how tsquery uses these weights.  Here's a query to demonstrate using weights in tsvector:

```sql

=# SELECT ts_rank(
           setweight(to_tsvector('With great power'), 'A') ||
           setweight(to_tsvector('comes great responsibility!'), 'D'),
           to_tsquery('power'))
  AS rank;

   rank
----------
 0.607927
(1 row)

=# SELECT ts_rank(
           setweight(to_tsvector('With great power'), 'A') ||
           setweight(to_tsvector('comes great responsibility!'), 'D'),
           to_tsquery('responsibility'))
   AS rank;

   rank
----------
 0.0607927
(1 row)

```

Our document here is made up of two parts, "with great power", tagged A, and "comes great responsibility", tagged D.

The rank for "power" is 10 times higher than "responsibility" when searching this document, because "power" is in group A, while responsibility is in group D.  Without the weights, or with same weights to all components, they will have the same rank.

The default assignment of weights A = 1, B = 0.4, C = 0.2 and D = 0.1 can be changed.  Refer to the documentation for variations of `ts_rank` that accept weights as an argument.  Playing around with the values in the psql console would give you an idea what works best for you.  I've often found that the default values really do work the best.

Coming back to the tags analogy, these same weights can also be assigned to a tsquery object, and the query would then match only amongst the given weight groups. Quite like a "filter".  This allows for features like "match this, but not that".  Have a look:

```sql

=# SELECT setweight(to_tsvector('With great power'), 'A')
     || setweight(to_tsvector('comes great responsibility!'), 'A')
     @@
     to_tsquery('resp:*B')
  AS is_match;

 is_match
----------
 f
(1 row)

=# SELECT setweight(to_tsvector('With great power'), 'A')
     || setweight(to_tsvector('comes great responsibility!'), 'A')
     @@
     to_tsquery('resp:*A')
   AS is_match;

 is_match
----------
 t
(1 row)

```


The [official documentation][control-docs] about controlling text search is quite good, and detailed.  I highly recommend that you read at least this section of the documentation, if not all of it.

This concludes the post about basic full-text search in Postgres. Have fun searching!  Do let me know what curious cases you tried out with tsquery and tsvectors.

In a follow-up post, we'll look at improving search performance by indexing, fuzzy matching, text highlighting and supporting multiple languages.


<!-- Links -->

[goog-example]: https://www.google.co.in/#q=ruby+-jewel
[active-record]: http://guides.rubyonrails.org/active_record_querying.html
[docs]: https://www.postgresql.org/docs/current/static/textsearch-intro.html
[lexeme-wiki]: https://en.wikipedia.org/wiki/Lexeme
[coalesce]: https://www.postgresql.org/docs/9.6/static/functions-conditional.html
[control-docs]: https://www.postgresql.org/docs/current/static/textsearch-controls.html
