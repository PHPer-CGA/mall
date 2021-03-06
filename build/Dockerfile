FROM php:7.0-fpm-alpine

# using aliyun alpine mirror
RUN echo "http://mirrors.aliyun.com/alpine/v3.8/main"      >  /etc/apk/repositories \
 && echo "http://mirrors.aliyun.com/alpine/v3.8/community" >> /etc/apk/repositories
RUN apk add --no-cache \
    nginx \
    supervisor \
    libpng \
    libpng-dev \
    freetype \
    libjpeg-turbo \
    freetype-dev \
    libjpeg-turbo-dev \
# php-exts
 && /usr/local/bin/docker-php-ext-install -j "$(nproc)" mbstring mysqli pdo pdo_mysql shmop \
 && docker-php-ext-configure gd \
        --enable-gd-native-ttf \
        --with-freetype-dir=/usr/include/freetype2 \
        --with-png-dir=/usr/include \
        --with-jpeg-dir=/usr/include \
 && /usr/local/bin/docker-php-ext-install -j "$(nproc)" gd \
 && /usr/local/bin/docker-php-ext-enable opcache gd \
 && apk add --no-cache --virtual .build-deps \
    $PHPIZE_DEPS \
    tzdata \
# php-ext-ds
 && cd /tmp/ && pecl bundle ds && cd ds \
 && phpize \
 && ./configure --enable-ds \
 && make -j "$(nproc)" && make install \
 && docker-php-ext-enable ds \
# timezone
 && /bin/cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime \
 && echo 'Asia/Shanghai' > /etc/timezone \
 && echo '[date]'                        >> "$PHP_INI_DIR/conf.d/docker-php-ext-date.ini" \
 && echo 'date.timezone = Asia/Shanghai' >> "$PHP_INI_DIR/conf.d/docker-php-ext-date.ini" \
# cleanup
 && rundeps="$( \
    scanelf --needed --nobanner --format '%n#p' --recursive /usr/local/lib/php/extensions \
  | tr ',' '\n' \
  | sort -u \
  | awk 'system("[ -e /usr/local/lib/" $1 " ]") == 0 { next } { print "so:" $1 }' \
 )" \
 && apk add --virtual .phpext-rundeps $rundeps \
 && rm -rf /var/cache/apk/* \
 && rm -rf /tmp/* \
 && apk del .build-deps \
# nginx logs
 && ln -sf /dev/stdout /var/log/nginx/access.log \
 && ln -sf /dev/stderr /var/log/nginx/error.log

COPY build/etc /etc/
COPY dist      /var/www/html

RUN chmod -R 777 /var/www/html

EXPOSE 80

CMD ["supervisord", "-n", "-c", "/etc/supervisord.conf"]
