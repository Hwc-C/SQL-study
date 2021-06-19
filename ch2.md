# Chapter 2. Sorting Query Results

## 2.1 **Returning Query Results in aSpecified Order**
Want to display cols in order based on their salary (from lowest to highest).

```sql
SELECT ename, job, sal
  FROM emp
WHERE deptno=10
ORDER BY sal asc
```

- OR "DESC" (from highest to lowest)
- OR "ORDER BY 3 DESC", the third column is sal

## 2.2 **Sorting by Multiple Fields**
Want to sort the rows from EMP first by DEPTNO ascending, then by salary descending. 

```sql
SELECT empno, deptno, sal, ename, job
  FROM emp
ORDER BY deptno, sal desc
```

## 2.3 **Sorting by Substrings**
Want to sort the results of a query by specific parts of a string.

The last two characters in the JOB field. 
```sql
SELECT ename, job
  FROM emp
ORDER BY substr(job, length(job)-1)
```

- SQL Server, using *substring(job, len(job)-1, 2)*

## 2.4 **Sorting Mixed Alphanumeric Data**
Have mixed alphanumeric data and want to sort by either the numeric or character portion of the data.

Mixing example:
```sql
CREATE VIEW V
AS
SELECT ename||' '||deptno as data
  FROM emp
```

```sql
-- Oracle, SQL Server, PostgreSQL
/*ORDER BY DEPTNO*/
SELECT data
  FROM v
ORDER BY replace(
  data,
  replace(
    translate(data, '0123456789', '##########'), '#', ''
  ), ''
)
```

```DB2
-- DB2
/*ORDER BY DEPTNO*/
SELECT *
  FROM (
    SELECT ename||' '||cast(deptno as char(2)) as data
      FROM emp
  ) v
ORDER BY replace(
  data,
  replace(
    translate(data, '##########', '0123456789'), '#', ''
    ), ''
)
```

- MySQL, NO Solution (no TRANSLATE function)

```sql
/*PUT INFO INTO COLS*/
SELECT data, 
  replace(
    data,
    replace(
      translate(data, '0123456789', '##########'), '#', ''
    ), ''
  ) nums,
  replace(
    translate(data, '0123456789', '##########'), '#', ''
  ) chars
FROM v
```

## 2.5 **Dealing with Nulls When Sorting**
Want to sort results from EMP by COMM, but the field is nullable (they sort first or last). 

Define 0 or 1 value on the data.
```sql
-- DB2, MySQL, PostgreSQL, SQL Server
SELECT ename, sal, comm
  FROM (
    SELECT ename, sal, comm, 
      CASE when comm is null then 0 else 1 end as is_null
    FROM emp
  ) x
ORDER BY is_null desc, comm
```

- Oracle, *ORDER BY comm nulls last* OR *ORDER BY comm nulls first*

## 2.6 **Sorting on a Data-Dependent Key**
Want to sort based on some conditional logic. 

```sql
SELECT ename, job, comm
  FROM emp
ORDER BY case when job='SALESMAN' then comm else sal end
```




