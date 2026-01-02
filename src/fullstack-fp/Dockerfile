ARG NODE_VERSION='22.16.0'
ARG PHP_VERSION='8.3'
ARG BASE_IMAGE="serversideup/php:${PHP_VERSION}-frankenphp-bookworm"

FROM node:${NODE_VERSION}-slim AS node

FROM oven/bun:debian AS bun

FROM ${BASE_IMAGE}

LABEL maintainer="Qisthi Ramadhani (https://qisthi.dev)"

ENV PHP_OPCACHE_ENABLE=1

USER root

RUN install-php-extensions gd exif bcmath mongodb && \
    apt-get update && \
    apt-get install -y --no-install-recommends git unzip bash && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# Install node and npm
COPY --from=node /usr/lib /usr/lib
COPY --from=node /usr/local/lib /usr/local/lib
COPY --from=node /usr/local/include /usr/local/include
COPY --from=node /usr/local/bin /usr/local/bin
COPY --from=node /opt/yarn-v1.22.22 /opt/yarn-v1.22.22

# Install bun
COPY --from=bun /usr/local/bin /usr/local/bin

# Install sonar-scanner
RUN apt-get update && \
    apt-get install -y --no-install-recommends curl unzip && \
    if [ "$(uname -m)" = "aarch64" ]; then \
        # aarch64
        curl https://binaries.sonarsource.com/Distribution/sonar-scanner-cli/sonar-scanner-cli-6.2.1.4610-linux-aarch64.zip --output /tmp/sonar-scanner.zip && \
        SONAR_DIR="sonar-scanner-6.2.1.4610-linux-aarch64" ; \
    else \
        # x86_64
        curl https://binaries.sonarsource.com/Distribution/sonar-scanner-cli/sonar-scanner-cli-6.2.1.4610-linux-x64.zip --output /tmp/sonar-scanner.zip && \
        SONAR_DIR="sonar-scanner-6.2.1.4610-linux-x64" ; \
    fi && \
    unzip /tmp/sonar-scanner.zip -d /opt && \
    ln -s /opt/${SONAR_DIR}/bin/sonar-scanner /usr/local/bin/sonar-scanner && \
    rm /tmp/sonar-scanner.zip && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# Clean up
RUN rm -rf /tmp/* /var/tmp/*

# Drop back to our unprivileged user
USER www-data

RUN composer global config allow-plugins.gmta/composer-velocita true && \
    composer global require gmta/composer-velocita && \
    git config --global --add safe.directory /builds

# Copy shell aliases to both root and www-data user homes
COPY shell/aliases.sh /root/.aliases.sh
COPY shell/aliases.sh /var/www/.aliases.sh

# Copy shell configuration
COPY shell/bashrc /root/.sh
COPY shell/bashrc /var/www/.bashrc
