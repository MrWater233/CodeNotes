# 一.快速开始

## 1.1 最小的Python应用

```python
from flask import Flask
app = Flask(__name__)

@app.route('/')
def hello_world():
    return 'Hello World!'

if __name__ == '__main__':
    app.run()
```

1. 首先我们创建了一个类的实例，第一个参数是应用模块或者包的名称。如果使用单一的模块（如本例），应该使用` __name__ `，因为模块的名称将会因其作为单独应用启动还是作为模块导入而有不同（ 也即是 `__main__` 或实际的导入名），这样 Flask 才知道到哪去找模板、静态文件等等。
2. 然后使用了`route()`装饰器来告诉Flask什么样的URL能够触发我们的函数
3. 最后用 `run()` 函数来让应用运行在本地服务器上。 其中 `if __name__ == '__main__':` 确保服务器只会在该脚本被 Python 解释器直接执行的时候才会运行，而不是作为模块导入的时候。

## 1.2 调试模式

> 调试模式能够在代码修改以后让服务器自动重新载入

有两种方法来开启调试模式

- ```python
  app.debug = True
  app.run()
  ```

- ```pythob
  app.run(debug=True)
  ```

## 1.3 路由

`route()`装饰器能够把一个函数绑定到对应的URL上

基本使用：

```python
@app.route('/')
def index():
    return 'Index Page'
```

> 如果路由最后以`\`结尾，那么他就不是唯一URL，带不带`\`都能访问
>
> 如果路由最后不以`\`结尾，那么他是唯一URL，使用带`\`的URL无法访问

### 1.3.1 动态路由

在URL中使用`<variable_name>`来指定动态路由参数

并且使用`<converter:variable_name>`给参数指定转换器（类型）

```python
@app.route('/user/<username>')
def show_user_profile(username):
    # show the user profile for that user
    return 'User %s' % username

@app.route('/post/<int:post_id>')
def show_post(post_id):
    # show the post with the given id, the id is an integer
    return 'Post %d' % post_id
```

转换器类型：

| 类型  | 作用                       |
| ----- | -------------------------- |
| int   | 接受整数                   |
| float | 同 int ，但是接受浮点数    |
| path  | 和默认的相似，但也接受斜线 |

### 1.3.2 url_for()

> `url_for()`能够根据视图函数名指定url，只要视图函数不变，url随便变都不会影响

- 不带参数

   ![](https://img2018.cnblogs.com/blog/1406024/201910/1406024-20191031200831611-2135915178.png)

  ![](https://img2018.cnblogs.com/blog/1406024/201910/1406024-20191031200936290-993526078.png)

- 带参数

  ```python
  @app.route('/')
  def func1():
      return url_for('func2', page=1)
  
  
  @app.route('/url/<page>')
  def func2(page):
      return 'hello'
  ```

  ![](https://img2018.cnblogs.com/blog/1406024/201910/1406024-20191030223325868-170144027.png)

### 1.3.3 HTTP方法

> 通过`route()`装饰器传递methods参数来指定请求的方法

```python
@app.route('/login', methods=['GET', 'POST'])
def login():
    if request.method == 'POST':
        do_the_login()
    else:
        show_the_login_form()
```

## 1.4 访问请求数据

> Flask中使用全局的`request`对象来提供客户端发送给服务端的数据

### 1.4.1 请求对象

首先从flask模块中导入它

```python
from flask import request
```

- 通过`method`属性来访问请求的HTTP方法

- 通过`form`属性来访问表单属性

  ```python
  request.form['username']
  ```

  如果访问一个不存在的表单数据，那么会抛出一个`KeyError`异常，可以捕获它，如果不捕获它那么会显示HTTP400的错误页面

- 通过`args`属性来访问URL中提交的参数（`?key=value`）

  ```python
  searchword = request.args.get('q', '')
  ```

### 1.4.2  文件上传

首先确保表单中设置`enctype="multipart/form-data"` 属性

上传的文件会存储在内存或者文件系统中一个临时的位置，可以通过`request`的`files`属性来访问他们。通过`save()`方法来讲文件保存到文件系统上

```python
from flask import request

@app.route('/upload', methods=['GET', 'POST'])
def upload_file():
    if request.method == 'POST':
        f = request.files['the_file']
        f.save('/var/www/uploads/uploaded_file.txt')
    ...
```

可以通过文件的`filename`属性来获取它上传前的文件名，但是这个值是可以伪造的，所以要把文件按照客户端的文件名来存储建议通过`secure_filename()`函数

```python
secure_filename(f.filename))
```

### 1.4.3 Cookies

可以通过`request`的`cookies`属性来访问Cookies，用响应对象的`set_cookie`来设置Cookies

- 读取Cookies

  ```python
  from flask import request
  
  @app.route('/')
  def index():
      username = request.cookies.get('username')
  ```

- 存储Cookies

  ```python
  from flask import make_response
  
  @app.route('/')
  def index():
      resp = make_response(render_template(...))
      resp.set_cookie('username', 'the username')
      return resp
  ```

## 1.5 重定向和错误

可以使用`redirect()`函数来重定向，并用`abort()`函数来返回错误码

```python
from flask import abort, redirect, url_for

@app.route('/')
def index():
    return redirect(url_for('login'))

@app.route('/login')
def login():
    abort(401)
    # 接下去的方法不会执行
    this_is_never_executed()
```

默认，错误代码会显示一个黑白的错误页面，如果想要定制错误页面，可以使用`errorhandler()`装饰器

```python
from flask import render_template

@app.errorhandler(404)
def page_not_found(error):
    return render_template('page_not_found.html'), 404
```

注意 `render_template()` 调用之后的 `404` 。这告诉 Flask，该页的错误代码是 404 ，即没有找到。默认为 200，也就是一切正常。

## 1.6 会话

除了`request`对象还有`session`对象

如果需要使用会话，需要设置一个密钥

```python
from flask import Flask, session, redirect, url_for, escape, request

app = Flask(__name__)

@app.route('/')
def index():
    if 'username' in session:
        return 'Logged in as %s' % escape(session['username'])
    return 'You are not logged in'

@app.route('/login', methods=['GET', 'POST'])
def login():
    if request.method == 'POST':
        session['username'] = request.form['username']
        return redirect(url_for('index'))
    return '''
        <form action="" method="post">
            <p><input type=text name=username>
            <p><input type=submit value=Login>
        </form>
    '''

@app.route('/logout')
def logout():
    # remove the username from the session if it's there
    session.pop('username', None)
    return redirect(url_for('index'))

# 设置密钥
app.secret_key = 'A0Zr98j/3yX R~XHH!jmN]LWX/,?RT'
```

`escape()`函数能够在模版引擎外做转义

> 获得强壮的密钥：
>
> 操作系统可以基于密钥随机生成器来生成一个漂亮的随机值，这个值可以用来做密钥
>
> ```python
> >>> import os
> >>> os.urandom(24)
> '\xfd{H\xe5<\x95\xf9\xe3\x96.5\xd1\x01O<!\xd5\xa2\xa0\x9fR"\xa1\xa8'
> ```
>
> 复制这个值就能使用了

## 1.7 日志

通过`app.logger`来记录日志

```python
app.logger.debug('A value for debugging')
app.logger.warning('A warning occurred (%d apples)', 42)
app.logger.error('An error occurred')
```

