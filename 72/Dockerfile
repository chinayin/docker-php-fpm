#
#--------------------------------------------------------------------------
# Image Setup
#--------------------------------------------------------------------------
#

FROM php:7.2.34-fpm
LABEL maintainer="chinayin <whereismoney@qq.com>" Version="7.2.34"
ENV DEBIAN_FRONTEND=noninteractive FPM_IMAGE_VERSION="2020-12-17 18:02"

ARG CHANGE_SOURCE=true
ARG TIMEZONE=Asia/Shanghai
RUN if [ ${CHANGE_SOURCE} = true ]; then \
    sed -i 's/deb.debian.org/mirrors.aliyun.com/' /etc/apt/sources.list && \
    sed -i 's/security.debian.org/mirrors.aliyun.com/' /etc/apt/sources.list && \
    sed -i 's/security-cdn.debian.org/mirrors.aliyun.com/' /etc/apt/sources.list \
;fi && cp /usr/share/zoneinfo/${TIMEZONE} /etc/localtime && echo "${TIMEZONE}" > /etc/timezone;

COPY files/arms/72 /usr/local/arms/arms-php-agent

RUN set -eux \
    && apt-get update && apt-get autoremove -y && apt-mark hold tzdata && apt-get upgrade -y \
    ## alias
    && printf "alias ls='ls --color=auto'\nalias ll='ls -l --color=auto'\nalias l='ls -lA --color=auto'" >> /root/.bashrc \
    && apt-get install -y --no-install-recommends \
        curl iputils-ping wget telnet vim apt-utils net-tools procps unzip dnsutils \
        libssl-dev libmcrypt-dev libzip-dev \
    && pecl channel-update pecl.php.net \
    ## Install extensions
    && docker-php-ext-install -j$(nproc) pdo_mysql fileinfo exif zip bcmath\
    ## Install uuid
    && apt-get install -y --no-install-recommends uuid-dev && pecl install uuid && docker-php-ext-enable uuid \
    ## Install redis
    && pecl install -o -f redis && docker-php-ext-enable redis \
    ## Install memcached
    && apt-get install -y --no-install-recommends libmemcached-dev && pecl install igbinary msgpack memcached && docker-php-ext-enable memcached \
    ## Install aliyun arms
    && mv /usr/local/arms/arms-php-agent/arms.ini /usr/local/etc/php/conf.d/arms.ini \
    ## Clean up
    && pecl clear-cache && apt-get clean \
    && rm -rf /var/lib/apt/lists/* /var/cache/apk/* /tmp/* /var/tmp/* \
    && rm -f /var/log/lastlog /var/log/faillog \
    ## Check PHP version
    && php -v | head -n 1 | grep -q "PHP ${PHP_VERSION}"

## Configure non-root user.
ARG PUID=1000
ARG PGID=1000
ENV PUID=${PUID} PGID=${PGID}
RUN groupmod -o -g ${PGID} www-data && usermod -o -u ${PUID} -g www-data www-data
