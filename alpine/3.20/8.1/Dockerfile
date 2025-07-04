# syntax = docker/dockerfile:1.0.2-experimental
ARG SPRYKER_PHP_VERSION=8.1.30

FROM php:${SPRYKER_PHP_VERSION}-fpm-alpine3.20

ARG TARGETPLATFORM
ARG COMPOSER_VERSION=2.8.1
ARG BLACKFIRE_VERSION=1.87.1
ARG BLACKFIRE_PHP_VERSION=81
ARG NEWRELIC_VERSION=11.2.0.15
ARG TIDEWAYS_VERSION=5.5.14
ARG TIDEWAYS_PHP_VERSION=8.1

ENV srcRoot /data
RUN mkdir -p ${srcRoot}

ARG PHP_RUN_DEPS="\
    freetype \
    gmp \
    gnu-libiconv \
    icu-libs \
    libbz2 \
    libc6-compat \
    libjpeg-turbo \
    libpng \
    libwebp \
    libxml2 \
    libxpm \
    libxslt \
    libzip \
    rabbitmq-c"

ARG PHP_BUILD_DEPS="\
    autoconf \
    bzip2-dev \
    freetype-dev \
    gmp-dev \
    icu-dev \
    icu-data-full \
    libjpeg-turbo-dev \
    libpng-dev \
    libwebp-dev \
    libxml2-dev \
    libxpm-dev \
    libzip-dev \
    postgresql-dev \
    rabbitmq-c-dev \
    g++"

ARG PHP_EXTENSIONS="\
    bcmath \
    bz2 \
    gd \
    gmp \
    intl \
    mysqli \
    opcache \
    pcntl \
    pdo_mysql \
    pdo_pgsql \
    pgsql \
    soap \
    sockets \
    zip"

ARG PHP_PECL_EXTENSIONS="\
    apcu \
    redis"

ARG ADDITIONAL_PHP_PECL_EXTENSIONS="\
    pcov \
    xdebug \
    xhprof \
    amqp \
    excimer"

ARG GRAPHVIZ_BUILD_DEPS="\
    expat-dev"

ARG CFLAGS="-I/usr/src/php"
RUN apk update \
    && apk upgrade \
    && apk add --no-cache \
    bash \
    coreutils \
    curl \
    g++ \
    git \
    make \
    mysql-client \
    netcat-openbsd \
    openssh \
    postgresql-client \
    procps \
    python3 \
    shadow \
    unzip \
    linux-headers \
    libstdc++ \
    ${PHP_RUN_DEPS} \
    && apk add --no-cache --virtual .php-build-deps ${PHP_BUILD_DEPS} \
    && apk add --no-cache --virtual .graphviz-build-deps ${GRAPHVIZ_BUILD_DEPS} \
    && rm -rf /var/lib/apt/lists/ \
    && cd /tmp && curl -LO https://github.com/unicode-org/icu/releases/download/release-74-1/icu4c-74_1-src.tgz \
    && tar xzf icu4c-74_1-src.tgz \
    && cd icu/source && ./configure --prefix=/usr && make && make install \
    && docker-php-ext-configure gd \
        --disable-gd-jis-conv \
        --with-freetype=/usr \
        --with-jpeg=/usr \
        --with-webp=/usr \
        --with-xpm=/usr \
    && docker-php-ext-install -j5 ${PHP_EXTENSIONS} \
    && pecl install -o -f ${PHP_PECL_EXTENSIONS} ${ADDITIONAL_PHP_PECL_EXTENSIONS} \
    && rm -rf /tmp/pear \
    && docker-php-ext-enable ${PHP_EXTENSIONS} ${PHP_PECL_EXTENSIONS} \
    && apk del --no-cache .php-build-deps

# Related to https://github.com/docker-library/php/issues/240
ENV LD_PRELOAD /usr/lib/preloadable_libiconv.so

# Blackfire
ENV BLACKFIRE_AGENT_SOCKET=''
RUN architecture=$(case ${TARGETPLATFORM} in "linux/amd64") echo "amd64" ;; linux/arm64) echo "arm64" ;; *) echo "amd64" ;; esac) \
    && curl -A "Docker" -o /tmp/blackfire.so -D - -L -s https://packages.blackfire.io/binaries/blackfire-php/${BLACKFIRE_VERSION}/blackfire-php-alpine_${architecture}-php-${BLACKFIRE_PHP_VERSION}.so \
    && mv /tmp/blackfire.so $(php -r "echo ini_get ('extension_dir');")/blackfire.so

