---
layout: post
title: 在Ubuntu上使用Nginx部署django
---

最终完成软工项目后，接下来就是项目的部署。由于在软工项目里，我们使用的是django，加上我们用的是我之前搞的阿里云服务器，搭载的系统是Ubuntu 16.04。因此这篇文章将会介绍如何在Ubuntu 16.04上部署django。

### 为什么要部署

其实，看到有些（几乎所有）同学将django部署到服务器上时，仅仅是简单地用django自带的测试服务器打开，然后直接（或间接）用8000端口来从外部访问django应用，这样做的好处有两个：一是简单，二是容易调试。

然而，在实际上的生产环境中，这样部署很容易出问题。django自带的测试服务器是单线程的，当多个用户访问的时候，性能就很差，这一个测试服务器挂掉应用就完蛋了。所以，使用Nginx部署势在必行。

### 部署步骤

简单地说，部署django，我们需要做的只有三步：

    1. 环境配置
    2. 安装、配置uwsgi来作为nginx和django的中间件
    3. 安装、配置nginx来作为服务器的最前端

最后达成的效果大约是这样的：

```
    用户 <-> nginx <-> uWSGI <-> Django
```

接下来就是具体的三步如何配置。

### 环境配置

由于服务器是Ubuntu 16.04，自带了Python 2.7与Python 3.5。

如果对python的版本不满意，可以用pyenv来管理不同版本的python，灵活切换环境，很爽（本人用pyenv装了python36）。

一个大坑就是：如果用了pyenv，想保持当前python的环境，就不要用sudo语句了，不然pyenv就会失效。

另外，就是django和各个依赖库的安装，注意版本号。

简单用django自带的测试服务器测试一下是否能正常运行即可。

```shell
    python manage.py runserver 0.0.0.0:8000
```

### 安装、配置uwsgi

uwsgi的安装也非常简单，用pip安装即可。

```shell
    pip install uwsgi
```

安装完uwsgi之后，可以使用如下语句测试uwsgi（到项目目录下，把下面的PROJECT替换成你的项目名）：

```shell
    uwsgi --http :8001 --plugin python --module PROJECT.wsgi
```

然后看能否正常访问项目。

接下来我们写uwsgi的配置文件（我们项目的配置文件，仅供参考）：

```ini
    [uwsgi]
    # Django-related settings
    #plugins = python3
    socket = :8001

    # the base directory (full path)
    chdir           = /root/Friend-Reader

    # Django s wsgi file
    module          = FriendReader.wsgi

    # process-related settings
    # master
    master          = true

    # maximum number of worker processes
    processes       = 4

    # ... with appropriate permissions - may be needed
    chmod-socket    = 777
    # clear environment on exit
    vacuum          = true
    buffer-size = 32768
```

如上，项目文件夹在/root/Friend-Reader，项目名为FriendReader，因此/root/Friend-Reader/FriendReader/下有个wsgi.py文件。

如下语句即可在后台运行uwsgi:

```shell
    uwsgi --ini uwsgi.ini
```

### 安装、配置nginx

nginx的安装十分简单，只需要用apt-get即可。

```shell
    sudo apt-get install nginx
```

这时候打开服务器地址，应该会看到nginx的欢迎页面。

![好友设置]({{ site.baseurl }}/public/images/deploy-django-by-nginx/nginx.png)

好，接下来配置nginx，打开/etc/nginx/nginx.conf，在http那个大括号里，我们加上这段：

```conf
    server {
        # the port your site will be served on
        listen      80;
        # the domain name it will serve for
        server_name reader.qwertier.cn; # substitute your machine's IP address or FQDN
        charset     utf-8;
        
        # max upload size
        client_max_body_size 75M;   # adjust to taste
        
        # Django media
        location /media  {
            alias /root/Friend-Reader/media;  # your Django project's media files - amend as required
        }
        
        location /static {
            alias /root/Friend-Reader/static; # your Django project's static files - amend as required
        }
        
        # Finally, send all non-media requests to the Django server.
        location / {
            include     uwsgi_params; # the uwsgi_params file you installed
            uwsgi_pass 127.0.0.1:8001;
        }
    }
```

首先，“location /media”和“location /static”那两段都是用来配置静态文件的，静态文件是指不经django处理的文件，只需要nginx给用户就可以。

然而，对于其他请求，“location /”是将这些请求交给uwsgi处理，且设置了uwsgi的监听地址：127.0.0.1:8001。

这样，我们的django就部署好了，访问就可以使用了。

最后，我们的在线演示地址：[http://reader.qwertier.cn](http://reader.qwertier.cn)，我们的GitHub项目地址：[https://github.com/HIT-Three-Friends/Friend-Reader/](https://github.com/HIT-Three-Friends/Friend-Reader/)。
