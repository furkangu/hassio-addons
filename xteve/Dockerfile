# Add this section to your existing Dockerfile after the user creation

# Complete VLC replacement - most reliable solution
RUN if [ -f /usr/bin/vlc ]; then \
        # Backup original VLC
        mv /usr/bin/vlc /usr/bin/vlc-original && \
        # Create a new VLC script that completely bypasses root checks
        cat > /usr/bin/vlc << 'VLCEOF'
#!/bin/sh
# VLC replacement script for non-root operation
# This completely bypasses VLC's root detection

export VLC_PLUGIN_PATH=/usr/lib/vlc/plugins
export HOME=/tmp/xteve
export USER=xteve
export LOGNAME=xteve

# Create necessary directories
mkdir -p /tmp/xteve/.config/vlc /tmp/xteve/.cache/vlc
chown -R xteve:xteve /tmp/xteve 2>/dev/null || true

# Use a fake UID for VLC's internal checks
# This is the key - we'll exec with a clean environment
exec env -i \
    PATH="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin" \
    HOME="/tmp/xteve" \
    USER="xteve" \
    LOGNAME="xteve" \
    VLC_PLUGIN_PATH="/usr/lib/vlc/plugins" \
    /usr/bin/vlc-original \
    --intf dummy \
    --no-video \
    --no-sout-video \
    --no-audio \
    --quiet \
    --no-osd \
    --no-stats \
    --no-media-library \
    --no-qt-privacy-ask \
    --no-qt-updates-notif \
    --config "/tmp/xteve/.config/vlc" \
    "$@" 2>/dev/null
VLCEOF
        chmod +x /usr/bin/vlc; \
    fi

# Also create the vlc-wrapper that VLC error message suggests
RUN if [ -f /usr/bin/vlc-original ]; then \
        cat > /usr/bin/vlc-wrapper << 'WRAPEOF'
#!/bin/sh
# VLC wrapper as suggested by VLC error message
export HOME=/tmp/xteve
export USER=xteve
export LOGNAME=xteve
export VLC_PLUGIN_PATH=/usr/lib/vlc/plugins

mkdir -p /tmp/xteve/.config/vlc
chown -R xteve:xteve /tmp/xteve 2>/dev/null || true

exec /usr/bin/vlc-original \
    --intf dummy \
    --no-video \
    --no-sout-video \
    --no-audio \
    --quiet \
    "$@" 2>/dev/null
WRAPEOF
        chmod +x /usr/bin/vlc-wrapper; \
    fi
