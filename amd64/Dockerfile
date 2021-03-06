FROM debian:10.6-slim

# Build arguments
ARG VERSION
ARG CHANNEL

# Runtime environment variables
ENV DEBIAN_FRONTEND=noninteractive \
    DECONZ_VERSION=${VERSION} \
    DECONZ_WEB_PORT=80 \
    DECONZ_WS_PORT=443 \
    DEBUG_INFO=1 \
    DEBUG_APS=0 \
    DEBUG_ZCL=0 \
    DEBUG_ZDP=0 \
    DEBUG_OTAU=0 \
    DECONZ_DEVICE=0 \
    DECONZ_VNC_MODE=0 \
    DECONZ_VNC_DISPLAY=0 \
    DECONZ_VNC_PASSWORD=changeme \
    DECONZ_VNC_PORT=5900 \
    DECONZ_UPNP=1

# Install deCONZ dependencies
RUN apt-get update && \
    apt-get install -y \
        git \
        gnupg \
        build-essential \
        libx11-xcb-dev \
        libglu1-mesa-dev \
        curl \
        kmod \
        wget \
        libcap2-bin \
        libqt5core5a \
        libqt5gui5 \
        libqt5network5 \
        libqt5serialport5 \
        libqt5sql5 \
        libqt5websockets5 \
        libqt5widgets5 \
        lsof \
        sqlite3 \
        tigervnc-standalone-server \
        tigervnc-common \
        wmii \
        xfonts-base \
        xfonts-scalable && \
    wget http://phoscon.de/apt/deconz.pub.key && \
    apt-key add deconz.pub.key && \
    sh -c "echo 'deb http://phoscon.de/apt/deconz buster-beta main' > /etc/apt/sources.list.d/deconz.list" && \
    apt-get update && \
    apt-get install -y \
        deconz-dev && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# Workaround required on amd64 to address issue #292
RUN apt-get update && \
    apt-get install -y \
        binutils && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/* && \
    strip --remove-section=.note.ABI-tag /usr/lib/x86_64-linux-gnu/libQt5Core.so.5

# Add start.sh and Conbee udev data; set execute permissions
COPY root /
RUN chmod +x /start.sh && \
    chmod +x /firmware-update.sh

RUN git clone --branch revolt https://github.com/Smanar/deconz-rest-plugin deconz-rest-plugin && \
    cd deconz-rest-plugin && \
    qmake && make -j2 && \
    cp ../libde_rest_plugin.so /usr/share/deCONZ/plugins && \
    mkdir /root/otau && \
    chown root:root /usr/bin/deCONZ* && \
    rm -rf /deconz-rest-plugin

VOLUME [ "/root/.local/share/dresden-elektronik/deCONZ" ]

EXPOSE ${DECONZ_WEB_PORT} ${DECONZ_WS_PORT} ${DECONZ_VNC_PORT}

ENTRYPOINT [ "/start.sh" ]
