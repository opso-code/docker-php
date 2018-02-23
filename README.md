#  Docker化PHP环境

使用docker-compose 快速搭建php环境

## 环境构成

### php

php镜像来自官方 `php:fpm`，目前最新稳定版本是`7.2.1`
在此基础上添加了以下扩展：

- swoole2.0
- redis
- mysqli
- pdo_mysql
- mongodb
- GD
- memcached
- xdebug

手动添加了 `composer` 并替换了国内源

### nginx

直接使用 `nginx:1.12.2-alpine` 镜像

### mongodb

直接使用 `mongodb:3.4.11` 镜像

## 镜像版本查询工具

可以列出[dockerhub镜像仓库](https://hub.docker.com)中相关镜像的版本列表：`docker_show_tags.sh`

使用示例：
```sh
$ sudo chmod +x docker_show_tags.sh
$ ./docker_show_tags.sh php
php:5.6-fpm
php:5-fpm
php:7.0.27-fpm
php:7.0.27-fpm-jessie
php:7.0-fpm
php:7.0-fpm-jessie
php:7.1.13-fpm
php:7.1.13-fpm-jessie
php:7.1-fpm
php:7.1-fpm-jessie
```


## 运行

```sh
$ cd docker-php/
$ docker-compose up
```
