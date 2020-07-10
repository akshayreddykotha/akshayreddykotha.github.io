---
title: "15 days of learning SQL"
date: 2020-07-07
tags: [sql]
excerpt: "My approach to the hackerrank final SQL challenge - 15 days of learning SQL"
comments: true
mathjax: true

---


This approach is driven by the thought to break down the problem (one of the hackerrank challenges named as title) as much as you can and finally synthesizing back to get the final result. 
Syntax used is with respect to MS SQL server and limits to using: `DATEDIFF`, `COALESCE`, `OVER()`, `PARTITION BY`, `WITH... AS(...)`. These are some uncommon clauses used in SQL. Using variables (`@var`) might make the solution much similar to something written in Python or R.

There are two parts I was able to clearly see. Part 1: the left chunk of the table, Part 2: the right chunk. Part 2 is more or less straightforward whereas Part 1 is slightly challenging but the order I am presenting here is 1 then 2 as it is easy to follow.

### Part 1

Getting to the part where you have to find out what is the unique number of users till that particular `submission_date` in continuity.

There are multiple tables defined which are used as subqueries in getting to the final output. The subqueries are short and easy to grasp on one read and will synthesize why they are written in the first place at the end. So, hang on!

#### `dates_hackers` 

This is the table view which gives a count of submissions grouped by submission_date and hacker_id.

```
WITH dates_hackers AS(
SELECT submission_date, hacker_id, COUNT(*) subs
FROM Submissions
GROUP BY submission_date, hacker_id
),
```

#### `day1_hackers`

This is core to the part 1 as we want to see the count of the users till any particular date starting from day 1 (2016-03-01).

```
day1_hackers AS (
SELECT DISTINCT(hacker_id)
FROM Submissions
WHERE submission_date = '2016-03-01'
),
```

#### `min_max` 

This helps us understand what are the first, last and no. of days of difference betweeen first and last dates of submissions by the hackers.

```
min_max AS (
SELECT hacker_id, MIN(submission_date) first_sub, MAX(submission_date) last_sub, 
DATEDIFF(day, MIN(submission_date), MAX(submission_date)) diff
FROM Submissions
GROUP BY hacker_id
),
```

#### `cumuCountLag`

This is a table to get the cumulative number of days submitted by each hacker and also a lagged date (prevDay).

```
cumuCountLag AS (
SELECT hacker_id, submission_date, LAG(submission_date) OVER(PARTITION BY hacker_id ORDER BY submission_date) prevDay, 
COUNT(*) OVER(PARTITION BY hacker_id ORDER BY submission_date) cumuCount
FROM dates_hackers
),
```
We have the `prevDay` column here to check the difference in days consecutively.

#### `p1_interm`

This is a derived table from the above tables by applying all the conditions. Remember synthesis, that's what this does for part 1.

```
p1_interm AS (
SELECT cumuCountLag.hacker_id, cumuCountLag.submission_date, prevDay, first_sub, cumuCount,
    CASE WHEN (cumuCountLag.hacker_id IN (SELECT * FROM day1_hackers))
    AND (cumuCount = DATEDIFF(day, first_sub, cumuCountLag.submission_date) + 1)
    THEN 1
ELSE 0 END AS partic_till
FROM cumuCountLag
LEFT JOIN  min_max
ON cumuCountLag.hacker_id = min_max.hacker_id
),
```

There are 2 main conditions which are the crux of part 1 (written in that order in the above snippet):
1. Check whether the hacker_id is in the first day of hackers who made submissions
2. We need to make sure if the hacker is making submissions from the day 1 and there is no break in between

#### `p1`

It's own table (the left chunk) to finally join into the final output.

```
p1 AS (
SELECT submission_date, SUM(partic_till) no_hackers_till
FROM p1_interm
GROUP BY submission_date
),
```

**A short snippet to debug part 1 while building.**

```
-- -- Debug 1
-- SELECT *
-- FROM p1
-- ORDER BY submission_date
```

### Part 2

Getting to find out the hacker with highest number of submissions with ascending order of `hacker_id` when there is a tie.

#### `topHackers`

This table view assigns ranks to hackers based on number of submissions in each day.

```
-- Part 2
topHackers AS(
SELECT dates_hackers.submission_date, dates_hackers.hacker_id, h.name, dates_hackers.subs, 
RANK() OVER(PARTITION BY dates_hackers.submission_date ORDER BY dates_hackers.subs DESC, 
dates_hackers.hacker_id) AS sRank
FROM dates_hackers
JOIN Hackers h
ON dates_hackers.hacker_id = h.hacker_id
), 
```

#### `p2` - It's own table, the right chunk to finally join

```
-- A separate output for part 2
p2 AS (
SELECT submission_date, hacker_id, name
FROM topHackers
WHERE sRank = 1
)
```

**A short snippet to debug part 2 while building. It's always variable based on the situation you face to quickly do some checks. But having a short snippet as a part of your code is very helpful to spot the run-time errors.**


```
-- -- Debug 2
-- SELECT *
-- FROM p1
-- ORDER BY submission_date
```

#### Final join of two parts

```
-- Final output
SELECT p1.submission_date, p1.no_hackers_till, p2.hacker_id, p2.name
FROM p1
JOIN p2
ON p1.submission_date = p2.submission_date
```

*Trivia*:

* `--` is the way we comment in SQL. There is `/*...*/` this too.
* Writing subqueries is a great way to breakdown the problem but there is trade-off run time, single batch queries run faster.

In order to get the output, add snippet by snippet in your environment in the same order as presented. You reached the end of the article, kudos to you. Let me know your thoughts or ideas to optimize this further. I am all ears!!!