ARG PHP_VERSION='8.0'

FROM php:${PHP_VERSION}-fpm-alpine

RUN apk add --no-cache \
        git \
		acl \
		fcgi \
		file \
		gettext \
	;

COPY --from=mlocati/php-extension-installer /usr/bin/install-php-extensions /usr/local/bin/

ARG APP_ENV
ARG APP_DEBUG

COPY /docker/fpm/conf.d/ $PHP_INI_DIR/conf.d/

RUN set -exu; \
    if [ "${APP_ENV}" = "dev" ]; then \
        ln -sf $PHP_INI_DIR/php.ini-development $PHP_INI_DIR/php.ini; \
        ln -sf $PHP_INI_DIR/conf.d/custom.ini-development $PHP_INI_DIR/custom.ini; \
    else \
        ln -sf $PHP_INI_DIR/php.ini-production $PHP_INI_DIR/php.ini; \
        ln -sf $PHP_INI_DIR/conf.d/custom.ini-production $PHP_INI_DIR/custom.ini; \
    fi; \
    install-php-extensions gd intl zip pcntl apcu opcache @composer; \
    rm /usr/local/bin/install-php-extensions;

WORKDIR /symfony

## Remove it and copy your source here.
RUN if [ "${APP_ENV}" = "prod" ]; then \
	composer create-project symfony/skeleton . --prefer-dist --no-dev --no-progress --no-interaction; \
	else \
    composer create-project symfony/skeleton . --prefer-dist --no-progress --no-interaction; \
    fi

RUN mkdir -p var/cache var/log;

RUN if [ "${APP_ENV}" = "prod" ]; then \
	composer install --prefer-dist --no-dev --no-scripts --no-progress --no-suggest; \
	composer dump-autoload --classmap-authoritative --no-dev; \
	else \
    composer install --prefer-dist --no-scripts --no-progress --no-suggest; \
    composer dump-autoload --classmap-authoritative; \
    fi

RUN	set -eux; \
    composer symfony:dump-env "${APP_ENV}"; \
	composer run-script --no-dev post-install-cmd; \
	chmod +x bin/console; sync;
RUN { \
		echo '[www]'; \
		echo 'ping.path = /ping'; \
	} | tee /usr/local/etc/php-fpm.d/docker-healthcheck.conf

VOLUME /symfony/var

COPY /docker/fpm/docker-entrypoint.sh /usr/local/bin/docker-entrypoint
COPY /docker/fpm/docker-healthcheck.sh /usr/local/bin/docker-healthcheck

RUN chmod +x /usr/local/bin/docker-entrypoint /usr/local/bin/docker-healthcheck

ENTRYPOINT ["docker-entrypoint"]

CMD ["fpm"]