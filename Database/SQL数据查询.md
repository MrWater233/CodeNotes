```sql
SELECT [ALL|DISTINCT] 列名1，列名2...
FROM ((表名/视图名)|(SELECT语句 [AS] 别名))...
[WHERE 条件]
[GROUP BY 列名 [HAVING 条件表达式]]
[ORDER BY 列名 [ASC|DESC]];
```

# 一.单表查询

## 1.选择表中若干列

### (1)查询指定列

```sql
SELECT Sno,Sname FROM Student;
```

执行过程：从`Student`表中取出一个元组，取出该元组`Sno`和`Sname`上的值，形成一个新的元组输出。对表中其他元组也做同样操作，最后形成结果输出。

### (2)查询全部列

```sql
SELECT * FROM Student
```

### (3)查询经过计算的值

> `SELECT`查询的不只是表中的属性列，还能是**表达式**

```sql
SELECT Sname,2014-Sage FROM Student;  /*结果第二列为2014-年龄的结果*/
SELECT Sname,LOWER(Sdept) FROM Student;  /*将所属系名小写输出*/
SELECT Sname,LOWER(Sdept) DEPARTMENT FROM Student;  /*将所属系名小写并另取别名输出*/
```

## 2.选择表中若干元组

### (1)消除取值的重复行

```sql
SELECT DISTINCT Sno FROM SC;
```

- 如果没有指定`DISTINCT`关键字，默认为`ALL`即保留重复行

### (2)查询满足条件的元组

> 查询满足条件的元组通过`WHERE`来实现

#### ①比较大小

`=,>,<,>-,<=,!或<>,!>(不大于),!<(不小于)`

```sql
SELECT Sname FROM Student WHERE Sdept='CS';
```

#### ②确定范围

`[NOT] BETWEEN...(下限)AND...(上限)`（包括上下限）

```sql
/*查找年龄在20-23岁（包括20和23）之间的学生姓名*/
SELECT Sname FROM Student WHERE Sage BETWEEN 20 AND 23;
```

#### ③确定集合

用谓词`[NOT]IN`来查找属性值(不)属于指定集合的元组

```sql
SELECT Sname FROM Student WHERE Sdept IN('CS','MA');
```

#### ④字符匹配

使用`[not] LIKE`来进行字符串匹配

```sql
[NOT] LIKE '匹配串'
```

- 匹配串可以是一个完整的字符串，也可以含有通配符`%和_`
  - `%`代表任意长度的字符串（长度可以为0）
  - `_`代码任意单个字符，ASCII字符集时一个汉字需要两个`_`，GBK时只需要一个
- 如果匹配串不含有通配符，则可以用`=` 代替`LIKE`
- 如果查询得字符串本身含有通配符，需要进行转义`\%或\_`

```sql
/*查询所有姓刘的同学的姓名*/
SELECT Sname FROM Student WHERE Sname LIKE '刘%';
```

#### ⑤涉及空值的查询

`IS [NOT] NULL`

```sql
/*查询确少成绩学生的学号和课程号*/
SELECT Sno,Cno FROM SC WHERE Grade IS NULL;
```

#### ⑥多重条件查询

使用`AND`和`OR`，其中`AND`的优先级高于`OR`，但是可以用括号改变优先级

`IN`谓词实际上是多个`OR`的缩写

```sql
SELECT Sname FROM Student WHERE Sdept='CS' AND Sage<20;
```

## 3.ORDER BY子句

> 对查询结果按照**一个**或**多个**属性列升序（`ASC`）或降序（`DESC`）排列，默认为升序

```sql
/*查询全体学生情况，结果按照系号升序配列，同一系中的学生按照年龄降序排列*/
SELECE * FROM Student ORDER BY Sdept,Sage DESC;
```

- 对于空值，排序显示的次序由具体系统实现来决定

## 4.聚集函数

| 聚集函数                      | 功能                                 |
| ----------------------------- | ------------------------------------ |
| `COUNT(*)`                    | 统计元组个数                         |
| `COUNT([DISTINCT|ALL]<列名>)` | 统计一列中值的个数                   |
| `SUM([DISTINCT|ALL]<列名>)`   | 计算一列值的总和（此列必是数值型）   |
| `AVG([DISTINCT|ALL]<列名>)`   | 计算一列值的平均数（此列必是数值型） |
| `MAX([DISTINCT|ALL]<列名>)`   | 求一列值中的最大值                   |
| `MIN([DISTINCT|ALL]<列名>)`   | 求一列值中的最小值                   |

- 如果指定了`DISTINCT`短语，则表示计算式要取消指定列中的重复值
- 当聚集函数遇到空值时（除了`COUNT(*)`）都跳过空值只处理非空值

## 5.GROUP BY子句

> 将查询结果按照一列或多列的分组，值相等的为一组

对查询结果分组的目的是为了**细化聚集函数的作用对象**

- 如果未对查询结果分组，聚集函数将作用于整个查询结果
- 分组后聚集函数将作用于每一个组，即每一组都有一个函数值

