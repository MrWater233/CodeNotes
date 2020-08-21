# 一.插入数据

`INSERT INTO 表名 [属性值1,属性值2...] VALUES(常量1,常量2...);`

## 1.插入元组

```sql
/*插入一条选课记录*/
/*Grade列上自动赋空值*/
INSERT INTO SC(Sno,Cno) VALUES('291215128','1');
/*没有指定SC的属性名，需要在Grade上明确给出空值*/
INSERT INTO SC VALUES('201215128','1',NULL);
```

- 字符串常用单引号括起来

## 2.插入子查询结果

```sql
/*对每一个系，求学生的平均年龄，并把结果存入数据库*/
INSERT INTO Dept_age(Sdept,Avg_age) SELECT Sdept,AVG(Sage) FROM Student GROUP BY Sdept;
```

# 二.修改数据

`UPDATE 表名 SET 列名=表达式,列名=表达式... [WHERE 条件];`

## 1.修改某一个元组的值

```sql
/*修改学生年龄*/
UPDATE Student SET Sage=22 WHERE Sno='201215121';
```

## 2.修改多个元组的值

```sql
/*将所有学生的年龄增加一岁*/
UPDATE Student SET Sage=Sage+1;
```

## 3.带子查询的修改语句

```sql
/*将计算机科学系全体学生的成绩置0*/
UPDATE SC SET Grade=0 WHERE Sno IN (SELECT Sno FROM Student WHERE Sdept='CS');
```

# 三.删除数据

`DELETE FROM 表名 [WHERE 条件];`

## 1.删除某一个元组的值

```sql
/*删除学号为201215128学生的记录*/
DELETE FROM Student WHERE Sno='201215128';
```

## 2.删除多个元组的值

```sql
/*删除所有学生的选课记录*/
DELETE FROM SC;
```

## 3.带子查询的删除语句

```sql
/*删除计算机科学系所有学生的选课记录*/
DELETE FROM SC WHERE Sno IN (SELECT Sno FROM Student WHERE Sdept='CS');
```

