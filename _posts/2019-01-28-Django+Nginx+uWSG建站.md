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

### 安装python3.6

```bash
apt-get update
apt-get install software-properties-common
add-apt-repository ppa:jonathonf/python-3.6
apt-get update
apt-get install python3.6
```

### 创建软链接

```bash
cd /usr/bin
rm python
ln -s python3.6 python
rm python3
ln -s python3.6 python3
```

### 安装pip3

方法一：

```bash
apt-get install python3-pip
pip3 install --upgrade pip
```

方法二：

```bash
wget https://bootstrap.pypa.io/get-pip.py
sudo python get-pip.py
```

### 安装虚拟环境

```bash
pip3 install virtualenv
```

查看已安装的包

```bash
pip3 list
```

### 创建虚拟环境

```bash
virtualenv mysite_env --python=python3.6
source mysite_env/bin/activate
deactivate
```

### 安装git

```bash
apt-get install git
```

### 安装环境包

```bash
pip install Django==2.0
pip install django-ckeditor==5.4.0
pip install django-js-asset==1.0.0
pip install Pillow==5.0.0
pip install pytz==2017.3
```

### 安装MySQL

```bash
wget https://dev.mysql.com/get/mysql-apt-config_0.8.12-1_all.deb
dpkg -i mysql-apt-config_0.8.12-1_all.deb # 选择8.0
apt-get update
apt-get install mysql-server
```

### 安装mysqlclient包

```bash
wget https://dev.mysql.com/get/Downloads/MySQL-8.0/libmysqlclient-dev_8.0.14-1ubuntu16.04_amd64.deb
dpkg -i libmysqlclient-dev_8.0.14-1ubuntu16.04_amd64.deb 
apt-get update
apt-get remove libmysqlclient-dev
apt-get install libmysqlclient-dev
apt-get install python3.6-dev
apt-get install libssl-dev
pip install mysqlclient
```

### 安装Nginx

```bash
sudo apt-get install nginx -y
```

### 重启Nginx

```bash
nginx -s reload
systemctl restart nginx
```