---
title: flask练习
tags: ['flask']
categories: ['python基础']
copyright: true
---
# template
html默认必须放在项目目录下的templates文件夹下。flask会默认自动去这个文件夹下找定义的html，使用html时不需要加templates这层目录。

# 前后端传递参数：
  参考：https://www.jianshu.com/p/4350065bdffe
（1）直接取放在链接里的数据，如 http://localhost:5000/home/lili/  想直接取到字符串'lili'。
```python
@app.route("/home/<name>", methods=['GET', 'POST'])
def home(name):
    return render_template("home.html", name=name)  #  直接用name就可以了。
  （2）get请求，通过request.args.get

@app.route("/test", methods=['GET'])
def test_strap():
    name = request.args.get('name')     #    http://localhost:5000/test?name=lili
```
（3）post请求
```python
@app.route("/login", methods=['POST'])
def login():
    name = request.form['name']
    password = request.form['password']
    if password.strip():
        return render_template("home.html", name=name)
    return render_template("login.html", message="wrong password", name=name)
```
'name'和'password'是form表单里input的name值。

# bootstrap
flask  template使用bootstrap和普通使用bootstrap的方式不同。
参考：http://flask-bootstrap-zh.readthedocs.io/zh/latest/basic-usage.html
（1）安装
`pip install flask-bootstrap`
（2）使用教程可以参考上面的链接。
```python
from flask import Flask
from flask import request, render_template
from flask_bootstrap import Bootstrap

app = Flask(__name__)
app.config['BOOTSTRAP_SERVER_LOCAL'] = True   # 使用本地
bootstrap = Bootstrap(app)
```
# 代码
例子：[template](https://github.com/meihuakaile/template)
执行 template_test.py 后，在浏览器输入 http://127.0.0.1:5000/
