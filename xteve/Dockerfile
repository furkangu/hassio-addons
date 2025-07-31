####################
# 1. Build image   #
####################
ARG BUILD_FROM
ARG BUILD_VERSION
ARG BUILD_UPSTREAM="2.5.3"
FROM ${BUILD_FROM}

####################
# 2. Base tweaks   #
####################
# S6-overlay timing
ENV S6_CMD_WAIT_FOR_SERVICES=1 \
    S6_CMD_WAIT_FOR_SERVICES_MAXTIME=0 \
    S6_SERVICES_GRACETIME=0

# Operate as root while we install things
USER root

# ─────────────────────────────────────────────────────────────────────────────
# Minimal VLC root-check workaround
# ─────────────────────────────────────────────────────────────────────────────
RUN apk add --no-cache su-exec                               \
 && addgroup -S xteve                                        \
 && adduser  -S -G xteve xteve                               \
 && mkdir -p /etc/opt/xteve /tmp/xteve /data /config         \
 && chown -R xteve:xteve /etc/opt/xteve /tmp/xteve /data /config \
 && chmod -R 755          /etc/opt/xteve /tmp/xteve /config  \
 && if [ -f /usr/bin/vlc ]; then                             \
        mv /usr/bin/vlc /usr/bin/vlc-original               && \
        cat >/usr/bin/vlc <<'EOF'                            \
#!/bin/sh
# /usr/bin/vlc – thin wrapper that drops privileges
exec su-exec xteve \
     env HOME=/tmp/xteve \
         VLC_PLUGIN_PATH=/usr/lib/vlc/plugins \
     /usr/bin/vlc-original --intf dummy --no-video --no-audio --quiet "$@"
EOF
        chmod 755 /usr/bin/vlc;                             \
    fi
# ─────────────────────────────────────────────────────────────────────────────

########################################################################
# 3. Everything below is *exactly* what you already had – modules, etc #
########################################################################

# Copy xteve data if it exists in the build context
RUN cp -vnpr /home/xteve* /data || true

#-------------------------------------------------------------------------
# 3-A Auto-modules / Auto-apps templates pulled from alexbelgium repo
#-------------------------------------------------------------------------
# Symlinks fallback shell
RUN if [ ! -f /bin/sh ]  && [ -f /usr/bin/sh  ]; then ln -s /usr/bin/sh  /bin/sh;  fi && \
    if [ ! -f /bin/bash ]&& [ -f /usr/bin/bash]; then ln -s /usr/bin/bash /bin/bash; fi

# Modules to auto-install
ARG MODULES="00-banner.sh 01-custom_script.sh"
ADD "https://raw.githubusercontent.com/alexbelgium/hassio-addons/master/.templates/ha_automodules.sh" "/ha_automodules.sh"
RUN chmod 744 /ha_automodules.sh && /ha_automodules.sh "$MODULES" && rm /ha_automodules.sh

# Optional extra Alpine packages
ENV PACKAGES=""
ADD "https://raw.githubusercontent.com/alexbelgium/hassio-addons/master/.templates/ha_autoapps.sh" "/ha_autoapps.sh"
RUN chmod 744 /ha_autoapps.sh && /ha_autoapps.sh "$PACKAGES" && rm /ha_autoapps.sh

#-------------------------------------------------------------------------
# 3-B  Entry-point hooks for Home-Assistant Supervisor / S6
#-------------------------------------------------------------------------
ENV S6_STAGE2_HOOK=/ha_entrypoint.sh
ADD "https://raw.githubusercontent.com/alexbelgium/hassio-addons/master/.templates/ha_entrypoint.sh"        "/ha_entrypoint.sh"
ADD "https://raw.githubusercontent.com/alexbelgium/hassio-addons/master/.templates/ha_entrypoint_modif.sh"  "/ha_entrypoint_modif.sh"
RUN chmod 777 /ha_entrypoint.sh /ha_entrypoint_modif.sh \
 && /ha_entrypoint_modif.sh \
 && rm /ha_entrypoint_modif.sh

#-------------------------------------------------------------------------
# 3-C  S6 service that launches xteve itself (still using su-exec)
#-------------------------------------------------------------------------
RUN mkdir -p /etc/services.d/xteve
COPY <<'EOF' /etc/services.d/xteve/run
#!/usr/bin/execlineb -P
with-contenv

export HOME=/tmp/xteve
export VLC_PLUGIN_PATH=/usr/lib/vlc/plugins

# Guarantee writable config dir at runtime
mkdir -p /tmp/xteve/.config/vlc
chown xteve:xteve /tmp/xteve/.config/vlc

# Launch xteve as the non-root user
su-exec xteve /opt/xteve/xteve
EOF
RUN chmod +x /etc/services.d/xteve/run

####################
# 4. Image labels  #
####################
ARG BUILD_ARCH
ARG BUILD_DATE
ARG BUILD_DESCRIPTION
ARG BUILD_NAME
ARG BUILD_REF
ARG BUILD_REPOSITORY
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
# 5. Final tweaks  #
####################
# Environment for any process that *does* inherit root (s6, etc.)
ENV VLC_PLUGIN_PATH=/usr/lib/vlc/plugins \
    HOME=/tmp/xteve

# At image-build time we are finished; run-time starts as the unprivileged user
USER xteve
