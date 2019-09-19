# 项目目录介绍及服务开发指南

## 代码目录结构

### 项目代码目录 说明

![1](http://tcloud-static.oss-cn-beijing.aliyuncs.com/tcloud_git/all.png)

| 目录名称 | 说明 |
| :--- | :--- |
| TcloudServer | 根目录 |
| apps | 多服务目录 |
| deploy | 部署文件存放目录 |
| library | 通用库 |
| apidoc.json | 用于生产 apidoc 文档 |
| docker-compose.yml | docker compose 配置文件 |
| local\_config.py | 配置文件（本地） |
| Pipfile | pipenv 使用的 包管理 配置 |
| public\_config.py | 通用配置文件 |
| requirement.txt | pip -r 使用的 配置 |

#### apps

要添加新的模块或者服务，需要放置在本目录下。

![2](http://tcloud-static.oss-cn-beijing.aliyuncs.com/tcloud_git/apps.png)

| 目录名称 | 说明 |
| :--- | :--- |
| author | 用户以及鉴权相关 |
| autotest | 自动化测试 |
| extention | 工具 |
| flow | 流程 |
| interface | 接口测试 |
| jobs | 定时任务 |
| message | 站内信 |
| project | 项目 |
| public | 公共 |
| tcdevices | 云真机 |
| ws | web socket |

#### library

通用的方法提取出来分类放到对应文件中。

![3](http://tcloud-static.oss-cn-beijing.aliyuncs.com/tcloud_git/library.png)

| 目录名称 | 说明 |
| :--- | :--- |
| api | 接口使用相关 |
| notification.py | 用来发送信息（站内信和企业微信通知） |
| oss.py | oss 相关 |
| tlogger.py | 日志相关 |
| trpc.py | rpc 用于服务间调用，目前使用 http 协议 |

> api

| 目录名称 | 说明 |
| :--- | :--- |
| db.py | 数据库相关，包括 db 和 model 定义 |
| exceptions.py | 异常定义 |
| parse.py | request 请求参数处理 |
| render.py | response 数据格式化 |
| security.py | request 请求参数格式校验 |
| tBlueprint.py | 定义 blueprint，并对 get 和 delete 进行权限校验 |
| tFlask.py | 实现 TFlask 继承自 Flask，重写了 run 用于使用 gunicorn 启动，重写了 make\_response 用于处理 response 格式化 |
| tMiddleware.py | 定义了用于处理 before\_request 和 error\_request 的中间函数 |
| transfer.py | 用于 格式化 数据库获取的结果 |

> 其他文件

* notification.py
* 包装了 发送站内信和发送微信消息 的方法，可以在当中添加其他发信方式。这里发送的两种方式 通过 其他服务发送。

```python
class Notification:
    def __init__(self):
        self.public_trpc = Trpc('public')
        self.message_trpc = Trpc('message')

    def send_notification(self, user_ids, text, creator=None, send_type=None):
        # 发送站内信
        # 发送企业微信
        # 自定义发送方式

notification = Notification()
```

1. 使用方法

```python
from library.notification import notification

notification.send_notification([1,2,3], "helloworld")
```

* oss.py

  用于 oss 上传 和 下载 文件 ,可以自定义添加其他方法。

* tlogger.py
* 日志相关,定义了 InfoFilter\(logging.Filter\) 作为 log 等级显示筛选。定义了 TloggerFormatter\(logging.Formatter\) 显示定制的 错误和警告信息。
* 使用方法 : 在 flask app 初始化后 使用 logger\_create\(\)

```python
    logger_create("hello-world", app)
```

* trpc.py
* 作为 rpc ，目前使用的是 http 协议进行服务间调用
* 使用方法：在上面的 notification 中已经使用过了

```python
from library.trpc import Trpc
# 使用服务名称获取对应的端口信息
public_trpc = Trpc('public')
public_trpc.requests('post', '/public/wxmessage', body={'user_ids': [1,2,4], 'text': "helloworld")
```

### 单个服务的项目结构，以 auth 为例子

* 目前整个项目后端是 多服务 结构，每个服务负责的内容相对独立。
* 由于项目是 前后端分离的，所以后端只提供接口供前端进行调用。 
* 整体架构是基于 flask、flask-sqlalchemy 进行构建的 非典型的 mvc 架构（model-view-controller）

#### 目录结构

![3](http://tcloud-static.oss-cn-beijing.aliyuncs.com/tcloud_git/auth-1.png)

| 目录名称 | 说明 |
| :--- | :--- |
| auth | 服务名称/服务根目录 |
| business | controller 所在目录，用于具体的逻辑构建 |
| models | 存放所有的 model，用于定义数据模型和数据库进行对应 |
| settings | 每个 服务独有 的配置信息 |
| validatons | 用于前面的 parse 和 security 的数据获取和数据校验 配置文件 |
| views | 存放所有 views |
| auth\_require.py | 鉴权相关 |
| extentins.py | 用于定义服务中使用的其他扩展 |
| run.py | 服务启动 |

![4](http://tcloud-static.oss-cn-beijing.aliyuncs.com/tcloud_git/auth-2.png)

## 创建一个简单地新的服务，以 auth 中的 user 为例子，选取 `/v1/user/add` 接口

### 创建目录结构

![5](http://tcloud-static.oss-cn-beijing.aliyuncs.com/tcloud_git/template.png)

### 配置 config

> apps/user/settings/config.py

```python
try:
    from public_config import *
except ImportError:
    pass

# 端口号是自定义的，需要规划好扩展及其他的服务端口
PORT = 9999
# 服务名称与 目录名称 相同：user
SERVICE_NAME = 'user'
```

### 添加 extentions

> apps/user/extentions.py

这个文件大部分是通用的库初始化的过程，主要有 初始化 validation，parse 等工具

```python
import os
from os.path import join, abspath, dirname, splitext

import yaml

from library.api.parse import TParse
from library.api.security import Validation

current_path = dirname(abspath(__file__))

yml_json = {}

try:
    validate_yml_path = join(current_path, 'validations')
    for fi in os.listdir(validate_yml_path):
        if splitext(fi)[-1] != '.yml':
            continue
        with open(join(validate_yml_path, fi), 'rb') as f:
            yml_json.update(yaml.safe_load(f.read()))
except yaml.YAMLError as e:
    print(e)

v = Validation(yml_json)
validation = v.validation

p = TParse(yml_json)
parse_list_args = p.parse_list_args
parse_list_args2 = p.parse_list_args2
parse_json_form = p.parse_json_form
parse_pwd = p.parse_pwd
```

### 定义接口

> /user/add

| 属性 | 值 |
| :--- | :--- |
| 路径 | /user/add |
| 方法 | POST |
| 参数 | {"name":"hello", "nickname":"world", "password":"helloworld"} |

#### 首先定义 user 的 model

> apps/user/models/users.py

```python
from library.api.db import db, EntityWithNameModel

# 使用 含有 name 的模型进行继承，这样默认便会含有 name，creation_time, modified_time 。
# 数据库表名 与定义的模型名称的小写全拼相同，但驼峰式命名会转化为 _ 的格式。
# User == user, UserDetail == user_detail


# 定义 User 模型
class User(EntityWithNameModel):
    ACTIVE = 0
    DISABLE = 1

    nickname = db.Column(db.String(100), nullable=True)  # 昵称
    password = db.Column(db.String(100), nullable=False)  # 密码
    status = db.Column(db.Integer, default=ACTIVE)  # 状态，0：存在，1：删除
```

#### 然后编写 用户添加数据库逻辑

> apps/user/business/users.py

```python
from flask import current_app

from apps.user.extentions import parse_pwd
from apps.user.models.users import User
from library.api.db import db


# 创建 UserBusiness
class UserBusiness(object):

    @classmethod
    def create_new_user_and_bind_roles(cls, username, nickname, password):
        # 传入 需要的参数 
        try:
            # 查询是否已经存在当前用户名的用户
            ret = User.query.filter(User.name == username, User.status == User.ACTIVE).first()
            if ret:
                return 103, None
            # 构建 User，使用 parse_pwd 加密 password 为密文
            n = User(
                name=username,
                nickname=nickname,
                password=parse_pwd(password)
            )
            # 使用 db 添加到数据库
            db.session.add(n)
            db.session.commit()
            return 0, None
        except Exception as e:
            current_app.logger.error(str(e))
            return 102, str(e)
```

#### 添加 需要校验数据的 配置文件

> apps/user/validations/users.yml

```yaml
adduser:
  username:
    required: True
    type: basestring
  nickname:
    required: True
    type: basestring
  password:
    required: True
    type: basestring
  returnvalue:
    - username
    - nickname
    - password
```

#### 然后编写 用户添加接口 逻辑

> apps/user/views/users.py

```python
from apps.user.business.users import UserBusiness
from apps.user.extentions import validation, parse_json_form
from library.api.tBlueprint import tblueprint

# 创建 blueprint
user = tblueprint("user", __name__, bpname="user", has_view=False)


# 定义 /add 路径 和 请求方法 POST
@user.route('/add', methods=['POST'])
@validation('POST:adduser') # 用于数据校验，主要从 validations/users.yml 中读取
def user_index_handler():
    # 从 validations/users.yml 中读取
    username, nickname, password = parse_json_form('adduser')
    ret, msg = UserBusiness.create_new_user_and_bind_roles(username, nickname, password)
    return {
        "code": ret,
        "data": [],
        "message": msg
    }
```

### 定义 app

> apps/user/run.py

```python
from apps.user.settings import config

if config.SERVER_ENV != 'dev':
    # 猴子补丁，用于 gunicorn 启动。
    from gevent import monkey
    monkey.patch_all()
else:
    pass

from apps.user.views.users import user
from library.api.tFlask import tflask


# 使用 create_app 创建 app
def create_app():
    app = tflask(config)
    register_blueprints(app)
    return app


# 注册 blueprint 到 app
def register_blueprints(app):
    app.register_blueprint(user, url_prefix="/v1/user")


# 启动应用
if __name__ == '__main__':
    create_app().run(port=config.PORT)
```

### 启动 app

```text
   python3 -m apps.user.run
```

![](http://tcloud-static.oss-cn-beijing.aliyuncs.com/tcloud_git/startapp.png)

### 访问 POST

![](http://tcloud-static.oss-cn-beijing.aliyuncs.com/tcloud_git/get.png)

### 登录一下，需要启动全部服务，将返回的 token 填入 header 的 Authorization 中

![](http://tcloud-static.oss-cn-beijing.aliyuncs.com/tcloud_git/login.png)

### 再次访问

> 可以看到报错信息为 数据不对

![](http://tcloud-static.oss-cn-beijing.aliyuncs.com/tcloud_git/invalid-data.png)

> 后台报错信息

![](http://tcloud-static.oss-cn-beijing.aliyuncs.com/tcloud_git/invalid-data-cmd.png)

### 填写 请求的 Body \(缺少参数\)

> 类型为 json 格式

```javascript
{
    "username": "test"
}
```

> 可以看到 返回为 缺少参数

![](http://tcloud-static.oss-cn-beijing.aliyuncs.com/tcloud_git/miss-param.png)

> 后台报错信息

![](http://tcloud-static.oss-cn-beijing.aliyuncs.com/tcloud_git/miss-param-cmd.png)

### 填写 完整的 body

```javascript
{
    "username": "hello",
    "password": "helloworld",
    "nickname": "world"
}
```

![](http://tcloud-static.oss-cn-beijing.aliyuncs.com/tcloud_git/success.png)

