ARG BUILD_ARCH=amd64
ARG BUILD_FROM=ghcr.io/home-assistant/${BUILD_ARCH}-base-debian:bookworm
FROM ${BUILD_FROM}

# Propagate add-on version into image labels via build arg
ARG ADDON_VERSION="dev"
LABEL io.hass.version="${ADDON_VERSION}" io.hass.type="addon" io.hass.arch="armhf|aarch64|i386|amd64" \
      org.opencontainers.image.version="${ADDON_VERSION}"

SHELL ["/bin/bash", "-o", "pipefail", "-c"]

# Optimize APT for faster, smaller builds
RUN echo 'APT::Install-Recommends "false";' > /etc/apt/apt.conf.d/99no-recommends \
    && echo 'APT::Install-Suggests "false";' >> /etc/apt/apt.conf.d/99no-recommends \
    && echo 'APT::Get::Clean "always";' >> /etc/apt/apt.conf.d/99auto-clean \
    && echo 'DPkg::Post-Invoke {"/bin/rm -f /var/cache/apt/archives/*.deb || true";};' >> /etc/apt/apt.conf.d/99auto-clean

# Copy rootfs before installation
COPY rootfs /

# Single optimized layer with package installation and scanservjs setup
RUN set -e \
    # Package installation
    && apt-get update \
    && DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends \
        # Core system packages
        sudo \
        nano \
        netcat-openbsd \
        nodejs \
        curl \
        # CUPS printing packages
        cups \
        cups-pdf \
        colord \
        # Minimal printer drivers (configurable at runtime)
        printer-driver-hpcups \
        printer-driver-brlaser \
        # Network discovery packages
        avahi-daemon \
        libnss-mdns \
        dbus \
        # SANE scanning packages
        sane \
        libsane-common \
        sane-utils \
        sane-airscan \
        imagemagick \
        ipp-usb \
        # OCR support (minimal, configurable)
        tesseract-ocr \
        tesseract-ocr-eng \
    # Download and install scanservjs with retry logic
    && (curl -fsSL "https://github.com/sbs20/scanservjs/releases/download/v3.0.3/scanservjs_3.0.3-1_all.deb" -o /tmp/scanservjs.deb || \
        curl -fsSL "https://github.com/sbs20/scanservjs/releases/download/v3.0.3/scanservjs_3.0.3-1_all.deb" -o /tmp/scanservjs.deb) \
    && dpkg -i /tmp/scanservjs.deb \
    # Create user with minimal setup
    && useradd --groups=sudo,lp,lpadmin --create-home --home-dir=/home/print --shell=/bin/bash print \
    && echo 'print:print' | chpasswd \
    && sed -i '/%sudo[[:space:]]/ s/ALL[[:space:]]*$/NOPASSWD:ALL/' /etc/sudoers \
    # Fast cleanup for quicker builds
    && apt-get autoremove -y \
    && apt-get clean \
    && rm -rf \
        /var/lib/apt/lists/* \
        /var/cache/apt/archives/* \
        /tmp/* \
        /var/tmp/* \
        /usr/share/doc/* \
        /usr/share/man/* \
        /usr/share/info/* \
        /var/cache/debconf/* \
        /var/lib/dpkg/info/*.list \
    && rm -rf /root/.npm /home/*/.npm \
    && rm -rf /var/cache/* /root/.cache \
    && chmod a+x /run.sh

EXPOSE 631 8080
CMD ["/run.sh"]
