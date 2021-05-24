# Copyright 2021 The Chromium OS Authors. All rights reserved.
# Use of this source code is governed by a BSD-style license that can be
# found in the LICENSE file.

FROM alpine:3.7 as chromiumos_overlays
RUN apk add --no-cache git
RUN git clone https://chromium.googlesource.com/chromiumos/overlays/chromiumos-overlay

######################################################################

FROM python:3.7.10-buster

# Install python packages.
COPY requirements.txt /opt/requirements.txt
RUN pip3 install -r /opt/requirements.txt

# Install servo_firmware.
RUN mkdir -p /usr/share/servo_updater/firmware/
WORKDIR /usr/share/servo_updater/firmware/
COPY servo_firmware_provision.py servo_firmware_provision.py
RUN python servo_firmware_provision.py && rm servo_firmware_provision.py

RUN apt-get update \
    && apt-get install -y \
        dfu-util \
        flashrom \
        ftdi-eeprom \
        gcc \
        libboost-all-dev \
        libssl-dev \
        libusb-1.0-0-dev \
        make \
        protobuf-compiler \
        cmake \
        sudo \
        swig \
        tidy \
        upx \
        uhubctl \
        unzip \
        vim \
        zlib1g-dev \
    && rm -rf /var/lib/apt/lists/*

# TODO (anhdle): Land libconfuse into a separate container.
WORKDIR /
RUN wget https://github.com/libconfuse/libconfuse/releases/download/v3.2.2/confuse-3.2.2.tar.gz \
    && tar xf ./confuse-3.2.2.tar.gz
RUN cd /confuse-3.2.2 && ./configure && make -j9 && make install
RUN rm -rf /confuse-3.2.2 && rm /confuse-3.2.2.tar.gz

# Install fmap.
# TODO (anhdle): Land libconfuse into a separate container.
WORKDIR /
RUN git clone https://chromium.googlesource.com/chromiumos/third_party/flashmap
RUN cd /flashmap && make && make install
RUN rm -rf /flashmap

# Install libftdi.
# TODO (anhdle): Land libftdi into a separate container.
RUN wget https://www.intra2net.com/en/developer/libftdi/download/libftdi1-1.2.tar.bz2 \
    && tar xf libftdi1-1.2.tar.bz2 \
    && rm libftdi1-1.2.tar.bz2
COPY --from=chromiumos_overlays \
    /chromiumos-overlay/dev-embedded/libftdi/files/libftdi-1.2-getopts.patch \
    /libftdi1-1.2/
RUN cd /libftdi1-1.2 \
    && git apply libftdi-1.2-getopts.patch \
    && mkdir build \
    && cd build \
    && cmake -DCMAKE_INSTALL_PREFIX="/usr" ../ \
    && make \
    && make install
RUN rm -rf /libftdi1-1.2
COPY --from=chromiumos_overlays \
    /chromiumos-overlay/dev-embedded/libftdi/files/confs/*.conf \
    /usr/share/libftdi/

COPY dobin /usr/bin/dobin
COPY dosbin /usr/sbin/dosbin

# Install ec-utils.
# TODO (anhdle): Land ec-utils into a separate container.
RUN git clone https://chromium.googlesource.com/chromiumos/platform/ec
WORKDIR /ec
RUN make utils-host \
    && make -C extra/usb_updater usb_updater2
RUN dosbin /ec/build/bds/util/ectool
RUN dosbin /ec/build/bds/util/ec_parse_panicinfo
RUN dosbin /ec/build/bds/util/ec_sb_firmware_update
RUN dobin /ec/build/bds/util/stm32mon
RUN dobin /ec/build/bds/util/ec_parse_panicinfo
RUN dobin /ec/build/bds/util/uartupdatetool
RUN dobin /ec/build/bds/util/iteflash
RUN dobin /ec/util/flash_ec
RUN dobin /ec/util/uart_stress_tester.py
RUN mkdir -p /usr/share/ec-devutils
RUN cp /ec/util/openocd/* /usr/share/ec-devutils/
RUN dosbin /ec/extra/usb_updater/usb_updater2
RUN curl https://raw.githubusercontent.com/kward/shflags/master/shflags --output /usr/share/misc/shflags
RUN python3 setup.py build
RUN python3 setup.py install
RUN mkdir -p /usr/share/servo_updater/configs
RUN cp /ec/extra/usb_updater/*.json /usr/share/servo_updater/configs/

# Install hdctools
WORKDIR /
RUN git clone https://chromium.googlesource.com/chromiumos/third_party/hdctools
WORKDIR /hdctools
RUN echo 'HOSTOS_CWARN=' >> 'defs/c-Linux.mk'
RUN make && make install
RUN python3 setup.py build
RUN python3 setup.py install

CMD tail -f /dev/null