#ARG PHP_VERSION=7.4
FROM php:7.4-fpm

ARG APCU_VERSION=5.1.19
ARG APCU_BC_VERSION=1.0.5

ENV DEBIAN_FRONTEND noninteractive

ENV DOCKERIZE_VERSION v0.6.1

# add extensions
RUN	apt-get -o 'Acquire::CompressionTypes::Order::="gz"' update \
	&& apt-get install -y --no-install-recommends \
	  libbz2-dev \
	  libfreetype6-dev \
	  libpng-dev \
	  libjpeg-dev \
	  libicu-dev \
	  libzip-dev \
	  unzip \
	  curl \
	  wget \
	  git \
	&& docker-php-ext-install -j$(nproc) \
	  bz2 \
	  gd \
	  zip \
	  pdo_mysql \
	  mysqli \
	  pcntl \
	  bcmath \
	  opcache \
	  intl \
	&& pecl install \
	  apcu-${APCU_VERSION} \
	  apcu_bc-${APCU_BC_VERSION} \
	&& pecl clear-cache \
	&& docker-php-ext-enable --ini-name 0-apc.ini apcu apc \
	&& apt-get purge --auto-remove -y \
	  libbz2-dev \
	  libfreetype6-dev \
	  libpng-dev \
	  libicu-dev \
	  libzip-dev \
	&& apt-get install -y --no-install-recommends \
	  libbz2-1.0 \
	  libpng16-16 \
	  libfreetype6 \
	  libicu63 \
	  libzip4 \
	&& rm -r /var/lib/apt/lists/* \
	&& wget https://github.com/jwilder/dockerize/releases/download/$DOCKERIZE_VERSION/dockerize-linux-amd64-$DOCKERIZE_VERSION.tar.gz \
	  && tar -C /usr/local/bin -xzvf dockerize-linux-amd64-$DOCKERIZE_VERSION.tar.gz \
	  && rm dockerize-linux-amd64-$DOCKERIZE_VERSION.tar.gz

COPY \
    memory_limit.ini \
    upload_size.ini \
    short_tag.ini \
    opcache.ini \
    log.ini \
    no-expose.ini \
        /usr/local/etc/php/conf.d/

# Install symfony-cli
RUN wget https://get.symfony.com/cli/installer -O - | bash \
    && mv ~/.symfony/bin/symfony /usr/local/bin/symfony

# Install and configure composer
COPY --from=composer:2 /usr/bin/composer /usr/bin/composer

# https://getcomposer.org/doc/03-cli.md#composer-allow-superuser
ENV COMPOSER_ALLOW_SUPERUSER=1 \
    PATH=/composer/vendor/bin:$PATH \
    COMPOSER_HOME=/composer \
    COMPOSER_MEMORY_LIMIT=-1

RUN composer global require \
        maglnet/composer-require-checker \
        insolita/unused-scanner \
            --prefer-dist --no-progress --no-suggest --classmap-authoritative \
        && composer clear-cache \
	&& chmod -R a+w ${COMPOSER_HOME}
