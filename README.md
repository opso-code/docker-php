##  Docker化PHP环境

使用docker-compose 快速搭建php环境

### php

php镜像来自官方`php:fpm`，目前最新稳定版本是`7.2.1`
在此基础上添加了以下扩展：

- swoole2.0
- redis
- mysqli
- mongodb
- GD
- imagick
- xdebug

添加了 `composer`

### nginx

直接使用`nginx:1.12.2-alpine`镜像

### mongodb

直接使用`mongodb:3.2`镜像

使用方式：
```sh
$ cd docker-php/
$ docker-compose up
```
