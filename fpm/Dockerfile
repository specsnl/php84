# syntax=docker/dockerfile:1
# check=error=true

# Latest version of PHP base image: https://hub.docker.com/_/php/tags
FROM php:8.4.21-fpm-trixie AS runtime

ARG DEBIAN_FRONTEND=noninteractive

ARG XDG_CONFIG_HOME=/config
ENV XDG_CONFIG_HOME=$XDG_CONFIG_HOME

ARG XDG_DATA_HOME=/data
ENV XDG_DATA_HOME=$XDG_DATA_HOME

ARG XDG_CACHE_HOME=/cache
ENV XDG_CACHE_HOME=$XDG_CACHE_HOME

# Latest version of event-extension: https://packagist.org/packages/osmanov/pecl-event
ARG PHP_EVENT_VERSION=3.1.5
# Latest version of igbinary-extension: https://packagist.org/packages/igbinary/igbinary
ARG PHP_IGBINARY_VERSION=3.2.17RC1
# Latest version of redis-extension: https://packagist.org/packages/phpredis/phpredis
ARG PHP_REDIS_VERSION=6.3.0
# Latest version of amqp-extension: https://packagist.org/packages/php-amqp/php-amqp
ARG PHP_AMQP_VERSION=2.2.0

ENV SMTPHOST=mail
ENV SMTPEHLO=localhost

WORKDIR /var/www

# Latest version of Pie: https://github.com/php/pie/releases
COPY --from=ghcr.io/php/pie:1.4.5-bin /pie /usr/bin/pie

