---
title: Flask初体验——搭建Android登录后端
date: 2016-12-29 16:32:56
tags: [Android,Flask]
subtitle: 初次使用Flask搭建Android后端，记录用户名密码数据
---

# 前言

​	Flask是一个使用Python编写的轻量级 Web 应用框架，对于像我这种想偷懒，不想自己从头学后端的人来说是再合适不过了，为了使用Flask，我在十一月份就已经开始了学习Flask必备的Python知识。

# 明确目标

​	首先要明确目标，我要做的是在客户端输入用户名密码，然后选择登录还是注册，在服务器端进行检验，并存储数据到服务器端的数据库（为了简便使用SQLite），并且返回Response消息回客户端，客户端再根据返回的Response消息进行相应的动作。由于在大一的时候用C语言做过一个京东购物管理系统，里面涉及了在本地登录，所以我对登录页面的逻辑还算是轻车熟路了。

# 首先写Android端逻辑

​	首先需要在APP端构造一个简单的登录界面，只需要两个EditText和两个按钮就行了。如图所示

![登陆界面](/image/登陆界面.png)

​	然后写代码从两个输入框中将用户输入的用户名和密码提取出来。

```java
@Override
    public void onClick(View v)
    {
        String userName = userNameEdit.getText().toString();
        String passWord = passWordEdit.getText().toString();
        if(userName.equals("")||passWord.equals(""))
        {
            showWarnSweetDialog("账号密码不能为空");
            return;
        }
        switch (v.getId())
        {
            case R.id.log_Button:
                String url = "http://192.168.253.1:5000/user";/*在此处改变你的服务器地址*/
                getCheckFromServer(url,userName,passWord);
                break;
            case R.id.Sign_Button:
                String url2 = "http://192.168.253.1:5000/register";/*在此处改变你的服务器地址*/
                registeNameWordToServer(url2,userName,passWord);
                break;
        }
    }
```

​	有一点重要的是要判断不为空，由于这个应用不牵涉到大规模使用，所以不对密码安全性进行限制。

​	接下来先写注册模块，注册模块由registeNameWordToServer()函数来实现，这个模块功能就是将用户名密码发送到url对应的服务器地址。在网络方面为了简便，使用了一个OKHttp的开源网络框架。

```java
 private void registeNameWordToServer(String url,final String userName,String passWord)
    {
        OkHttpClient client = new OkHttpClient();
        FormBody.Builder formBuilder = new FormBody.Builder();
        formBuilder.add("username", userName);
        formBuilder.add("password", passWord);
        Request request = new Request.Builder().url(url).post(formBuilder.build()).build();
        Call call = client.newCall(request);
        call.enqueue(new Callback()
        {
            @Override
            public void onFailure(Call call, IOException e)
            {
                runOnUiThread(new Runnable()
                {
                    @Override
                    public void run()
                    {
                        showWarnSweetDialog("服务器错误");
                    }
                });
            }

            @Override
            public void onResponse(Call call, final Response response) throws IOException
            {
                final String res = response.body().string();
                runOnUiThread(new Runnable()
                {
                    @Override
                    public void run()
                    {
                        if (res.equals("0"))
                        {
                            showWarnSweetDialog("该用户名已被注册");
                        }
                        else
                        {
                            showSuccessSweetDialog(res);
                            sharedPreferences = getSharedPreferences("UserIDAndPassword", MODE_PRIVATE);
                            SharedPreferences.Editor editor = sharedPreferences.edit();
                            editor.putString("username", userName);
                            editor.apply();
                        }

                    }
                });
            }
        });

    }
```

只需要调用Request的post方法就能够进行post动作。在Call里收到服务器的返回消息，重写CallBack接口，写出成功响应和失败响应的逻辑。在接口中由于OkHttp已经封装了子线程操作，所以这是在子线程中的动作，如果需要弹出对话框需要返回UI线程。

接下来的登录模块跟注册模块大同小异。

# 服务器端逻辑