```sql
/*求各个课程号及相应的选课人数*/
/*先对Cno的值分组再对每组进行计算(使用COUNT(*)也可以)*/
SELECT Cno,COUNT(Sno) FROM SC GROUP BY Cno;
```

### HAVING

> 对每一组进行筛选

```sql
/*查询选修三门以上课程的学生学号*/
/*这里先用GROUP BY对Sno进行分组，再对每一组进行计数*/
SELECT Sno FROM SC GROUP BY Sno HAVING COUNT(*)>3;
```

### WHERE和HAVING的区别

- `WHERE`作用于基本表或视图
- `HAVING`作用于组，从中选择满足条件的组

# 二.连接查询

> 同时涉及两个以上的表称为连接查询

## 1.等值与非等值连接查询

### 格式

连接查询的`WHERE`子句中用来连接两个表的**条件**称为**连接条件**或者**连接谓词**，一般格式为：

- `表名1.列名1<比较运算符>表名2.列名2`
- `表名1.列名1 BETWEEN 表名2.列名2 AND 表名2.列名3`

当连接运算符为`=`时为**等值连接**，其他运算符为**非等值连接**

### 例子

```sql
/*查询每个学生及其选修课情况*/
SELECT Student.*,SC.* FROM Student,SC WHERE Student.Sno=SC.Sno;
```

- `SELECT`和`WHERE`后的属性中如果属性名在参加连接的各表中是唯一的，可以省略表名前缀

### 执行过程

现在`Student`表中找到第一个元组，然后从头扫描`SC`，逐一查找与`Student`第一个元组`Sno`相等的`SC`元组，找到后就将`Student`的第一个元组与该元组进行拼接，形成表的第一个元组，以此类推。

### 自然连接

> 在等值连接中去除**结果的重复属性列**就是自然连接

```sql
/*将SC.Sno从结果中剔除*/
SELECT Student.Sno,Sname,Sex,Sage,Sdept,Cno,Grade FROM Student,SC WHERE Student.Sno=SC.Sno;
```

### 优化

当`WHERE`子句中既有**连接谓词**又有**选择谓词**时先进行**选择**，再进行连接能够提高速度

## 2.自身连接

> 一个表与自己进行连接成为表的**自身连接**
>
> 通常需要为自己的表取两个**别名**进行区分

```sql
/*查询和刘晨在同一个系学习的学生*/
SELECT S1.Sno,S1.Sname,S1.Sdept FROM Student S1,Student S2
WHERE S1.Sdept=S2.Sdept AND S2.Sname='刘晨';
```

## 3.外连接

> 通常连接操作中，只有满足连接条件的两个元组才会被连接，不满足的会被舍弃
>
> 外连接就是保留这些被舍弃的**悬浮元组**

```sql
/*查询每个学生的选课情况并保留没有选课的学生*/
SELECT Student.Sno,Sname,Ssex,Sage,Sdept,Cno,Grade FROM Student LEFT OUTER JOIN SC ON (Student.Sno=SC.Sno);
```

## 4.多表连接

> 连接除了可以两表连接，还可以两个以上的表进行连接，称为多表连接

```sql
/*查询每个学生学号，姓名，选修课程名及成绩*/
SELECT Student.Sno,Sname,Cname,Grade FROM Student,SC,Course WHERE Student.Sno=SC.Sno AND SC.Cno=Course.Cno;
```

- 数据库进行多表连接的时候，通常先进行两个表的连接操作，再将连接结果与第三个表进行连接

# 三.嵌套查询

> SQL中一个`SELECT-FROM-WHERE`语句称为一个**查询块**，将一个查询块嵌套再另一个查询块的`WHERE`子句或者`HAVING`短语的条件中的查询称为**嵌套查询**

```sql
SELECT Sname FROM Student WHERE Sno IN  /*外层查询或父查询*/
(SELECT Sno FROM SC WHERE Cno='2');     /*内层查询或子查询*/
```

- 子查询不能使用`ORDER BY`子句，`ORDER BY`只能对最终查询结果进行排序

## 1.带有IN谓词的子查询

> 在嵌套查询中，子查询结果往往是一个**集合**，所以谓词IN是嵌套查询中最经常使用的谓词

```sql
/*查询和刘晨在同一个系学习的学生*/
SELECT Sno,Same,Sdept FROM Student WHERE Sdept IN
(SELECT Sdept FROM Student WHERE Sname='刘晨');
```

- 子查询不依赖父查询，称为**不相关子查询**

## 2.带有比较运算符的子查询

> 带有比较运算符的子查询是指父查询与子查询之间用比较运算符进行连接
>
> 当确切知道**内层查询返回的是单个值**时，可以用比较运算符

```sql
/*找出每个学生超过他自己选修课程平均成绩的课程号*/
SELECT Sno,Cno FROM SC x WHERE Grade>=(SELECT AVG(Grade) FROM SC y WHERE y.Sno=x.Sno);
```

- 子查询依赖父查询的值，称为**相关子查询**

### 相关子查询的步骤

