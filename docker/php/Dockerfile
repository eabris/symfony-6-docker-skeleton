FROM php:8.2-apache

# set main params
ARG BUILD_ARGUMENT_ENV=dev
ENV ENV=$BUILD_ARGUMENT_ENV
ENV APP_HOME /var/www/html
ARG HOST_UID=1000
ARG HOST_GID=1000
ENV USERNAME=www-data
ARG INSIDE_DOCKER_CONTAINER=1
ENV INSIDE_DOCKER_CONTAINER=$INSIDE_DOCKER_CONTAINER
ARG XDEBUG_CONFIG=main
ENV XDEBUG_CONFIG=$XDEBUG_CONFIG

# check environment
RUN if [ "$BUILD_ARGUMENT_ENV" = "default" ]; then echo "Set BUILD_ARGUMENT_ENV in docker build-args like --build-arg BUILD_ARGUMENT_ENV=dev" && exit 2; \
    elif [ "$BUILD_ARGUMENT_ENV" = "dev" ]; then echo "Building development environment."; \
    elif [ "$BUILD_ARGUMENT_ENV" = "prod" ]; then echo "Building production environment."; \
    else echo "Set correct BUILD_ARGUMENT_ENV in docker build-args like --build-arg BUILD_ARGUMENT_ENV=dev. Available choices are dev,test,staging,prod." && exit 2; \
    fi

