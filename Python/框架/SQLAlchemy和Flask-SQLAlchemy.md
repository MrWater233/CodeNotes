> SQLAlchemy是一个关系型数据库框架，它提供了高层的 ORM 和底层的原生数据库的操作，让开发者不用直接和 SQL 语句打交道，而是通过 Python 对象来操作数据库，在舍弃一些性能开销的同时，换来的是开发效率的较大提升。
>
> Flask-SQLAlchemy是一个简化了 SQLAlchemy 操作的flask扩展，是SQLAlchemy的具体实现，封装了对数据库的基本操作。

# 一.SQLAlchemy

> SQL Alchemy是python中最著名的ORM(Object Relationship Mapping)框架。
>
> **ORM：**对象关系映射。即将用户定义的Python类与数据库表相关联，并将这些类（对象）的实例与其对应表中的行相关联。

## 1.1 安装

```shell
pip install sqlalchemy,pymysql
```

## 1.2 基本使用

### 1.2.1 连接数据库

`main.py`

```python
from sqlalchemy import create_engine

engine = create_engine("mysql+pymysql://{username}:{password}@{host}:{port}/{db_name}?charset=utf8mb4")
```

参数的含义：`数据库+数据库连接框架://用户名:密码@IP地址:端口号/数据库名称?连接参数`

### 1.2.2 声明映射模型

`model/user.py`

```python
from sqlalchemy import Column, String, Integer
from sqlalchemy.ext.declarative import declarative_base

# 声明性基类
Base = declarative_base()


# 构建数据库表的映射模型
class User(Base):
    __tablename__ = 'user'

    id = Column(Integer, primary_key=True)
    username = Column(String)
    password = Column(String)

    # 这个是可选的方法，这里自定义在显示User对象时的格式
    def __repr__(self):
        return "<User(id='%s', username='%s', password='%s')>" % (
            self.id, self.username, self.password)

```

- 使用Declarative系统映射的类是根据基类定义的，该基类维护相对于该基类的累和表的目录-这称为声明性基类。我们的应用程序通常在一个常用的模块中只有一个这个基础的实例。我们使用`declarative_base()`函数创建基类。
- `__tablename__`变量声明了在根据这个类创建表时表的名字。

现在我们有了一个类`Base`，我们可以根据它定义任意数量的映射类。上面我们根据它定义了一个`user`表，表中含有字段`id`、`username`、`password`

### 1.2.3 创建映射类的实例

`main.py`

```python
user1 = User(id=1, username="jack", password="123456")
user2 = User(id=2, username="kitty", password="1234")
```

### 1.2.4 创建会话

上面我们创建了连接数据库的引擎（`engine`），这里需要引擎来创建一个`session`，后面才能通过`session`与数据库进行交互。

```python
Session = sessionmaker(engine)
db_session = Session()
```

### 1.2.5 单表的增删改查

以下每个代码块的前缀：

```python
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker

from model.user import User

# 准备工作
engine = create_engine("mysql+pymysql://{username}:{password}@{host}:{port}/{db_name}?charset=utf8mb4")

Session = sessionmaker(engine)
db_session = Session()
```

#### 1.2.5.1 add

- 增加一条数据

   ```python
   db_session.add(User(username="Jack", password="123456"))
   db_session.commit()
   # 关闭session
   db_session.close()
   ```

- 增加多条数据

  ```python
  user1 = User(username="Tom", password="123456")
  user2 = User(username="Kitty", password="123456")
  db_session.add_all([user1, user2])
  db_session.commit(）
  # 关闭session
  db_session.close()
  ```

若`db_session.add()`将错误信息插入到了数据库中，可以使用`db_session.rollback()`进行回滚

#### 1.2.5.2 query

- 查询SQL语句

  ```python
  res = db_session.query(User)
  print(res)
  # 关闭session
  db_session.close()
  ```

  ```shell
  SELECT user.id AS user_id, user.username AS user_username, user.password AS user_password 
  FROM user
  ```

- 查询表中所有数据

  ```python
  res = db_session.query(User).all()
  print(res)
  # 关闭session
  db_session.close()
  ```

  ```shell
  [<User(id='1', username='Jack', password='123456')>, <User(id='2', username='Tom', password='123456')>, <User(id='3', username='Kitty', password='123456')>]
  ```

