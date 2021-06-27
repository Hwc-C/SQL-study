# Chapter 4. Inserting, Updating, and Deleting

## 4.1 **Inserting a New Record**
Want to insert a new reord into a table. 

```sql
insert into dept (deptno, dname, loc)
values (50, 'PROGRAMMING', 'BALTIMORE')
```

## 4.2 **Inserting Default Values** 
Want to insert a row of default values without having to specify those values. 

```sql
-- default keyward
insert into D (id) values (default)
create table D (id integer default 0, foo varchar(10))
insert into D (name) values ('Bar')
```

## 4.3 **Overriding a Default Value with NULL**
Want to override that default value by setting the column to NULL. 

```sql
-- the table
create table D (id integer default 0, foo VARCHAR(10))
-- then
insert into d (id, foo) values (null, 'Brighten')
```

## 4.4 **Copying Rows from One Table into Another**
Want to copy rows from one table to another by using a query. 

```sql
insert into dept_east (deptno, dname, loc)
select deptno, dname, loc
  from dept
 where loc in ('NEW YORK', 'BOSTON')
```

## 4.5 **Copying a Table Definition**
Want to create a new table having the same set of columns as an existing table. 

```sql
-- DB2 Ver
create table dept_1 like dept
-- Oracle, MySQL,PostgreSQL Ver
create table dept_2
as 
select *
  from dept
 where 1=0
-- SQL Server Ver
select *
  into dept_2
  from dept
 where 1=0
```

## 4.6 **Inserting into Multiple Tables at Once**
Want to take rows returned by a query and insert those rows into multiple target tables. 

```sql
-- Oracle
insert all
  when loc in ('NEW YORK', 'BOSTON') then
  into dept_east (deptno, dname, loc) value (deptno, dname, loc)
  when loc='CHICAGO' then
    into dept_mid (deptno, dname, loc) value (deptno, dname, loc)
  else 
    into dept_west (deptno, dname, loc) value (deptno, dname, loc)
  select deptno, dname, loc
    from dept
```

## 4.7 **Blocking Inserts to Certain Columns**
Want to prevent users, or an errant software application, from inserting values into certain table columns. 

```sql
create view new_emps
as
select empno, ename, job
  from emp
```

## 4.8 **Modifying Records in a Table**
Want to modify values for some or all rows in a table. 

```sql
-- bump all the SAL values by 10%
update emp
  set sal=sal*1.10
 where deptno=20
```

## 4.9 **Updating When Corresponding Rows Exist**
Want to update rows in one table when corresponding rows exist in another. 

```sql
update emp
  set sal = sal*1.20
 where empno in (select empno from emp_bonus)
```

## 4.10 **Updating with Values from Another Table**
Want to update rows in one table usin values from another. 

```sql
-- MySQL Ver
update emp e, new_sal ns
set e.sal=ns.sal, e.comm-ns.sal/2
where e.deptno=ns.deptno
```

## 4.11 **Merging Records**
Want to conditionally insert, update, or delete records in a table depending on whether corresponding records exist. 

```sql
-- MySQL might not have a MERGE statement
merge into emp_commission ec
using (select * from emp) emp
  on (ec.empno=emp.empno)
 when matched then
  update set ec.comm=1000
  delete where (sal < 2000)
 when not matched then
  insert (ec.empno, ec.ename, ec.deptno, ec.comm)
  values (emp.empno, emp.ename, emp.deptno, emp.comm)
```

## 4.12 **Deleting All Records from a Table**
Want to delete all the records from a table. 

```sql
delete from emp
```

## 4.13 **DeletingSpecific Records**
Want to delete records meeting a specific criterion from a table. 

```sql
delete from emp where deptno=10
```

## 4.14 **Deleting a Single Record**
Want to delete a single record from a table. 

```sql
delete from emp where empno=7782
```

## 4.15 **Deleting Referential Integrity Violations**
Want to delete records from a table when those records refer to nonexistent records in some other table. 

```sql
-- not exists
delete from emp
  where not exists (
    select * from dept
     where dept.deptno=emp.deptno
  )

-- OR using NOT IN
delete from emp
  where deptno not in (select deptno from dept)
```

## 4.16 **Deleting Duplicate Records**
Want to delete duplicate records from a table. 

```sql
delete from dupes
 where id not in (
   select min(id)
     from  (select id, name from dupes) tmp
     group by name
 )
```
> Note::MySQL cannot reference the same table twice in a delete. 

## 4.17 **Deleting Records Referenced from Another Table**
Want to delete records from one table when those records are referenced from some other table. 

```sql
delete from emp
 where deptno in (
   select deptno
     from dept_accidents
    group by deptno
    having count(*)>=3
 )
```
