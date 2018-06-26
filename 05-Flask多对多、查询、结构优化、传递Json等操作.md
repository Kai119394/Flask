###创建多对多关系表
####创建学生表
```
class Student(db.Model):
    s_id = db.Column(db.Integer, primary_key=True, autoincrement=True)  # 设置主键
    s_name = db.Column(db.String(20), unique=True)
    s_age = db.Column(db.Integer, default=18)

	__tablename__ = 'student'

    def __init__(self, name, age):  # 初始化（为了后面传参）
        self.s_name = name
        self.s_age = age
```

####创建学生、课程表的中间表
```
sc = db.Table('sc',
    # 字段s_id， 外键关联学生表的s_id
    db.Column('s_id', db.Integer, db.ForeignKey('student.s_id'), primary_key=True),
    # 字段c_i， 外键关联课程表的c_id
    db.Column('c_id', db.Integer, db.ForeignKey('course.c_id'), primary_key=True)
              )
```

####创建课程表
```
class Course(db.Model):
    c_id = db.Column(db.Integer, primary_key=True, autoincrement=True)
    c_name = db.Column(db.String(10), unique=True)
    students = db.relationship('Student', secondary=sc, backref='cou')  
    # 与学生表建立联系，backref表示可以通过学生.cou方法回查课程,secondary指定副表名称

    __tablename__ = 'course'

    def __init__(self, name):
    # 初始化方法
        self.c_name = name
```
####向课程表中插入数据
```
@stu.route('/addcourse/')
def add_course():
    courses = ['高数', '思修', '英语', '造价', '结构']
    course_list = []
    for course in courses:
        cou = Course(course)
        course_list.append(cou)

    db.session.add_all(course_list)
    db.session.commit()

    return 'ok'
```

####选择学生和课程信息并存储进中间表
```
@stu.route('/stucourse/', methods=['GET', 'POST'])
def stu_cou():
    if request.method == 'GET':
		stus = Student.query.all()
        cous = Course.query.all()
        
        return render_template('stu_cou.html', stus=stus, cous=cous)
	else:
		s_id = request.form.get('student')
		c_id = request.form.get('course')  # 注意：直接获取的就是id值

		# 第一种方法(原生sql语句)
		sql = 'insert into (s_id, c_id) values (%s, %s)' % (s_id, c_id)
		db.session.add(sql)
		db.session.commit()

		# 第二种方式
		stu = Student.query.get(s_id)
		cou = Course.query.get(c_id)

		cou.students.append(stu)

		db.session.add(cou)
        db.session.commit()

```

####新建stu_cou.html页面
```
<form action="" method="post">
    <h2>学生信息：</h2>
    <br>
    <select name="student" id="">
        <option value="">选择学生信息</option>  # option为下拉选项标签
        {% for stu in stus %}
            <option value="{{ stu.s_id }}">{{ stu.s_name }}</option>
        {% endfor %}
    </select>
    <br>
    <h2>课程信息：</h2>
    <br>
    <select name="course" id="">
        <option value="">选择课程信息</option>
        {% for cou in cous %}
            <option value="{{ cou.c_id }}">{{ cou.c_name }}</option>
        {% endfor %}
    </select>
    <br>
    <input type="submit" value="提交">
</form>
```

####查找所有学生及对应所选课程
1.views方法
```
@stu.route('/allstu/')
def all_stu():
    stus = Student.query.all()
    return render_template('all_stu.html', stus=stus)
```
2.all_stu.html网页
```
<ul>
   {% for stu in stus %}
       <li>
           id：{{ stu.s_id }}
           姓名：{{ stu.s_name }}
           <a href="/stu/selectcoursebystu/{{ stu.s_id }}">课程</a>
           # 点击跳转至学生所选的课程详情
       </li>
   {% endfor %}
</ul>
```
![这里写图片描述](https://img-blog.csdn.net/20180518191146600?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTc4MjA1MA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

3.通过学生查找对应所选课程
```
@stu.route('/selectcoursebystu/<int:id>/')
def select_course_by_stu(id):
    stu = Student.query.get(id)
    cous = stu.cou  # 注意：cou为学生的一个方法，在课程表中设置

    return render_template('stucourse.html', cous=cous, stu=stu)
```
4.stucourse.html页面
```
{% for cou in cous%}
	<li>
		{{ stu.s_name }}所选课程：
	    课程id：{{ cou.c_id }}
        课程名：{{ cou.c_name }}
        <a href="/stu/deletecoursebyid/{{ stu.s_id }}/{{ cou.c_id }}">删除</a>
	</li>
{% endfor %}
```
5.删除对应课程方法
```
@stu.route('/deletecoursebyid/<int:s_id>/<int:c_id>')
def delete_course_by_id(s_id, c_id):
    stu = Student.query.get(s_id)
    cou = Course.query.get(c_id)

	cou.students.remove(stu)
    db.session.commit()

    return redirect(url_for('stu.all_stu'))
```

###结构优化、项目重构
####安装相关包
1.pip install DebugToolbar  ——  安装DebugToolbar，用于调试(页面左边增加一栏调试栏)

2.pip install RESTful  ——  安装RESTful，用于api接口

3.pip install marshmallow  ——  安装棉花糖，序列化数据

####结构优化
1.修改文件结构
  新建static静态文件夹
  新建template网页文件夹
  新建utils文件夹
  2.在utils文件夹中
  新建__init__.py / App.py / function.py / setting.py文件
  ![这里写图片描述](https://img-blog.csdn.net/20180519140058128?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTc4MjA1MA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

3.manage.py
```
from flask_script import Manager
from utils.APP import create_app  # 到处APP中的create_app方法


app = create_app()

manager = Manager(app=app)


if __name__ == '__main__':
	manager.run()
```
4.APP.py
```
from flask import Flask
from utils.setting import templates_dir, static_folder=static_dir, SQLALCHEMY_DATABASE_URI
from utils.function import init_ext


def create_app():
	# 创建app对象
	app = Flask(__name__, template_folder=templates_dir, static_folder=static_dir)

	# 连接数据库，从setting中导入SQLALCHEMY_DATABASE_URI
	app.config['SQLALCHEMY_DATABASE_URI'] = SQLALCHEMY_DATABASE_URI  
	app.config['SQLALCHEMY_TRACK_MODIFICATIONS'] = False

	init_ext(app)  # 从function中导入init_ext方法

	return app
```

5.setting.py
```
import os

BASE_DIR = os.path.dirname(os.path.dirname(os.path.abspath(__file__)))
templates_dir = os.path.join(BASE_DIR, 'templates')
static_dir = os.path.join(BASE_DIR, 'static')

DATABASE = {
	'USER': 'root',
    'PASSWORD': '123456',
    'HOST': 'localhost',
    'PORT': '3306',
    'DB': 'mysql',
    'DRIVER': 'pymysql',
    'NAME': 'flask3'
}

SQLALCHEMY_DATABASE_URI = get_db(DATABASE) # 在function中导入get_db方法
```
6.function.py
```
from flask_sqlalchemy import SQLAlchemy

db = SQLAlchemy()  # 创建模型使用


def get_db(DATABASE):
	user = DATABASE.get('USER')
    password = DATABASE.get('PASSWORD')
    host = DATABASE.get('HOST')
    port = DATABASE.get('PORT')
    name = DATABASE.get('NAME')
    db = DATABASE.get('DB')
    driver = DATABASE.get('DRIVER')

	return '{}+{}://{}:{}@{}:{}/{}'.format(db, driver, user, password, host, port, name)


	def iinit_ext(app):
		db.init_app(app=app)
	
```
