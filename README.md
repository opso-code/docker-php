#  Docker化PHP环境

使用 **docker-compose** 快速搭建php环境

## 环境构成

将 `php-fpm` 和 `nginx` 容器分开，通过 `php:9000` 端口通信

### PHP

`php-fpm` 镜像来自官方 `php:fpm`，目前最新稳定版本是 `7.2.12`

在此基础上添加或编译开启了以下等扩展：

- swoole
- redis/hiredis
- mysqli
- pdo_mysql
- mongodb
- GD
- memcached
- xdebug
- bcmath
- sockets
- pcntl
- sysvmsg
- sysvsem
- sysvshm
- ...

说明：

- `swoole` 版本默认`4.4.0`，编译参数可自行修改，默认是 `--enable-async-redis --enable-mysqlnd --enable-sockets --enable-openssl`
- 添加了 `composer` 支持，并替换了国内源，修改了时区（`Asia/Shanghai`）
- 添加了 `xdebug` 支持，默认指定 `xdebug.idekey = PHPSTORM`，端口是 `9001`，可以在 `phpstorm` 中配置调试，在此就不详述了。

### Nginx

使用的 `nginx:stable-alpine` 镜像，替换了国内源，修改了时区（`Asia/Shanghai`），可以在 `.env` 中设置。

### Mongodb

待续

### MySQL

待续

### Redis

待续

## 如何使用

### .env 配置

我们需要复制一份 `.env` 配置，根据自己的需求修改。

```bash
$ cp example.env .env
```

变量说明：

- WWWROOT php代码目录
- TIMEZONE 时区设置
- CHANGE_SOURCE 是否改为国内源地址（true为修改）
- PHP_VERSION php版本
- NGINX_VERSION nginx版本
- SWOOLE_VERSION swoole扩展版本
- INSTALL_XDEBUG 是否安装xdebug（注意，与swoole协程冲突）

### nginx站点配置

为了方便本地开发调试，建议在本地使用hosts域名方式区分多站点。

比如取一个虚拟站点名字 `local.app`，然后设置好站点配置。

```nginx
$ cp nginx/conf.d/example.conf nginx/conf.d/local.app.conf
$ vim nginx/conf.d/local.app.conf
server {
    listen       80;
    server_name  local.app;
    root  /data/wwwroot/;
    location / {
        index index.html index.htm index.php;
    }
    error_page  500 502 503 504  /50x.html;
    location = /50x.html {
        root  /usr/share/nginx/html;
    }
    access_log  /var/log/nginx/${host}_access.log  main;
    location ~ \.php$ {
        fastcgi_pass   php:9000;
        fastcgi_index  index.php;
        fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
        include        fastcgi_params;
    }
}
```

在 `/etc/hosts` 文件中加入一条记录，根据宿主机的实际IP设置，我的是 `192.168.33.10`

```hosts
192.168.33.10 local.app
```

### 运行

```bash
$ docker-compose build
$ docker-compose up
```

浏览器打开 http://local.app 就可以看到php站点了。 :tada::tada::tada: