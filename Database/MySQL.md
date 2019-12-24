启动/关闭服务:

```mysql
net start mysql
```

```mysql
net stop mysql
```

登陆

```mysql
mysql -u root -p
```

创建/删除数据库

```mysql
create database name;
```

```mysql
drop database name;
```

查询数据库

```mysql
show databases;
```

查看当前数据库

```mysql
select database();
```

查看数据库下的表

```mysql
show tables;
```

创建表

```mysql
create table name(
    -> id int primary key auto_increment,--主键约束：primary key，被主键修饰过的字段，唯一非空。一张表只能有一个主键，但是主键可以包含多个字段；auto_increment自增。
    -> name varchar(20),--数据类型varchar为可变长度的字符串。
    -> chinese double(5,2),--该参数长度为5，小数位占两个，最大值：999.99
    -> english double(5,2),
    -> math double(5,2)
)
```

查看表的结构

```mysql
desc name;
```

修改表

```mysql
alter table 表名 change 字段名称 新的字段描述;
alter table 表名 modify 字段名称 字段类型;
```

添加数据

```mysql
insert into scores value(字段值1，字段值2，字段值3...);
```

查看数据

```mysql
select *from 表名;--查看整个表
select 字段一,字段二 from 表名;
```

更新数据

```mysql
update 表名 set 字段名=字段值 where 条件;
```

清空数据库

```sql
truncate TABLE 表名;
```