- 查询第一个数据

  ```shell
  res = db_session.query(User).first()
  print(res)
  # 关闭session
  db_session.close()
  ```

  ```shell
  <User(id='1', username='Jack', password='123456')>
  ```

- 条件查询

  ```python
  res = db_session.query(User).filter(User.id < 2, User.username == "Jack").all()
  print(res)
  # 关闭session
  db_session.close()
  ```

  ```shell
  [<User(id='1', username='Jack', password='123456')>]
  ```

#### 1.2.5.3 update

- 全部修改

  ```python
  res = db_session.query(User).update({"password": "111"}) # 修改符合条件的数据，返回修改的数据个数
  print(res)
  db_session.commit()
  # 关闭session
  db_session.close()
  ```

  ```shell
  3
  ```

- 根据条件修改

  ```python
  res = db_session.query(User).filter(User.id == 1).update({"username": "kite"}) # 全部修改，返回修改的数据个数
  print(res)
  db_session.commit()
  # 关闭session
  db_session.close()
  ```

  ```shell
  1
  ```

#### 1.2.5.4 delete

- 全部删除

  ```python
  res = db_session.query(User).delete() # 删除表中所有数据，并返回删除数量
  print(res)
  db_session.commit()
  # 关闭session
  db_session.close()
  ```

  ```shell
  3
  ```

- 根据条件进行删除

  ```python
  res = db_session.query(User).filter(User.id == 1).delete() # 根据条件删除，比返回删除的数量
  print(res)
  db_session.commit()
  # 关闭session
  db_session.close()
  ```

  ```shell
  1
  ```

## 1.3 复杂关系

https://blog.csdn.net/tianpingxian/article/details/82720442

### 1.3.1 一对多

#### 1.3.1.1 表设计

- `parent`

  ```sql
  CREATE TABLE `parent` (
    `id` int(11) NOT NULL AUTO_INCREMENT,
    `name` varchar(64) NOT NULL,
    PRIMARY KEY (`id`)
  ) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4
  ```

- `child`

  ```sql
  CREATE TABLE `child` (
    `id` int(11) NOT NULL AUTO_INCREMENT,
    `name` varchar(64) NOT NULL,
    `parent_id` int(11) NOT NULL,
    PRIMARY KEY (`id`)
  ) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4
  ```

父亲：Jack

儿子：Tom、Kitty

#### 1.3.1.2 映射关系

- `model/base.py`

  ```python
  from sqlalchemy.ext.declarative import declarative_base
  
  Base = declarative_base()
  ```

- `model/parent.py` （一）

  ```python
  from sqlalchemy import Column, String, Integer
  
  from sqlalchemy.orm import relationship
  
  from model.base import Base
  
  
  class Parent(Base):
      __tablename__ = "parent"
      id = Column(Integer, primary_key=True)
      name = Column(String(64), nullable=False)
      children = relationship("Child")
  ```

- `model/child.py`（多）

  ```python
  from sqlalchemy import Column, String, Integer, ForeignKey
  
  from model.base import Base
  
  
  class Child(Base):
      __tablename__ = "child"
      id = Column(Integer, primary_key=True)
      name = Column(String(64), nullable=False)
      parent_id = Column(Integer, ForeignKey('parent.id'))
      
      def __repr__(self):
          return self.name
  ```

**注意这里的`parent`和`child`使用的是统一的  `Base`**

#### 1.3.1.3 增加

`main.py`

```python
parent = Parent(name="Lucy")
parent.children = [Child(name="Shanks"), Child(name="Betty")]
db_session.add(parent)
db_session.commit()
```

#### 1.3.1.4 查询

`main.py`

```python
parent = db_session.query(Parent).filter(Parent.name == "Jack").first()
print(parent.children)  # [Tom, Kitty]
parent = db_session.query(Parent).filter(Parent.children.any(Child.name == "Tom")).first()
print(parent.children)  # [Tom, Kitty]
```

### 1.3.2 多对一

相比一对多而言多对一是双向关系

#### 1.3.2.1 表设计

同上

#### 1.3.2.2 映射关系

- `model/base.py`同上

- `model/parent.py`

  ```python
  class Parent(Base):
      __tablename__ = "parent"
      id = Column(Integer, primary_key=True)
      name = Column(String(64), nullable=False)
      # 这儿增加了back_populates和lazy参数
      children = relationship("Child", back_populates="parent", lazy="dynamic")
  
      def __repr__(self):
          return self.name
  ```