​	服务器端要用到Flask，所以当然首先创建一个Flask项目，由于我用的IDE是JetBrain的PyCharm，创建Flask项目非常方便。本来我以为服务器端会很难，还专门买了一本Flask的书，结果还没看几页的知识就完全够用了。先上代码：

```python
from flask import Flask
from flask import make_response
from flask import request
from flask.ext.script import Manager
import os
from flask.ext.sqlalchemy import SQLAlchemy

app = Flask(__name__)
basedir=os.path.abspath(os.path.dirname(__file__))
app.config['SQLALCHEMY_DATABASE_URI']='sqlite:///'+os.path.join(basedir,'userConfigBase.sqlite')
app.config['SQLALCHEMY_COMMIT_ON_TEARDOWN']=True
app.config['SQLALCHEMY_TRACK_MODIFICATIONS']=True
userdb=SQLAlchemy(app)
manager=Manager(app)
@app.route('/')
def test():
    return '服务器正常运行'

class userInfoTable(userdb.Model):
    __tablename__='userInfo'
    id=userdb.Column(userdb.Integer,primary_key=True)
    username=userdb.Column(userdb.String,unique=True)
    password=userdb.Column(userdb.String)

    def __repr__(self):
        return 'table name is '+self.username

#此方法处理用户登录 返回码为0无注册 返回码为1密码错误
@app.route('/user',methods=['POST'])
def check_user():
    haveregisted = userInfoTable.query.filter_by(username=request.form['username']).all()
    if haveregisted.__len__() is not 0: # 判断是否已被注册
        passwordRight = userInfoTable.query.filter_by(username=request.form['username'],password=request.form['password']).all()
        if passwordRight.__len__() is not 0:
            return '登录成功'
        else:
            return '1'
    else:
        return '0'


#此方法处理用户注册
@app.route('/register',methods=['POST'])
def register():
    userdb.create_all()
    haveregisted = userInfoTable.query.filter_by(username=request.form['username']).all()
    if haveregisted.__len__() is not 0: # 判断是否已被注册
        return '0'
    userInfo=userInfoTable(username=request.form['username'],password=request.form['password'])
    userdb.session.add(userInfo)
    userdb.session.commit()
    return '注册成功'

if __name__ == '__main__':
    manager.run()


class userInfoTable(userdb.Model):
    __tablename__='userInfo'
    id=userdb.Column(userdb.Integer,primary_key=True)
    username=userdb.Column(userdb.String,unique=True)
    password=userdb.Column(userdb.String)

    def __repr__(self):
        return 'table name is '+self.username
```

​	@app.route('/')为装饰器，起作用就是在客户端访问“服务器地址/”是可以调用相应的函数。比如@app.route('/register',methods=['POST'])就是当访问 "服务器地址/register"时调用的。

​	Python使用SQLAlchemy操作数据库很方便，几句代码就创建完毕了，一看就懂，当创建表时就要注意了，需要自己写一个类来实现这个表里所需的列。这个类需要继承自你的数据库.Model，userdb.Column(userdb.String,unique=True)就相当于创建了一个String类型的不可重复的列，用来储存用户名在合适不过了。

​	接下来就要在函数里处理登录注册的逻辑，注册逻辑是这样的：取出Request中的username，将其作为关键字放进数据库中搜索，若没有，则将其用户名和密码都储存下来，若已经存在了，则返回一个错误信号通知客户端已经存在了。

​	登陆逻辑就更简单了，只需要取出Request中的username，若数据库中不存在，就通知用户请先注册，若存在但密码不正确，则通知用户密码错误，若密码正确，则返回正常信息。

# 总结

​	通过这次Flask的后端实践，我深刻明白了一个道理，往往你认为难的东西，你真正去做了，往往会发现并没有这么难。以前我总是把搭建服务器和Android网络编程想的很难，其实真正去做了，并不是很难。而那些你认为很简单那的东西，如果不考虑周到，往往会出岔子。