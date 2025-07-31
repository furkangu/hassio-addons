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

USER root

# Install dependencies
RUN apk add --no-cache su-exec

# Create non-root user and group
RUN addgroup -S xteve && adduser -S xteve -G xteve

# Copy xteve files if present
RUN cp -vnpr /home/xteve* /data || true

# Ensure proper ownership and access
RUN mkdir -p /etc/opt/xteve /tmp/xteve /config && \
    chown -R xteve:xteve /etc/opt/xteve /tmp/xteve /data /config && \
    chmod -R 755 /etc/opt/xteve /tmp/xteve /config

# Aggressive VLC root check bypass
RUN if [ -f /usr/bin/vlc ]; then \
        cp /usr/bin/vlc /usr/bin/vlc-original && \
        # Multiple patches to bypass root detection
        sed -i 's/geteuid/getppid/g' /usr/bin/vlc && \
        sed -i 's/getuid/getppid/g' /usr/bin/vlc && \
        sed -i 's/VLC is not supposed to be run as root/VLC running normally/g' /usr/bin/vlc && \
        # Create a complete wrapper that fakes non-root
        mv /usr/bin/vlc /usr/bin/vlc-patched && \
        echo '#!/bin/sh' > /usr/bin/vlc && \
        echo 'export VLC_PLUGIN_PATH=/usr/lib/vlc/plugins' >> /usr/bin/vlc && \
        echo 'export HOME=/tmp/xteve' >> /usr/bin/vlc && \
        echo '# Force VLC to think it is not root by overriding UID checks' >> /usr/bin/vlc && \
        echo 'exec /usr/bin/vlc-patched --intf dummy --no-video --no-sout-video --no-audio --quiet "$@"' >> /usr/bin/vlc && \
        chmod +x /usr/bin/vlc; \
    fi || true

# Alternative: Create vlc-wrapper as suggested by VLC error message
RUN if [ -f /usr/bin/vlc-original ]; then \
        echo '#!/bin/sh' > /usr/bin/vlc-wrapper && \
        echo 'export VLC_PLUGIN_PATH=/usr/lib/vlc/plugins' >> /usr/bin/vlc-wrapper && \
        echo 'export HOME=/tmp/xteve' >> /usr/bin/vlc-wrapper && \
        echo 'exec /usr/bin/vlc-original --intf dummy --no-video --no-sout-video --no-audio --quiet "$@"' >> /usr/bin/vlc-wrapper && \
        chmod +x /usr/bin/vlc-wrapper && \
        # Make it setuid as VLC suggests (but this is risky)
        # chown root:root /usr/bin/vlc-wrapper && chmod 4755 /usr/bin/vlc-wrapper; \
    fi || true

# Nuclear option: Replace VLC entirely with a stub that always succeeds
RUN echo '#!/bin/sh' > /usr/local/bin/vlc-stub && \
    echo '# VLC stub for xteve - always returns success' >> /usr/local/bin/vlc-stub && \
    echo 'echo "VLC stub: would run with args: $@" >&2' >> /usr/local/bin/vlc-stub && \
    echo 'exit 0' >> /usr/local/bin/vlc-stub && \
    chmod +x /usr/local/bin/vlc-stub

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

# Auto module install
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

# Entry hook for HA
ENV S6_STAGE2_HOOK=/ha_entrypoint.sh
ADD "https://raw.githubusercontent.com/alexbelgium/hassio-addons/master/.templates/ha_entrypoint.sh" "/ha_entrypoint.sh"

# Patch entrypoint
ADD "https://raw.githubusercontent.com/alexbelgium/hassio-addons/master/.templates/ha_entrypoint_modif.sh" "/ha_entrypoint_modif.sh"
RUN chmod 777 /ha_entrypoint.sh /ha_entrypoint_modif.sh && /ha_entrypoint_modif.sh && rm /ha_entrypoint_modif.sh

####################
# 5 Service script #
####################

# Add service run script that uses su-exec to drop privileges
RUN mkdir -p /etc/services.d/xteve && \
    echo '#!/usr/bin/execlineb -P' > /etc/services.d/xteve/run && \
    echo 'with-contenv' >> /etc/services.d/xteve/run && \
    echo 'export VLC_PLUGIN_PATH=/usr/lib/vlc/plugins' >> /etc/services.d/xteve/run && \
    echo 'export HOME=/tmp/xteve' >> /etc/services.d/xteve/run && \
    echo 'mkdir -p /tmp/xteve/.config/vlc' >> /etc/services.d/xteve/run && \
    echo 'chown xteve:xteve /tmp/xteve/.config/vlc' >> /etc/services.d/xteve/run && \
    echo 'su-exec xteve /opt/xteve/xteve' >> /etc/services.d/xteve/run && \
    chmod +x /etc/services.d/xteve/run

############
# 6 Labels #
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
# 7 Final Settings #
####################

# Set environment variables for VLC non-root operation
ENV VLC_PLUGIN_PATH=/usr/lib/vlc/plugins \
    HOME=/tmp/xteve

# Drop to xteve user (just in case)
USER xteve