RUN apt-get update \
    && apt-get install --assume-yes --no-install-recommends \
        apt-transport-https \
        ca-certificates \
        openssl \
        curl \
        msmtp-mta \
        unzip \
        # Dependency of the PHP intl-extension
        libicu76 \
        # Dependency of the PHP gd-extension
        libpng16-16t64 \
        libwebp7 \
        libjpeg62-turbo \
        libfreetype6 \
        # Dependency of PHP zip-extension
        libzip5 \
        # Dependency of PHP event-extension
        libevent-2.1-7t64 \
        libevent-openssl-2.1-7t64 \
        libevent-extra-2.1-7t64 \
        # Dependency of PHP pdo_pgsql-extension
        libpq5 \
        # Dependency of PHP amqp-extension
        librabbitmq4 \
        # Dependency of PHP xsl-extension
        libxslt1.1 \
    # Install packages that are needed for building PHP extensions
    && apt-get install --assume-yes --no-install-recommends \
        $PHPIZE_DEPS \
        # Dependency of the PHP intl-extension
        libicu-dev \
        # Dependencies of PHP gd-extension
        libpng-dev \
        libwebp-dev \
        libjpeg62-turbo-dev \
        libfreetype6-dev \
        # Dependency of PHP zip-extension
        libzip-dev \
        # Dependency of PHP event-extension
        libevent-dev \
        libssl-dev \
        # Dependency of PHP soap-extension
        libxml2-dev \
        # Dependency of PHP pdo_pgsql-extension
        libpq-dev \
        # Dependency of PHP amqp-extension
        librabbitmq-dev \
        # Dependency of PHP xsl-extension
        libxslt1-dev \
    # Configure PHP gd-extension
    && docker-php-ext-configure gd \
        --enable-gd \
        --with-jpeg \
        --with-freetype \
        --with-webp \
    # Install PHP extensions
    && docker-php-ext-install -j "$(nproc --all)" \
        pdo_mysql \
        pdo_pgsql \
        intl \
        opcache \
        pcntl \
        gd \
        bcmath \
        zip \
        soap \
        xsl \
        # Dependency of PHP event-extension
        sockets \
    && pie install --skip-enable-extension osmanov/pecl-event:$PHP_EVENT_VERSION \
    && pie install --skip-enable-extension igbinary/igbinary:$PHP_IGBINARY_VERSION \
    && pie install --skip-enable-extension php-amqp/php-amqp:$PHP_AMQP_VERSION \
    && pie install --skip-enable-extension phpredis/phpredis:$PHP_REDIS_VERSION --enable-redis-igbinary \
    && docker-php-ext-enable --ini-name docker-php-ext-zz-custom.ini \
        event \
        igbinary \
        redis \
        amqp \
    && cp "$PHP_INI_DIR/php.ini-production" "$PHP_INI_DIR/php.ini" \
    # Purge packages that where only needed for building php extensions
    && apt-get purge --assume-yes \
        $PHPIZE_DEPS \
        libicu-dev \
        libpng-dev \
        libwebp-dev \
        libjpeg62-turbo-dev \
        libfreetype6-dev \
        libzip-dev \
        libevent-dev \
        libssl-dev \
        libxml2-dev \
        libpq-dev \
        librabbitmq-dev \
        libxslt1-dev \
    && mkdir -p \
        $XDG_CONFIG_HOME \
        $XDG_DATA_HOME \
        $XDG_CACHE_HOME \
    && rm -rf /var/www/* \
    # Cleanup
    && apt-get autoremove --assume-yes \
    && apt-get clean --assume-yes \
    && rm -rf /var/lib/apt/lists/* \
    && rm -rf /tmp/*

COPY files/general /

FROM runtime AS builder

ARG TARGETARCH
ARG DEBIAN_FRONTEND=noninteractive

# Latest version of Phive: https://api.github.com/repos/phar-io/phive/releases/latest
ARG PHIVE_VERSION=0.16.0
# Latest version of Composer: https://getcomposer.org/download
ARG COMPOSER_VERSION=2.10.1
# Latest version of Xdebug: https://packagist.org/packages/xdebug/xdebug
ARG XDEBUG_VERSION=3.5.1
# Latest version of pcov: https://packagist.org/packages/pecl/pcov
ARG PCOV_VERSION=1.0.12


RUN apt-get update \
    && apt-get install --assume-yes --no-install-recommends \
        # Needed for xdebug extension configuration
        $PHPIZE_DEPS \
        vim \
        git \
        sqlite3 \
        wait-for-it \
        p7zip-full \
        # Needed for phive:
        gnupg \
    # Install Phive
    && curl -fsSLo /usr/local/bin/phive "https://github.com/phar-io/phive/releases/download/$PHIVE_VERSION/phive-$PHIVE_VERSION.phar" \
    && curl -fsSLo /tmp/phive.phar.asc "https://github.com/phar-io/phive/releases/download/$PHIVE_VERSION/phive-$PHIVE_VERSION.phar.asc" \
    && gpg --keyserver keys.openpgp.org --recv-keys 0x9D8A98B29B2D5D79 \
    && gpg --verify /tmp/phive.phar.asc /usr/local/bin/phive \
    && chmod +x /usr/local/bin/phive \
    && phive update-repository-list \
    # Install Composer using Phive
    && phive install --global composer:$COMPOSER_VERSION --trust-gpg-keys CBB3D576F2A0946F \
    && rm -rf /root/.phive \
    # Download Composer keys
    && mkdir -p "$XDG_CONFIG_HOME/composer" \
    && curl -fsSLo "$XDG_CONFIG_HOME/composer/keys.dev.pub" https://composer.github.io/snapshots.pub \
    && curl -fsSLo "$XDG_CONFIG_HOME/composer/keys.tags.pub" https://composer.github.io/releases.pub \
    && chmod 644 "$XDG_CONFIG_HOME/composer/keys.dev.pub" "$XDG_CONFIG_HOME/composer/keys.tags.pub" \
    # Install Xdebug and pcov PHP extensions
    && pie install xdebug/xdebug:$XDEBUG_VERSION \
    && pie install pecl/pcov:$PCOV_VERSION \
    && cp "$PHP_INI_DIR/php.ini-development" "$PHP_INI_DIR/php.ini" \
    # Cleanup
    && apt-get purge --assume-yes $PHPIZE_DEPS \
    && apt-get autoremove --assume-yes \
    && apt-get clean --assume-yes \
    && rm -rf /var/lib/apt/lists/* \
    && rm -rf /tmp/*

# Latest version of fixuid: https://github.com/boxboat/fixuid/releases/latest
ARG FIXUID_VERSION=0.6.0

ARG USER=code
ARG GROUP=code
ARG USER_UID=1000
ARG USER_GID=1000
ARG TARGETARCH

SHELL ["/bin/bash", "-o", "pipefail", "-c"]

RUN curl -fsSLo /tmp/fixuid.tar.gz "https://github.com/boxboat/fixuid/releases/download/v$FIXUID_VERSION/fixuid-${FIXUID_VERSION}-linux-${TARGETARCH}.tar.gz" \
    && tar -xf /tmp/fixuid.tar.gz -C /usr/local/bin fixuid \
    && chown root:root /usr/local/bin/fixuid \
    && chmod 4755 /usr/local/bin/fixuid \
    && rm /tmp/fixuid.tar.gz \
    # Create user and group, and setup fixuid configuration
    && existing_group="$(getent group "$USER_GID" | cut -d: -f1 || true)" \
    && if [ -z "$existing_group" ]; then \
           groupadd --gid "$USER_GID" "$GROUP"; \
           existing_group="$GROUP"; \
       fi \
    && useradd --uid "$USER_UID" --gid "$USER_GID" --create-home "$USER" --shell /bin/bash \
    && mkdir -p \
        /etc/fixuid \
        "/data" \
        "/config" \
        "/cache/npm" \
    && chown -R "$USER_UID":"$USER_GID" \
        "/data" \
        "/config" \
        "/cache" \
    && cat <<EOF > /etc/fixuid/config.yml
user: $USER
group: $existing_group
paths:
  - /var/www
  - /home/$USER
  - /data
  - /config
  - /cache

EOF

FROM builder AS builder_nodejs

ARG TARGETARCH
ARG DEBIAN_FRONTEND=noninteractive

ENV COREPACK_ENABLE_DOWNLOAD_PROMPT=0
ENV npm_config_cache="$XDG_CACHE_HOME/npm"

SHELL ["/bin/bash", "-o", "pipefail", "-c"]

# Latest version of Node.js: https://nodejs.org
ARG NODE_MAJOR=24

RUN apt-get update \
    && apt-get install --assume-yes --no-install-recommends \
        gcc \
        g++ \
        make \
    && mkdir -p /usr/share/keyrings \
    && curl -fsSL https://deb.nodesource.com/gpgkey/nodesource-repo.gpg.key | gpg --dearmor -o /usr/share/keyrings/nodesource.gpg \
    && chmod 644 /usr/share/keyrings/nodesource.gpg \
    && echo "deb [arch=$TARGETARCH signed-by=/usr/share/keyrings/nodesource.gpg] https://deb.nodesource.com/node_$NODE_MAJOR.x nodistro main" | tee /etc/apt/sources.list.d/nodesource.list \
    && echo "Package: nodejs" | tee /etc/apt/preferences.d/nodejs > /dev/null \
    && echo "Pin: origin deb.nodesource.com" | tee -a /etc/apt/preferences.d/nodejs > /dev/null \
    && echo "Pin-Priority: 600" | tee -a /etc/apt/preferences.d/nodejs > /dev/null \
    && apt-get update \
    && apt-get install --assume-yes --no-install-recommends \
        nodejs \
    && npm uninstall --global npm \
    && corepack install --global npm@11.x yarn@4.x pnpm@10.x \
    && corepack enable npm yarn pnpm \
    # Cleanup
    && apt-get autoremove --assume-yes \
    && apt-get clean --assume-yes \
    && rm -rf /var/lib/apt/lists/* \
    && rm -rf /tmp/*
