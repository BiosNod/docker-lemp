FROM php:7.2-fpm

# Prerequisites
RUN apt-get update && apt-get install -y libmcrypt-dev nano vim unzip screen inotify-tools
RUN pecl install mcrypt-1.0.2 && docker-php-ext-enable mcrypt
RUN apt-get install -y \
        curl \
        wget \
        git \
        libfreetype6-dev \
        libjpeg62-turbo-dev \
        libmcrypt-dev \
        libpng-dev \
    && docker-php-ext-install -j$(nproc) iconv mbstring mysqli pdo_mysql zip \
    && docker-php-ext-configure gd --with-freetype-dir=/usr/include/ --with-jpeg-dir=/usr/include/ \
    && docker-php-ext-install -j$(nproc) gd bcmath
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer
ADD ./php.ini /usr/local/etc/php/conf.d/40-custom.ini

# Install necessary locales
RUN apt-get install -y locales \
    && echo "ru_RU.UTF-8 UTF-8" > /etc/locale.gen \
    && locale-gen

# XDebug
RUN pecl install xdebug-2.7.0
RUN docker-php-ext-enable xdebug
RUN echo "xdebug.remote_enable=1" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini && \
    # Эта опция позволяет НЕ запускать скрипт с отладчиком если отладка НЕ используется
    # иначе сайт будет медленно выполняться, нужно будет поставить Xdebug helper расширение
    # чтобы включать отладку вручную, но зато без отладки сайт будет выполнятся быстро
    echo "xdebug.remote_autostart=0" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini && \
    echo "xdebug.remote_handler=dbgp" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini && \
    echo "xdebug.remote_mode=req" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini && \
    # Не путать с портом DBGP proxy в PHPSTORM
    echo "xdebug.remote_port=9000" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini && \
    #RUN echo "xdebug.remote_log=/var/log/xdebug_remote.log" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini
    # Как показала практика idekey как будто бы ни на что не влияет, подключения всё равно происходят если использовать слушатель в PHPSTORM
    echo "xdebug.idekey=PHPSTORM" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini && \
    # В случае 1 ip компа с PHPSTORM будет определяться автоматически
    echo "xdebug.remote_connect_back=1" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini && \
    # Работает только если remote_connect_back равен 0, подключение будет строго к этому ip
    echo "xdebug.remote_host=192.168.0.2" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini


# Bcgen
RUN git clone https://github.com/nanosonde/bcgen.git && cd bcgen && \
    phpize && \
    ./configure && \
    make && \
    make install ; \
    # Теперь увидим куда установилась .so-шка, копируем путь вида zend_extension=/usr/lib/php/20170718/bcgen.so
    # И вставляем его в /etc/php/7.2/apache2/php.ini и в /etc/php/7.2/cli/php.ini и в /etc/php/7.2/fpm/php.ini
    # Получается так (будет добавлено автоматически с помощью echo):
    # [PHP]
    # zend_extension=/usr/lib/php/20170718/bcgen.so
    echo 'zend_extension=/usr/local/lib/php/extensions/no-debug-non-zts-20170718/bcgen.so' >> /usr/local/etc/php/conf.d/bcgen.ini

# MSSQL
RUN apt-get install unixodbc-dev -y && pecl install sqlsrv-4.1.6.1 && pecl install pdo_sqlsrv-4.1.6.1
RUN echo 'extension=/usr/local/lib/php/extensions/no-debug-non-zts-20170718/sqlsrv.so' >> /usr/local/etc/php/conf.d/mssql.ini
RUN echo 'extension=/usr/local/lib/php/extensions/no-debug-non-zts-20170718/pdo_sqlsrv.so' >> /usr/local/etc/php/conf.d/mssql.ini

# Memcached
RUN apt-get install -y \
    libmemcached11 \
    libmemcachedutil2 \
    libmemcached-dev \
    libz-dev \
    git \
    && cd /root \
    && git clone -b php7 https://github.com/php-memcached-dev/php-memcached \
    && cd php-memcached \
    && phpize \
    && ./configure \
    && make \
    && make install \
    && cd .. \
    && rm -rf  php-memcached \
    && echo extension=memcached.so >> /usr/local/etc/php/conf.d/memcached.ini

# Memcache
RUN apt-get install -y --no-install-recommends unzip libssl-dev libpcre3 libpcre3-dev \
    && cd /tmp \
    && curl -sSL -o php7.zip https://github.com/websupport-sk/pecl-memcache/archive/php7.zip \
    && unzip php7 \
    && cd pecl-memcache-php7 \
    && /usr/local/bin/phpize \
    && ./configure --with-php-config=/usr/local/bin/php-config \
    && make \
    && make install \
    && echo "extension=memcache.so" > /usr/local/etc/php/conf.d/ext-memcache.ini \
    && rm -rf /tmp/pecl-memcache-php7 php7.zip

CMD ["php-fpm"]

#COPY ./src/composer.json /var/www
#VOLUME ["/var/www"]
#WORKDIR /var/www
#RUN cd /var/www && mkdir tests && mkdir database && composer install --prefer-source --no-interaction