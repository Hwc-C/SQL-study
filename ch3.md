# Chapter 3. Working with Multiple Tables

## 3.1 **Stacking One Rowset atop Another**
Want to return data stored in more than one table. 

"UNION ALL" to combine rows from multiple tables,
```sql
SELECT ename as ename_and_dname, deptno
  FROM emp
WHERE deptno=10
union all
SELECT '---------', null
  FROM t1
  union all
SELECT dname, deptno
  FROM dept
```


## 3.2 **Combining Related Rows**
Want to return rows from multiple tables by joining on a known common column or joining on columns that share common values. 

```sql
SELECT e.ename, d.loc
  FROM emp e, dept d
WHERE e.deptno=d.deptno
  and e.deptno=10
```

## 3.3 **Finding Rows in Common Between Two Tables**
Want to find common rows between two tables, but there are multiple columns on which you can join. 

```sql
CREATE view V
as
SELECT ename, job, sal
  FROM emp
WHERE job='CLERK'
```

```sql
-- MySQL, SQL Server
SELECT e.empno, e.ename, e.job, e.sal, e.deptno
  FROM emp e, V
WHERE e.ename=v.ename
  and e.job=v.job
  and e.sal=v.sal
```

```sql
-- Another version
SELECT e.empno, e.ename, e.job, e.sal, e.deptno
  FROM emp e join V
    on (
        e.ename=v.ename
        and e.job=v.job
        and e.sal=v.sal
    )
```

```sql
-- DB2, Oracle, PostgreSQL
SELECT empno, ename, job, sal, deptno
  FROM emp
WHERE (ename, job, sal) in (
    SELECT ename, job, sal FROM emp
    intersect
    SELECT ename, job, sal FROM V
)
```

## 3.4 **Retrieving Values from One Table That Do Not Exist in Another**
Want to find those values in one table, call it the source table, that do not also exist in some target table. 

```sql
-- MySQL Ver
SELECT deptno
  FROM dept
WHERE deptno not in (SELECT deptno FROM emp)
```

```sql
-- DB2, PostgreSQL, SQL Server
SELECT deptno FROM dept
  EXCEPT
SELECT deptno FROM emp
```

```sql
-- Oracle
SELECT deptno FROM dept
  minus
SELECT deptno FROM emp
```

> "True or NULL" is True, "False or NULL" is NULL. 

## 3.5 **Retrieving Rows from One Table That Do Not Correspond to Rows in Another**
Want to find rows that are in one table that do not have a match in another table. 

```sql
SELECT d.*
  FROM dept d left outer join emp e
WHERE e.deptno is null
```

## 3.6 **Adding Joins to a Query Without Interfering with Other Joins**
Have a query that returns the results you want. Need additional info, but when trying to get it, we lose data from the original result set. 

```sql
-- Start with
SELECT e.name, d.loc
  FROM emp e, dept d
 WHERE e.deptno=d.deptno
```

```sql
-- DB2, MySQL, PostgreSQL, SQL Server Ver
select e.ename, d.loc, eb.received
  from emp e join dept d
    on (e.deptno=d.deptno)
  left join emp_bonus eb
    on (e.empno=eb.empno)
 order by 2

-- OR use a scalar subquery
select e.ename, d.loc,(
  select eb.received from emp_bonus eb
   where eb.empno=e.empno
) as received
  from emp e, dept d
 where e.deptno=d.deptno
 order by 2
```

## 3.7 **Determining Whether Two Tables Have the Same Data**
Want to know whether two tables or views have the same data (cardinality and values). 

Consider
```sql
create view V
as 
select * from emp where deptno!=10
 union all
select * from emp where ename='WARD'
```

