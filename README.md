# Docker化PHP环境

:whale: 使用 **docker-compose** 快速搭建 **php** 开发环境。

## 环境构成

将 `php-fpm` 和 `nginx` 容器分开，通过 `php:9000` 端口通信。

同时添加常见数据库支持。

> `docker-compose.yaml` 默认只开启 `php-fpm` 和 `nginx`，需要其他服务解开注释就好。

- [x] MySQL
- [x] MongoDB

Docker环境安装可以参考我的博客 ：[Docker化PHP环境](https://opso-code.github.io/post/docker-php/)

> 注：国内环境下，推荐使用Docker加速下载。

### PHP

使用的官方 `php:fpm` 镜像，可在 `.env` 中定义具体版本，默认 `7.3-fpm`。

> 默认替换了国内源，修改了时区（`Asia/Shanghai`），可在 `.env` 中自定义。

在此基础上添加或编译开启了以下扩展：

- swoole
- redis/hiredis
- mysqli
- pdo_mysql
- mongodb
- memcached
- GD
- imagick
- gmp
- xdebug
- bcmath
- sockets
- pcntl
- sysvmsg
- sysvsem
- sysvshm
- ...

说明：

- 添加了 `swoole` 扩展，默认最新版本，可以在 `.env` 文件中指定版本
- 添加了 `composer` 支持
- 添加了 `xdebug` 支持，默认指定 `xdebug.idekey = PHPSTORM`，端口是 `9001`，可以在 `phpstorm` 中配置，实现逐步调试功能。注意，与swoole协程功能冲突。

### Nginx

使用的 `nginx:stable-alpine` 镜像（默认），可在 `.env` 中定义版本。

> 默认替换了国内源，修改了时区（`Asia/Shanghai`），见 [.env 配置说明](#.env 配置说明)

### Mongodb

默认使用 `mongo:3.0.15` 镜像，可在 `.env` 中定义版本。

如果需要初始化一些脚本，需要在 `./mongo/initdb.d/`中放入 `.sh` 脚本文件。

数据库文件默认放到了 `/data/db/mongodb` 可以根据实际情况修改

启动后可以使用 `wwwroot` 中的 `adminer.php` 通过web访问mongo数据库，登录时`host`输入填写 `mongo` 就可以连接了。

### MySQL

默认使用 `mysql:5.6` 镜像，可在 `.env` 中定义版本。

构建时候已经将 `./mysql/my.conf` 加入到镜像中了，可自行加入mysql配置项。

如果需要默认导入数据库数据，需要在 `./mysql/initdb.d/ `中放入SQL文件。

数据库文件默认放到了 `/data/db/mysql` 可以根据实际情况修改；注意，如果在Windows虚拟机下，不要放到虚拟机共享目录下，否则mysql启动报错，不支持 `Linux aio`。

启动后可以使用 `wwwroot` 中的 `adminer.php` 通过web访问mysql数据库，登录`host`填写 `mysql` 就可以了。

## .env 配置说明

我们需要复制一份 `.env` 配置，根据自己的需求修改。

```bash
$ cp env-example .env
```

变量说明：

| 环境变量            | 说明                                                 |
| ------------------- | ---------------------------------------------------- |
| WORK_DIR            | Web项目目录                                          |
| TZ                  | 系统时区设置                                         |
| CHANGE_SOURCE       | 是否改为国内源地址（true为修改）                     |
| PHP_VERSION         | php指定版本                                          |
| NGINX_VERSION       | nginx指定版本                                        |
| EXT_SWOOLE_VERSION  | swoole扩展指定版本                                   |
| EXT_MONGODB_VERSION | mongodb扩展指定版本                                   |
| INSTALL_XDEBUG      | 是否安装xdebug（注意，命令行下运行与swoole协程冲突） |
| MYSQL_ROOT_PASSWORD | mysql管理员密码                                      |
| MYSQL_USER          | 需要新加的用户名                                     |
| MYSQL_PASS          | 需要新加的用户的密码                                 |
| MYSQL_PORT          | mysql映射端口                                        |
| MYSQL_DATABASE      | 登录数据库名                                         |
| MONGO_VERSION       | mongodb版本                                          |
| DB_MONGO_PATH       | mongodb存储目录                                      |
| MYSQL_PORT          | mongodb映射端口                                      |
| MONGO_ROOT_USER     | 初始化管理员用户名                                   |
| MONGO_ROOT_PASS     | 初始化管理员密码                                     |
| MONGO_DATABASE      | 登录数据库名                                         |

## Nginx 站点配置

为了方便本地开发调试，建议在本地使用`hosts`域名方式区分多站点。

比如取一个虚拟站点名字 `local.app`，然后设置好站点配置。

```nginx
$ cp nginx/conf.d/local.app.conf.example nginx/conf.d/local.app.conf
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

在 **宿主机** 的 `/etc/hosts` 文件中加入一条记录，根据宿主机的实际IP设置，我的是 `192.168.33.10`。

```bash
192.168.33.10 local.app
```

这样访问`local.app`域名会重定向到你的虚拟机里的nginx服务了。

## 运行

```bash
$ docker-compose build
// 后台运行
$ docker-compose up -d
// 进入php容器bash环境
$ docker-compose exec -it compose-php bash
```

浏览器打开 http://local.app 就可以看到php站点了 :tada::tada::tada: