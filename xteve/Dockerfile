#============================#
#  now its my dockerfile yarek  #
#============================#

#=== Home Assistant Addon ===#

#################
# 1 Build Image #
#################

ARG BUILD_FROM
ARG BUILD_VERSION
ARG BUILD_UPSTREAM="2.5.3"
FROM $BUILD_FROM

##################
# 2 Modify Image #
##################

# Set S6 wait time
ENV S6_CMD_WAIT_FOR_SERVICES=1 \
    S6_CMD_WAIT_FOR_SERVICES_MAXTIME=0 \
    S6_SERVICES_GRACETIME=0

# Root operations
USER root

# Install php7-fpm and su-exec
RUN apk add --no-cache php7-fpm su-exec

# Create non-root user and group
RUN addgroup -S xteve && adduser -S xteve -G xteve

# Copy xteve binaries or configs if present
RUN cp -vnpr /home/xteve* /data || true

# Ensure folder permissions
RUN mkdir -p /etc/opt/xteve /tmp/xteve /config /run/php && \
    chown -R xteve:xteve /etc/opt/xteve /tmp/xteve /data /config /run/php && \
    chmod -R 755 /etc/opt/xteve /tmp/xteve /config /run/php

# Fallback patch: replace geteuid with getppid in VLC (avoids root warning)
RUN sed -i 's/geteuid/getppid/' /usr/bin/vlc || true

##################
# 3 Install apps #
##################

# Copy addon files
COPY rootfs/ /

# Symlink fallback
RUN if [ ! -f /bin/sh ] && [ -f /usr/bin/sh ]; then ln -s /usr/bin/sh /bin/sh; fi && \
    if [ ! -f /bin/bash ] && [ -f /usr/bin/bash ]; then ln -s /usr/bin/bash /bin/bash; fi

# Modules
ARG MODULES="00-banner.sh 01-custom_script.sh"

# Automatic module install
ADD "https://raw.githubusercontent.com/alexbelgium/hassio-addons/master/.templates/ha_automodules.sh" "/ha_automodules.sh"
RUN chmod 744 /ha_automodules.sh && /ha_automodules.sh "$MODULES" && rm /ha_automodules.sh

# Apps (manual)
ENV PACKAGES=""

# Auto app install
ADD "https://raw.githubusercontent.com/alexbelgium/hassio-addons/master/.templates/ha_autoapps.sh" "/ha_autoapps.sh"
RUN chmod 744 /ha_autoapps.sh && /ha_autoapps.sh "$PACKAGES" && rm /ha_autoapps.sh

################
# 4 Entrypoint #
################

# Entrypoint
ENV S6_STAGE2_HOOK=/ha_entrypoint.sh
ADD "https://raw.githubusercontent.com/alexbelgium/hassio-addons/master/.templates/ha_entrypoint.sh" "/ha_entrypoint.sh"

# Modify entrypoint
ADD "https://raw.githubusercontent.com/alexbelgium/hassio-addons/master/.templates/ha_entrypoint_modif.sh" "/ha_entrypoint_modif.sh"
RUN chmod 777 /ha_entrypoint.sh /ha_entrypoint_modif.sh && /ha_entrypoint_modif.sh && rm /ha_entrypoint_modif.sh

# Final CMD (php-fpm optional fallback — modify or remove if unneeded)
# Use `su-exec` to ensure non-root PHP-FPM, if you want this running
# alongside xteve, you may choose to add it to your `01-custom_script.sh`
CMD ["su-exec", "xteve", "/usr/sbin/php-fpm7", "--nodaemonize", "--force-stderr"]

# Set working user
USER xteve

############
# 5 Labels #
############

ARG BUILD_ARCH
ARG BUILD_DATE
ARG BUILD_DESCRIPTION
ARG BUILD_NAME
ARG BUILD_REF
ARG BUILD_REPOSITORY
ARG BUILD_VERSION
ENV BUILD_VERSION="${BUILD_VERSION}"
LABEL \
    io.hass.name="${BUILD_NAME}" \
    io.hass.description="${BUILD_DESCRIPTION}" \
    io.hass.arch="${BUILD_ARCH}" \
    io.hass.type="addon" \
    io.hass.version=${BUILD_VERSION} \
    maintainer="alexbelgium (https://github.com/alexbelgium)" \
    org.opencontainers.image.title="${BUILD_NAME}" \
    org.opencontainers.image.description="${BUILD_DESCRIPTION}" \
    org.opencontainers.image.vendor="Home Assistant Add-ons" \
    org.opencontainers.image.authors="alexbelgium (https://github.com/alexbelgium)" \
    org.opencontainers.image.licenses="MIT" \
    org.opencontainers.image.url="https://github.com/alexbelgium" \
    org.opencontainers.image.source="https://github.com/${BUILD_REPOSITORY}" \
    org.opencontainers.image.documentation="https://github.com/${BUILD_REPOSITORY}/blob/main/README.md" \
    org.opencontainers.image.created=${BUILD_DATE} \
    org.opencontainers.image.revision=${BUILD_REF} \
    org.opencontainers.image.version=${BUILD_VERSION}

####################
# 6 HealthcheckNOT #
####################
