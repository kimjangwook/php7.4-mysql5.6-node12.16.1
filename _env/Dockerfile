FROM php:7.3.16-apache

COPY ./php.ini /usr/local/etc/php/
COPY ./conf/*.conf /etc/apache2/sites-available/
COPY ./apache2.conf /etc/apache2/

ARG TZ=Asia/Tokyo
ARG NODE_VERSION=12.x

RUN set -xe; \
    apt-get update -yqq && \
    apt-get install -yqq --no-install-recommends \
      apt-utils vim gettext git \
      default-mysql-client \
      # for gd
      libfreetype6-dev \
      libjpeg62-turbo-dev \
      libpng-dev \
      libwebp-dev \
      libxpm-dev \
      # for ImageMagick
      libmagickwand-dev \
      # for zip
      libzip-dev zip unzip \
      # for oniguruma
      libonig-dev \
    && docker-php-ext-install bcmath gettext mbstring mysqli pdo pdo_mysql zip \
    && docker-php-ext-configure mbstring --disable-mbregex \
    && docker-php-ext-install calendar \
    && docker-php-ext-configure calendar \
    # gd
    && docker-php-ext-install -j$(nproc) iconv \
    && docker-php-ext-configure gd \
    && docker-php-ext-install -j$(nproc) gd \
    # imagick
    && pecl install imagick \
    && docker-php-ext-enable imagick \

    # iden_to_ascii()
    && docker-php-ext-install intl \
    && docker-php-ext-configure intl \
    # enable mod_rewrite
    && a2enmod rewrite \
    # enable ssl
    && a2enmod ssl \
    # enable proxy
    && a2enmod proxy \
    && a2enmod proxy_http \
    # enable env
    && a2enmod env \
    # enable headers
    && a2enmod headers \
    # enable site
    && a2ensite * \
    # for ssl
    && apt-get update \
    && apt-get -y install ngrep \
    && openssl req -new -newkey rsa:4096 -days 365 -nodes -x509 -subj "/C=JP/CN=localhost" \
        -keyout /etc/ssl/private/apache-selfsigned.key -out /etc/ssl/certs/apache-selfsigned.crt \
    # set timezone
    && ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone \
    # node
    && apt-get install gcc g++ make \
    && curl -sL https://deb.nodesource.com/setup_12.x | bash - \
    && apt-get install -y nodejs \
    # install composer
    && cd '/' \
    && php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');" \
    && php composer-setup.php \
    && php -r "unlink('composer-setup.php');" \
    && mv composer.phar /usr/local/bin/composer

ENV COMPOSER_ALLOW_SUPERUSER 1
ENV COMPOSER_HOME /composer
ENV PATH $PATH:/composer/vendor/bin


WORKDIR /var/www/html