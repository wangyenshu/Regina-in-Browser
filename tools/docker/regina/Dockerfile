FROM i386/debian:bookworm-slim

WORKDIR /root/build
ENV DEBIAN_FRONTEND=noninteractive

# 1. Base Setup
RUN rm -f /etc/apt/sources.list.d/* && \
    echo "deb http://mirrors.ustc.edu.cn/debian/ bookworm main contrib non-free non-free-firmware\n\
    deb http://mirrors.ustc.edu.cn/debian/ bookworm-updates main contrib non-free non-free-firmware\n\
    deb http://mirrors.ustc.edu.cn/debian/ bookworm-backports main contrib non-free non-free-firmware\n\
    deb http://security.debian.org/debian-security bookworm-security main contrib non-free non-free-firmware" > /etc/apt/sources.list && \
    apt-get update && \
    apt-get install -y --no-install-recommends \
    linux-image-686 \
    systemd-sysv \
    locales \
    libterm-readline-perl-perl \
    && rm -rf /var/lib/apt/lists/*

# 2. Configure Locales & Shell
RUN echo "en_US.UTF-8 UTF-8" >> /etc/locale.gen && \
    locale-gen && \
    echo 'LANG="en_US.UTF-8"' > /etc/default/locale && \
    chsh -s /bin/bash root  

# 3. Install Minimal X11, Input Drivers, Fonts, UI Libraries, and Regina Build Dependencies
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    xserver-xorg-core xserver-xorg-video-fbdev xinit fluxbox \
    xserver-xorg-input-mouse xserver-xorg-input-kbd xserver-xorg-input-evdev \
    libxext6 libxrender1 libxtst6 libxi6 libx11-xcb1 libxrender-dev \
    libfontconfig1 fonts-dejavu-core \
    wget tar ca-certificates xterm \
    g++ make cmake libgmp-dev libxml2-dev libtokyocabinet-dev \
    qt6-base-dev libqt6svg6-dev python3-all-dev doxygen \
    libgraphviz-dev pkg-config xsltproc \
    && rm -rf /var/lib/apt/lists/*

# 4. Compile and Install Regina
COPY regina-7.4.1.tar.gz /tmp/regina.tar.gz
RUN mkdir -p /opt && \
    tar -xzf /tmp/regina.tar.gz -C /opt && \
    rm /tmp/regina.tar.gz && \
    cd /opt/regina-7.4.1 && \
    mkdir build && \
    cd build && \
    cmake .. && \
    make -j$(nproc) && \
    make install && \
    cd / && rm -rf /opt/regina-7.4.1

RUN passwd -d root && \
    sed -i 's/nullok_secure/nullok/' /etc/pam.d/common-auth

# 6. Configure Serial Console
COPY getty-noclear.conf getty-override.conf /etc/systemd/system/getty@tty1.service.d/
COPY getty-autologin-serial.conf /etc/systemd/system/serial-getty@ttyS0.service.d/

RUN systemctl enable serial-getty@ttyS0.service

# 7. Disable Unnecessary Services
RUN systemctl disable systemd-timesyncd.service && \
    systemctl disable apt-daily.timer && \
    systemctl disable apt-daily-upgrade.timer

# ------------------------------------------------------------------------------
# 8. 9p Filesystem Boot Configuration
# ------------------------------------------------------------------------------
RUN printf '%s\n' 9p 9pnet 9pnet_virtio virtio virtio_ring virtio_pci | tee -a /etc/initramfs-tools/modules

RUN echo '#!/bin/sh' > /etc/initramfs-tools/scripts/boot-9p && \
    echo 'case $1 in prereqs) exit 0;; esac' >> /etc/initramfs-tools/scripts/boot-9p && \
    echo '. /scripts/functions' >> /etc/initramfs-tools/scripts/boot-9p && \
    echo 'mkdir -p ${rootmnt}' >> /etc/initramfs-tools/scripts/boot-9p && \
    echo 'mount -n -t 9p -o trans=virtio,version=9p2000.L,cache=loose,rw host9p ${rootmnt}' >> /etc/initramfs-tools/scripts/boot-9p && \
    chmod +x /etc/initramfs-tools/scripts/boot-9p

RUN echo 'BOOT=boot-9p' | tee -a /etc/initramfs-tools/initramfs.conf
RUN update-initramfs -u

# 9. Configure X11 and Auto-start
# Modified to launch the installed Regina GUI
RUN echo 'regina-gui & \n\
(sleep 5 && echo -e "\n\nGUI_READY\n" > /dev/ttyS0) & \n\
exec fluxbox' > /root/.xinitrc

RUN { \
    echo 'if [ -z "$DISPLAY" ]; then'; \
    echo '  echo "127.0.0.1 localhost" > /etc/hosts'; \
    echo '  echo "localhost" > /etc/hostname'; \
    echo '  hostname localhost'; \
    echo '  startx -- -ac'; \
    echo 'fi'; \
} >> /root/.bashrc

# 10. Force Xorg to use the FBDEV driver
RUN mkdir -p /etc/X11/xorg.conf.d && \
    echo 'Section "Device"\n\
    Identifier "Card0"\n\
    Driver "fbdev"\n\
EndSection' > /etc/X11/xorg.conf.d/10-fbdev.conf

WORKDIR /root