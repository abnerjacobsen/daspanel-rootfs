FROM alpine:3.6
MAINTAINER Abner G Jacobsen - http://daspanel.com <admin@daspanel.com>

# Parse arguments for the build command.
ARG VERSION
ARG VCS_URL
ARG VCS_REF
ARG BUILD_DATE

# Set default env variables
ENV \
    # Stop container initialization if error occurs in cont-init.d, fix-attrs.d script's
    S6_BEHAVIOUR_IF_STAGE2_FAILS=2 \

    # Timezone
    TZ="UTC" 

# A little bit of metadata management.
# See http://label-schema.org/  
LABEL org.label-schema.schema-version="1.0" \
      org.label-schema.build-date=$BUILD_DATE \
      org.label-schema.version=$VERSION \
      org.label-schema.vcs-url=$VCS_URL \
      org.label-schema.vcs-ref=$VCS_REF \
      org.label-schema.name="daspanel/engine-php71" \
      org.label-schema.description="This service provides HTTP php 7.1 engine server to Daspanel sites."

# PHP modules to install
ARG PHP_MODULES="php7-common php7-ctype php7-cli php7-curl php7-fileinfo \
    php7-fpm php7-gd php7-iconv php7-intl php7-json php7-mbstring php7-mcrypt \
    php7-mysqli php7-mysqlnd php7-opcache php7-openssl php7-pdo php7-pdo_mysql \
    php7-pdo_pgsql php7-pdo_sqlite php7-phar php7-posix php7-redis@testing \
    php7-session php7-soap php7-tokenizer php7-xml php7-xmlreader \
    php7-xmlwriter php7-zip php7-zlib php7-dom \
    php7-pear php7-pgsql php7-sqlite3 \
    php7-pcntl php7-ftp php7-gettext php7-imap \
    php7-bcmath php7-bz2 php7-exif php7-gmp \
    php7-imagick \
    php7-apcu php7-memcached php7-mongodb@testing \
    php7-sockets php7-pspell"

ARG CADDY_PLUGINS="http.cors,http.expires,http.filemanager"
ARG CADDY_URL="https://caddyserver.com/download/linux/amd64?plugins=${CADDY_PLUGINS}"
    
ENV VAR_PREFIX=/var/run
ENV LOG_PREFIX=/var/log/php-fpm
ENV TEMP_PREFIX=/tmp
ENV CACHE_PREFIX=/var/cache

RUN set -x \
    && addgroup -g 82 -S www-data \
    && adduser -u 82 -D -S -h /var/cache/php-fpm -s /sbin/nologin -G www-data www-data \
    && apk add --no-cache --virtual .build-deps \
        wget \
        linux-headers \
        curl \
        unzip \
    && echo '@testing http://dl-cdn.alpinelinux.org/alpine/edge/testing' >> /etc/apk/repositories \
    && echo '@community http://nl.alpinelinux.org/alpine/edge/community' >> /etc/apk/repositories \
    && apk add --no-cache --update \
        libcap mailcap ffmpeg php7-fpm php7 $PHP_MODULES \
        curl \
        monit \
        bash \
        xz \
        ca-certificates \
        openssl \
        tar \
    && mkdir -p /var/run \

    # Install Composer
    && curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/bin --filename=composer \

    # Install Caddy
    && curl --silent --show-error --fail --location \
        --header "Accept: application/tar+gzip, application/x-gzip, application/octet-stream" -o - \
        "${CADDY_URL}" \
        | tar --no-same-owner -C /usr/sbin/ -xz caddy \
    && chmod 0755 /usr/sbin/caddy \
    && setcap "cap_net_bind_service=+ep" /usr/sbin/caddy \

    # Install daspanel-base
    && wget -O /tmp/opt-daspanel.zip "https://github.com/abnerjacobsen/daspanel-rootfs/releases/download/0.4.0/opt-daspanel.zip" \
    && unzip -d / /tmp/opt-daspanel.zip \
    && wget -O /tmp/alpine-s6.zip "https://github.com/abnerjacobsen/daspanel-rootfs/releases/download/0.4.0/alpine-s6.zip" \
    && unzip -d / /tmp/alpine-s6.zip \

    # Cleanup    
    && rm -rf /tmp/* \
    && rm -rf /var/cache/apk/*

COPY conf/monit/ /etc/monit.d/
COPY docker-entrypoint.sh /docker-entrypoint.sh
COPY check_wwwdata.sh /usr/bin/check_wwwdata
RUN chmod +x /docker-entrypoint.sh /usr/bin/check_wwwdata

ENTRYPOINT ["/docker-entrypoint.sh"]
CMD ["php-fpm7", "-g", "/var/run/php-fpm.pid"]
