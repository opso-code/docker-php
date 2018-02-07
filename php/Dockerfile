FROM php:fpm

ENV TIMEZONE Asia/Shanghai
ENV PHP_EXT_SWOOLE=swoole-2.0.12
ENV HIREDIS=v0.13.3
ENV LIBMEMCACED=libmemcached-1.0.18

RUN sed -i 's/deb.debian.org/mirrors.163.com/g' /etc/apt/sources.list && \
    apt-get update && apt-get install -y  --no-install-recommends \
    curl git unzip wget \
    apt-utils \
    build-essential \
    libssl-dev \
    libjpeg-dev \
    libfreetype6-dev \
    libpng-dev && \
    cp /usr/share/zoneinfo/${TIMEZONE} /etc/localtime && \
    echo "${TIMEZONE}" > /etc/timezone && \
    docker-php-ext-configure gd \
        --with-freetype-dir=/usr/include/ \
        --with-jpeg-dir=/usr/include/ \
        --with-png-dir=/usr/include/ && \
    docker-php-ext-install -j$(nproc) bcmath mysqli pdo_mysql gd && \
    pecl install mongodb && \
    docker-php-ext-enable mongodb && \
    pecl install xdebug && \
    docker-php-ext-enable xdebug && \
    pecl install redis && \
    docker-php-ext-enable redis && \
    # 安装composer
    php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');" && \
    php composer-setup.php && \
    php -r "unlink('composer-setup.php');" && \
    mv composer.phar /usr/local/bin/composer && \
    composer config -g repositories.packagist composer http://packagist.phpcomposer.com && \
    composer self-update && \
    apt-get remove build-essential -y && \
    apt-get clean && apt-get autoclean && apt-get autoremove

# 安装swoole
RUN apt-get update && \
    apt-get install -y wget build-essential apt-utils libsasl2-dev && \
    # hiredis
    cd /tmp && \
    wget -q https://github.com/redis/hiredis/archive/${HIREDIS}.tar.gz && \
    mkdir -p /tmp/${HIREDIS} && \
    tar -zxf ${HIREDIS}.tar.gz -C /tmp/$HIREDIS --strip-components=1 && \
    cd /tmp/$HIREDIS && \
    make -j && \
    make install && \
    ldconfig && \
    rm -rf /tmp/${HIREDIS}* && \
    # swoole
    cd /tmp && \
    pecl channel-update pecl.php.net && \
    pecl download $PHP_EXT_SWOOLE && \
    mkdir -p /tmp/$PHP_EXT_SWOOLE && \
    tar -xf ${PHP_EXT_SWOOLE}.tgz -C /tmp/$PHP_EXT_SWOOLE --strip-components=1 && \
    docker-php-ext-configure /tmp/$PHP_EXT_SWOOLE --enable-async-redis --enable-openssl --enable-sockets=/usr/local/include/php/ext/sockets && \
    docker-php-ext-install /tmp/$PHP_EXT_SWOOLE && \
    rm -rf /tmp/${PHP_EXT_SWOOLE}* && \
    # libmemcached
    wget -q https://launchpadlibrarian.net/165454254/${LIBMEMCACED}.tar.gz && \
    mkdir /tmp/$LIBMEMCACED && \
    tar -zxf $LIBMEMCACED.tar.gz -C /tmp/$LIBMEMCACED --strip-components=1 && \
    cd /tmp/$LIBMEMCACED && ./configure && make -j && make install && cd /tmp &&\
    pecl install memcached && \
    docker-php-ext-enable memcached && \
    rm -rf /tmp/$LIBMEMCACED* && \
    rm -rf /tmp/pear/* && \
    apt-get remove build-essential -y && \
    apt-get clean && apt-get autoclean && apt-get autoremove -y

# php-fpm使用以下配置
EXPOSE 9000
CMD ["php-fpm"]