- `model/child.py`

  ```python
  class Child(Base):
      __tablename__ = "child"
      id = Column(Integer, primary_key=True)
      name = Column(String(64), nullable=False)
      parent_id = Column(Integer, ForeignKey('parent.id'))
      # 增加了parent属性
      parent = relationship("Parent", order_by="Parent.id", back_populates="children")
  
      def __repr__(self):
          return self.name
  ```

> 1. `backref`和`back_populates`
>
>    两个参数的效果完全一致，就是让我们能通过`child.parent`这样的方法来的到`child`的`parent`。
>
>    两个参数的不同点是`backref` 只需要在 `Parent` 类中声明 `children`，`Child.parent` 会被动态创建。而 `back_populates` 必须在两个类中显式地使用 `back_populates`，更显繁琐。（但是也更清晰？）
>
> 2. `lazy`
>
>    http://shomy.top/2016/08/11/flask-sqlalchemy-relation-lazy/
>
>    该属性指定sqlalchemy数据库什么时候加载数据
>
>    - `select`：就是访问到属性的时候，就会全部加载该属性的数据
>    - `joined`：对关联的两个表使用联接
>    - `subquery`：与joined类似，但使用子子查询
>    - `dynamic`：不加载记录，但提供加载记录的查询，也就是生成query对象。可以接着使用`.all()`或者`.first()`之类的查询执行器来获得记录

#### 1.3.2.3 增加

同上

#### 1.3.2.4 查询

`main.py`

```python
# 通过父亲来查询儿子，lazy="dynamic"的作用
parent = db_session.query(Parent).get(1)
print(parent.children)  # SELECT child.id AS child_id, child.name AS child_name, child.parent_id AS child_parent_id FROM child WHERE %(param_1)s = child.parent_id
print(parent.children.all())  # [Tom, Kitty]
print(parent.children.filter(Child.name == 'Tom').first())  # Tom

# 通过儿子来查询父亲
child = db_session.query(Child).get(1)
print(child.parent)  # Jack

# 多对一查询相同父亲的儿子
children = db_session.query(Child).filter(Child.parent.has(Parent.name == 'Jack')).all()
print(children)  # [Tom, Kitty]
```

### 1.3.3 多对多

#### 1.3.3.1 表设计

多对多查询需要增加中间表

`parent_child`

```sql
CREATE TABLE `parent_child` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `parent_id` int(11) NOT NULL,
  `child_id` int(11) NOT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4
```

#### 1.3.3.2 映射关系

- `model/base.py`同上

- `parent.py`

  ```python
  class Parent(Base):
      __tablename__ = "parent"
      id = Column(Integer, primary_key=True)
      name = Column(String(64), nullable=False)
      # 增加了secondary属性
      children = relationship("Child", secondary=parent_child_association_table, back_populates="parents")
      # 也可以使用backref属性来替代back_populates，这样child中就不需要写parents属性了
      # children = relationship("Child",secondary=association_table, backref="parents")
  
      def __repr__(self):
          return self.name
  ```

- `child.py`

  ```python
  class Child(Base):
      __tablename__ = "child"
      id = Column(Integer, primary_key=True)
      name = Column(String(64), nullable=False)
      # 增加了secondary属性
      parents = relationship("Parent", secondary=parent_child_association_table, back_populates="children")
  
      def __repr__(self):
          return self.name
  ```

- `parent_child.py`

  ```python
  parent_child_association_table = Table('parent_child', Base.metadata,
                                         Column('parent_id', Integer, ForeignKey('parent.id')),
                                         Column('child_id', Integer, ForeignKey('child.id')))
  ```

#### 1.3.3.3 增加

`main.py`

```python
parent = Parent(name="Mike")
parent.children = [Child(name="Kite"), Child(name="Steve")]
db_session.add(parent)
db_session.commit()
# 也可以通过以下方式添加
child = Child(name="Clid")
child.parents = [Parent(name="Blank"), Parent(name="Bland")]
db_session.add(child)
db_session.commit()
```

#### 1.3.3.4 查询

`main.py`

```python
parent = db_session.query(Parent).get(1)
print(parent.children)  # [Tom, Kitty]

child = db_session.query(Child).get(1)
print(child.parents)  # [Jack, Jessy]
```

#### 1.3.3.5 删除

- 删除中间关系

  ```python
  parent = db_session.query(Parent).filter(Parent.name == 'Bland').one()
  parent.children = []
  db_session.commit()
  ```

