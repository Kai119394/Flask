### 样式继承（挖坑填坑）
#### 创建新项目
1. __init__.py文件中
```
import os
from flask import Flask
from Stu.views import stu

def create_app():
	BASE_DIR = os.path.dirname(os.path.dirname(os.path.abspatn(__file__)))
	template_dir = os.path.dirname(BASE_DIR, 'templates')
	static_dir = os.path.dirname(BASE_DIR, 'static')
	app = Flask(__name__, templeta_folder=template_dir, static_folder=static_dir)
	
	app.register_blueprint(blueprint=stu)  # 注册路由

	reutrn app
```
2.views文件中
```
from flask import Blueprint, render_template

stu = Blueprint('stu', __name__)  # 别名


@stu.route('/index/')
def index():
	return render_template('index.html')
```
3.manage.py文件中
```
from Stu import create_app
from flask_script import Manager


app = create_app()
manage = Manage(app=app)

if __name__ == '__main__':
    manage.run()
```
#### 样式继承
1. 新建base.html文件
```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>
        {% block title %}{% endblock %}
    </title>
    {% block extCSS %}{% endblock %}

</head>
<body>

{% block header %}{% endblock %}
{% block content %}{% endblock %}
{% block footer %}{% endblock %}
{% block extJS %}{% endblock %}

</body>
</html>
```
2.新建base_main.py文件
```
{% extends 'base.html' %}

{% block extCSS %}
    {#    第一种引入css文件方法#}
    <link rel="stylesheet" href="/static/css/index.css">
    {#    第二种引入css文件方法#}
{#  <link rel="stylesheet" href="{{ url_for('static', filename='css/index.css') }} ">#}
{% endblock %}
```
#### 过滤器
1. safe —— 渲染标签
2. striptages —— 渲染前去掉标签
3. trime —— 去掉空格
4. length —— 计算长度
5. i | first —— 第一个字符
6. i | last —— 最后一个字符
7. {i | lower} —— 小写
8. {i | upper} —— 大写
9. {i | capitallize} —— 首字母大写

#### 数据库
1. pip install flask-sqlalchemy  —— 安装flask-sqlalchemy
2. pip install pymysql  ——  安装pymysql
3. 新建models.py文件创建学生模型
```
from flask_sqlalchemy import SQLAlchemy

db = SQLAlchemy()


class Student(db.Model):
	s_id = db.Column(db.Integer, primary_key=True, autoincrement=True)  # 设置主键
	s_name = db.Column(db.String(20), unique=True)
	s_age = db.Column(db.Integer, default=18)

	__tablename__ = 'student'
	
```
4.在__init__.py文件中添加两句话
```
app.config['SQLALCHEMY_DATABASE_URI'] = 'mysql+pymysql://root:123456@localhost:3306/flask3'
 # 依次对应数据库密码、本机端口号（远端服务器公网）、数据库名
 
app.config['SQLALCHEMY_TRACK_MODIFICATIONS'] = False
```

#### 数据库的增删改查等基本操作
1. 创建学生表
```
@stu.route('/createtable/')
def create_db():
	db.create_all()
	return '创建成功'
```

2.删除学生表
```
@stu.route('/droptable/')
def drop_db():
	db.drop_all()
	return '删除成功'
```
3.创建学生对象
```
from Stu.models import Student
import random

@stu.route('createstu')
def create_stu():
	stu = Student()
	stu.s_name = '小帅%d' % random.randrange(1000)
	stu.s_age = '%d' % random.randrange(40)

	db.session.add(stu)  # 添加
	try:
		db.session.commit()  # 提交
	except:
		# 回滚
		db.session.rollback()
	return '创建学生成功'
```

4.展示全部学生
```
@stu.route('/stulist/')
def stu_all():
	stus = Student.query.all()

	return render_template('stulist.html', stus=stus)
```

5.查询学生
  方法(1)使用原生sql
```
@stu.route('/studetail/')
def stu_detail():
	sql = 'selecte * from student where s_name="学生名";'
	stus = db.session.execute(sql)
	# 注意：学生名后必须接上";"分号

	return render_template('stulist.html', stus= stus)
```
方法(2)使用filter
```
stus = Student.query.filter(Student.s_name == '学生名')
```
方法(3)使用filter_by
```
stus = Student.query.filter_by(s_name='学生名')
```

6.更新数据
  方法(1)
```
from flask import redirect, url_for

@stu.route('/updatestu/')
def update_stu():
	stu = Student.query.filter_by(s_id=5).first()
    stu.s_name = '王大锤'

	db.session.commit()
	return redirect(url_for('stu.stu_all'))
```
方法(2)
```
Student.query.filter_by(s_id=1).update({'s_name': '海飞丝'})

db.session.commit()
```

7.删除数据
```
@stu.route('/deletestu/')
def delete_stu():
	stu = Student.query.filter(Student.s_id==2).first()
	db.session.delete(stu)
	db.session.commit()
	
	return redirect(url_for('stu.stu_all'))
```

