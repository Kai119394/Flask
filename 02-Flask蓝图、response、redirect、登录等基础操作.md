### flask基础操作
##### CSS层叠样式显示
1. 在views中
```
@blue.route('/index/')
def index():
	# 方法一
    return send_file('../templates/hello.html')
    # 方法二
    # return render_template('hello.html')
```

2. 在init.py中
```
def create_app():
	# 方法一
    app = Flask(__name__)
	# 方法二
	# BASE_DIR = os.path.dirname(os.path.dirname(os.path.abspath(__file__)))
    # templates_dir = os.path.join(BASE_DIR, 'templates')
    # start = os.path.join(BASE_DIR, 'static')
    # app = Flask(__name__, template_folder=templates_dir, static_folder=start)

    # 路由注册
    app.register_blueprint(blueprint=blue)
    return app 
```

3. 在hello.html中
```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>欢迎</title>
    <link rel="stylesheet" href="/static/css/hello.css">
</head>
<body>
    <h3>欢迎来到王者峡谷</h3>
</body>
</html>
```

4. 在hello.css中
```
h3 {
    color: red;
}
```

##### request请求

args —— GET请求，获取参数
form —— POST请求，获取参数
files —— 上传file文件
method —— 请求方式

1. views中
```
@blue.route('/getrequest/', methods=['GET', 'POST'])
def get_request():
    if request.method == 'GET':
        args = request.args
    else:
        form = request.form

    return '获取request'
```
2. 打开postman执行GET
```
http://127.0.0.1:8080/getrequest/?name=coco&name=hehe&id=1
```
3. console中效果
![这里写图片描述](https://img-blog.csdn.net/20180515113922838?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTc4MjA1MA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

##### 蓝图前缀
在init.py中添加url_prefix='/hello'
```
app.register_blueprint(blueprint=blue, url_prefix='/hello')
```

##### response响应

```
@blue.route('/makeresponse/')
def make_res():

    temp = render_template('hello.html')
    response = make_response(temp, 400)
    # response = make_response('<h2>呵呵哒</h2>')
    return response
    
```

##### redirect重定向

```
@blue.route('/redirect/')
def make_redirect():
    # 方法一
    # return redirect('/index/')
    # 方法二
    return redirect(url_for('first.index'))
```

##### 异常捕获
```
@blue.route('/makeabort/')
def make_abort():
    abort(404 )
    return '终结者'


@blue.errorhandler(404)
def get_error(exception):
    return '捕捉异常: %s' % exception
```

##### 搭建一个全新flask项目结构
1. 新建flask项目（pycharm新建）
2. 修改文件结构（见day1内容）
3. 配置虚拟环境和debug模式（见day1内容）
4. init.py
```
import os
from flask import Flask
from App.views import blue


def create_app():
    BASE_DIR = os.path.dirname(os.path.dirname(os.path.abspath(__file__)))  # homework目录
    templates_dir = os.path.join(BASE_DIR, 'templates')
    static_dir = os.path.join(BASE_DIR, 'static')
    app = Flask(__name__, template_folder=templates_dir, static_folder=static_dir)

	# 注册路由， url_prefix表示设置前缀
    app.register_blueprint(blueprint=blue, url_prefix='/app')
	

    return app
```

5. views.py
```
from flask import Blueprint, make_response


# 将app别名为blue
blue = Blueprint('app', __name__)

@blue.route('/hello/')
def hello():
    response = make_response('<h1>fuck</h1>')
    return response

```

6.manage.py
```
from flask_script import Manager
from App import create_app

app = create_app()
manager = Manager(app=app)


if __name__ == '__main__':
    manager.run()
```

##### 用户登录（session）
1. pip install flask-session —— 安装session
2. pip install redis —— 安装redis
3. 在views.py中添加
```
from flask import request, session, render_template, redirect, url_for

@blue.route('/login/', methods=['GET', 'POST'])
def login():
    if request.method == 'GET':
        username = session.get('username')
        return render_template('login.html', username=username)
    else:
        username = request.form.get('username')
        session['username'] = username
        return redirect(url_for('app.login'))
        
```
4.在init.py中添加
```
from flask_session import Session
import redis

	#密钥
    app.config['SECRET_KEY'] = 'secret_key'
    # 使用redis存储信息，默认访问redis，127.0.0.1:6379
    app.config['SESSION_TYPE'] = 'redis'
    # 访问redis
    app.config['SESSION_REDIS'] = redis.Redis(host='127.0.0.1', port='6379')
    # 定义前缀
    app.config['SECRET_KEY_PREFIX'] = 'flask'
    # 初始化app
    Session(app)  
```
##### ticket / cookie
```
@blue.route('/getresponse/')
def get_response():

    response = make_response('<h2>香蕉巴拉</h2>')
    ticket = ''
    s = 'qwertyuiopasdfghjklzxcvbnm'
    for i in range(20):
        ticket += random.choice(s)

    response.set_cookie('ticket', ticket)

    return response


@blue.route('/deletecookie/')
def del_cookie():
    response = make_response('<h2>香蕉巴拉</h2>', 200)
    response.delete_cookie('ticket')

    return response
```




