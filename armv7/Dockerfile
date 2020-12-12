FROM multiarch/qemu-user-static:arm as qemu
FROM arm32v7/debian:10.6-slim
COPY --from=qemu /usr/bin/qemu-arm-static /usr/bin

# Build arguments
ARG VERSION
ARG CHANNEL

# Runtime environment variables
ENV DEBIAN_FRONTEND=noninteracive \
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

# Install deCONZ dependencies (except for WiringPi)
RUN apt-get update && \
    apt-get install -y \
        curl \
        kmod \
        git \
        libcap2-bin \
        libqt5core5a \
        libqt5gui5 \
        libqt5network5 \
        libqt5serialport5 \
        libqt5sql5 \
        libqt5websockets5 \
        libqt5widgets5 \
        lsof \
        gnupg \
        build-essential \
        libx11-xcb-dev \
        libglu1-mesa-dev \
        sqlite3 \
        tigervnc-standalone-server \
        tigervnc-common \
        wmii \
        xfonts-base \
        wget \
        xfonts-scalable && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/* && \
    wget http://phoscon.de/apt/deconz.pub.key && \
    apt-key add deconz.pub.key && \
    sh -c "echo 'deb http://phoscon.de/apt/deconz buster-beta main' > /etc/apt/sources.list.d/deconz.list" && \
    apt-get update && \
    apt-get install -y \
        deconz-dev && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# Add WiringPi, install WiringPi
ADD https://project-downloads.drogon.net/wiringpi-latest.deb /wiringpi.deb
RUN dpkg -i /wiringpi.deb && \
    rm -f /wiringpi.deb

# Add start.sh and Conbee udev data; set execute permissions
COPY root /
RUN chmod +x /start.sh && \
    chmod +x /firmware-update.sh

# Add deCONZ, install deCONZ, make OTAU dir
#ADD http://deconz.dresden-elektronik.de/raspbian/${CHANNEL}/deconz-${DECONZ_VERSION}-qt5.deb /deconz.deb
#RUN dpkg -i /deconz.deb && \
#    rm -f /deconz.deb && \
#    mkdir /root/otau && \
#    chown root:root /usr/bin/deCONZ*

RUN git clone https://github.com/kgorszczyk/deconz-rest-plugin && \
    cd deconz-rest-plugin && \
    qmake && make -j2 && \
    cp ../libde_rest_plugin.so /usr/share/deCONZ/plugins && \
    mkdir /root/otau && \
    chown root:root /usr/bin/deCONZ* && \
    rm -rf /deconz-rest-plugin

#RUN git clone https://github.com/dresden-elektronik/deconz-rest-plugin && \
#    cd deconz-rest-plugin && \
#    qmake && make -j2 && \
#    cp ../libde_rest_plugin.so /usr/share/deCONZ/plugins && \
#    mkdir /root/otau && \
#    chown root:root /usr/bin/deCONZ* && \
#    rm -rf /deconz-rest-plugin

VOLUME [ "/root/.local/share/dresden-elektronik/deCONZ" ]

EXPOSE ${DECONZ_WEB_PORT} ${DECONZ_WS_PORT} ${DECONZ_VNC_PORT}

ENTRYPOINT [ "/start.sh" ]
