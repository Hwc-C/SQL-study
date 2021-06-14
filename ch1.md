# Chapter 1. Retrieving Records

## 1.1 **Retrieving All Rows and Columns from a Table**
Want to see all of the data in a table. 

```sql
SELECT *
  FROM emp
```

It is better to specify each column individually when writing program code. 

## 1.2 **Retrieving a Subset of Rows from a Table**
Want to see only rows that satisfy a specific condition. 

```sql
SELECT *
  FROM emp
WHERE deptno=10
```

Common operators such as =, <, >, <=, >=, !, <>. 


## 1.3 **Finding Rows That Satisfy Multiple Conditions**
Want to return rows that satisfy multiple conditions. 

```sql
SELECT *
  FROM emp
WHERE (
    deptno=10
    or comm is not null
    or sal <= 2000 and deptno=20
)
```

## 1.4 **Retrieving a Subset of Columns from a Table**
Want to see values for specific columns rather than for all the columns. 

```sql
SELECT ename, deptno, sal
  FROM emp
```

## 1.5 **Providing Meaningful Names for Columns**
Consider the query the returns more readable and understandable result. 

```sql
SELECT sal as salary, comm as commision
  FROM emp
```

## 1.6 **Referencing an Aliased Column in the WHERE Clause**
Have used aliases to provide meaningful column names, then meet fails when using WHERE. 

```sql
SELECT *
  FROM (
      SELECT sal as salary, comm as commision
        FROM emp
  ) x
WHERE salary<5000
```

## 1.7 **Concatenating Column Values**
Want to return values in multiple columns as one column. 

```sql
-- MySQL
SELECT concat(ename, ' WORKS AS A ', job) as msg
  FROM emp
WHERE deptno=10
```

- In DB2, Oracle and PostgreSQL, using ||. 
- In SQL Server, using +. 

## 1.8 **Using Conditional Logic in a SELECT Statement**
Want to perform IF-ELSE operations on values in SELECT statement. 

```sql
-- Set some status
SELECT ename, sal, 
    case when sal <= 2000 then 'UNDERPAID'
         when sal >= 4000 then 'OVERPAID'
         else 'OK'
    end as status
FROM emp
```

CASE expression -> perform condition logic on values

## 1.9 **Limiting the Number of Rows Returned**
Want to limit the number of rows returned in the query. 

```sql
-- MySQL and PostgreSQL Ver
SELECT *
  FROM emp limit 5
```

- DB2, using "FETCH first 5 rows only" at the end. 
- Oracle, using "WHERE rownum <= 5".
- SQL Server, using "top 5" in SELECT section.

## 1.10 **Returning n Random Records from a Table**
Want to return a specific number of random record from the table.

```sql
-- MySQL and PostgreSQL Ver
SELECT ename, job
  FROM emp
ORDER BY rand() limit 5
```

- DB2, similarly "ORDER BY rand() ...".
- Oracle, modify the FROM part, using SELECT and "ORDER BY dbms_random.value()".
- SQL Server, using "ORDER BY newid()".

## 1.11 **Finding Null Values**
Want to find all rows that are NULL for a particular column.

```sql
SELECT *
  FROM emp
WHERE comm is null
```

OR using "IS NOT NULL" to find rows without a NULL in a given column. 

## 1.12 **Transforming Nulls into Real Values**
Rows contain NULLs and want to return non-null values in place of those NULLs. 

```sql
SELECT coalesce(comm, 0)
  FROM emp
```

ALSO, it is possible to use CASE. 

## 1.13 **Searching for Patterns**
Want to return rows that match a particular substring OR pattern. 

```sql
SELECT ename, job
  FROM emp
WHERE deptno in (10, 20)
  and (ename like '%I%' or job like '%ER')
```

Using LIKE pattern-match operation
- "%", any sequence
- "_", single character


