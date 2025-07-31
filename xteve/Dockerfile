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

# Enhanced VLC fixes for non-root user
RUN if [ -f /usr/bin/vlc ]; then \
        # Method 1: Patch the geteuid check
        sed -i 's/geteuid/getppid/' /usr/bin/vlc && \
        # Method 2: Create a wrapper script that bypasses root check
        mv /usr/bin/vlc /usr/bin/vlc-real && \
        cat > /usr/bin/vlc << 'EOF'
#!/bin/sh
# VLC wrapper to run as non-root
export VLC_PLUGIN_PATH=/usr/lib/vlc/plugins
exec /usr/bin/vlc-real --intf dummy --no-video --no-sout-video "$@"
EOF
        chmod +x /usr/bin/vlc; \
    fi || true

# Alternative: Create VLC wrapper that forces non-interactive mode
RUN cat > /usr/local/bin/vlc-wrapper << 'EOF'
#!/bin/sh
# Enhanced VLC wrapper for xteve
export HOME=/tmp/xteve
export VLC_PLUGIN_PATH=/usr/lib/vlc/plugins
export VLC_DATA_PATH=/usr/share/vlc

# Create temp directory for VLC
mkdir -p /tmp/xteve/.config/vlc
chown xteve:xteve /tmp/xteve/.config/vlc

# Run VLC with safe options for non-root
exec /usr/bin/vlc-real \
    --intf dummy \
    --no-video \
    --no-sout-video \
    --no-audio \
    --no-osd \
    --no-stats \
    --no-media-library \
    --play-and-exit \
    --no-loop \
    --no-repeat \
    --no-random \
    --no-qt-privacy-ask \
    --no-qt-updates-notif \
    --config=/tmp/xteve/.config/vlc \
    "$@"
EOF
RUN chmod +x /usr/local/bin/vlc-wrapper

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
RUN mkdir -p /etc/services.d/xteve
COPY <<EOF /etc/services.d/xteve/run
#!/usr/bin/execlineb -P
with-contenv
# Set VLC environment variables for non-root operation
export VLC_PLUGIN_PATH=/usr/lib/vlc/plugins
export HOME=/tmp/xteve
# Create VLC config directory
mkdir -p /tmp/xteve/.config/vlc
chown xteve:xteve /tmp/xteve/.config/vlc
# Run xteve as non-root user
su-exec xteve /opt/xteve/xteve
EOF
RUN chmod +x /etc/services.d/xteve/run

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
