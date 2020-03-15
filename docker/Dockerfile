ARG PHP_VERSION
ARG NGINX_VERSION

# "php" stage
FROM php:${PHP_VERSION}-fpm-alpine AS php

ARG APP_ENV
ARG APP_DEBUG

# extensions default version
ARG APCU_VERSION=5.1.18
ARG AMQP_VERSION=1.9.4

RUN apk add --no-cache \
		acl \
		fcgi \
		file \
		gettext \
	;

RUN set -eux; \
	apk add --no-cache --virtual .build-deps \
		$PHPIZE_DEPS \
		icu-dev \
		libzip-dev \
		zlib-dev \
		rabbitmq-c-dev \
	; \
	apk add --no-cache python3; \
	pip3 install supervisor; \
	docker-php-ext-configure zip; \
	docker-php-ext-install -j$(nproc) \
		intl \
		zip \
		pdo_mysql \
	; \
	pecl install \
		apcu-${APCU_VERSION} \
		amqp-${AMQP_VERSION} \
	; \
	pecl clear-cache; \
	docker-php-ext-enable \
		apcu \
		opcache \
		amqp \
	; \
	\
	runDeps="$( \
		scanelf --needed --nobanner --format '%n#p' --recursive /usr/local/lib/php/extensions \
			| tr ',' '\n' \
			| sort -u \
			| awk 'system("[ -e /usr/local/lib/" $1 " ]") == 0 { next } { print "so:" $1 }' \
	)"; \
	apk add --no-cache --virtual .api-phpexts-rundeps $runDeps; \
	apk del .build-deps

COPY --from=composer:latest /usr/bin/composer /usr/bin/composer

RUN chmod +x /usr/bin/composer; sync

RUN if [ "${APP_ENV}" = 'prod' ]; then \
    ln -sf $PHP_INI_DIR/php.ini-production $PHP_INI_DIR/php.ini; \
    else \
    ln -sf $PHP_INI_DIR/php.ini-development $PHP_INI_DIR/php.ini; \
    fi

COPY /docker/php/conf.d/$APP_ENV.ini $PHP_INI_DIR/conf.d/config.ini
COPY /docker/php/supervisord /var/supervisord

WORKDIR /app

# Copy needed resources of app.
COPY /symfony/composer.json /app/composer.json
COPY /symfony/composer.lock /app/composer.lock
COPY /symfony/symfony.lock /app/symfony.lock
COPY /symfony/.env /app/.env
COPY /symfony/bin /app/bin
COPY /symfony/config /app/config
COPY /symfony/src /app/src
COPY /symfony/public /app/public

ENV SYMFONY_PHPUNIT_VERSION=8.3

# Install Symfony Flex globally to speed up download of Composer packages (parallelized prefetching)
RUN set -eux; \
	composer global require "symfony/flex" --prefer-dist --no-progress --no-suggest --classmap-authoritative; \
	composer clear-cache

ENV PATH="${PATH}:/root/.composer/vendor/bin"

RUN if [ "${APP_ENV}" = 'prod' ]; then \
	composer install --prefer-dist --no-dev --no-scripts --no-progress --no-suggest; \
	else \
    composer install --prefer-dist --no-scripts --no-progress --no-suggest; \
    fi

# do not use .env files
RUN composer clear-cache; \
    composer dump-env $APP_ENV; \
	rm .env

RUN mkdir -p /app/var/cache /app/var/log;

RUN	if [ "$APP_ENV" = 'prod' ]; then \
	composer dump-autoload --classmap-authoritative --no-dev; \
	composer run-script --no-dev post-install-cmd; \
	else \
	composer dump-autoload --classmap-authoritative; \
	composer run-script post-install-cmd; \
	fi

VOLUME /app/var

RUN chmod +x bin/console; sync

RUN set -eux; \
	{ \
		echo '[www]'; \
		echo 'ping.path = /ping'; \
	} | tee /usr/local/etc/php-fpm.d/docker-healthcheck.conf

COPY /docker/php/docker-entrypoint.sh /usr/local/bin/docker-entrypoint
COPY /docker/php/docker-healthcheck.sh /usr/local/bin/docker-healthcheck

RUN chmod +x /usr/local/bin/docker-entrypoint /usr/local/bin/docker-healthcheck

ENTRYPOINT ["docker-entrypoint"]

# "nginx" stage
FROM nginx:${NGINX_VERSION}-alpine AS nginx

COPY /docker/nginx/conf.d/default.conf /etc/nginx/conf.d/default.conf
COPY --from=php /app/public /app/public

WORKDIR /app/public