- 删除两表的对应关系

  这儿删除了Mike和Kite的对应关系

  ```python
  parent = db_session.query(Parent).filter(Parent.name == 'Mike').one()
  child = db_session.query(Child).filter(Child.name == 'Kite').one()
  parent.children.remove(child)
  db_session.commit()
  ```

- 删除实体和对应关系删除关系

  这儿删除了`child`和`parent_child`表中的数据，`parent`表的数据并未被删除

  ```python
  child = db_session.query(Child).filter(Child.name == 'Steve').one()
  db_session.delete(child)
  db_session.commit()
  ```

# 二.Flask-SQLAlchemy

> Flask-SQLAlchemy 是一个为您的 Flask 应用增加 SQLAlchemy 支持的扩展。它需要 SQLAlchemy 0.6 或者更高的版本。它致力于简化在 Flask 中 SQLAlchemy 的使用，提供了有用的默认值和额外的助手来更简单地完成常见任务。

## 2.1 安装

```shell
pip install flask-sqlalchemy, pymysql
```

## 2.2 基本使用

### 2.2.1 连接数据库

`database.py`

```python
from flask import Flask
from flask_sqlalchemy import SQLAlchemy

# 实例化SQLAlchemy
db = SQLAlchemy()
# PS : 实例化SQLAlchemy的代码必须要在引入蓝图之前

app = Flask(__name__)

# 配置数据库连接
app.config["SQLALCHEMY_DATABASE_URI"] = "mysql+pymysql://{username}:{password}@{host}:{port}/{db_name}?charset=utf8mb4"
app.config["SQLALCHEMY_TRACK_MODIFICATIONS"] = False
app.app_context().push()
# 初始化SQLAlchemy , 本质就是将以上的配置读取出来
db.init_app(app)

```

### 2.2.2 创建映射模型

`model/user.py`

```python
from database import db


class User(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(64), unique=True)
    password = db.Column(db.String(64))

    def __repr__(self):
        return '<User %r>' % self.username
```

### 2.2.3 单表的增删改查

#### 2.2.3.1 add

```python
guest = User(username="guest", password="123456")
db.session.add(guest)
db.session.commit()
```

增加数据的步骤：

1. 创建模型对象
   语法：模型对象 = 模型类(字段名=字段值)

2. 将模型对象添加到会话中
   语法：组件对象.session.add(模型对象)

3. 提交会话
   语法：组件对象.session.commit()

#### 2.2.3.2 query

```python
users = User.query.all()
print(users)  # [<User 'Tom'>, <User 'Kitty'>, <User 'admin'>, <User 'guest'>]

user = User.query.filter(User.username == 'guest').first()
print(user)  # <User 'guest'>
```

