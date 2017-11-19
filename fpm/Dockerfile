# You can set "cli" for this ARG and then use this image for cron containers
ARG PHP_MODE=fpm
FROM php:5.6-${PHP_MODE}
LABEL maintainer="sergey.nevmerzhitsky@gmail.com"

ARG FPM_GROUP=www-data
ARG FPM_USER=www-data
ARG PECL_APCU_VERSION=4.0.11
ARG PECL_MONGO_VERSION=1.6.14
ARG PHP_MODE
ARG ZEND_GUARD_LOADER_URL=http://downloads.zend.com/guard/7.0.0/zend-loader-php5.6-linux-x86_64_update1.tar.gz

RUN set -xe; \
    apt-get update; \
    # apt-get install -y --no-install-recommends ...
    DEBIAN_FRONTEND=noninteractive apt-get install -y \
        gettext-base \
        libfreetype6-dev \
        libicu-dev \
        libjpeg62-turbo-dev \
        libldap2-dev \
        libldb-dev \
        libmcrypt-dev \
        libpng12-dev \
        libpq-dev \
        libsasl2-dev \
        libtidy-dev \
        libxml2-dev \
        libxslt1-dev \
        # Temporary for installing Zend Guard Loader
        wget \
    ; \
    # Fix of https://bugs.php.net/bug.php?id=49876
    ln -fs /usr/lib/x86_64-linux-gnu/libldap.so /usr/lib/; \
    docker-php-ext-configure ldap \
        --with-ldap-sasl \
    ; \
    docker-php-ext-configure gd \
        --with-freetype-dir=/usr/include/ \
        --with-jpeg-dir=/usr/include/ \
        --with-png-dir=/usr/include/ \
    ; \
    # Possible values for ext-name:
    # bcmath bz2 calendar ctype curl dba dom enchant exif fileinfo filter ftp gd
    # gettext gmp hash iconv imap interbase intl json ldap mbstring mcrypt mssql
    # mysql mysqli oci8 odbc opcache pcntl pdo pdo_dblib pdo_firebird pdo_mysql
    # pdo_oci pdo_odbc pdo_pgsql pdo_sqlite pgsql phar posix pspell readline
    # recode reflection session shmop simplexml snmp soap sockets spl standard
    # sybase_ct sysvmsg sysvsem sysvshm tidy tokenizer wddx xml xmlreader xmlrpc
    # xmlwriter xsl zip
    docker-php-ext-install -j$(nproc) \
        bcmath \
        # Really required?
        calendar \
        # Really required?
        intl \
        gd \
        ldap \
        mcrypt \
        opcache \
        pcntl \
        pgsql \
        soap \
        sockets \
        tidy \
        xsl \
        zip \
    ; \
    # @TODO Is really APCu required?
    pecl install apcu-$PECL_APCU_VERSION dom_varimport mongo-$PECL_MONGO_VERSION; \
    docker-php-ext-enable apcu dom_varimport mongo; \
    ( \
        mkdir /tmp/zend_guard_loader; \
        cd /tmp/zend_guard_loader; \
        wget -nv $ZEND_GUARD_LOADER_URL; \
        tar xzvf $(basename $ZEND_GUARD_LOADER_URL) --strip-components=1; \
        chmod -R 755 .; \
        cp ZendGuardLoader.so /usr/local/lib/php/extensions/no-debug-non-zts-20131226; \
        cp opcache.so /usr/local/lib/php/extensions/no-debug-non-zts-20131226; \
        rm -rf /tmp/zend_guard_loader \
    ); \
    DEBIAN_FRONTEND=noninteractive apt-get remove -y wget; \
    apt-get autoremove -y; \
    apt-get clean all; \
    rm -rf /var/lib/apt/lists/*

# It's important to name this file with "00-" prefix.
# This forces PHP to load ZendGuardLoader BEFORE opcache and avoids segfaults.
COPY 00-php.over.ini $PHP_INI_DIR/conf.d
COPY apcu.ini $PHP_INI_DIR/conf.d/docker-php-ext-apcu.ini
COPY mongo.ini $PHP_INI_DIR/conf.d/docker-php-ext-mongo.ini
COPY opcache.ini $PHP_INI_DIR/conf.d/docker-php-ext-opcache.ini
COPY www.conf /tmp/www.conf

RUN set -xe; \
    if [ "$PHP_MODE" = "fpm" ]; then \
        envsubst < /tmp/www.conf > /usr/local/etc/php-fpm.d/www.conf; \
    fi

VOLUME /var/lib/php/sessions
VOLUME /tmp
