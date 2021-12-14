> pydantic是一种常用的接口数据定义与校验的库

# 一.基本使用

## 1.1 schema基本定义方法

pydantic库的数据定义方式是通过`BaseModel`类来进行定义的，所有基于pydantic的数据类型本质上都是一个`BaseModel`类，它最基本的使用方式如下：

```python
from pydantic import BaseModel

class Person(BaseModel):
    name: str
```

## 1.2 schema基本实例化方法

需要校验调用时，我们直接对它进行实例化即可，实例化的方法有如下几种：

- 直接传值

  ```python
  p = Person(name="Tom")
  print(p.json())
  ```

- 通过字典传入

  ```python
  p = {"name": "Tom"}
  p = Person(**p)
  print(p.json())
  ```

  这也是在Web开发中校验入参的常用方式，比如在Flask中：

  ```python
  @app.route("/test")
  def test():
    req_json = reqeust.json
    p = Person(**req_json)
    print(p.json())
  ```

- 通过其他的实例化对象传入

  ```python
  p = Person(name="Tom")
  p2 = Person.copy(p)
  print(p2.json())
  ```



当传入值错误的时候，pydantic就会抛出错误，如：

```python
Person(age=18)
```

pydantic就会抛出异常：

```shell
pydantic.error_wrappers.ValidationError: 1 validation error for Person
name
  field required (type=value_error.missing)
```

我们可以通过异常的捕获来处理抛出的异常

```python
try:
		p = Person(age=18)
except pydantic.ValidationError as e:
    print(e.json())
```



另一方面，如果传入的值多于定义值，BaseModel也会对它进行过滤，比如：

```python
p = Person(name="Tom", gender="man", age=24)
print(p.json())  # {"name": "Tom"}
```

通过这种方式，**数据的传递将会更加的安全**

此外，pydantic在数据传输时会直接进行数据类型的转换，例如：

```python
p = Person(name=123)
print(p.json())  # {"name": "123"}
```

## 1.3 pydantic基本数据类型

```python
from pydantic import BaseModel
from typing import Dict, List, Sequence, Set, Tuple

class Demo(BaseModel):
    a: int  # 整型
    b: float  # 浮点型
    c: str  # 字符串
    d: bool  # 布尔型
    e: List[int]  # 整型列表
    f: Dict[str, int]  # 字典型，key为str，value为int
    g: Set[int]  # 集合
    h: Tuple[str, int]  # 元组
```

# 二.高级数据结构

这里我们给出一些较为复杂的数据结构实现：

## 2.1 可选数据类型

如果一个数据类型不是必须的，那么我们可以使用typing库中的Optional方法进行实现。

例子如下：

```python
from typing import Optional
from pydantic import BaseModel

class Person(BaseModel):
    name: str
    age: Optional[int]
```

需要注意的是设置为可选之后，数据中仍然会有age字段，但是默认值为None。即当不传入age字段时，Person仍然可以取到age，只是其值为None

例子如下：

```python
p = Person(name="Tom")
print(p.json())  # {"name": "Tom", "age": None}
```

## 2.2 数据默认值的设置

```python
from pydantic import BaseModel

class Person(BaseModel):
    name: str
    gender: str = "man"

p = Person(name="Tom")
print(p.json())  # {"name": "Tom", "gender": "man"}
```

## 2.3 允许多种数据类型

如果一个数据可以允许多种数据类型，我们可以通过typing库中的Union方法进行实现。

例子如下：

```python
from typing import Union
from pydantic import BaseModel

class Time(BaseModel):
    time: Union[int, str]
        
t = Time(time=12345)
print(t.json())  # {"time": 12345}
t = Time(time = "2020-7-29")
print(t.json())  # {"time": "2020-7-29"}
```

## 2.4 异名数据传递

有如下场景，我们之前定义了一个schema，我们将其中一个参数命名为了A，但是在后续的定义中，我们希望这个参数被命名为B。那么我们如何完成这两个不同名称参数的相互传递呢？

我们可以通过Field方法来实现这个操作

例子如下：

```python
from pydantic import BaseModel, Field

class Password(BaseModel):
    password: str = Field(alias = "key")
```

在传入的时候，我们需要使用`key`关键字来传入`password`变量

代码如下：

```python
p = Password(key="123456")
print(p.json()) # {"password": "123456"}
```

# 三.多级schema定义

这里给出一个复杂的多级的公司的定义：

```python
from enum import Enum
from typing import List, Union
from datetime import date
from pydantic import BaseModel

class Gender(str, Enum):
    man = "man"
    women = "women"

class Person(BaseModel):
    name : str
    gender : Gender
        
class Department(BaseModel):
    name : str
    lead : Person
    cast : List[Person]
        
class Group(BaseModel):
    owner: Person
    member_list: List[Person] = []

class Company(BaseModel):
    name: str
    owner: Union[Person, Group]
    regtime: date
    department_list: List[Department] = []
```

除了一步一步实例化意外，如果我们已经有了一个完整的Company字典，我们可以一步到位地进行实例化

如：

```python
sales_department = {
    "name": "sales",
    "lead": {"name": "Sarah", "gender": "women"},
    "cast": [
        {"name": "Sarah", "gender": "women"},
        {"name": "Bob", "gender": "man"},
        {"name": "Mary", "gender": "women"}
    ]
}

research_department = {
    "name": "research",
    "lead": {"name": "Allen", "gender": "man"},
    "cast": [
        {"name": "Jane", "gender": "women"},
        {"name": "Tim", "gender": "man"}
    ]
}

company = {
    "name": "Fantasy",
    "owner": {"name": "Victor", "gender": "man"},
    "regtime": "2020-7-23",
    "department_list": [
        sales_department,
        research_department
    ]
}

company = Company(**company)
```

# 四.数据检查方法

pydantic本身提供了上述基本类型的数据检查， 除此之外，我们也可以使用`validator`和`config`方法来实现较为复杂的数据类型定义和检查

## 4.1 validator

使用validator方法，我们可以对数据进行更为复杂的数据检查

```python
import re
from pydantic import BaseModel, validator

class Password(BaseModel):
    password: str

    @validator("password")
    def password_rule(cls, password):
        def is_valid(password):
            if len(password) < 6 or len(password) > 20:
                return False
            if not re.search("[a-z]", password):
                return False
            if not re.search("[A-Z]", password):
                return False
            if not re.search("\d", password):
                return False
            return True

        if not is_valid(password):
            raise ValueError("password is invalid")
```

## 4.2 config

如果要对BaseModel中某一基本类型进行统一的格式要求，我们可以使用Conifg方法来实现

```python
from pydantic import BaseModel

class Password(BaseModel):
    password: str
        
    class Config:
        min_anystr_length = 6 # 令Password类中所有的字符串长度均要不少于6
        max_anystr_length = 20 # 令Password类中所有的字符串长度均要不大于20
```

这里特殊关键关键字用法只给出了两个，更多可以参考[官方文档](https://pydantic-docs.helpmanual.io/usage/model_config/)

# 五.参考

- https://pydantic-docs.helpmanual.io/
- https://blog.csdn.net/codename_cys/article/details/107675748