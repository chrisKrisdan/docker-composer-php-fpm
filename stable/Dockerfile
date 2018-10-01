FROM php:7.3-rc-fpm-alpine

RUN apk --no-cache add git subversion openssh mercurial tini bash patch libzip-dev su-exec

RUN echo "memory_limit=-1" > "$PHP_INI_DIR/conf.d/memory-limit.ini" \
 && echo "date.timezone=${PHP_TIMEZONE:-UTC}" > "$PHP_INI_DIR/conf.d/date_timezone.ini"

RUN apk add --no-cache --virtual .build-deps zlib-dev \
 && docker-php-ext-install zip \
 && runDeps="$( \
    scanelf --needed --nobanner --format '%n#p' --recursive /usr/local/lib/php/extensions \
    | tr ',' '\n' \
    | sort -u \
    | awk 'system("[ -e /usr/local/lib/" $1 " ]") == 0 { next } { print "so:" $1 }' \
    )" \
 && apk add --virtual .composer-phpext-rundeps $runDeps \
 && apk del .build-deps

RUN addgroup -g 1000 -S composer  && adduser -u 1000 -S composer -G composer

USER composer

ENV COMPOSER_ALLOW_SUPERUSER 1
ENV COMPOSER_HOME /home/composer
ENV COMPOSER_VERSION 1.7.2

RUN curl --silent --fail --location --retry 3 --output /home/composer/installer.php --url https://getcomposer.org/installer \
 && php -r " \
    \$signature = '$(wget -q -O - https://composer.github.io/installer.sig)'; \
    \$hash = hash('SHA384', file_get_contents('/home/composer/installer.php')); \
    if (!hash_equals(\$signature, \$hash)) { \
        unlink('/home/composer/installer.php'); \
        echo 'Integrity check failed, installer is either corrupt or worse.' . PHP_EOL; \
        exit(1); \
    }"

USER root

RUN chown -R composer:root /home/composer \
 && chmod -R 770 /home/composer \
 && php /home/composer/installer.php --no-ansi --install-dir=/usr/local/bin --filename=composer --version=${COMPOSER_VERSION} \
 && composer --ansi --version --no-interaction \
 && rm -rf /home/composer/* /home/composer/.htaccess

COPY docker-entrypoint.sh /docker-entrypoint.sh

WORKDIR /app

RUN composer global require laravel/installer \
 && ln -s /home/composer/vendor/bin/laravel /usr/local/bin/laravel

ENTRYPOINT ["/bin/sh", "/docker-entrypoint.sh"]

CMD ["php-fpm"]
