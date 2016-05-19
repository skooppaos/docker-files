FROM alpine:latest

MAINTAINER Scott Molinari

# install core things
RUN apk --update add bash wget curl git grep zlib tar make libxml2 readline \
    freetype openssl libjpeg-turbo libpng libmcrypt libwebp icu

# compile and configure PHP
ENV PHP_VERSION 7.0.6
ADD etc /etc

# run everything here in one RUN command so the layer is committed AFTER cleanup
# which will drastically shrink the image size

# install PHP dev dependencies
RUN buildDeps=" build-base re2c file readline-dev autoconf binutils bison \
        libxml2-dev curl-dev freetype-dev openssl-dev \
        libjpeg-turbo-dev libpng-dev libwebp-dev libmcrypt-dev \
        gmp-dev icu-dev libmemcached-dev" \
    && apk --update add $buildDeps \

    # download unpack php-src
    && mkdir /tmp/php && cd /tmp/php \
    && wget https://github.com/php/php-src/archive/php-${PHP_VERSION}.tar.gz \
    && tar xzf php-${PHP_VERSION}.tar.gz \
    && cd php-src-php-${PHP_VERSION} \

    # compile
    && ./buildconf --force \
    && ./configure \
        --prefix=/usr \
        --sysconfdir=/etc/php \
        --with-config-file-path=/etc/php \
        --with-config-file-scan-dir=/etc/php/conf.d \
        --enable-fpm --with-fpm-user=root --with-fpm-group=root \
        --enable-cli \
        --enable-mbstring \
        --enable-zip \
        --enable-ftp \
        --enable-bcmath \
        --enable-opcache \
        --enable-pcntl \
        --enable-mysqlnd \
        --enable-gd-native-ttf \
        --enable-sockets \
        --enable-exif \
        --enable-soap \
        --enable-calendar \
        --enable-intl \
        --enable-json \
        --enable-dom \
        --enable-libxml --with-libxml-dir=/usr \
        --enable-xml \
        --enable-xmlreader \
        --enable-phar \
        --enable-session \
        --enable-sysvmsg \
        --enable-sysvsem \
        --enable-sysvshm \
        --disable-cgi \
        --disable-debug \
        --disable-rpath \
        --disable-static \
        --disable-phpdbg \
        --with-libdir=/lib/x86_64-linux-gnu \
        --with-curl \
        --with-mcrypt \
        --with-iconv \
        --with-gd --with-jpeg-dir=/usr --with-webp-dir=/usr --with-png-dir=/usr \
        --with-freetype-dir=/usr \
        --with-zlib --with-zlib-dir=/usr \
        --with-openssl \
        --with-mhash \
        --with-pcre-regex \
        --with-pdo-mysql \
        --with-mysqli \
        --with-readline \
        --with-xmlrpc \
        --with-pear \
    && make \
    && make install \
    && make clean \

    # strip debug symbols from the binary (GREATLY reduces binary size)
    && strip -s /usr/bin/php \

    # install xdebug
    && cd /tmp \
    && git clone https://github.com/xdebug/xdebug.git \
    && cd xdebug \
    && git checkout master \
    && phpize && ./configure --enable-xdebug && make \
    && cp modules/xdebug.so /usr/lib/php/extensions/no-debug-non-zts-20151012 \

    # remove PHP dev dependencies
    && apk del $buildDeps \

    # install composer
    && cd /tmp \
    && curl -sS https://getcomposer.org/installer | php \
    && mv composer.phar /usr/local/bin/composer \

    # install phpunit
    && mkdir /spider \
    && cd /spider \
    && composer global require "phpunit/phpunit:~5.1.0" --prefer-source --no-interaction  \
    && ln -s /root/.composer/vendor/bin/phpunit /bin \

    # other clean up
    && cd / \
    && rm -rf /var/cache/apk/* \
    && rm -rf /tmp/*

EXPOSE 9000

CMD ["sh"]