- 常用查询执行器

  ![](https://gitee.com/mrwater233/PictureHost/raw/master/2021/202112131752815.png)

  执行器特点：

  - 将整个查询语句转换为SQL语句并执行查询
  - 在查询语句的末尾设置, 每条查询语句只能设置一个执行器

- 常用查询过滤器

  ![](https://gitee.com/mrwater233/PictureHost/raw/master/2021/202112131753089.png)

  过滤器特点：

  - 只负责设置过滤条件, 不会执行查询(查询由执行器来完成)
  - 允许续接其他过滤器 或 执行器

#### 2.2.3.3 update

- 先查询再更新

  ```python
  user = User.query.filter(User.username == 'guest').first()
  user.password = "123"
  db.session.add(user)
  db.session.commit()
  ```

  缺点：

  - 查询更新分为两条语句，效率低
  - 如果并发更新，可能出现更新丢失的问题

- 基于过滤条件的更新

  ```python
  User.query.filter(User.username == 'guest').update({"password": User.password + "123"})
  db.session.commit()
  ```

#### 2.2.3.4 delete

- 先查询再删除

  ```python
  user = User.query.filter(User.username == 'guest').first()
  db.session.delete(user)
  db.session.commit()
  ```

  缺点：效率低

- 基于过滤条件的删除

  ```python
  User.query.filter(User.username == 'admin').delete()
  db.session.commit()
  ```

  优点：

  - 效率高
  - 能够批量删除

## 2.3 复杂关系

### 2.3.1 一对多

#### 2.3.1.1 表设计

- `person`

  ```sql
  CREATE TABLE `person` (
    `id` int(11) NOT NULL AUTO_INCREMENT,
    `name` varchar(64) NOT NULL,
    PRIMARY KEY (`id`)
  ) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4
  ```

- `address`

  ```sql
  CREATE TABLE `address` (
    `id` int(11) NOT NULL AUTO_INCREMENT,
    `email` varchar(64) NOT NULL,
    `person_id` int(11) NOT NULL,
    PRIMARY KEY (`id`)
  ) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4
  ```

#### 2.3.1.2 映射关系

- `model/person.py`

  ```python
  from database import db
  # 这儿需要导入，不然会因为找不到类而报错
  from model.address import Address
  
  
  class Person(db.Model):
      id = db.Column(db.Integer, primary_key=True)
      name = db.Column(db.String(64))
      addresses = db.relationship('Address', backref='person', lazy='dynamic')
  
      def __repr__(self):
          return self.name
  ```

- `model/address.py`

  ```python
  from database import db
  
  
  class Address(db.Model):
      id = db.Column(db.Integer, primary_key=True)
      email = db.Column(db.String(64))
      person_id = db.Column(db.Integer, db.ForeignKey("person.id"))
  
      def __repr__(self):
          return self.email
  ```

#### 2.3.1.3 增加

`main.py`

```python
person = Person(name="Deny")
person.addresses = [Address(email="1@qq.com"), Address(email="45@qq.com")]
db.session.add(person)
db.session.commit()
```

#### 2.3.1.4 查询

`main.py`

```python
person = Person.query.filter(Person.name == 'mike').first()
print(person.addresses.all())  # [1234@qq.com]
```

### 2.3.2 多对多

#### 2.3.2.1 表设计

- `article`

  ```sql
  CREATE TABLE `article` (
    `id` int(11) NOT NULL AUTO_INCREMENT,
    `title` varchar(64) DEFAULT NULL,
    PRIMARY KEY (`id`)
  ) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4
  ```

- `tag`

  ```sql
  CREATE TABLE `tag` (
    `id` int(11) NOT NULL AUTO_INCREMENT,
    `name` varchar(64) NOT NULL,
    PRIMARY KEY (`id`)
  ) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4
  ```

- `article_tag`

  ```sql
  CREATE TABLE `article_tag` (
    `id` int(11) NOT NULL AUTO_INCREMENT,
    `article_id` int(11) NOT NULL,
    `tag_id` int(11) NOT NULL,
    PRIMARY KEY (`id`)
  ) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4
  ```

#### 2.3.2.2 映射关系

- `model/article.py`

  ```python
  from database import db
  from .article_tag import article_tag
  from .tag import Tag
  
  
  class Article(db.Model):
      id = db.Column('id', db.Integer, primary_key=True)
      title = db.Column('title', db.String(64), nullable=False)
      # 假如拿到了一个标签Tag,怎么拿到标签下的所有文章呢.反向引用Article这时用backref。并给tag的articles加上lazy属性（并不是给tags加上lazy）
      tags = db.relationship('Tag', secondary=article_tag, backref=db.backref('articles', lazy='dynamic'))
  
      def __repr__(self):
          return self.title
  ```

- `model/tag.py`

  ```python
  from database import db
  
  
  class Tag(db.Model):
      id = db.Column('id', db.Integer, primary_key=True)
      name = db.Column('name', db.String(64), nullable=False)
  
      def __repr__(self):
          return self.name
  ```

- `model/article_tag.py`

  ```python
  from database import db
  
  article_tag = db.Table('article_tag',
                         db.Column('id', db.Integer, primary_key=True),
                         db.Column('article_id', db.Integer, db.ForeignKey('article.id')),
                         db.Column('tag_id', db.Integer, db.ForeignKey('tag.id')))
  ```

#### 2.3.2.3 增加

`main.py`

```python
article = Article(title="文章1")
article.tags = [Tag(name='标签1'), Tag(name='标签2')]
db.session.add(article)
db.session.commit()
```

#### 2.3.2.4 查询

`main.py`

```python
if __name__ == '__main__':
    article = Article.query.filter(Article.title == '文章1').first()
    print(article.tags)  # [标签1, 标签2]
    tag = Tag.query.filter(Tag.name == '标签1').first()
    # 因为backref的里面加上了lazy，所以还需要用all()来取出数据
    print(tag.articles.all())  # [文章1]
```

# 三.参考

- https://www.cnblogs.com/ChangAn223/p/11272532.html