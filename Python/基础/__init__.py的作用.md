# 一.`__init__.py`的主要的作用

- Python中package的标识，不能删除

  `__init__.py`文件的作用是将文件夹变为一个Python模块,Python 中的每个模块的包中，都有`__init__.py`文件。

  通常`__init__.py` 文件为空，但是我们还可以为它增加其他的功能。我们在导入一个包时，实际上是导入了它的`__init__.py`文件。这样我们可以在`__init__.py`文件中批量导入我们所需要的模块，而不再需要一个一个的导入。

  `model/__init__.py`

  ```python
  import re
  import urllib
  import sys
  import os
  ```

  `main.py`

  ```python
  import model 
  print(package.re, package.urllib, package.sys, package.os)
  ```

  注意这里访问`__init__.py`文件中的引用文件，需要加上包名

- 定义`__all__`用来模糊导入

  `model/__init__.py`

  ```python
  import re
  import urllib
  import sys
  import os
  
  __all__ = ['os', 'sys', 're', 'urllib']
  ```

  `main.py`

  ```python
  from package import *
  ```

  这时就会把注册在__init__.py文件中__all__列表中的模块和包导入到当前文件中来

- 编写Python代码（但是不建议在`__init__`中编写Python模块，可以在包中另外新建py文件，应该尽量保证`__init__.py`文件的简单）

# 二.设计`__init__.py`的目的

将多个py文件组成一个包，便于维护和使用，这样能有限地避免命名冲突

比如有如下包结构：

```python
package
  |- subpackage1
      |- __init__.py
      |- a.py
  |- subpackage2
      |- __init__.py
      |- b.py
```

那么有如下几种导入方式

```python
import subpackage1.a # 将模块subpackage.a导入全局命名空间，例如访问a中属性时用subpackage1.a.attr
from subpackage1 import a #　将模块a导入全局命名空间，例如访问a中属性时用a.attr_a
from subpackage.a import attr_a # 将模块a的属性直接导入到命名空间中，例如访问a中属性时直接用attr_a
```

# 三.如何编写`__init__.py`

可以什么都不写，但如果想使用`from package1 import *`这种写法的话，需要在__init__.py中加上：

```python
__all__ = ['file1','file2'] #package1下有file1.py,file2.py
```