FROM php:8.3-apache AS backend_php

ARG NODE_VERSION
ARG BACKEND_PHP_IP

RUN echo "ServerName $BACKEND_PHP_IP" >> /etc/apache2/apache2.conf
CMD apachectl -DFOREGROUND

RUN addgroup --gid 1000 appuser
RUN adduser --uid 1000 --gid 1000 --disabled-password appuser
RUN adduser www-data appuser
RUN mkdir /home/appuser/bin

ENV ACCEPT_EULA=Y
ENV APACHE_ROOT_DIR=/var/www/html
ENV APACHE_DOCUMENT_ROOT=/var/www/html
ENV APACHE_LOG_DIR=/var/log
ENV APACHE_RUN_USER=appuser
ENV APACHE_RUN_GROUP=appuser

RUN a2dismod mpm_event
RUN a2enmod mpm_prefork rewrite expires ssl headers

RUN DEBIAN_FRONTEND=noninteractive apt-get update && apt-get install -y \
        git \
        zip \
        unzip \
        vim \
        curl \
        zlib1g-dev \
        libzip-dev \
        libpng-dev \
        libssl-dev \
        libicu-dev \
        libxml2-dev \
        libonig-dev \
        libxslt-dev \
        freetds-dev \
        apache2-utils \
        wget \
        libpcre3 \
        libpcre3-dev \
        apt-utils \
        xvfb \
        sqlite3 \
        sqlite3-pcre \
        libfontconfig \
        libltdl-dev \
        libodbc1 \
        gnupg \
        redis-tools \
        zlib1g-dev

        RUN DEBIAN_FRONTEND=noninteractive apt-get update \
        && curl https://packages.microsoft.com/keys/microsoft.asc | apt-key add - \
        && curl https://packages.microsoft.com/config/debian/11/prod.list \
        > /etc/apt/sources.list.d/mssql-release.list \
        && apt-get install -y --no-install-recommends locales \
        apt-transport-https \
        && echo "en_US.UTF-8 UTF-8" > /etc/locale.gen \
        && locale-gen \
        && apt-get update \
        && apt-get install -y msodbcsql18 mssql-tools18 unixodbc-dev \
        && docker-php-ext-install pdo \
        && echo "en_US.UTF-8 UTF-8" > /etc/locale.gen \
        && pecl install sqlsrv pdo_sqlsrv || \
        apt-get install -y --allow-downgrades \
                odbcinst=2.3.7 \
                odbcinst1debian2=2.3.7 \
                unixodbc=2.3.7 \
                unixodbc-dev=2.3.7

RUN docker-php-ext-install mysqli \
	&& docker-php-ext-enable mysqli

RUN docker-php-ext-install pdo pdo_mysql

RUN docker-php-ext-install pdo intl gd exif bcmath mbstring xsl  \
        && pecl install sqlsrv pdo_sqlsrv  \
        && docker-php-ext-enable sqlsrv pdo_sqlsrv

RUN ln -s /usr/include /usr/local/incl

RUN apt-get install -y libmemcached-dev libmemcached11 \
	&& apt-get install -y memcached libmemcached-tools \
	&& pecl install -f memcached-3.2.0 \
	&& docker-php-ext-enable memcached

RUN docker-php-ext-configure pdo_dblib --with-libdir=lib/x86_64-linux-gnu \
        && docker-php-ext-install pdo_dblib

RUN docker-php-source extract
RUN docker-php-ext-configure pdo_odbc --with-pdo-odbc=unixODBC,/usr/
RUN docker-php-ext-install pdo_odbc \
        && cd /usr/src/php/ext/odbc \
        && phpize \
        && sed -ri 's@^ *test +"\$PHP_.*" *= *"no" *&& *PHP_.*=yes *$@#&@g' configure \
        && ./configure --with-unixODBC=shared,/usr \
        && docker-php-ext-install odbc

RUN ln -s /usr/include /usr/local/incl

RUN apt-get -y autoremove \
&& apt-get clean \
&& rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* /usr/share/doc/*

COPY --from=composer:latest /usr/bin/composer /usr/local/bin/composer

COPY laravel.conf /etc/apache2/sites-available/

RUN a2dissite 000-default.conf
RUN a2ensite laravel.conf

EXPOSE 80

USER appuser

RUN curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.1/install.sh | bash
ENV NVM_DIR=/home/appuser/.nvm
RUN . "$NVM_DIR/nvm.sh" && nvm install ${NODE_VERSION}
RUN . "$NVM_DIR/nvm.sh" && nvm use v${NODE_VERSION}
RUN . "$NVM_DIR/nvm.sh" && nvm alias default v${NODE_VERSION}
RUN . "$NVM_DIR/nvm.sh" && nvm install-latest-npm
ENV PATH="/home/appuser/.nvm/versions/node/v${NODE_VERSION}/bin/:${PATH}"

WORKDIR /var/www/html