1. 从外层循环中取出一个元组，将内层需要的属性值传递进去
2. 执行内层查询，得到值并将得到的值代入外层查询，外层执行查询
3. 反复执行外部查询并带入内部查询再执行外部查询

求解相关子查询不能像求解不相关子查询那样一次将子查询求解出来，然后求解父查询。内层查询由于与外层求解有关，所以必须**反复求值**

## 3.带有ANY(SOME)或ALL谓词的子查询

> 子查询返回单值时可以用比较运算符，但是返回多值时要用`ANY`（有的系统用`SOME`）或`ALL`谓词修饰符
>
> 使用`ANY`或`ALL`谓词时则**必须同时使用比较运算符**

![](https://raw.githubusercontent.com/MrWater233/PictureHost/master/20200807234047.png)

```sql
/*查询非计算系中比计算机系中任意一个学生年龄小的学生*/
/*通俗来说就是找出非计算机系中年龄比计算机系最大年龄小的学生*/
SELECT Sname,Sage FROM Student WHERE Sage<ANY(SELECT Sage FROM Student WHERE Sdept='CS') AND Sdept<>'CS'

/*用聚集函数实现上面的相同的查询*/
SELECT Sname,Sage FROM Student WHERE Sage<(SELECT MAX(Sage) FROM Student WHERE Sdept='CS') AND Sdept<>'CS';
```

- 用聚集函数实现子查询通常比用`ANY`或`ALL`效率高

- `ANY`,`ALL`谓词与聚集函数，`IN`谓词的转换关系

  ![](https://raw.githubusercontent.com/MrWater233/PictureHost/master/20200807234858.png)

  举例：`=ANY` 等价于`IN`谓词

## 4.带有EXISTS谓词的子查询

> `EXISTS`代表量词`∃`（存在），带有`EXISTS`谓词的子查询不反悔任何数据，只产生逻辑真值`true`或者逻辑假值`false`

```sql
/*查询所有选修了1号课程的学生姓名*/
SELECT Sname FROM Student WHERE EXISTS(SELECT * FROM SC WHERE Sno=Student.Sno AND Cno='1');
```

- 若内层查询非空，则外层的`WHERE`子句返回真值，否则返回假值
- `EXISTS`引出的子查询，子查询目标列表达式通常用`*`，因为子查询只返回真假，给出列名无意义
- 上面例子是**相关子查询**，会从外层逐一取值带入内层最后带入外层

```sql
/*查询没有选修1号课程的学生姓名*/
SELECT Sname FROM Student WHERE NOT EXISTS (SELECT * FROM SC WHERE Sno=Student.Sno AND Cno='1');
```

- `NOT EXISTS` 当内层为空外层`WHERE`返回真值，否则假值

```sql
/*查询选修了全部课程的学生姓名*/
/*等价于：没有一门课程他不选修*/
/*先取出一个学生，再从课程表中取所有课程看该学生学习的课程是否在里面*/
SELECT Sname
FROM Student
WHERE NOT EXISTS
		  (SELECT *
           FROM Course
           WHERE NOT EXISTS
          			 (SELECT *
                      FROM SC
                      WHERE Sno=Student.Sno
                      AND Cno=Course.Cno));
```

# 四.集合查询

> 多个`SELECT`语句的结果可以进行集合操作
>
> 集合操作主要包括`UNION`（并）`INTERSECT`（交）`EXCEPT`（差）

- 参加集合操作的各个查询结果的**列数**必须相同，对应项的**数据类型**也必须相同

## 1.并

```sql
/*查询计算机系的学生和年龄不大于19岁的学生*/
SELECT * FROM Student WHERE Sdept='CS' UNION SELECT * FROM Student WHERE Sage<=19;
```

- `UNION`会将多个查询结果合并然后自动去掉重复的元组，如果要保留重复元组则用`UNION ALL`操作符

## 2.交

```sql
/*查询计算机系学生与年龄不大于19岁学生的交集*/
/*也就是擦汗寻计算机系中年龄不大于19岁的学生*/
SELECT * FROM Student WHERE Sdept='CS' INTERSECT SELECT * FROM Student WHERE Sage<=19;
```

## 3.差

```sql
/*查询计算机系学生与年龄不大于19岁的学生的差集*/
/*也就是查询计算机系中年龄大于19岁的学生*/
SELECT * FROM Student WHERE Sdept='CS' EXCEPT SELECT * FROM Student WHERE Sage<=19;
```

# 五.基于派生表的查询

> 子查询出现在`FROM`中，这时子查询生成的**临时派生表**称为主查询的查询对象

```sql
/*找出每个学生超过他自己选修课平均成绩的课程号*/
SELECT Sno,Cno FROM SC,(SELECT Sno,AVG(Grade) FROM SC GROUP BY Sno) AS Avg_sc(avg_sno,avg_grade) WHERE SC.Sno=Avg_sc.avg_sno AND SC.Grade>=Avg_sc.avg_grade;
```

- `AS`可以省略，但是必须为派生表指定别名
- 如果子查询中没有聚集函数，派生表可以不指定属性列，子查询`SELECT`子句后面的列名为其默认属性(???书p114)