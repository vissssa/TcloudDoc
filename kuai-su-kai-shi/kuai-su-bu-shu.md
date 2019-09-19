---
description: 建议部署在Linux或者Macos上，Windows不适合本项目的调试、部署
---

# 快速部署



> 建议部署在Linux或者Macos上，Windows不适合本项目的调试、部署
>
> **docker部署**

点击查看: [deploy](https://github.com/bigbaser/TcloudServer/wiki/Docker%E9%83%A8%E7%BD%B2)

**环境**

python =&gt; 3.7

**服务端口**

| id | service name | port |
| :--- | :--- | :--- |
| 1 | auth | 9020 |
| 2 | autotest | 9022 |
| 3 | extention | 9024 |
| 4 | flow | 9026 |
| 5 | interface | 9028 |
| 6 | message | 9030 |
| 7 | project | 9032 |
| 8 | public | 9034 |
| 9 | tcdevices | 9036 |
| 10 | jobs | 9038 |
| 11 | ws | 9040 |

**服务间调用**

> trpc

* now : communicate with http request by `requests`

```text
    user_trpc = Trpc('auth')   # using with service name
    user_trpc.requests(method='get', path='/user')
```

**一、下载代码**

```text
$ git clone https://github.com/bigbaser/TcloudServer.git
```

**二、配置nginx**

```text
server {
    listen 80;
    server_name 127.0.0.1 localhost;

  location /v1/datashow/ {
        proxy_pass http://127.0.0.1:9022;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        add_header 'Access-Control-Allow-Origin' '*';
        add_header 'Access-Control-Allow-Headers' 'Content-Type, Authorization, projectid';
        add_header 'Access-Control-Allow-Methods' 'POST, GET, DELETE, OPTIONS';

    }
    location /v1/jobs/ {
        proxy_pass http://127.0.0.1:9038;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        add_header 'Access-Control-Allow-Origin' '*';
        add_header 'Access-Control-Allow-Headers' 'Content-Type, Authorization, projectid';
        add_header 'Access-Control-Allow-Methods' 'POST, GET, DELETE, OPTIONS';

    }
       location /v1/message/ {
        proxy_pass http://127.0.0.1:9030;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        add_header 'Access-Control-Allow-Origin' '*';
        add_header 'Access-Control-Allow-Headers' 'Content-Type, Authorization, projectid';
        add_header 'Access-Control-Allow-Methods' 'POST, GET, DELETE, OPTIONS';

    }

    location /v1/tcdevices/ {
        proxy_pass http://127.0.0.1:9036;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        add_header 'Access-Control-Allow-Origin' '*';
        add_header 'Access-Control-Allow-Headers' 'Content-Type, Authorization, projectid';
        add_header 'Access-Control-Allow-Methods' 'POST, GET, DELETE, OPTIONS';

    }

    location  /v1/public/ {
        proxy_pass http://127.0.0.1:9034;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        add_header 'Access-Control-Allow-Origin' '*';
        add_header 'Access-Control-Allow-Headers' 'Content-Type, Authorization, projectid';
        add_header 'Access-Control-Allow-Methods' 'POST, GET, DELETE, OPTIONS';

    }

    location  /v1/monkey/ {
        proxy_pass http://127.0.0.1:9022;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        add_header 'Access-Control-Allow-Origin' '*';
        add_header 'Access-Control-Allow-Headers' 'Content-Type, Authorization, projectid';
        add_header 'Access-Control-Allow-Methods' 'POST, GET, DELETE, OPTIONS';

    }

    location ~* /v1/(flow|deploy)/ {
        proxy_pass http://127.0.0.1:9026;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        add_header 'Access-Control-Allow-Origin' '*';
        add_header 'Access-Control-Allow-Headers' 'Content-Type, Authorization, projectid';
        add_header 'Access-Control-Allow-Methods' 'POST, GET, DELETE, OPTIONS';

    }

    location ~* /v1/(cidata|tool)/ {
        proxy_pass http://127.0.0.1:9024;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        add_header 'Access-Control-Allow-Origin' '*';
        add_header 'Access-Control-Allow-Headers' 'Content-Type, Authorization, projectid';
        add_header 'Access-Control-Allow-Methods' 'POST, GET, DELETE, OPTIONS';

    }

    location ~* /v1/interface.* {
        proxy_pass http://127.0.0.1:9028;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        add_header 'Access-Control-Allow-Origin' '*';
        add_header 'Access-Control-Allow-Headers' 'Content-Type, Authorization, projectid';
        add_header 'Access-Control-Allow-Methods' 'POST, GET, DELETE, OPTIONS';

    }

    location ~* /v1/(user|track|role|ability|feedback|wxlogin)/ {
        proxy_pass http://127.0.0.1:9020;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        add_header 'Access-Control-Allow-Origin' '*';
        add_header 'Access-Control-Allow-Headers' 'Content-Type, Authorization, projectid';
        add_header 'Access-Control-Allow-Methods' 'POST, GET, DELETE, OPTIONS';

    }

    location / {
        proxy_pass http://127.0.0.1:9032;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        add_header 'Access-Control-Allow-Origin' '*';
        add_header 'Access-Control-Allow-Headers' 'Content-Type, Authorization, projectid';
        add_header 'Access-Control-Allow-Methods' 'POST, GET, DELETE, OPTIONS';

    }

}
```

**三、Data Migration**

将我们预先准备好的[sql文件](https://github.com/bigbaser/TcloudServer/blob/master/deploy/init/init.sql)执行到mysql数据库中

**四、python环境**

1、准备好一个python3.7的环境，且安装好pip 2、推荐使用pipenv管理python环境，安装Pipfile中的依赖包，注意版本问题，  
或者直接使用`pip install -r requirement.txt -i https://mirrors.aliyun.com/pypi/simple`安装依赖包，最好先新建一个虚拟环境`virtualenv`。

**五、Config setting**

```text
$ vim local_config.py
```

将其中的数据库连接等修改成自己的，_注意_，不要修改秘钥相关代码，否则无法登陆预先设定的账号密码，如果有需要的，可以查看代码中加密密码的代码来修改SALT等，修改属于自己的密码

**六、Start**

linux下可以使用`sh start.sh`来nohup启动，否则可以使用

```text
$ python -m apps.auth.run
```

创建多个terminal窗口来启动多个服务

**七、前端**

1、安装node环境

2、在前端项目根目录下`npm install`，稍等片刻安装依赖包

3、修改`config/dev.env.js`中的`BASE_URL`地址为上面的后端地址`http://localhost:80`

4、运行`npm run dev`即可打开本项目