```sql
-- MySQL, SQL Server Ver
select *
  from (
    select e.empno, e.ename, e.job, e.mgr, e.hiredate, e.sal, e.comm, e.deptno, count(*) as cnt
      from emp e
     group by empno, ename, job, mgr, hiredate, sal, comm, deptno
  ) as e
 where not exists (
   select null
     from (
       select v.empno, v.ename, v.job, v.mgr, v.hiredate, v.sal, v.comm, v.deptno, count(*) as cnt
        from v
       group by empno, ename, job, mgr, job, hiredate, sal, comm, deptno
     ) v
    where v.empno=e.empno
      and v.ename=e.ename
      and v.job=e.job
      and coalesce(v.mgr, 0)=coalesce(e.mgr, 0)
      and v.hiredate=e.hiredate
      and v.sal=e.sal
      and v.deptno=e.deptno
      and v.cnt=e.cnt
      and coalesce(v.comm, 0)=coalesce(e.comm, 0)
 )
  union all
select *
  from (
    select v.empno, v.ename, v.job, v.mgr, v.hiredate, v.sal, v.comm, v.deptno, count(*) as cnt
      from v
     group by empno, ename, job, mgr, hiredate, sal, comm, deptno
  ) as v
 where not exists (
   select null
     from (
       select e.empno, e.ename, e.job, e.mgr, e.hiredate, e.sal, e.comm, e.deptno, count(*) as cnt
        from emp e
       group by empno, ename, job, mgr, job, hiredate, sal, comm, deptno
     ) e
    where v.empno=e.empno
      and v.ename=e.ename
      and v.job=e.job
      and coalesce(v.mgr, 0)=coalesce(e.mgr, 0)
      and v.hiredate=e.hiredate
      and v.sal=e.sal
      and v.deptno=e.deptno
      and v.cnt=e.cnt
      and coalesce(v.comm, 0)=coalesce(e.comm, 0)
 )
```
1. Find rows in table EMP that do not exist in view V. 
2. Combine those rows with rows from view V that do not exist in table EMP. 

> Note: "coalesce(expression1, expression2, ...)" means return expression1 when it is not null, or return expression2 when it is not null ... 

## 3.8 **Identifying and Avoiding Cartesian Products**
Want to return the name of each employee in department 10 along with the location of the department. 

```sql
select e.ename, d.loc
  from emp e, dept d
 where e.deptno=10
  and d.deptno=e.deptno
```

## 3.9 **Performing Joins When Using Aggregates**
Want to perform an aggregation, but your query involves multiple tables. Want to ensure that joins do not disrupt the aggregation. 

Be careful about the duplicates. Simply using DISTINCT or aggregation first. 
```sql
-- MySQL, PostgreSQL Ver
select deptno, sum(distinct sal) as total_sal, sum(bonus) as total_bonus
  from (
    select e.empno, e.ename, emsal, e.deptno, 
      e.sal * case when eb.type=1 then .1
                   when eb.type=2 then .2
                   else .3
              end as bonus
      from emp e, emp_bonus eb
      where e.empno=eb.empno
        and e.deptno=10
  ) x
 group by deptno
```

## 3.10 **Performing Outer Joins When Using Aggregates**
The same problem as in 3.9, but modify table EMP_BONUS such that the difference in this case is not all employees in department 10 have been given bonuses. 

```sql
select deptno, sum(distinct sal) as total_sal, sum(bonus) as total_bonus
  from (
    select e.empno, e.ename, e.sal, e.deptno, 
      e.sal*case when eb.type is null then 0
                 when eb.type=1 then .1
                 when eb.type=2 then .2
                 else .3 
                 end as bonus
      from emp e left outer join emp_bonus eb
        on (e.empno=eb.empno)
    where e.deptno=10
  )
 group by deptno
```

```sql
-- SUM OVER func
select distinct deptno, total_sal, total_bonus
  from (
    select e.empno, e.ename, 
      sum(distinct e.sal) over
      (partition by e.deptno) as total_sal, 
      e.deptno, 
      sum(
        e.sal*case when eb.type is null then 0
                   when eb.type=1 then .1
                   when eb.type=2 then .2
                   else .3
                   end
      ) over
      (partition by deptno) as total_bonus
  from emp e left outer join emp_bonus eb
    on (e.empno=eb.empno)
  where e.deptno=10
  ) x
```

## 3.11 **Returning Missing Data from Multiple Tables**
Want to return missing data from multiple tables simultaneously. 

```sql
select d.deptno, d.dname, e.ename
  from dept d full outer join emp e
    on (d.deptno=e.deptno)

-- equals to
select d.deptno, d.dname, e.ename
  from dept d right outer join emp e
    on (d.deptno=e.deptno)
  union
select d.deptno, d.dname, e.ename
  from dept d left outer join emp e
    on (d.deptno=e.deptno)
```

## 3.12 **Using NULLs in Operations and Comparisons**
NULL is never equal to or not equal to any value, not even itself, but you want to evaluate values returned by a nullable column like you would evaluate real values. 

Example, find all employees in EMP whose comm is less than the commission of employee WARD. So, using **coalesce** for null. 
```sql
select ename, comm
  from emp
 where coalesce(comm, 0) < (
  select comm
    from emp
   where ename='WARD'  
 )
```
