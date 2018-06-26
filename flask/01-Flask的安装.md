### Flask的概念
1. Flask ‘微’框架的微不代表 Flask 的功能上是不足的，而是指 Flask 旨在保持代码简洁且易于扩展。
2. Flask 并不包含数据库抽象层，表单验证或者任何其它现有的库( Django )能够处理的。相反，Flask 支持扩展，这些扩展能够添加功能到你的应用，像是 Flask 本身实现的一样。众多的扩展提供了数据库集成，表单验证，上传处理，多种开放的认证技术等功能。Flask 可能是“微”型的，但是已经能够在各种各样的需求中生产使用。
3. Flask与Django的区别：
  django 为完善完整高集成的框架；
  Flask 不包含数据库抽象层微框架，database，templetes需要自己去组装。

### Flask的安装
1. 新建flaskenv文件夹
2. cd flaskenv
3. virtualenv --no-site-packages flaskenv  创建虚拟环境
4. cd flaskenv
5. cd Scripts
6. activate 进入虚拟环境
7. pip install flask 安装flask

### Flask的第一个应用
1. 打开pycharm，新建flask项目
2. 配置虚拟环境：
  file  ——  setting  ——  Add local  ——  Exisiting envirment  ——  Scripts  ——  python.py
3. 新建hello.py文件
  ```
  from flask import Flask
  app = Flask(__name__)  #初始化，__name__代表主模块名或者包

  @app.route('/')  # 默认路由--127.0.0.1:5000
  def hello():  # 视图函数
	  return 'Hello World'


  @app.route('/hello/<name>/')
  def helloname(name):
	  return 'hello %s' % name


  @app.route('/helloint/<int:id>/')
  def hello_int(id):
	  return 'hello int: %s' % (id)


  if __name__ == '__main__':
	  app.run(debug=True, port=8000, host='0.0.0.0')  # debug=True设置调试，port指定端口, host指定IP
  
  # 在Terminal中执行python hello.py可以启动应用。
  ```

4. 修改启动方式
  pip install Flask-Script  安装flask-script包
  ```
  from flask_script import Manager

  manager = Manager(app=app)

  manager.run()
  # 注意：写了manage.run()后，不需要写app.run()
  ```

5. 配置debug
   Run ——  Debug  ——  Edit  ——  + python  ——  设置Scripts path / runserver -p 8080 -d



### Flask的结构修改
1.
![改变后结构](https://img-blog.csdn.net/20180514145827863?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTc4MjA1MA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

2. pip install flask-blueprint 安装蓝图（用于管理urls ）
在views中：
```
from flask import Blueprint

# 初始化(蓝图管理url)
blue = Blueprint('first', __name__)

@blue.route('/')
def hello():
    # 视图函数
    return 'Hello World'
```
在init.py中：
```
from flask import Flask
from App.views import blue


def create_app():

    app = Flask(__name__)
    # 路由注册
    app.register_blueprint(blueprint=blue)

    return app

```

### Flask的route变量规则
1.  route() 装饰器是用于把一个函数绑定到一个 URL 上
2. int (整型)/ float(浮点型) / str(字符串) / path / uuid
```
@blue.route('/helloint/<int:id>/')
def hello_int(id):
    return 'hello int: %s' % (id)


@blue.route('/getfloat/<float:price>/')
def hello_float(price):
    return 'float: %s' % price


@blue.route('/getstr/<string:name>/')
def hello_name(name):
    return 'hello name: %s' % name


@blue.route('/getpath/<path:url_path>/')
def hello_path(url_path):
    return 'path: %s' % url_path


@blue.route('/getuuid/')
def hello_get_uuid():
    a = uuid.uuid4()
    return str(a)
```