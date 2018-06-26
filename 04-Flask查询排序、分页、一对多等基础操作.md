###创建学生数据
####添加一条学生数据
```
from flask import request

@stu.route('/createstu/', methods=['GET', 'POST'])
def create_stu():
	if request.method == ''GET:
		return rener_template('create_stu.html')

	if request.method == 'POST':
		username = request.form.get('username')
		age = request.form.get('age')

		stus = Student(username, age)
		db.session.add(stus)
		db.session.commit()

		return '创建成功'
```

####添加多条学生数据
```
@stu.route('/createstus/', methods=['GET',  'POST'])
def create_stus():
    if request.method == 'GET':
        return render_template('create_stus.html')

    else:
        stus_list = []
        username1 = request.form.get('username1')
        age1 = request.form.get('age1')
        username2 = request.form.get('username2')
        age2 = request.form.get('age2')

        stu1 = Student(username1, age1)
        stu2 = Student(username2, age2)

        stus_list.append(stu1)
        stus_list.append(stu2)

        db.session.add_all(stus_list)
        db.session.commit()

        return '创建成功'
```

####查询数据（常用语句）
1.查询年龄小于16的学生(方法一)
```
stus = Student.query.filter(Student.s_age<16)
```
2.查询年龄小于16的学生(方法2)
```
stus = Student.query.filter_by(Studnet.s_age.__lt__(16))

# 注意：__lt__表示小于；__le__表示小于等于；__gt__表示大于；__ge__表示大于等于
```
3.查询规定的年龄
```
stus = Student.query.filter(Student.s_age.in_([12, 13, 14]))
```
4.查询所有学生
```
sql = 'select * from student;'
stus = db.session.execute(sql)
```
5.按照id降序排列
```
stus = Student.query.order_by('-s_id')
```
6.按降序获取3个对象
```
stus = Student.query.order_by('-s_id').limit(3)
```
7.获取年龄最大的学生
```
stus = Student.query.order_by('-s_age').first()
```
8.跳过两个数据，获取后两个数据
```
stus = Student.query.order_by('-s_age').offset(2).limit(2)
```
9.跳过两个数据，查询后面所有的数据
```
stus = Student.query.order_by('-s_age').offset(2)
```
10.获取对应id号的学生
方法(1)
```
stus = Student.query.filter(Student.s_id == 14)
```
方法(2)
```
stus = Studnet.query.get(14)
# 注意get()方法自动获取主键
```
11.多条件查询
方法(1)
```
stus = Student.query.filter(Student.s_age == 1, Student.s_name == '小帅450')
```
方法(2) —— and_
```
stus = Student.query.filter(and_(Student.s_age == 1), (Student.s_name == '小帅450'))
```
12.or_ ,not_
```
stus = Student.query.filter(or_(Student.s_age == 14), (Student.s_name == 'keke'))

stus = Student.query.filter(not_(Student.s_age == 0))
```

####分页操作
1.views.py中写方法
```
@stu.route('/stupage/')
def stu_page():
	
	page = int(request.args.get('page', 1))  # 当前页数
	per_page = int(request.args.get('per_page', 5))  # 设置每页数量
	paginate = Studnet.query.order_by('-s_id').paginate(page, per_page, error_out=False)
	stus = paginate.items # 获取当前页数据

	return render_template('stupage.html', paginate=paginate, stus=stus)

```
2.stupage.html中
```
{% extends 'base_main.html' %}

{% block title %}
    学生分页页面
{% endblock %}

{% block content %}

    <h2>学生信息</h2>
    {% for stu in stus %}
        id：{{ stu.s_id }}
        姓名：{{ stu.s_name }}
        年龄：{{ stu.s_age }}
        <br>
    {% endfor %}

	总页数：{{paginate.pages}}
	
	统计数据数量：{{paginate.total}}

	当前页数：{{paginate.page}}

	{% if paginate.has_prev %} # 如果有上一页
		上一页：<a href="/shupage/?page={{ paginate.prev_num }}">{{ paginate.prev_num }}</a>
	{% endif %}

	{% if paginate.has_next %}
        下一页：<a href="/shupage/?page={{ paginate.next_num }}">{{ paginate.next_num }}</a>
    {% endif %}

	页码：
	{% for i in paginate.iter_pages() %}
	    <a href="/shupage/?page={{ i }}">{{ i }}</a>
    {% endfor %}
```

#### 创建一对多数据结构表(举例班级、学生表)
1.设置主键(班级表)
```
class Grade(db.Model):
    g_id = db.Column(db.Integer, primary_key=True, autoincrement=True)  # 设置主键
    g_name = db.Column(db.String(10), unique=True)
    g_desc = db.Column(db.String(100), nullable=True)
    g_time = db.Column(db.Date, default=datetime.now)

    students = db.relationship('Student', backref='grade', lazy=True)  # 建立关系

    __tablename__ = 'grade'

    def __init__(self, name, desc):
        self.g_name = name
        self.g_desc = desc
```

2.设置外键(学生表）
```
class Student(db.Model):
    s_id = db.Column(db.Integer, primary_key=True, autoincrement=True)
    s_name = db.Column(db.String(20), unique=True)
    s_age = db.Column(db.Integer, default=18)

    s_g = db.Column(db.Integer, db.ForeignKey('grade.g_id'), nullable=True)  # 设置外键

    __tablename__ = 'student'

    def __init__(self, name, age):
        self.s_name = name
        self.s_age = age
```

3.sql语句设置表关联(主外键)
```
alter table student add s_g int;

alter table student add foreign key(s_g) references grade(g_id);
```

4.注册班级路由
```
app.register_blueprint(blueprint=grade, url_prefix='/grade')
```

5.通过班级查询学生(一对多)
```
@grade.route('/selectstubygrade/<int:id>/')
def select_stu_by_grade(id):
	grade = Grade.query.get(id)
	stus = grade.students

	return render_template('grade_student.html', stus=stus, grade=grade)

```
6.通过学生查询对应班级(多对一)
```
@stu.route('/selectgradebystu/<int:id>/')
def select_grade_by_stu(id):
	stu = Student.query.get(id)
    grade = stu.grade

    return render_template('student_grade.html', grade=grade, stu=stu)

```