# New Relic
ENV NEWRELIC_ENABLED=0
ENV NEWRELIC_LICENSE=''
ENV NEWRELIC_APPNAME='Spryker'
ENV NEWRELIC_DISTRIBUTED_TRACING_ENABLED=0
ENV NEWRELIC_TRANSACTION_TRACER_ENABLED=0
ENV NEWRELIC_SPAN_EVENTS_ENABLED=0
ENV NEWRELIC_TRANSACTION_TRACER_THRESHOLD=0
ENV NEWRELIC_DISTRIBUTED_TRACING_EXCLUDE_NEWRELIC_HEADER=0

RUN bash -c 'curl -L "https://download.newrelic.com/php_agent/archive/${NEWRELIC_VERSION}/newrelic-php5-${NEWRELIC_VERSION}-linux-musl.tar.gz" | tar -C /tmp -zx \
  && export NR_INSTALL_USE_CP_NOT_LN=1 \
  && export NR_INSTALL_SILENT=1 \
  && /tmp/newrelic-php5-*/newrelic-install install \
  && rm -rf /tmp/newrelic-php5-* /tmp/nrinstall* \
  && /usr/bin/install -d -m 777 /var/log/newrelic/ \
  && rm /usr/local/etc/php/conf.d/newrelic.ini'

# Tideways
ENV TIDEWAYS_APIKEY=''
ENV TIDEWAYS_DAEMON_URI=''
ENV TIDEWAYS_CLI_ENABLED=0
RUN bash -c 'if [ "${TARGETPLATFORM}" == "linux/amd64" ]; then \
    wget https://s3-eu-west-1.amazonaws.com/tideways/extension/${TIDEWAYS_VERSION}/tideways-php-${TIDEWAYS_VERSION}-alpine.tar.gz \
    && tar xvzf tideways-php-${TIDEWAYS_VERSION}-alpine.tar.gz \
    && cp ./build/dist/tideways-php-alpine-${TIDEWAYS_PHP_VERSION}.so $(php-config --extension-dir)/tideways.so; fi'

RUN bash -c 'if [ "${TARGETPLATFORM}" == "linux/arm64" ]; then \
    wget https://s3-eu-west-1.amazonaws.com/tideways/extension/${TIDEWAYS_VERSION}/tideways-php-${TIDEWAYS_VERSION}-alpine-arm64.tar.gz \
    && tar xvzf tideways-php-${TIDEWAYS_VERSION}-alpine-arm64.tar.gz \
    && cp ./build/dist/tideways-php-alpine-${TIDEWAYS_PHP_VERSION}.so $(php-config --extension-dir)/tideways.so; fi'

# GraphViz 7.0.5
RUN cd /tmp && \
    curl -L https://gitlab.com/api/v4/projects/4207231/packages/generic/graphviz-releases/7.0.5/graphviz-7.0.5.tar.gz | tar -zx && \
    cd /tmp/graphviz-7.0.5 && \
    ./configure && \
    make && \
    make install && \
    rm -rf /tmp/graphviz-7.0.5 && \
    apk del --no-cache .graphviz-build-deps

# Opcache
RUN /usr/bin/install -d -m 777 /var/run/opcache

# Remove default FPM pool
RUN rm /usr/local/etc/php-fpm.d/www.conf && \
    rm /usr/local/etc/php-fpm.d/docker.conf && \
    rm /usr/local/etc/php-fpm.d/zz-docker.conf

# Add FPM configs
COPY context/php/php-fpm.d/worker.conf /usr/local/etc/php-fpm.d/worker.conf
COPY context/php/php-fpm.conf  /usr/local/etc/php-fpm.conf

# Copy php.ini configuration
COPY context/php/php.ini /usr/local/etc/php/
COPY context/php/conf.d/90-opcache.ini /usr/local/etc/php/conf.d/
COPY context/php/conf.d/92-session.ini /usr/local/etc/php/conf.d/
COPY context/php/disabled /usr/local/etc/php/disabled

WORKDIR /data

# Install composer
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/bin --filename=composer --version=${COMPOSER_VERSION}

# Create application user 'spryker'
RUN addgroup spryker && \
    adduser -h /home/spryker -s /bin/sh -G www-data -D spryker && \
    chown spryker:spryker ${srcRoot}

USER spryker
ENV COMPOSER_MEMORY_LIMIT=-1
RUN mkdir -p /home/spryker/.composer
RUN bash -c '[[ $COMPOSER_VERSION == "1"* ]] && composer global require hirak/prestissimo && rm -rf /home/spryker/.composer/cache || true'

USER root
