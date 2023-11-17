ARG PHP_VERSION=7.3
FROM php:${PHP_VERSION}-fpm
LABEL maintainer="opso <945206413@qq.com>"

# replace sources.list
ARG CHANGE_SOURCE=false
RUN if [ ${CHANGE_SOURCE} = true ]; then \
    cp /etc/apt/sources.list /etc/apt/sources.list.back \
    && sed -i 's/deb.debian.org/mirrors.163.com/g' /etc/apt/sources.list \
    && sed -i 's/security.debian.org/mirrors.163.com/g' /etc/apt/sources.list \
    && sed -i 's/archive.ubuntu.com/mirrors.163.com/g' /etc/apt/sources.list \
;fi

# timezone
ARG TZ=Asia/Shanghai
RUN cp /usr/share/zoneinfo/${TZ} /etc/localtime \
    && echo "${TZ}" > /etc/timezone

# init
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    vim \
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
    libmagickwand-dev \
    imagemagick \
    libgmp-dev \
    libzip-dev \
    apt-utils \
    && rm -rf /var/lib/apt/lists/*

# install php extentions
RUN pecl channel-update pecl.php.net \
    && docker-php-ext-configure gd \
    --with-jpeg-dir=/usr/include/ \
    --with-freetype-dir=/usr/include/ \
    && docker-php-ext-install -j$(nproc) zip bz2 bcmath mysqli pdo_mysql gd gmp sockets pcntl sysvmsg sysvsem sysvshm \
    && pecl install memcached \
    && docker-php-ext-enable memcached \
    && pecl install redis \
    && docker-php-ext-enable redis \
    && pecl install imagick \
    && docker-php-ext-enable imagick \
    && pecl clear-cache

# install mongodb extention
ARG EXT_MONGODB_VERSION=""
RUN if [ -z ${EXT_MONGODB_VERSION} ]; then \
    pecl install https://pecl.php.net/get/mongodb \
    && pecl clear-cache \
    && docker-php-ext-enable mongodb \
;else \
    pecl install https://pecl.php.net/get/mongodb-${EXT_MONGODB_VERSION}.tgz \
    && pecl clear-cache \
    && docker-php-ext-enable mongodb \
;fi

# install swoole extention
ARG EXT_SWOOLE_VERSION=""
RUN if [ -z ${EXT_SWOOLE_VERSION} ]; then \
    pecl install https://pecl.php.net/get/swoole \
    && pecl clear-cache \
    && docker-php-ext-enable swoole \
;else \
    pecl install https://pecl.php.net/get/swoole-${EXT_SWOOLE_VERSION}.tgz \
    && pecl clear-cache \
    && docker-php-ext-enable swoole \
;fi

# install xdebug and config phpstom xdebug
ARG INSTALL_XDEBUG
RUN if [ ${INSTALL_XDEBUG} = true ]; then \
    pecl install xdebug && pecl clear-cache \
    && docker-php-ext-enable xdebug \
    && echo "xdebug.idekey = PHPSTORM" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini \
    && echo "xdebug.remote_enable = 1" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini \
    && echo "xdebug.remote_handler = dbgp" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini \
    && echo "xdebug.remote_mode = req" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini \
    && echo "xdebug.remote_connect_back = on" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini \
    && echo "xdebug.remote_autostart = off" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini \
    && echo "xdebug.remote_host = host.docker.internal" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini \
    && echo "xdebug.remote_port = 9001" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini \
;fi

# install composer
RUN curl -sfL --connect-timeout 5 --max-time 15 --retry 5 --retry-delay 2 --retry-max-time 60 https://install.phpcomposer.com/installer | php -- --install-dir="/usr/bin" --filename=composer \
    && chmod +x "/usr/bin/composer" \
    && composer self-update --clean-backups

# clean up
RUN apt-get clean && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* && \
    rm /var/log/lastlog /var/log/faillog

EXPOSE 9000
CMD ["php-fpm"]