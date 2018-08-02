#  Docker化PHP环境

使用 **docker-compose** 快速搭建php环境

## 环境构成

将 `php-fpm` 和 `nginx` 容器分开，通过 `php:9000` 端口通信

### php

php镜像来自官方 `php:fpm`，目前最新稳定版本是 `7.2.8`

在此基础上添加了以下等扩展：

- swoole-4.0.3
- redis/hiredis
- mysqli
- pdo_mysql
- mongodb
- GD
- memcached
- ...

手动添加了 `composer` 并替换了国内源，修改了时区（`Asia/Shanghai`）

### nginx

直接使用的 `nginx:1.12.2-alpine` 镜像

### mongodb

直接使用的 `mongodb:3.4.11` 镜像，根据具体情况修改 `/data/mongodb` 本地映射的数据库文件夹，如不需要可注释掉，其他数据库同理。

## 运行

```sh
$ cd docker-php/
// 后台运行
$ docker-compose up -d
// 进入php容器bash环境
$ docker exec -it compose-php bash
```

浏览器打开 http://127.0.0.1 Bingo!
