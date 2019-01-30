---
layout:     post
title:      "Django+Nginx+uWSG建站"
subtitle:   "Linux系统环境太复杂了。。。 "
date:       2019-01-28
author:     "ZZH"
header-img: "img/post-bg-2015.jpg"
tags:
    - Linux
    - Django
    - Nginx
---

### 环境准备

> Ubuntu 16.04 LTS
>
> Python3.6
>
> Django2.0

#### 安装python3.6

```shell
apt-get update
apt-get install software-properties-common
add-apt-repository ppa:jonathonf/python-3.6
apt-get update
apt-get install python3.6
```

#### 创建软链接

```shell
cd /usr/bin
rm python
ln -s python3.6 python
rm python3
ln -s python3.6 python3
```

#### 安装pip3

方法一：

```shell
apt-get install python3-pip
pip3 install --upgrade pip
```

方法二：

```shell
wget https://bootstrap.pypa.io/get-pip.py
sudo python get-pip.py
```

#### 安装虚拟环境

```shell
pip3 install virtualenv
```

查看已安装的包

```shell
pip3 list
```

#### 创建虚拟环境

```shell
virtualenv mysite_env --python=python3.6
source mysite_env/bin/activate
deactivate
```

#### 安装git

```shell
apt-get install git
```

#### 安装环境包

```shell
pip install Django==2.0
pip install django-ckeditor==5.4.0
pip install django-js-asset==1.0.0
pip install Pillow==5.0.0
pip install pytz==2017.3
```

#### 安装MySQL

```shell
wget https://dev.mysql.com/get/mysql-apt-config_0.8.12-1_all.deb
dpkg -i mysql-apt-config_0.8.12-1_all.deb # 选择8.0
apt-get update
apt-get install mysql-server
```

#### 安装mysqlclient包

```shell
wget https://dev.mysql.com/get/Downloads/MySQL-8.0/libmysqlclient-dev_8.0.14-1ubuntu16.04_amd64.deb
dpkg -i libmysqlclient-dev_8.0.14-1ubuntu16.04_amd64.deb 
apt-get update
apt-get remove libmysqlclient-dev
apt-get install libmysqlclient-dev
apt-get install python3.6-dev
apt-get install libssl-dev
pip install mysqlclient
```

#### MySQL配置

##### 登录

```
mysql -u root -p
```

##### 创建数据库

```
# 使用utf8编码，否则中文会有问题
CREATE DATABASE customize_user character set utf8;
```

### Django入门项目

#### 创建Django项目

`django-admin startproject <项目名>`

#### setting.py配置

`ALLOWED_HOSTS = ['*']`

#### 启动Django应用

进入项目目录，执行命令：`python manage.py runserver `，访问127.0.0.1:8000查看

创建超级理员用户：`python manage.py createsupperuser`，访问127.0.0.1:8000/admin查看

### uWSGI

#### 安装uwsgi

```shell
pip3 install uwsgi
```

#### uwsgi基础测试

创建一个test.py测试文件（路径任意） `vim test.py`

键盘按i进入编辑模式，输入以下内容：

```python
def application(env, start_response):
    start_response('200 OK', [('Content-Type','text/html')])
    return [b"Hello World"]
```

运行uwsgi：   `uwsgi --http :8000 --wsgi-file test.py`

**解释**：  --http :8000 使用http协议 端口8000  --wsgi-file test.py 加载指定文件

在本地浏览器进行访问 输入服务器ip地址或域名（已进行域名解析）+:8000 例如  *.*.*.*:8000 或 *.com:8000

如果没问题。你将在浏览器中看到 Hello World，这证明 web客户端--uwsgi--python 的连接是正常的

#### 使用uwsgi测试你的项目

启动uWSGI服务器的示例命令：

```shell
uwsgi --chdir=/path/to/your/project \
    --module=mysite.wsgi:application \
    --env DJANGO_SETTINGS_MODULE=mysite.settings \
    --master --pidfile=/tmp/project-master.pid \
    --socket=127.0.0.1:49152 \      # can also be a file
    --processes=5 \                 # number of worker processes
    --uid=1000 --gid=2000 \         # if root, uwsgi can drop privileges
    --harakiri=20 \                 # respawn processes taking more than 20 seconds
    --max-requests=5000 \           # respawn processes after serving 5000 requests
    --vacuum \                      # clear environment on exit
    --home=/path/to/virtual/env \   # optional path to a virtualenv
    --daemonize=/var/log/uwsgi/yourproject.log      # background the process
```

```shell
uwsgi --chdir /path/to/your/project/mysite --home /path/to/virtual/env --http :8020 --module mysite.wsgi:application
```

用本地浏览器进行访问，访问正常，但无法加载静态文件。所以需要Nginx了。

#### 查看uwsgi进程

```she
ps -aux | grep uwsgi
```

### Nginx

#### 安装Nginx

```shell
sudo apt-get install nginx
sudo /etc/init.d/nginx start    # start nginx
```

现在，通过在一个web浏览器上通过端口80访问它，来检查nginx是否正常 - 你应该会从nginx获得一个消息：”Welcome to nginx!”. 那意味着整个栈的这些模块都能一起正常工作:

`the web client <-> the web server`

如果有其他的东东已经提供端口80的服务了，并且你想要在那里使用nginx，那么你将必须重新配置nginx来提供另一个端口的服务。

#### 为你的站点配置nginx

在`/etc/nginx/sites-available/`下创建一个名为mysite.conf的文件，然后将这个写入到它里面:

```
# mysite.conf

# the upstream component nginx needs to connect to
upstream django {
    # server unix:///path/to/your/mysite/mysite.sock; # for a file socket
    server 127.0.0.1:8001; # for a web port socket (we'll use this first)
}

# configuration of the server
server {
    # the port your site will be served on
    listen      80;
    # the domain name it will serve for
    server_name .example.com; # substitute your machine's IP address or FQDN
    charset     utf-8;

    # max upload size
    client_max_body_size 75M;   # adjust to taste

    # Django media
    location /media  {
        alias /path/to/your/mysite/media;  # your Django project's media files - amend as required
    }

    location /static {
        alias /path/to/your/mysite/static; # your Django project's static files - amend as required
    }

    # Finally, send all non-media requests to the Django server.
    location / {
        uwsgi_pass  django;
        include     /path/to/your/mysite/uwsgi_params; # the uwsgi_params file you installed
    }
}
```

这个配置文件告诉nginx提供来自文件系统的媒体和静态文件，以及处理那些需要Django干预的请求。对于一个大型部署，让一台服务器处理静态/媒体文件，让另一台处理Django应用，被认为是一种很好的做法，但是现在，这样就好了。

将这个文件链接到/etc/nginx/sites-enabled，这样nginx就可以看到它了:

```shell
sudo ln -s /etc/nginx/sites-available/mysite.conf /etc/nginx/sites-enabled/
```

输入`nginx -t`显示成功，重启Nginx服务

#### 部署静态文件

在运行nginx之前，你必须收集所有的Django静态文件到静态文件夹里。首先，必须编辑mysite/settings.py，添加:

```python
STATIC_ROOT = os.path.join(BASE_DIR, "static/")
```

然后运行

```shell
python manage.py collectstatic
```

#### 基本的nginx测试

测试 nginx 的配置文件的语法是否正确

```shell
sudo nginx -t
```

重启nginx:

```shell
sudo /etc/init.d/nginx restart
```

要检查是否正确的提供了媒体文件服务，添加一个名为 `media.png` 的图像到 `/path/to/your/project/project/media directory` 中，然后访问http://example.com/media/media.png - 如果这能正常工作，那么至少你知道nginx正在正确的提供文件服务。

注意不仅仅是重启nginx，而是实际停止然后再次启动它，这将会通知是否有问题，以及问题在哪里。

#### 重启Nginx

```shell
nginx -s reload
systemctl restart nginx
sudo /etc/init.d/nginx restart
```

### 配置uWSGI服务

#### nginx和uWSGI以及test.py

让nginx对 `test.py` 应用说句”hello world”吧。

```shell
uwsgi --socket :8001 --wsgi-file test.py
```

这几乎与之前相同，除了这次有一个选项不同：

- `socket :8001`: 使用uwsgi协议，端口为8001

同时，已经配置了nginx在那个端口与uWSGI通信，而对外使用8000端口。访问:

<http://example.com:8000/>

来检查。而这是我们的栈:

```
the web client <-> the web server <-> the socket <-> uWSGI <-> Python
```

同时，你可以试着看看在http://example.com:8001的uswgi输出 - 但很有可能它不会正常工作，因为你的浏览器使用http，而不是uWSGI，但你应该能够在终端上看到来自uWSGI的输出。

#### 使用Unix socket而不是端口

目前，我们使用了一个TCP端口socket，因为它简单些，但事实上，使用Unix socket会比端口更好 - 开销更少。

编辑 `mysite_nginx.conf`, 修改它以匹配:

```
server unix:///path/to/your/mysite/mysite.sock; # for a file socket
# server 127.0.0.1:8001; # for a web port socket (we'll use this first)
```

然后重启nginx.

再次运行uWSGI:

```shell
uwsgi --socket mysite.sock --wsgi-file test.py
```

这次， `socket` 选项告诉uWSGI使用哪个文件。

在浏览器中尝试访问http://example.com:8000/。

**如果那不行**

检查nginx错误日志(/var/log/nginx/error.log)。如果你看到像这样的信息:

```
connect() to unix:///path/to/your/mysite/mysite.sock failed (13: Permission
denied)
```

那么可能你需要管理这个socket上的权限，从而允许nginx使用它。

尝试:

```shell
uwsgi --socket mysite.sock --wsgi-file test.py --chmod-socket=666 # (very permissive)
```

或者:

```shell
uwsgi --socket mysite.sock --wsgi-file test.py --chmod-socket=664 # (more sensible)
```

你可能还必须添加你的用户到nginx的组 (可能是 www-data)，反之亦然，这样，nginx可以正确地读取或写入你的socket。

值得保留nginx日志的输出在终端窗口中滚动，这样，在解决问题的时候，你就可以容易的参考它们了。

#### 使用uwsgi和nginx运行Django应用

运行我们的Django应用：

```
uwsgi --socket mysite.sock --module mysite.wsgi --chmod-socket=664
```

现在，uWSGI和nginx应该不仅仅可以为一个”Hello World”模块服务，还可以为你的Django项目服务。

#### 配置uWSGI以允许.ini文件

我们可以将用在uWSGI上的相同的选项放到一个文件中，然后告诉 uWSGI使用该文件运行。这使得管理配置更容易。

在Django同级目录下创建一个`mysite_uwsgi`文件夹。进入该文件夹，创建一个名为 ``mysite_uwsgi.ini`` 的文件:

```
# mysite_uwsgi.ini file
[uwsgi]

# Django-related settings
# the base directory (full path)
chdir           = /path/to/your/project
# Django's wsgi file
module          = project.wsgi
# the virtualenv (full path)
home            = /path/to/virtualenv

# process-related settings
# master
master          = true
# maximum number of worker processes
processes       = 10
# the socket (use the full path to be safe
socket          = /path/to/your/project/mysite.sock
# ... with appropriate permissions - may be needed
# chmod-socket    = 664
# clear environment on exit
vacuum          = true

pidfile=/path/to/mysite_uwsgi/master.pid
daemonize=/path/to/mysite_uwsgi/mysite.log
```

然后使用这个文件运行uswgi：

```
uwsgi --ini mysite_uwsgi.ini # the --ini option is used to specify a file
```

再次，测试Django站点是否如预期工作。

使用如下命令，查询指定端口的进程id

```shell
ps -aux | grep uwsgi
```

#### uwsgi 启动与结束

```shell
# 启动
uwsgi --ini uwsgi.ini
# 重启
uwsgi --reload /path/to/mysite_uwsgi/master.pid
# 结束
uwsgi --stop /path/to/mysite_uwsgi/master.pid
```

如果出现错误 `signal_pidfile()/kill(): No such process [core/uwsgi.c line 1627]，`是由于 master.pid 的进程ID不正确，修改/path/to/mysite_uwsgi/master.pid为正确的pid就可以。pid可通过命令`ps -aux | grep uwsgi`查询得到，第二个数字即是。