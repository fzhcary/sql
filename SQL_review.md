# SQL

## Video Review
[This](https://www.youtube.com/watch?v=2Fn0WAyZV0E) is nice review of the SQL 

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
SELECT DENSE_RANK() OVER ( ORDER BY sid) FROM enrolled
-- output 1,2,2,3,3 , has duplicates, numbers are consecutive
```