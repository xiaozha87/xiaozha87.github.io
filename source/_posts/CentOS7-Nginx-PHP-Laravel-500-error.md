---
title: Centos7 Nginx+PHP框架laravel状态码500错误解决！
date: 2018-08-25 09:02:16
tags:
- Laravel
- PHP
category:
- PHP
- Laravel
---

Laravel安装完成后的权限设置，要给`storage`，`bootstrap/cache`目录可写权限，`chmod +x storage`。

我用的环境是lnmp一键安装,配置完成之后我还是不能访问，报500错误。

首先开启php.ini中的错误提示：`display_error=on;`（PHP7.x的调试默认是OFF）。开启后发现，lnmp一键安装的环境中，在nginx的配置文件中有两处设置了open_basedir参数。

由于laravel框架的入口文件不再项目根目录，而在public目录下，当我在lnmp中用lnmp vhost add命令添加虚拟主机并将域名制定到public目录下时，会在public目录下生成.user.ini文件，
<!-- more -->
里边的内容是：`open_basedir=/home/wwwroot/blog/public:/tmp/:/proc/`，所以导致laravel请求不到public目录意外的文件而报错。

还有一处实在nginx的配置文件中/usr/local/nginx/conf/fastcgi.conf的最后有类似的配置：`fastcgi_param PHP_ADMIN_VALUE "open_basedir=$document_root/:/tmp/:/proc/";`，

这句代表open_basedir的目录跟你的站点目录相同（public），跟上边一样的效果，可以直接对这句进行了注释，或者将项目目录加入`fastcgi_param PHP_ADMIN_VALUE "open_basedir=$document_root/:/tmp/:/proc/:/home/wwwroot/laravel/";`，

然后重启lnmp即可解决问题。