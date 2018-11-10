FROM php:fpm

LABEL maintainer="opso <945206413@qq.com>"

ARG timezone=Asia/Shanghai
ARG swoole_version=4.2.2

# replace sources.list
RUN sed -i 's/deb.debian.org/mirrors.163.com/g' /etc/apt/sources.list \
    && sed -i 's/security.debian.org/mirrors.163.com/g' /etc/apt/sources.list \
    && cp /usr/share/zoneinfo/${timezone} /etc/localtime \
    && echo "${timezone}" > /etc/timezone

# init
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    curl \
    wget \
    zip \
    git \
    unzip \
    procps \
    libz-dev \
    libpq-dev \
    libjpeg-dev \
    libpng-dev \
    libfreetype6-dev \
    libssl-dev \
    libmcrypt-dev \
    libmemcached-dev \
    apt-utils \
    && rm -rf /var/lib/apt/lists/*

# install hiredis
RUN wget https://github.com/redis/hiredis/archive/v0.13.3.tar.gz -O hiredis.tar.gz \
    && mkdir -p hiredis \
    && tar -xf hiredis.tar.gz -C hiredis --strip-components=1 \
    && rm hiredis.tar.gz \
    && ( \
        cd hiredis \
        && make -j$(nproc) \
        && make install \
        && ldconfig \
    ) \
    && rm -r hiredis

# install php extentions
RUN pecl channel-update pecl.php.net \
    && docker-php-ext-configure gd \
    --enable-gd-native-ttf \
    --with-jpeg-dir=/usr/lib \
    --with-freetype-dir=/usr/include/freetype2 \
    && docker-php-ext-install -j$(nproc) bcmath mysqli pdo_mysql gd sockets\
    && pecl install mongodb \
    && docker-php-ext-enable mongodb \
    && pecl install memcached \
    && docker-php-ext-enable memcached \
    && pecl install redis \
    && docker-php-ext-enable redis \
	&& pecl install xdebug \
    && docker-php-ext-enable xdebug \
    && pecl clear-cache

RUN echo "xdebug.idekey = PHPSTORM" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini \
    && echo "xdebug.remote_enable = 1" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini \
    && echo "xdebug.remote_handler = dbgp" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini \
    && echo "xdebug.remote_mode = req" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini \
    && echo "xdebug.remote_connect_back = on" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini \
    && echo "xdebug.remote_autostart = off" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini \
    && echo "xdebug.remote_host = 192.168.33.1" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini \
    && echo "xdebug.remote_port = 9001" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini
	
# install swoole extention
RUN wget https://github.com/swoole/swoole-src/archive/v${swoole_version}.tar.gz -O swoole.tar.gz \
    && mkdir -p swoole \
    && tar -xf swoole.tar.gz -C swoole --strip-components=1 \
    && rm swoole.tar.gz \
    && ( \
        cd swoole \
        && phpize \
        && ./configure --enable-async-redis --enable-mysqlnd --enable-coroutine --enable-openssl \
        && make -j$(nproc) \
        && make install \
    ) \
    && rm -r swoole \
    && docker-php-ext-enable swoole

# install composer
RUN curl -sS https://install.phpcomposer.com/installer | php \
    && mv composer.phar /usr/local/bin/composer \
    && composer config -g repositories.packagist composer https://packagist.phpcomposer.com \
    && composer self-update --clean-backups

EXPOSE 9000
CMD ["php-fpm"]