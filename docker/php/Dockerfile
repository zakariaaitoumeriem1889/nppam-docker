FROM php:8.5-fpm

RUN apt-get update && apt-get install -y --no-install-recommends \
    vim curl debconf subversion git apt-transport-https apt-utils bash \
    build-essential locales acl mailutils wget zip unzip libpng-dev \
    gnupg gnupg1 gnupg2 libonig-dev zlib1g-dev libxml2-dev libzip-dev \
    libfreetype6-dev libjpeg62-turbo-dev libcurl4-openssl-dev libssl-dev \
    libreadline-dev libxslt1-dev libgmp-dev libkrb5-dev libldap2-dev \
    libpq-dev libsqlite3-dev libmemcached-dev libmagickwand-dev libbz2-dev \
    libffi-dev libtidy-dev libenchant-2-dev libsodium-dev libwebp-dev sudo \
    && apt-get clean && rm -rf /var/lib/apt/lists/*

RUN docker-php-ext-configure gd --with-jpeg --with-freetype --with-webp
RUN docker-php-ext-configure ldap --with-libdir=/usr/lib/x86_64-linux-gnu

RUN docker-php-ext-install \
    bcmath bz2 calendar dba enchant exif ffi ftp gd gettext gmp intl \
    ldap mysqli pcntl pdo pdo_mysql pdo_pgsql pdo_sqlite pgsql shmop \
    soap sysvmsg sysvsem sysvshm tidy xsl zip

RUN pecl install apcu imagick memcached mongodb redis xdebug mailparse \
    && docker-php-ext-enable apcu imagick memcached mongodb redis mailparse xdebug

COPY php.ini /usr/local/etc/php/php.ini

COPY --from=composer:latest /usr/bin/composer /usr/bin/composer

RUN wget -q https://phar.phpunit.de/phpunit-13.phar && \
    chmod +x phpunit-13.phar && \
    mv phpunit-13.phar /usr/local/bin/phpunit

RUN curl -fsSL https://deb.nodesource.com/setup_lts.x | bash - \
    && apt-get install -y nodejs \
    && npm -v && npm install -g npm@latest && npm -v

RUN curl -1sLf 'https://dl.cloudsmith.io/public/symfony/stable/setup.deb.sh' | sudo -E bash \
    && sudo apt install -y --no-install-recommends symfony-cli

RUN groupadd -g 999 dev \
    && useradd -g dev -d /home/dev -m -s /bin/bash dev \
    && passwd -d dev

RUN echo "en_US.UTF-8 UTF-8" > /etc/locale.gen && \
    echo "fr_FR.UTF-8 UTF-8" >> /etc/locale.gen && \
    echo "ar_MA.UTF-8 UTF-8" >> /etc/locale.gen && \
    locale-gen

RUN echo "dev ALL=(ALL) NOPASSWD: ALL" > /etc/sudoers.d/dev \
    && chmod 0440 /etc/sudoers.d/dev

ENV COMPOSER_HOME=/usr/local/composer
ENV COMPOSER_CACHE_DIR=/var/www/.composer/cache
ENV NPM_CONFIG_CACHE=/var/www/.npm
ENV PATH="$COMPOSER_HOME/vendor/bin:$PATH"

RUN composer global require laravel/installer

RUN mkdir -p \
    /var/www/.npm \
    /var/www/.composer/cache \
    /var/www/.config \
    /var/www/html \
    /var/log/xdebug \
    /var/log/php \
    && touch /var/log/xdebug/xdebug.log \
    && touch /var/log/php/error.log

RUN chown -R www-data:www-data \
    /var/www \
    /usr/local/composer \
    /var/log/xdebug \
    /var/log/php

RUN chmod -R 755 /var/www \
    && chmod 664 /var/log/xdebug/xdebug.log \
    && chmod 664 /var/log/php/error.log

RUN npm config set cache /var/www/.npm --global

WORKDIR /var/www/html

EXPOSE 9000
CMD ["php-fpm"]