# Update
RUN apt-get -y update --fix-missing && \
    apt-get upgrade -y && \
    apt-get --no-install-recommends install -y apt-utils && \
    rm -rf /var/lib/apt/lists/*
# Install useful tools and install important libaries
RUN apt-get -y update && \
    apt-get -y --no-install-recommends install nano wget \
    dialog \
    libsqlite3-dev \
    libsqlite3-0 && \
    apt-get -y --no-install-recommends install default-mysql-client \
    zlib1g-dev \
    libzip-dev \
    libicu-dev && \
    apt-get -y --no-install-recommends install --fix-missing apt-utils \
    build-essential \
    git \
    curl \
    libonig-dev && \
    apt-get install -y iputils-ping && \
    apt-get -y --no-install-recommends install --fix-missing libcurl4 \
    libcurl4-openssl-dev \
    zip \
    openssl

RUN apt-get install -y supervisor
RUN apt-get install -y unzip

RUN apt-get install -y libssl-dev
RUN apt-get install -y librabbitmq4
RUN apt-get install -y librabbitmq-dev
RUN apt-get install -y autotools-dev
RUN apt-get install -y automake
RUN apt-get install -y autoconf
RUN apt-get install -y libtool
RUN apt-get install -y cmake

#install amqp
RUN cd /usr/local/src && \
    git clone https://github.com/alanxz/rabbitmq-c.git && \
    cd rabbitmq-c/ && \
    pwd && \
    mkdir build && cd build && \
    cmake .. && \
    cmake --build .
RUN pecl install amqp

# Install xdebug
RUN pecl install xdebug-3.2.1 && \
    docker-php-ext-enable xdebug && \
    mkdir /var/log/xdebug

# Install redis
RUN pecl install redis-5.3.7 && \
    docker-php-ext-enable redis

# Install imagick
RUN apt-get update && \
    apt-get -y --no-install-recommends install --fix-missing libmagickwand-dev && \
    rm -rf /var/lib/apt/lists/*

# Workarround until imagick is available in pecl with php8 support
# Imagick Commit to install
# https://github.com/Imagick/imagick
ARG IMAGICK_COMMIT="661405abe21d12003207bc8eb0963fafc2c02ee4"

RUN cd /usr/local/src && \
    git clone https://github.com/Imagick/imagick && \
    cd imagick && \
    git checkout ${IMAGICK_COMMIT} && \
    phpize && \
    ./configure && \
    make && \
    make install && \
    cd .. && \
    rm -rf imagick && \
    docker-php-ext-enable imagick

# Other PHP8 Extensions

RUN docker-php-ext-install pdo_mysql && \
    docker-php-ext-install pdo_sqlite && \
    docker-php-ext-install bcmath && \
    docker-php-ext-install mysqli && \
    docker-php-ext-install curl && \
    docker-php-ext-install zip && \
    docker-php-ext-install -j$(nproc) intl && \
    docker-php-ext-install mbstring && \
    docker-php-ext-install gettext && \
    docker-php-ext-install calendar && \
    docker-php-ext-enable amqp && \
    docker-php-ext-install exif


# Install Freetype
RUN apt-get -y update && \
    apt-get --no-install-recommends install -y libfreetype6-dev \
libjpeg62-turbo-dev \
libpng-dev && \
    rm -rf /var/lib/apt/lists/* && \
    docker-php-ext-configure gd --enable-gd --with-freetype --with-jpeg && \
    docker-php-ext-install gd

RUN rm -rf /tmp/*
RUN rm -rf /var/list/apt/*
RUN apt-get clean

# disable default site and delete all default files inside APP_HOME
RUN a2dissite 000-default.conf
RUN rm -r $APP_HOME

# create document root, fix permissions for www-data user and change owner to www-data
RUN mkdir -p $APP_HOME/public && \
    mkdir -p /home/$USERNAME && chown $USERNAME:$USERNAME /home/$USERNAME \
    && usermod -o -u $HOST_UID $USERNAME -d /home/$USERNAME \
    && groupmod -o -g $HOST_GID $USERNAME \
    && chown -R ${USERNAME}:${USERNAME} $APP_HOME

# put apache and php config for Symfony, enable sites
COPY ./docker/general/symfony.conf /etc/apache2/sites-available/symfony.conf
COPY ./docker/general/symfony-ssl.conf /etc/apache2/sites-available/symfony-ssl.conf
RUN a2ensite symfony.conf && a2ensite symfony-ssl
COPY ./docker/$BUILD_ARGUMENT_ENV/php.ini /usr/local/etc/php/php.ini

# enable apache modules
RUN a2enmod rewrite
RUN a2enmod ssl

# install Xdebug in case dev/test environment
#COPY ./docker/general/do_we_need_xdebug.sh /tmp/
#COPY ./docker/dev/xdebug-${XDEBUG_CONFIG}.ini /tmp/xdebug.ini
#RUN chmod u+x /tmp/do_we_need_xdebug.sh && /tmp/do_we_need_xdebug.sh

# install composer
COPY --from=composer:latest /usr/bin/composer /usr/bin/composer
RUN chmod +x /usr/bin/composer
ENV COMPOSER_ALLOW_SUPERUSER 1

# add supervisor
RUN mkdir -p /var/log/supervisor
COPY --chown=root:root ./docker/general/supervisord.conf /etc/supervisor/conf.d/supervisord.conf
COPY --chown=root:crontab ./docker/general/cron /var/spool/cron/crontabs/root
RUN chmod 0600 /var/spool/cron/crontabs/root

# generate certificates
# TODO: change it and make additional logic for production environment
RUN openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/ssl/private/ssl-cert-snakeoil.key -out /etc/ssl/certs/ssl-cert-snakeoil.pem -subj "/C=HU/ST=Budapest/L=Budapest/O=Security/OU=Development/CN=example.com"

# set working directory
WORKDIR $APP_HOME

USER ${USERNAME}

# copy source files
COPY --chown=${USERNAME}:${USERNAME} . $APP_HOME/

#COPY . $APP_HOME
#USER root
#RUN chmod 0777 -R $APP_HOME/var/cache/dev
#RUN chmod 0777 -R $APP_HOME/var/log
#RUN chown ${USERNAME}:${USERNAME} -R $APP_HOME/vendor
#USER ${USERNAME}

WORKDIR $APP_HOME

# install all PHP dependencies
RUN if [ "$BUILD_ARGUMENT_ENV" = "dev" ]; then COMPOSER_MEMORY_LIMIT=-1 composer install --optimize-autoloader --no-interaction; \
    else export APP_ENV=$BUILD_ARGUMENT_ENV && COMPOSER_MEMORY_LIMIT=-1 composer install --optimize-autoloader --no-interaction --no-progress --no-dev; \
    fi

# create cached config file .env.local.php in case of prod environment
RUN if [ "$BUILD_ARGUMENT_ENV" = "prod" ]; then composer dump-env $BUILD_ARGUMENT_ENV; \
    fi

USER root







