# SQL

## Video Review
[This](https://www.youtube.com/watch?v=2Fn0WAyZV0E) is nice review of the SQL 

- Important rule: output of other columns outside of an aggregate is undefined. ex.
```
SELECT AVG(s.gpa), e.cid
FROM enrolled AS e, student AS s
WHERE e.sid=s.sid
```
## tips
you can use rollup in group by clause to aggregate along the dimensions. so you can see the rollup values in addition.
cube is similar.
There is only one major difference between the functionality of the ROLLUP operator and the CUBE operator. ROLLUP operator generates aggregated results for the selected columns in a hierarchical way. On the other hand, CUBE generates a aggregated result that contains all the possible combinations for the selected columns.

you can use transaction and rollback if you are not sure about delete

```
SELECT cr.country_name, cr.region_name, count(e.*)
FROM employees e
JOIN company_regions cr
ON e.region_id = cr.id
GROUP BY rollup(cr.country_name, cr.region_name)
ORDER BY cr.country_name, cr.region_Name
```




### create tables
Let's go to sqlfiddle.com and use "Text to DDL" to paste a csv text, with the header as the first row and build the schema by click the button. 
Once you create one table, you can append another one and build schema again. In the end you get something like this
```
CREATE TABLE student
    ("sid" int, "name" varchar(6), "login" varchar(10), "age" int, "gpa" float)
;
    
INSERT INTO student
    ("sid", "name", "login", "age", "gpa")
VALUES
    (53666, 'Kanye', 'kayne@cs', 39, 4.0),
    (53688, 'Bieber', 'jbieber@cs', 22, 3.9),
    (53655, 'Tupac', 'shkur@cs', 26, 3.5)
;


CREATE TABLE enrolled
    ("sid" int, "cid" varchar(6), "grade" varchar(1))
;
    
INSERT INTO enrolled
    ("sid", "cid", "grade")
VALUES
    (53666, '15-445', 'C'),
    (53688, '15-721', 'A'),
    (53688, '15-826', 'B'),
    (53655, '15-445', 'B'),
    (53666, '15-721', 'C')
;


CREATE TABLE course
    ("cid" varchar(6), "name" varchar(28))
;
    
INSERT INTO course
    ("cid", "name")
VALUES
    ('15-445', 'Database Systems'),
    ('15-721', 'Advanced Database Systems'),
    ('15-826', 'Data Mining'),
    ('15-823', 'Advanced Topics in Databases')
;

```

### pratice problems

With these tables, you can practice sql. I like to use postgres 9.6 engine.

- find student record with the highest id that is enrolled in at least one course
```
SELECT sid, name
FROM student S
WHERE sid=
(SELECT MAX(e.sid)
FROM enrolled e)
```

- find all courses that has no students enrolled in it (you can refer outer query from inner query.)
```
SELECT *
FROM course c
WHERE NOT EXISTS
(SELECT cid FROM enrolled e WHERE e.cid=c.cid)
```

- window functions
```
SELECT cid, sid, ROW_NUMBER() OVER ( ORDER BY sid) FROM enrolled
-- row number is the rank by sid start from 1. no duplicates
SELECT cid, sid, ROW_NUMBER() OVER ( PARTITION BY cid ORDER BY sid) FROM enrolled
-- row number reset after each partition
SELECT RANK() OVER ( ORDER BY sid) FROM enrolled
-- output 1, 2, 2, 4, 4. has duplicates, numbers are not consecutive
-- the difference between ROW_NUMBER and RANK() is that rank has tie, row number doesn't produce tie.
SELECT DENSE_RANK() OVER ( ORDER BY sid) FROM enrolled
-- output 1,2,2,3,3 , has duplicates, numbers are consecutive
-- the difference between rank and dense_rank is that rank is not consecutive after a tie. dense_rank is.
```

- find student with highest grade for each course
```
SELECT * FROM 
(
SELECT *, RANK() OVER (PARTITION BY cid ORDER BY grade) FROM enrolled
) AS ranking
WHERE ranking.rank=1
```

- same as above, but also output name, cid, and grade
```
SELECT s.name, ranking.cid, ranking.grade FROM 
(
SELECT *, RANK() OVER (PARTITION BY cid ORDER BY grade) FROM enrolled
) AS ranking
--WHERE ranking.rank=1
JOIN student s
ON s.sid=ranking.sid
AND ranking.rank=1
```

- CTE (common table expressions ): a way to write auxiliary statements for use in a larger query. ex
```
WITH temp (col1, col2) AS (
  SELECT 1, 2
  )
  SELECT col1 + col2 FROM temp;
```
-- the select statement after the cte can refer to table as if it exists

-- difference between CTE and nest query is that CTE can do recursion. ex
```
WITH RECURSIVE temp ( counter ) AS (
    (SELECT 1)
    UNION ALL
    (SELECT counter + 1 FROM temp
     WHERE counter < 3)
  )
  SELECT * FROM temp
```
-- this will produce 3 rows, with rows have value from 1 to 3. first execution is 1 row with 1, 2nd is add 1 to counter 1, produce row with 2 then union the 1, so row 1 and 2. 3nd time, first have two rows of 1, 2, add 1 to each row becomes 2, 3 and unionall with 1, so final output is 1,2,3

--btw, this produce the same result if we take the parenthesis out:
```
WITH RECURSIVE temp ( counter ) AS (
    SELECT 1
    UNION ALL
    SELECT counter + 1 FROM temp
     WHERE counter < 3
  )
  SELECT * FROM temp
```
