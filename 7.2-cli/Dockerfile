FROM ubuntu:bionic
LABEL maintainer="Trac Nguyen <npbtrac@yahoo.com>"

ENV OS_LOCALE="en_US.UTF-8" \
    DEBIAN_FRONTEND=noninteractive
RUN apt-get update && apt-get install -y locales && locale-gen ${OS_LOCALE}
ENV LANG=${OS_LOCALE} \
    LANGUAGE=${OS_LOCALE} \
    LC_ALL=${OS_LOCALE}
ENV PHP_RUN_DIR=/run/php \
    PHP_LOG_DIR=/var/log/php \
    PHP_CONF_DIR=/etc/php/7.2 \
    PHP_DATA_DIR=/var/www/html

RUN \
    BUILD_DEPS='software-properties-common' \
    && dpkg-reconfigure locales \
    # Install common libraries
    && apt-get install --no-install-recommends -y $BUILD_DEPS \
    && add-apt-repository -y ppa:ondrej/php \
    && apt-get update -y

# Install PHP libraries
# php7.2-mcrypt not support and move to PECL. Use extension Sodium: http://php.net/manual/book.sodium.php
RUN apt-get install -y \
    build-essential curl git python libglib2.0-dev wget \
    make g++ bzip2 xz-utils pkg-config wget \
    php7.2-cli php7.2-apcu php7.2-dev php7.2-readline php7.2-mbstring php7.2-memcached php7.2-zip \
    php7.2-imagick php7.2-intl php7.2-json php7.2-curl php7.2-gd php7.2-pgsql php7.2-mysql php-pear \
    php7.2-psr php7.2-redis php7.2-xml php7.2-yaml \
    php7.0-phalcon \
    && mkdir -p ${PHP_LOG_DIR} ${PHP_RUN_DIR}
RUN apt-get update -y && apt-get install -y php7.2-memcache php7.2-mongodb php7.2-tidy php7.2-soap

# Install v8js (crazy one)
RUN add-apt-repository -y ppa:stesie/libv8 \
    && apt-get update -y \
    && apt-get install libv8-7.5 libv8-7.5-dev \
    && echo "/opt/libv8-7.5/lib" | tee -a /etc/ld.so.conf.d/libv8.conf \
    && ldconfig \
    && printf "\/opt/libv8-7.5\n" | CFLAGS=-w CPPFLAGS=-w pecl install v8js-2.1.1 \
    && echo extension=v8js.so > ${PHP_CONF_DIR}/cli/conf.d/99-v8js.ini

# Install ioncube, it needs to be loaded first so be sure to name it with correct prefix
COPY ioncube_loader_lin_7.2.so /tmp/
RUN mv /tmp/ioncube_loader_lin_7.2.so /usr/lib/php/20170718/ \
    && echo "zend_extension=ioncube_loader_lin_7.2.so" > /etc/php/7.2/cli/conf.d/00-ioncube_loader_lin_7.2.ini

# Install phpunit
COPY phpunit.phar /tmp/
RUN cd /tmp \
    && mv phpunit.phar /usr/local/bin/phpunit \
    && chmod +x /usr/local/bin/phpunit

# Install composer
RUN curl -sS https://getcomposer.org/installer | php -- --version=1.10.9 --install-dir=/usr/local/bin --filename=composer \
    && chmod +x /usr/local/bin/composer

# Node and yarn
RUN curl -sL https://deb.nodesource.com/setup_12.x  | bash -
RUN apt-get -y install nodejs
RUN npm install -g yarn

# Making composer faster
RUN mkdir ~/.composer/cache \
    && echo "{}" > ~/.composer/composer.json \
    && composer global require hirak/prestissimo

# Cleaning
RUN apt-get purge -y --auto-remove $BUILD_DEPS \
    && apt-get autoremove -y && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

WORKDIR ${PHP_DATA_DIR}